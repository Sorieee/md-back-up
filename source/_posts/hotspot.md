---
title: hotspot
date: 2021-05-01 08:52:35
tags: [jvm,java,hotspot]
---



# Metaspace

Note: this Wiki page describes Metaspace in its current form, which has substantially changed in the wake of [JEP 387 Elastic Metaspace](https://openjdk.java.net/jeps/387). Some information in this page may not be applicable for earlier JDK releases.

## 什么是Metaspace

​	Metaspace是在hotspot中一块本地(堆外)内存的管理者。

​	它用于管理class metadata的内存。Class metaData当类加载时被分配。其生命周期受限于其classLoader的加载(当loader被回收, 它所有加载的class metadata 都会被批量释放)。

​	它不必为了释放它们跟踪单个的分配。Hence, the metaspace allocator is an [Arena- or Region-Based Allocator](https://en.wikipedia.org/wiki/Region-based_memory_management).

​	它经过优化，可快速，低开销地分配本机内存，但代价是无法（轻松）删除任意块。

## 高级功能概述

​	每个CLD(ClassLoaderData)实例拥有一个MetaspaceArena。从该区域，它通过指针缓冲分配用于类元数据和其他目的的内存。当它耗尽时，arena会以semi-coarse步骤动态增长。当class loader被卸载，它的CLD会被删除，arena也会被删除，其内存返还给Metaspace。内存会被保存在metaspace等待之后的服用，但是metaspace可以看情况uncommit部分或者所有。

​	全局存在一个MetaspaceContext: 它在OS层面管理潜在内存。为了实现arenas，它提供了一个粗粒度的分配API，它以块的形式分发内存。它同事保存一份已故的arenas中已被释放的块的freelist。

​	只有一个全局context存在，如果压缩类指针被禁用并且我们没有compressed class space。如果压缩指针启用，我们将类空间分配从非类空间分配独立出来。所有我们就会有两个metaspace context: 一个包含所有类结构体的分配, 另外一个包含其他所有东西。每个CLD都将包含两个arenas。

不压缩的情况

```
  +--------+  +--------+  +--------+  +--------+
  |  CLD   |  |  CLD   |  |  CLD   |  |  CLD   |
  +--------+  +--------+  +--------+  +--------+
      |           |           |           |       
      |           |           |           |       allocates variable-sized,
      |           |           |           |       typically small-tiny metaspace blocks 
      v           v           v           v  
  +--------+  +--------+  +--------+  +--------+
  | arena  |  | arena  |  | arena  |  | arena  |
  +--------+  +--------+  +--------+  +--------+
      |           |           |           |       
      |           |           |           |       allocate and, on death, release-in-bulk
      |           |           |           |       medium-sized chunks (1k..4m)
      |           |           |           |       
      v           v           v           v  
  +--------------------------------------------+
  |                                            |
  |         Metaspace Context                  |
  |          (incl chunk freelist)             |
  |                                            |
  +--------------------------------------------+
         |            |            |
         |            |            |              map/commit/uncommit/release
         |            |            |
         v            v            v
    +---------+  +---------+  +---------+
    |         |  |         |  |         |
    | virtual |  | virtual |  | virtual |
    | space   |  | space   |  | space   |
    |         |  |         |  |         |
    +---------+  +---------+  +---------+
```

​	当压缩指针空间启用，我们拥有两个Metaspace contexts(一个是普通的，一个包装class space),每个CLD都有两个class arenas，一个关联non-class context, 一个关联class space。

```
 +--------+              +--------+
        |  CLD   |              |  CLD   |
        +--------+              +--------+
         /     \                 /     \          Each CLD has two arenas...             
        /       \               /       \       
       /         \             /         \      
      v           v           v           v             
  +--------+  +--------+  +--------+  +--------+
  | noncl  |  | class  |  | noncl  |  | class  |
  | arena  |  | arena  |  | arena  |  | arena  |
  +--------+  +--------+  +--------+  +--------+
      |              \      /            |       
      |               --------\          |        Non-class arenas take from non-class context,
      |                   /   |          |        class arenas take from class context
      |         /---------    |          |       
      v         v             v          v  
  +--------------------+  +------------------------+
  |                    |  |                        |
  | Metaspace Context  |  | Metaspace Context      |
  |     (nonclass)     |  |     (class)            |
  |                    |  |                        |
  +--------------------+  +------------------------+
         |            |            |
         |            |            |                    Non-class context: list of smallish mappings
         |            |            |                    Class context: one large mapping (the class space)
         v            v            v
  +--------+  +--------+  +----------------~~~~~~~-----+
  |        |  |        |  |                            |
  | virtual|  | virt   |  | virt space (class space)   |
  | space  |  | space  |  |                            |
  |        |  |        |  |                            |
  +--------+  +--------+  +----------------~~~~~~~-----+
```

## 核心概念

### 提交颗粒

​	Elastic Metaspace的关键点之一是弹性，即将不需要的内存返回给OS并仅按需提交内存的能力。

​	Metaspace地址空间被均匀地分配到2的指数大小的内存单元中，叫做提交颗粒。提交颗粒是Metaspace提交和撤销提交内存的基本单位，因此来确认提交的粒度。

​	尽管从技术上讲，提交粒度可能小到一页，但实际上它们更大(默认为64K)。当内存返还给metaspace时，完全取消占用的提交颗粒将取消提交。

​	提交粒度是在内存回收效率和与内存映射片段化相关的某些成本之间进行折衷的选择。 颗粒越小，就越有可能不被使用并且有资格进行取消提交，但同时，取消提交许多小区域将增加VM进程的映射数量。 默认大小为64K，这是一个折衷方案，效果似乎很好，只是适度增加了映射数，同时给了我们良好的弹性。 粒度可以通过MetaspaceReclaimStrategy开关间接影响（请参见下文）。 

### 元数据块和协同风格分配器(Metachunks and the Buddy Style Allocator)

​	Metaspace会以semi-coarse的方式动态增长.在内部，他们是被称为*Metachunk*的边长内存区域的列表([metachunk.hpp](https://github.com/openjdk/jdk/blob/master/src/hotspot/share/memory/metaspace/metachunk.hpp)）。

​	Arenas在各自的metaspace context中包含这些chunk。当他们"死亡"，会返回批量所有的chunks。

​	chunk是2的指数的变长的，从1k到4M(Root Chunk)的。

​	Chunks are managed by a power-two-based [buddy allocator](https://en.wikipedia.org/wiki/Buddy_memory_allocation). 伙伴分配器在防止碎片化方面非常有效，代价是将管理区域的大小限制为2的指数倍。这种限制在元空间中并不重要，因为这些块不是最终的用户级分配单元，只是一个中间单元。

​	在整个元空间实现中，块大小不是表示为大小，而是表示为“块级别”(`chunklevel_t`, see `chunklevel.hpp`)。根块具有块级别0，下一个较小的块级别1，依此类推，直到最小的块级别13。在chunk_level.hpp中可以找到用于chunk级别的Helper函数和常量。

```
+------ ~~~~ --------+
|                    | level 0: (4M root chunk)
+------ ~~~~ --------+
...
+--------------+
|              | level 9: 8K
+--------------+
+------+ 
|      | level 10: 4K
+------+
+--+
|  | level 11: 2K
+--+
++
|| level 12: 1K (smallest chunk)
++

```

​	在伙伴式分配中，除非区块是根区块，否则区块总是相邻区块对的一部分。在代码中，我们对地址较低的块使用leader一词，他的伙伴称为follower。

```
+-------------------+-------------------+ 
| Leader            | Follower          |
+-------------------+-------------------+
```

当然，其中一个或两个可以分成更小的块。

```
+-------------------+-------------------+ 
| Leader            |         |    |    |
+-------------------+-------------------+
```

#### 合并chunk

​	如果一个空闲块是空闲的并且没有被分割，那么它可以和它的伙伴合并。这是递归进行的，直到块对中的任一个伙伴没有空闲和未分割，或者直到达到最大的块大小-根块大小。

​	这就相当有效地将一系列空闲块具体化为一个更大的块。在下图中，块b变为自由块，与自由块A融合，然后与块C融合：

```
+---------+---------+-------------------+
|    A    |    b    |         C         |
+---------+---------+-------------------+
                    |
                    v
+-------------------+-------------------+
|         A`        |         C         |
+-------------------+-------------------+
                    |
                    v
+-------------------+-------------------+
|                  A``                  |
+-------------------+-------------------+
```

#### 分割chunk

为了从一个较大的块中得到一个较小的块，可以分割一个较大的块。分裂发生在2次方大小。分割操作产生所需的较小块以及1-n个分割块。

```
Step
     +---------------------------------------+
 0   |                   A                   |
     +---------------------------------------+

                         |
                         v
     +-------------------+-------------------+
 1   | d1 | D2 |    C    |         B         |
     +-------------------+-------------------+
       ^
       Result chunk
                         |
                         v
     +-------------------+-------------------+
 2   | d1 | d2 |    c    |         B         |
     +-------------------+-------------------+
            ^
            Result chunk
                         |
                         v
     +-------------------+-------------------+
 3   | d1 | d2 | c1 | C2 |         B         |
     +-------------------+-------------------+
                 ^
                 Result chunk
```

#### How it all looks in memory

​	分配的元空间块（用户级分配单元）驻留在块中；块驻留在称为VirtualSpaceNode的映射中，其中可能存在多个映射：

```
 +------------------+ <--- virtual memory region
 | +-------------+  | <--- chunk
 | | +---------+ |  | <--- block 
 | | |         | |  |
 | | +---------+ |  |
 | | +---------+ |  | <--- block 
 | | |         | |  |
 | | |         | |  |
 | | |         | |  |
 | | +---------+ |  |
 | | +---------+ |  | <--- block 
 | | +---------+ |  |
 | |             |  |
 | +-------------+  | <--- end: chunk
 | +-------------+  | <--- chunk
 | | +---------+ |  |
 | | |         | |  |
        ...
 +------------------+ <--- end: virtual memory region
  
  
 +------------------+ <--- next virtual memory region
 | +-------------+  |
 | | +---------+ |  |
 | | |         | |  |
 | | +---------+ |  |
       ...
```

## 子系统

​	元空间实现被划分为独立的子系统，每个子系统都与其对等系统隔离，并且有少量的任务。

### 虚拟内存子系统

Classes:

\- `VirtualSpaceList`

\- `VirtualSpaceNode`

\- `RootChunkArea` and `RootChunkAreaLUT`

\- `CommitMask`

\- `CommitLimiter`

虚拟内存层是最底层的子系统。它形成了元空间上下文的一半（上半部分是块管理器）。
它负责保留和提交内存。它知道什么。它与上层的外部接口是VirtualSpaceList，而一些操作也通过VirtualSpaceNode直接公开。

#### Essential operations

* "Allocate new root chunk"

  [Metachunk* VirtualSpaceList::allocate_root_chunk();](https://github.com/openjdk/jdk/blob/2c9dfc73f92bc8d1c38b4f2f595194fcc623aa11/src/hotspot/share/memory/metaspace/virtualSpaceList.hpp#L102)

  这将从底层的保留空间中划分出一个新的根块，并将其交给调用者（还没有提交任何内容，这纯粹是保留内存）。

* "commit this range"

  [bool VirtualSpaceNode::ensure_range_is_committed(MetaWord* p, size_t word_size);](https://github.com/openjdk/jdk/blob/2c9dfc73f92bc8d1c38b4f2f595194fcc623aa11/src/hotspot/share/memory/metaspace/virtualSpaceNode.hpp#L256)

  上层请求提交给定的任意地址范围。子系统找出哪些颗粒会受到影响，并确保这些颗粒已提交（如果它们以前提交过，则可能是noop）。
  提交时，子系统遵守VM限制（MaxMetaspaceSize resp。提交限制（commit gc threshold）。

* "uncommit this range"

  [void VirtualSpaceNode::uncommit_range(MetaWord* p, size_t word_size);](https://github.com/openjdk/jdk/blob/2c9dfc73f92bc8d1c38b4f2f595194fcc623aa11/src/hotspot/share/memory/metaspace/virtualSpaceNode.hpp#L261)

  类似于提交。子系统找出哪些提交颗粒受到影响，并取消这些颗粒的提交。

* “清除”

  [void VirtualSpaceList::purge()](https://github.com/openjdk/jdk/blob/2c9dfc73f92bc8d1c38b4f2f595194fcc623aa11/src/hotspot/share/memory/metaspace/virtualSpaceList.hpp#L107)

  这将取消映射所有完全空的内存区域，并取消提交所有未使用的提交颗粒。

#### Other operations

​	虚拟内存子系统代表上层区域负责伙伴分配器操作：

* "split this chunk, recursivly" 

  [void VirtualSpaceNode::split(chunklevel_t target_level, Metachunk* c, FreeChunkListVector* freelists);](https://github.com/openjdk/jdk/blob/2c9dfc73f92bc8d1c38b4f2f595194fcc623aa11/src/hotspot/share/memory/metaspace/virtualSpaceNode.hpp#L187)

* "merge up chunk with neighbors as far as possible"

  `Metachunk* VirtualSpaceNode::merge(Metachunk* c, FreeChunkListVector* freelists);`

* "enlarge chunk in place" 

  [bool VirtualSpaceNode::attempt_enlarge_chunk(Metachunk* c, FreeChunkListVector* freelists);](https://github.com/openjdk/jdk/blob/2c9dfc73f92bc8d1c38b4f2f595194fcc623aa11/src/hotspot/share/memory/metaspace/virtualSpaceNode.hpp#L209)

#### Classes

##### VirtualSpaceList

VirtualSpaceList（VirtualSpaceList.hpp）封装内存映射列表（VirtualSpaceNode的实例）。此列表可以扩展—可以按需添加新映射—也可以不扩展。
不可扩展的后一种情况用于表示类空间，该类空间必须是单个连续地址范围，这样压缩的Klass*指针编码才能工作。在这种情况下，类空间VirtualSpaceList只包含一个节点，它封装了整个类空间。这听起来可能很复杂，但只是代码重用的问题。

##### VirtualSpaceNode

​	VirtualSpaceNode（VirtualSpaceNode.hpp）管理元空间的一个连续内存映射。在压缩类空间的情况下，这包括类空间的整个预保留地址范围。在非类元空间的情况下，这些映射很小，默认情况下大小为两个根块。
​	VirtualSpaceNode知道提交颗粒，并且知道提交颗粒中的哪些颗粒（通过提交掩码）。
​	VirtualSpaceNode还知道根块：它的内存被划分为一系列根块大小的区域（RootChunkArea类）。为了保持编码简单，我们要求每个内存映射在起始地址和大小上都与根块大小对齐。
注意：块和提交颗粒的概念几乎是完全独立的。前者是一种分发/收回内存的方法，同时避免碎片化；后者是管理内存提交状态的一种方法。
​	综上所述，VirtualSpaceNode下面的内存如下所示：

```sh
base                                                                    end
 |                                                                       |
 v                                                                       v

 | root chunk area | root chunk area | root chunk area | root chunk area |

 |x| |x|x|x| | | | |x|x|x| | | |x|x| | | |x|x|x|x| | | | | |x| |x| | | | |    <-- commit granules (x=committed)
```

##### CommitMask

([commitMask.hpp](https://github.com/openjdk/jdk/blob/master/src/hotspot/share/memory/metaspace/commitMask.hpp)) 

只是VirtualSpaceNode中的一个位掩码，其中包含提交信息（每个颗粒一位）。

##### RootChunkArea and RootChunkAreaLUT

`(`[rootChunkArea.hpp](https://github.com/openjdk/jdk/blob/master/src/hotspot/share/memory/metaspace/rootChunkArea.hpp)) 

RootChunkArea包含伙伴分配器代码。它被包裹在单个根块的区域上。
它知道如何分割和合并块。它还引用了该区域中的第一个块（需要，因为Metachunk块头与它们的有效负载是分开的实体，请参见下文，而且从metaspace起始地址到它的Metachunk并不容易）。
RootChunkArea对象本身并不存在，而是作为VirtualSpaceNode中数组的一部分，用于描述节点的内存。
RootChunkAreaLUT（用于“lookup table”）只保存RootChunkArea类的序列，这些类覆盖VirtualSpaceNode的内存区域。

##### CommitLimiter

([commitLimiter.hpp](https://github.com/openjdk/jdk/blob/master/src/hotspot/share/memory/metaspace/commitLimiter.hpp)) 

在为元空间提交内存时，我们必须遵守两个限制：

* MaxMetaspaceSize（允许我们为metaspace提交的内存总量上限）
* GC阈值，它为元空间增长设置了一个权宜之计，在允许进一步增长之前，触发一个GC来尝试类卸载。

（注意：CompressedClassSpaceSize—类空间的保留大小—是另一个限制，但在当前实现中没有显式检查它，因为它检查自身。如果我们用完了课堂空间，我们会注意到）。
  检查这两个限制有点复杂，也不应该是一般分配器的一部分，因此限制检查被抽象到类CommitLimiter中。允许将来使用不同的极限逻辑重用元空间编码（也更容易测试）。
  在正常情况下，CommitLimiter只存在一个实例，请参见CommitLimiter:：globalLimiter（），它封装了GC threshold和MaxMetaspace查询。

# 《深入解析Java虚拟机Hotspot》

# Java大观园

![](https://pic.imgdb.cn/item/608cbce5d1a9ae528f74d8e4.jpg)

## 源码模块

```
├── cpu                   # 与CPU架构相关的代码
├── os                    # 与操作系统相关的代码
├── os_cpu                # 与CPU和操作系统相关的代码
└── share                          
    ├── adlc              # 平台描述语言编译器（编译cpu目录中的*.ad文件）
    ├── aot               # AOT支持，加载验证AOT库等
    ├── asm               # 宏汇编器，为宏形式的JIT代码生成机器代码
    ├── c1                # Client即时编译器（C1 JIT）
    ├── ci                # 编译器接口，定义JIT编译器通用的一些结构
    ├── classfile         # 字节码文件解析和处理
    ├── code              # 描述JIT编译后的代码结构等
    ├── compiler          # JIT编译器代理，虚拟机通过它选择特定的JIT编译器
    ├── gc                # 垃圾回收。gc/shared表示共享代码，gc/g1，gc/cms表示特定代码
    ├── include           # 一些JVM函数和常量的导出
    ├── interpreter       # 模板解释器和CPP解释器实现
    ├── jfr               # 诊断工具Java Flight Record
    ├── jvmci             # JVMCI编译器接口，可以开启Graal编译器代替C2
    ├── cpu                   # 与CPU架构相关的代码
    ├── os                    # 与操作系统相关的代码
    ├── os_cpu                # 与CPU和操作系统相关的代码
    └── share                          
    ├── adlc              # 平台描述语言编译器（编译cpu目录中的*.ad文件）
    ├── aot               # AOT支持，加载验证AOT库等
    ├── asm               # 宏汇编器，为宏形式的JIT代码生成机器代码
    ├── c1                # Client即时编译器（C1 JIT）
    ├── ci                # 编译器接口，定义JIT编译器通用的一些结构
    ├── classfile         # 字节码文件解析和处理
    ├── code              # 描述JIT编译后的代码结构等
    ├── compiler          # JIT编译器代理，虚拟机通过它选择特定的JIT编译器
    ├── gc                # 垃圾回收。gc/shared表示共享代码，gc/g1，gc/cms表示特定代码
    ├── include           # 一些JVM函数和常量的导出
    ├── interpreter       # 模板解释器和CPP解释器实现
    ├── jfr               # 诊断工具Java Flight Record
    ├── jvmci             # JVMCI编译器接口，可以开启Graal编译器代替C2
```

在linux编译jdk

```sh
$ yum install java-11-openjdk*  # 安装Bootstrap JDK
$ yum install  autoconfunzip zip alsa-lib-devel
$ yum install libXtst-devel libXt-devel libXrender-devel  
$ yum install cups-devel freetype-devel fontconfig-devel
$ cd openjdk12
$ chmod +x configure
$./configure --with-debug-level=fastdebug
$ make all
```

​	在Linux开发机上可以使用Visual Code进行调试。Visual Code也是笔者推荐使用的智能编辑器，它同时支持Linux/Windows/macOS三大平台，只需简单的launch.json配置即可进行断点调试。

​	具体操作是在Visual Code菜单中选择File→Open，打开OpenJDK 12源码目录，然后选择Debug→Start Debugging添加launch.json文件，如代码清单1-4所示：

```json
{
    "version": "0.2.0",
    "configurations": [{
        "cwd": "${workspaceFolder}",
        "name": "HotSpot Linux Debug",
        "type": "cppdbg",
        "request": "launch",
        "program": "<构建生成的JDK目录>",
        "args": [ "<JVM启动参数>" ],
        "setupCommands": [{
            "description": "ignore sigsegv",
            "ignoreFailures": false,
            "text": "handle SIGSEGV nostop"
        }]
    }]
}
```

​	打上断点后，点击调试按钮即可开始调试。-XX:+PauseAtStartup和-XX:+PauseAtExit参数分别代表让虚拟机在启动和退出的地方停顿。

​	随着社区的不断发展，JDK的构建愈发成熟和简单，读者如果在构建过程中遇到问题，可以尝试根据报错自行解决，可以参见官方提供的构建文档（openjdk/doc/building.html），也可以在互联网中寻求解决方案。构建一个可调试的虚拟机是探索虚拟机实现的第一步，也是必要的一步。

ubuntu编译jdk12

https://blog.csdn.net/qq_36434742/article/details/106670246

有报错的情况

```
./configure disable-warnings-as-errors --enable-debug --with-jvm-variants=server
```

回归测试

在openjdk/test/hotspot/jtreg/下新增测试文件TestDummy.java

```java
/*
 * @test TestDummy 
 * @summary Test whether flag -XX:+DummyPrint works correctly
 * @library /test/lib
 * @run main/othervm TestDummy 
 * @author kelthuzadx
 */
import jdk.test.lib.process.OutputAnalyzer;
import jdk.test.lib.process.ProcessTools;

public class TestDummy {
    static class Wrap{ public static void main(String... args){} }

    static void runWithFlag(boolean enableFlag) throws Throwable{
        ProcessBuilder pb = ProcessTools.createJavaProcessBuilder(
            enableFlag ? "-XX:+DummyPrint" : "-XX:-DummyPrint",
            Wrap.class.getName());
        OutputAnalyzer out = new OutputAnalyzer(pb.start());
        if(enableFlag){
            out.shouldContain("Hello World");
        }else{
            out.shouldNotContain("Hello World");
        }
	}
	public static void main(String[] args) throws Throwable{
        runWithFlag(true);  
        runWithFlag(false);
    }
}
```

自行构建jtreg或者下载预构建的jtreg，使用如代码清单1-8所示的命令进行测试：

```sh
 ./jtreg -jdk:<待测试的JDK路径> openjdk/test/hotspot/jtreg/TestDummy.java
Test results: passed: 1
```

# 类可用机制

## 类加载

### 字节码

​	由于字节码对于源码的描述是栈的形式，所以Java虚拟机属于栈式机器（Stack Machine）。与之相对的是寄存器机器（Register Machine）

​	寄存器机器的加法是直接使用add 0 0 -4指令完成的，它的操作数和指令组成一个整体，而栈式机器的iadd没有操作数，它隐式地假设了一个操作数栈，用于存放iadd需要的数据，这是两者的主要区别。

### 类加载器

​	类加载的过程位于ClassLoader::load_class()。

* BootStrap加载器
* Platform加载器
* Application加载器。



​	首先使用Application类加载器加载，然后调用Platform去加载，委派给BootStrap类去加载。

​	如果Bootstrap类加载器加载完成，那么加载任务就此终止。如果没有加载完成，它会将任务返还给Platform类加载器等待加载，如果Platform类加载器也无法加载则又会将任务返还给Application类加载器加载。

​	双亲委派加载模型避免了类被重复加载，而且保证了诸如java.lang.Object、java.lang.Thread等核心类只能被Bootstrap类加载器加载。

​	CLD结构

​	源码中很多CLD字样指的就是类加载器的数据。每个类加载器都有一个对应的CLD结构，这是一个重要的数据结构。

![](https://pic.imgdb.cn/item/608ce536d1a9ae528ff4d2d5.jpg)

​	CLD存放了所有被该ClassLoader加载的类、当前类加载器的Java对象表示、管理内存的metaspace等。另外CLD还指示了当前类加载器是否存活、是否需要卸载等。除此之外，CLD还有一个next字段指向下一个CLD，所有CLD连接起来构成一幅CLD图，即ClassLoaderDataGraph。通过调用ClassLoaderDataGraph::classes_do可以在垃圾回收过程中很容易地遍历该结构找到所有类加载器加载的所有类。

### 文件解析

​	ClassLoader::load_class()负责定位磁盘上字节码文件的位置，读取该文件的工作由类文件解析器ClassFileParser完成。

​	Java所有的类最终都继承自Object类，每个类的常量池都会包含诸如“[java/lang/Object;”的字符串。为了节省内存，HotSpot VM用Symbol唯一表示常量池中的字符串，所有Symbol统一存放到SymbolTable中。SymbolTable是一个并发哈希表，虚拟机会根据该表中Symbol的哈希值判断是返回已有的Symbol还是创建新的Symbol。

```c++
void ClassFileParser::parse_stream(const ClassFileStream* const stream,
                                   TRAPS) {

  assert(stream != NULL, "invariant");
  assert(_class_name != NULL, "invariant");

  // BEGIN STREAM PARSING
  stream->guarantee_more(8, CHECK);  // magic, major, minor
  // Magic value
  // 读取魔数 前32位0xcafebabe
  const u4 magic = stream->get_u4_fast();
  guarantee_property(magic == JAVA_CLASSFILE_MAGIC,
                     "Incompatible magic value %u in class file %s",
                     magic, CHECK);

  // Version numbers
  // 检查主版本号和次版本号
  _minor_version = stream->get_u2_fast();
  _major_version = stream->get_u2_fast();

  // Check version numbers - we check this even with verifier off
  verify_class_version(_major_version, _minor_version, _class_name, CHECK);

  // 读取常量池
  stream->guarantee_more(3, CHECK); // length, first cp tag
  // 读取常量池长度
  ...
  // 读取类和父类
  _this_class_index = stream->get_u2_fast();
  check_property(
    valid_cp_range(_this_class_index, cp_size) &&
      cp->tag_at(_this_class_index).is_unresolved_klass(),
    "Invalid this class index %u in constant pool in class file %s",
    _this_class_index, CHECK);
  
  Symbol* const class_name_in_cp = cp->klass_name_at(_this_class_index);
  ...
}
```

​	SymbolTable有个特别的地方：它使用引用计数管理Symbol。如果两个类常量池都包含字符串“hello world”，当两个类都卸载后该Symbol计数为0，且下一次垃圾回收的时候不会做可达性分析，而是直接清除。

​	在HotSpot VM中，SymbolTable还有个孪生兄弟StringTable。StringTable这个名字可能比较陌生，但是一定见过String.intern(), String.intern()底层依托的正是StringTable：

```c++
JVM_ENTRY(jstring, JVM_InternString(JNIEnv *env, jstring str))
  JvmtiVMObjectAllocEventCollector oam;
  if (str == NULL) return NULL;
  oop string = JNIHandles::resolve_non_null(str);
  oop result = StringTable::intern(string, CHECK_NULL);
  return (jstring) JNIHandles::make_local(THREAD, result);
JVM_END
```

​	String.intern()会返回一个字符串的标准表示。所谓标准表示是指对于相同字符串常量会返回唯一内存地址。StringTable则是用来存放这些标准表示的字符串的哈希容器。它没有使用引用计数管理，是众多类型的GC Root之一，在垃圾回收过程中会被当作根，以它为起点出发进行标记。

​	虚拟机用户可以使用参数-XX:+PrintStringTableStatistics在虚拟机退出时输出StringTable和SymbolTable的统计信息，或者使用`jcmd <pid> VM.stringtable`在运行时输出相关信息。

**字节码文件格式**

```c++
ClassFile {
    u4             magic;                  // 字节码文件魔数，0xcafebabe
    u2             minor_version;          // 主版本号
    u2             major_version;          // 次版本号
    u2             constant_pool_count;    // 常量池大小
	p_info       constant_pool[constant_pool_count-1]; // 常量池
    u2             access_flags;           // 该类是否public，是否final
    u2             this_class;             // 当前类在常量池的索引号
    u2             super_class;            // 父类在常量池的索引号
    u2             interfaces_count;       // 接口个数
    u2             interfaces[interfaces_count]; // 接口
    u2             fields_count;           // 字段个数
    field_info    fields[fields_count];    // 字段
    u2             methods_count;          // 方法个数
    method_info  methods[methods_count];   // 方法
    u2             attributes_count;       // 属性信息，比如是否内部类
    attribute_info attributes[attributes_count]; // 属性
}
```

## 类的链接

​	 HotSpot VM的执行模式是解释器与JIT编译器混合的模式，当一个Java方法/循环被探测到是“热点”，即执行了很多次时，就可能使用JIT编译器编译它然后从解释器切换到执行后的代码再执行它。那么，如何让方法同时具备可解释执行、可执行编译后的机器代码的能力呢？HotSpot VM的实现是在方法中放置解释器、编译器的入口地址，需要哪种模式就进入哪种入口。

​	在哪里设置这些入口呢？结合类的实现过程，在前面的类加载中没有提到，而后面的类初始化会执行代码，说明在执行代码时入口已设置，即它们是在类链接阶段设置的。类链接源码位于InstaceKlass::link_class_impl()，源码很长，主要有5个步骤：

* 字节码验证（verify_code）；
* 字节码重写（rewrite_class）；
* 方法链接（link_method）；
* 初始化vtable（虚表）和itable（接口表）；
* 链接完成（set_init_state）。

### 字节码验证

​	字节码验证可以确保字节码是结构性正确的。

​	感兴趣的读者请对照verifier源码和Java虚拟机文档4.9、4.10节（关于结构性正确的一些要求）阅读。

### 字节码重写

​	字节码重写器（Rewritter）位于interpreter/rewriter.cpp。

**finalize方法重写**

​	当某个类重写了Object.finalize()方法时，在运行时，最后一条字节码return会被重写器重写为`_return_register_finalizer`。这是一条非标准的字节码，在Java虚拟机规范中没有要求，是虚拟机独有的字节码，如果虚拟机在执行时发现是非标准的`_return_register_finalizer`，则会额外执行很多代码（代码清单2-7）：插入机器指令判断当前方法是否重写finalize，如果重写，则经过一个很长的调用链，最终调用`java.lang.ref.Finalizer`的register()。

```c++
instanceOop InstanceKlass::register_finalizer(...) {
    instanceHandle h_i(THREAD, i);  JavaValue result(T_VOID);
    JavaCallArguments args(h_i);
    // 对应java.lang.ref.Finalizer的register方法（该类为package-private）
    methodHandle mh (THREAD, Universe::finalizer_register_method());
    JavaCalls::call(&result, mh, &args, CHECK_NULL);
    return h_i();
}
```

​	register()会将重写了finalize()的对象放入一个链表，等待后面垃圾回收对链表每个对象执行finalize()方法。

**switch**

​	重写器会优化switch语句的性能。根据switch的case个数是否小于-XX:BinarySwitchThreshold（默认5）选择线性搜索switch或者二分搜索switch。

​	二分伪代码

```c++
int binary_search(int key, LookupswitchPair* array, int n) {
    int i = 0, j = n;
    while (i+1 < j) {
        int h = (i + j) >> 1;
        if (key < array[h].fast_match())
            j = h;
        else 
            i = h;
    }  
    return i;
}
```

### 方法链接

#### Method数据结构

​	OpenJDK 8以后的版本是用Method这个数据结构，在JVM层表示Java方法，位于oops/method.cpp，里面包含了解释器、编译器的代码入口和一些重要的用于统计方法性能的数据。

​	“HotSpot”的中文意思是“热点”，指的是它能对字节码中的方法和循环进行Profiling性能计数，找出热点方法或循环，并对其进行不同程度的优化。这些Profiling数据就存放在MethodData和MethodCounter中。

​	Method另一个重要的字段是`_intrinsic_id`。如果某方法的实现广为人知，或者某方法另有高效算法实现，对于它们，即便使用JIT编译性能也达不到最佳。为了追求极致的性能，可以将这些方法视作固有方法（Intrinsic Method）或者知名方法（Well-known Method），解放CPU指令集中所有支持的指令，由虚拟机工程师手写它们的实现。`_intrinsic_id`表示固有方法的id，如果该id有效，即该方法是固有方法，即便方法有对应的Java实现，虚拟机也不会走普通的解释执行或者编译Java方法，而是直接跳到该方法对应的手写的固有方法实现例程并执行。

​	所有固有方法都能在classfile/vmSymbols.hpp中找到，一个绝佳的例子是java.lang.Math。对于Math.sqrt()，用Java或者JNI均无法达到极致性能，这时可以将其置为固有方法，当虚拟机遇到它时只需要一条CPU指令fsqrt（代码清单2-9），用硬件级实现碾压软件级算法：

```c++
/ 32位：使用x87的fsqrt
void Assembler::fsqrt() {
    emit_int8((unsigned char)0xD9);
    emit_int8((unsigned char)0xFA); 
}
// 64位：使用SSE2的sqrtsd
void Assembler::sqrtsd(XMMRegister dst, XMMRegister src) {
    ...
    int encode = simd_prefix_and_encode(...);
    emit_int8(0x51);
    emit_int8((unsigned char)(0xC0 | encode));
}
```

#### 编译器、解释器入口

​	Method的其他数据字段会在后面陆续提到，目前方法链接需要用到的数据只是图2-2右侧的各个入口地址，具体如下所示。

* `_i2i_entry`：定点解释器入口。方法调用会通过它进入解释器的世界，该字段一经设置后面不再改变。通过它一定能进入解释器。
* `_from_interpreter_entry：`解释器入口。最开始与_i2i_entry指向同一个地方，在字节码经过JIT编译成机器代码后会改变，指向i2c适配器入口。
* `_from_compiled_entry：`编译器入口。最开始指向c2i适配器入口，在字节码经过编译后会改变地址，指向编译好的代码。
* ` _code`: 代码入口。当编译器完成编译后会指向编译后的本地代码。

![](https://pic.imgdb.cn/item/608d13e8d1a9ae528f20a35e.jpg)

```c++
void Method::link_method(const methodHandle& h_method, TRAPS) {
  // If the code cache is full, we may reenter this function for the
  // leftover methods that weren't linked.
  if (_i2i_entry != NULL) {
    return;
  }
  assert( _code == NULL, "nothing compiled yet" );

  // Setup interpreter entrypoint
  assert(this == h_method(), "wrong h_method()" );

  assert(adapter() == NULL, "init'd to NULL");
  address entry = Interpreter::entry_for_method(h_method);
  assert(entry != NULL, "interpreter entry must be non-null");
  // Sets both _i2i_entry and _from_interpreted_entry
  set_interpreter_entry(entry);

  // Don't overwrite already registered native entries.
  if (is_native() && !has_native_function()) {
    set_native_function(
      SharedRuntime::native_method_throw_unsatisfied_link_error_entry(),
      !native_bind_event_is_interesting);
  }

  // Setup compiler entrypoint.  This is made eagerly, so we do not need
  // special handling of vtables.  An alternative is to make adapters more
  // lazily by calling make_adapter() from from_compiled_entry() for the
  // normal calls.  For vtable calls life gets more complicated.  When a
  // call-site goes mega-morphic we need adapters in all methods which can be
  // called from the vtable.  We need adapters on such methods that get loaded
  // later.  Ditto for mega-morphic itable calls.  If this proves to be a
  // problem we'll make these lazily later.
  (void) make_adapters(h_method, CHECK);

  // ONLY USE the h_method now as make_adapter may have blocked
}
```

​	各种入口的地址不会是一成不变的，当编译/解释模式切换时，入口地址也会相应切换，如从解释器切换到编译器，编译完成后会设置新的_code、_from_compiled_entry和_from_interpreter_entry入口；如果发生退优化（Deoptimization），从编译模式回退到解释释模式，又会重置这些入口。

```c++
void Method::set_code(const methodHandle& mh, CompiledMethod *code) {
  assert_lock_strong(CompiledMethod_lock);
  assert( code, "use clear_code to remove code" );
  assert( mh->check_code(), "" );

  guarantee(mh->adapter() != NULL, "Adapter blob must already exist!");

  // These writes must happen in this order, because the interpreter will
  // directly jump to from_interpreted_entry which jumps to an i2c adapter
  // which jumps to _from_compiled_entry.
  // 设置编译好的代码
  mh->_code = code;             // Assign before allowing compiled code to exec

  int comp_level = code->comp_level();
  // In theory there could be a race here. In practice it is unlikely
  // and not worth worrying about.
  if (comp_level > mh->highest_comp_level()) {
    mh->set_highest_comp_level(comp_level);
  }

  OrderAccess::storestore();
  // 设置解释器入口点为编译后的机器代码
  mh->_from_compiled_entry = code->verified_entry_point();
  OrderAccess::storestore();
  // Instantly compiled code can execute.
  if (!mh->is_method_handle_intrinsic())
    mh->_from_interpreted_entry = mh->get_i2c_entry();
}
void Method::clear_code() {
  // this may be NULL if c2i adapters have not been made yet
  // Only should happen at allocate time.
   // 清除_from_interpreted_entry，使其再次指向c2i适配器
  if (adapter() == NULL) {
    _from_compiled_entry    = NULL;
  } else {
    _from_compiled_entry    = adapter()->get_c2i_entry();
  }
  OrderAccess::storestore();
  _from_interpreted_entry = _i2i_entry;
  OrderAccess::storestore();
  // 取消指向机器代码
  _code = NULL;
}
```

#### C2I/I2C适配器

​	在上述代码中多次提到c2i、i2c适配器，如图2-3所示。所谓c2i是指编译模式到解释模式（Compiler-to-Interpreter），i2c是指解释模式到编译模式（Interpreter-to-Compiler）。由于编译产出的本地代码可能用寄存器存放参数1，用栈存放参数2，而解释器都用栈存放参数，需要一段代码来消弭它们的不同，适配器应运而生。它是一段跳床（Trampoline）代码，以i2c为例，可以形象地认为解释器“跳入”这段代码，将解释器的参数传递到机器代码要求的地方，这种要求即调用约定（Calling Convention），然后“跳出”到机器代码继续执行。

![](https://pic.imgdb.cn/item/608d16d4d1a9ae528f40b305.jpg)

![](https://pic.imgdb.cn/item/608d177bd1a9ae528f479493.jpg)

void SharedRuntime::gen_i2c_adapter()方法

![](https://pic.imgdb.cn/item/608d1c66d1a9ae528f746b24.jpg)

#### CDS

​	方法链接还有个细节：在设置入口前，它会区分该方法是否是CDS（Class Data Sharing，类数据共享）方法，并据此设置不同的解释器入口。

​	CDS是JDK5引入的特性，它把最常用的类从内存中导出形成一个归档文件，在下一次虚拟机启动可使用mmap/MapViewOfFile等函数将该文件映射到内存中直接使用而不再加载解析这些类，以此加快Java程序启动。如果有多个虚拟机运行，还可以共享该文件，减小内存消耗。

​	但是CDS只允许Bootstrap类加载器加载类共享文件，适用场景非常有限，所以JEP 310于Java 10引入了新的AppCDS（Application Class Data Sharing，应用类数据共享），让Application类加载器和Platform类加载器甚至自定义类加载器也能拥有CDS。

​	AppCDS对于快速启动、快速执行、立即关闭的应用程序有不错的效果，

```sh
$java -Xshare:off -XX:DumpLoadedClassList=class.lit HelloWorld
$java -Xshare:dump -XX:SharedClassListFile=class.list -XX:SharedArchiveFile=hello.jsa HelloWorld
$java -Xshare:on -XX:SharedArchiveFile=hello.jsa HelloWorld
```

AppCDS并不是故事的全部，它虽然可以导出更多类，但是使用比较麻烦，需要三步：

* 运行第一次，生成类列表;
* 运行第二次，根据类列表从内存中导出类到归档文件;
* 附带归档文件运行第三次。



​	JEP 350于Java 13引入了DynamicCDS，它可以消除AppCDS的第一步，在第一次运行程序退出时将记录了本次运行加载的CDS没有涉及的类自动导出到归档文件，第二次直接附带归档文件运行即可。

## 类的初始化

​	类可用三部曲的最后一步是类初始化。《Java虚拟机规范》的第5章对初始化流程有非常详尽的描述，指出整个类的初始化流程有12步。

1. 获取类C的初始化锁LC。
2. 如果另外一个线程正在初始化C，那么释放锁，阻塞当前进线程，知道另一个线程初始化完成。
3. 如果当前线程正在初始化C，那么释放LC。
4. 如果C早已初始化，不需要做什么，释放LC。
5. 如果C处于错误状态，初始化不可能完成，则释放LC并报出NoClassDefFoundError。
6. 否则，标示当前线程正在初始化C，释放LC。然后初始化每个final static常量字段，初始化顺序遵照代码写的顺序。
7. 下一步，如果C是类而不是借口，初始化父类和父接口。
8. 下一步，查看C的断言是否开启。
9. 下一步，执行类或接口的初始化方法。
10. 如果初始化完成，那么获取锁LC，标识C已经完成初始化，通知所有等待线程，然后释放LC。
11. 否则，初始化一定会遇到类问题，抛出异常E。如果类E是Error或它的子类，那么创建一个ExceptionInitializationError对象，将E作为参数，然后该对象替代下一步的E。如果因为OutOfMemoryError原因不能创建ExceptionInitializationError实例，则使OutOfMemoryError实例作为下一步的E的替代品。
12. 获取LC，表示C为错误状态，通知所有线程，然后释放LC，以上一步的E作为本步的终止。



​	为了通用性和抽象性，可能《Java虚拟机规范》在语言描述方面比较学究。要想直观了解类初始化过程，可以阅读InstanceKlass::initialize_impl()源码实现。

​	不难看出，上面步骤很多都是为了处理错误和异常情况，真正意义上的初始化其实是第9步

```c++
void InstanceKlass::initialize_impl(TRAPS) {
    ...
    // Step 8 （虚拟机文档的第9步对应源码第8步，因为源码省略了文档第8步的处理）
    call_class_initializer(THREAD);
}
void InstanceKlass::call_class_initializer(TRAPS) {
    // 如果启用了编译重放则跳过初始化
	if (ReplayCompiles && ...){
        return;
    }
    // 获取初始化方法，包装成一个methodHandle
    methodHandle h_method(THREAD, class_initializer());
    // 调用初始化方法
    if (h_method() != NULL) {
        JavaCallArguments args; // <clinit>无参数
        JavaValue result(T_VOID);
        JavaCalls::call(&result, h_method, &args, CHECK); 
    }
}
```

​	类初始化首先会判断是否开启了**编译重放**（Replay Compile）。使用“-XX:CompileCommand=option,ClassName::MethodName,DumpInline”可以将一个方法的编译信息存放到文件，这样就可以在下一次运行时使用-XX:+ReplayCompiles -XX:ReplayDataFile=file从文件读取编译数据，并创建编译任务投入编译队列，然后进入阻塞状态，在编译完成后继续执行程序。这种“第一次运行存放编译任务→第二次运行获取编译任务→第二次执行编译”的过程就是编译重放。

​	编译重放固定了编译顺序，而固定的编译顺序减少了虚拟机的不确定性，可用于JIT编译器性能数据分析和GC性能数据分析等场景。除此之外，虚拟机参数`-XX:ReplaySuppressInitializers=<val>`的值还可以控制类初始化行为：

* 0：不做特殊处理；
* 1：将所有类初始化代码视为空；
* 2：将所有应用程序类初始化代码视为空；
* 3：允许启动时运行类初始化代码，但是在重放编译时忽略它们。

处理了编译重放后，虚拟机会调用class_initializer()函数，该函数返回当前类的`<clinit>`方法。类的构造函数和静态代码块在虚拟机中有特殊的名字，前者是`<init>`，后者则是`<clinit>`。

​	Java编译器会将静态字段的初始化代码也放入`<clinit>`，所以字段k和字段obj的赋值都是在类初始化阶段完成的，也正是因为赋值操作需要真实的执行代码，所以需要在链接阶段提前设置解释器入口，以便初始化代码的执行。在确认class_initializer()返回的当前类的`<clinit>`方法存在后，虚拟机会将其包装成methodHandle送入JavaCalls::call执行。

​	虚拟机和Java沟通的两座桥梁是JNI和JavaCalls，Java层使用JNI进入JVM层，而JVM层使用JavaCalls进入Java层。JavaCalls可以在HotSpot VM中调用Java方法，main方法执行也是使用这种JavaCalls实现的。

## 类重定义

​	加载、链接、初始化是标准的类可用机制，除此之外，Java提供了一个用于特殊场景的类重定义功能，由JDK 5引入的java.lang.instrument.Instrumentation实现。

​	Instrumentation可以在应用程序运行时修改或者增加类的字节码，然后替换原来的类的字节码，这种方式又称为热替换

```java
// Num.java
public class Num {
    public int getNum() { return 3; }
}
// java -javaagent:AgentMain.jar ...
import java.lang.instrument.Instrumentation;
public class AgentMain { 
    public static void premain(String args, Instrumentation inst) {
        inst.addTransformer((loader, className, classBeingRedefined, 
                             protectionDomain, byteCode) -> {
            // 修改Num.getNum()的字节码，使它返回1
            if("Num".equals(className)){ byteCode[261] = 4; }
            return byteCode;
        });
        try {
            inst.retransformClasses(Num.class);
        } catch (UnmodifiableClassException e) {
            e.printStackTrace();
        }
    } 
}
```

​	如果将Instrumentation与asm、cglib、Javaassist等字节码增强框架结合使用，开发者可以灵活地在运行时修改任意类的方法实现，这样无须修改源代码，也无须重编译运行就能改变方法的行为，达到近似热更新的效果。

​	如果类字节码转换器没有修改字节码，正确的做法是返回null，如果修改了字节码，应该创建一个新的byte[]数组，将原来的byteCode复制到新数组中，然后修改新数组，而不是像代码清单2-16一样修改原有的byteCode再返回。这样直接修改byteCode可能会造成虚拟机崩溃的情况。

​	Instrumentation的底层实现是基于JVMTI（Java虚拟机工具接口）的RedefineClasses。虚拟机创建VM_RedefineClasses，投递给VMThread，然后等待VMThread执行VM_RedefineClasses::redefine_single_class重定义一个类。类的重定义是一个烦琐的过程，它会移除原来类（the_class）中的所有断点，所有依赖原来类的编译后的代码都需要进行退优化，原来类的方法、常量池、内部类、虚表、接口表、调试信息、版本号、方法指纹等数据也会一并被替换为新的类定义（scratch_class）中的数据。

# 对象和类

## 对象和类

​	HotSpot VM使用oop描述对象，使用klass描述类，这种方式被称为对象类二分模型。理解对象类二分模型最好的方法是回归到编程语言本身来看。HotSpot VM是用C++编写的，C++的类是一个强大的抽象工具，HotSpot VM需要借助这个强大的工具，对Java各个方面做一个抽象。换句话说，用一个C++类描述一个Java语言组件。

![](https://pic.imgdb.cn/item/608d272fd1a9ae528fc7ef06.jpg)

​	普通对象（new Foo）是instanceOop，普通数组（new int[]）是typeArrayOop，对象数组（new Bar[]）是objArrayOop。这些类都继承自oop类，如果查看HotSpot VM源码会发现没有oop、instanceOop、objArrayOop等类，只有oopDesc、instanceOopDesc、objArrayOopDesc，其实后两者是一回事，instanceOop只是instanceOopDesc指针的别名（typedef）。Java层面的类、接口、枚举会被抽象成C++的klass类。对象的类（Foo.class）是instanceKlass，对象数组的类（Bar[].class）是objArrayKlass，普通数组的类（int[].class）是typeArrayKlass。

​	Java对象在虚拟机表示中除了字段外还有个对象头，里面有一个字段记录了对象的GC年龄、hash值等信息，这个字段被命名为markOop。另外，java.lang.ref.Reference及其子类不是用InstanceKlass描述而是用InstanceRefKlass描述，它们会被GC特殊对待。与之类似，java.lang.ClassLoader用InstanceClassLoaderKlass描述，java.lang.Class用InstanceMirrorKlass描述。以上便是对象和类的相关内容，它们的源码位于hotspot/share/oops，本章剩下的部分将首先讨论表示对象的oop，然后讨论表示类的klass。

## 对象

​	oop的全称是Ordinary Object Pointer，它来源于Smalltalk和Self语言，字面意思是“普通对象指针”，在HotSpot VM中表示受托管的对象指针。

### 创建对象

​	创建oop的蓝图是InstanceKlass。InstanceKlass了解对象所有信息，包括字段个数、大小、是否为数组、是否有父类，它能根据这些信息调用InstanceKlass::allocate_instance创建对应的instanceOop/arrayOop。

```c++
instanceOop InstanceKlass::allocate_instance(TRAPS) {
  // 是否重写finalize方法
  bool has_finalizer_flag = has_finalizer(); // Query before possible GC
  // 获取对象大小
  int size = size_helper();  // Query before forming handle.
  instanceOop i;
  // 在堆上分配对象
  i = (instanceOop)Universe::heap()->obj_allocate(this, size, CHECK_NULL);
  if (has_finalizer_flag && !RegisterFinalizersAtInit) {
    i = register_finalizer(i, CHECK_NULL);
  }
  return i;
}
inline oop CollectedHeap::obj_allocate(Klass* klass, int size, TRAPS) {
  ObjAllocator allocator(klass, size, THREAD);
  return allocator.allocate();
}
```

```c++
oop MemAllocator::allocate() const {
  oop obj = NULL;
  {
    Allocation allocation(*this, &obj);
    // 根据对象size分配一片内存
    HeapWord* mem = mem_allocate(allocation);

    if (mem != NULL) {
      // 初始化对象头
      obj = initialize(mem);
    } else {
      // The unhandled oop detector will poison local variable obj,
      // so reset it to NULL if mem is NULL.
      obj = NULL;
    }
  }
  return obj;
}
```

有很多方法可以查看oop对象布局，了解它有助于深刻理解HotSpot VM的对象实现。使用`-XX:+PrintFieldLayout`虚拟机参数可以输出对象字段的偏移，但是该参数的输出内容比较简略。要想获取详细的对象布局，可以使用JOL（Java Object Layout）工具，但JOL不是JDK自带的工具，需要自行下载。除了JOL外，还可以使用JDK自带的jhsdb工具获取。使用jhsdb hsdb命令打开HotSpot Debugger程序，可以查看oop的内部数据

![image-20210503125917969](C:\Users\81929\AppData\Roaming\Typora\typora-user-images\image-20210503125917969.png)

### 对象头

​	oop是指向一片内存的指针，只是将这片内存‘视作’（强制类型转换）Java对象/数组。

```c++
class oopDesc {
    volatile markOop _mark;
    union _metadata {
        Klass*      _klass;
        narrowKlass _compressed_klass;
    } _metadata;
};
```

​	对象头的第一个字段是_mark，也叫Mark Word，虽然由于历史原因带了个oop字样，但是它与oop并没有关系。它在形式上是一个指针，但是HotSpot VM把它当作一个整数来使用。

​	根据CPU位数，markOop表现为32位或者64位整数，不同位（bit）有不同意义。

![](https://pic.imgdb.cn/item/608f85c6d1a9ae528f26569f.jpg)

​	使用VM参数-XX:+UseCompressedOops还可以开启对象指针压缩，在64位机器上开启该参数后，可以用32位无符号整数值（narrowOop）来表示oop指针。压缩对象指针允许32位整数表示64位指针。对象引用位数的减少允许堆中存放更多的其他数据，继而提高内存利用率，但是随之而来的问题是64位指针的可寻址范围可能是0～242字节或0～248字节（一般64位CPU的地址总线到不了64位），压缩后只能寻址0～232字节，显然无法覆盖可能的内存范围。对于这个问题，HotSpot VM的应对方案如图3-4所示，其中压缩对象指针有三种寻址模式：

​	![](https://pic.imgdb.cn/item/608f86f5d1a9ae528f31a984.jpg)



* 如果堆的高位地址小于32GB，说明不需要基址（base）就能定位堆中任意对象，这种模式也叫作零地址Oop压缩模式（Zero-based Compressed Oops Mode）；
* 如果堆的高位大于等于32GB，说明需要基址，这时如果堆大小小于4GB，说明基址+偏移能定位堆中任意对象；
* 如果堆大小处于4～32GB，这时只能通过基址+偏移×缩放（scale）才能定位堆中任意对象。



​	这三种寻址模式最大支持32GB的堆，很显然，如果Java堆大于32GB，那么将无法使用压缩对象指针。

​	对象头的第二个字段`_metadata`表示对象关联的类（klass）。它是union类型，`_klass`表示正常的指针，另一个narrowKlass是针对64位CPU的优化。如果开启-XX:+UseCompressedClassPointers，虚拟机会将指向klass的指针压缩为一个无符号32位整数（`_compressed_klass`），剩下的32位则用于存放对象字段数据，如果是typeArrayOop或objArrayOop，还能存放数组长度。但是压缩klass指针也会遇到和压缩对象指针一样的问题，即寻址范围无法覆盖可能的内存区域，对此，HotSpot VM的解决方案也是使用基址+偏移×缩放进行定位，只是这时候32位无符号整数偏移是narrowKlass而不是narrowOop。

### 对象哈希值

​	_mark中有一个hash code字段，表示对象的哈希值。每个Java对象都有自己的哈希值如果没有重写Object.hashCode()方法，那么虚拟机会为它自动生成一个哈希值。

​	Java层调用Object.hashCode()或者System.identityHashCode()，最终会调用虚拟机层的runtime/synchronizer的get_next_hash()生成哈希值。get_next_hash内置六种可选方案，如代码清单3-4所示，可以使用-XX:hashCode=<val>指定生成策略。OpenJDK 12目前默认的策略是Marsaglia XOR-Shift随机数生成器，它通过重复异或和位移自身值，可以得到一个很长的随机数序列周期，生成的随机数序列通过了所有随机性测试。另外，它的速度也非常快，能达到每秒2亿次。

```c++
static inline intptr_t get_next_hash(Thread* current, oop obj) {
  intptr_t value = 0;
  if (hashCode == 0) {
    // This form uses global Park-Miller RNG.
    // On MP system we'll have lots of RW access to a global, so the
    // mechanism induces lots of coherency traffic.
    value = os::random();
  } else if (hashCode == 1) {
    // This variation has the property of being stable (idempotent)
    // between STW operations.  This can be useful in some of the 1-0
    // synchronization schemes.
    intptr_t addr_bits = cast_from_oop<intptr_t>(obj) >> 3;
    value = addr_bits ^ (addr_bits >> 5) ^ GVars.stw_random;
  } else if (hashCode == 2) {
    value = 1;            // for sensitivity testing
  } else if (hashCode == 3) {
    value = ++GVars.hc_sequence;
  } else if (hashCode == 4) {
    value = cast_from_oop<intptr_t>(obj);
  } else {
    // Marsaglia's xor-shift scheme with thread-specific state
    // This is probably the best overall implementation -- we'll
    // likely make this the default in future releases.
    unsigned t = current->_hashStateX;
    t ^= (t << 11);
    current->_hashStateX = current->_hashStateY;
    current->_hashStateY = current->_hashStateZ;
    current->_hashStateZ = current->_hashStateW;
    unsigned v = current->_hashStateW;
    v = (v ^ (v >> 19)) ^ (t ^ (t >> 8));
    current->_hashStateW = v;
    value = v;
  }

  value &= markWord::hash_mask;
  if (value == 0) value = 0xBAD;
  assert(value != markWord::no_hash, "invariant");
  return value;
}
```

## 类

​	Klass是一个抽象基类，它定义了一些接口（纯虚函数），由InstanceKlass继承并实现这些接口，两者结合可以描述一个Java类的方法有哪些、字段有哪些、父类是否存在等。Klass提供了相当多的关于类的信息，同样可以使用HotSpot Debugger可视化。

![](https://pic.imgdb.cn/item/608f9c3ed1a9ae528f2c36b8.jpg)

​	InstanceKlass在虚拟机层描述大部分的Java类，但有少部分Java类有特殊语意：普通类的对象在垃圾回收过程中只需要遍历所有实例字段；java.lang.Class的对象需要遍历实例字段和静态字段；`java.lang.ref.*`的对象需要处理被引用对象；java.lang.ClassLoader需要处理类加载数据。这些类的特殊行为不能用InstanceKlass统一表示，因此InstanceKlass之下派生出InstanceMirrorKlass描述java.lang.Class类，InstanceRefKlass描述java.lang.ref.*类，InstanceClassLoaderKlass描述java.lang.ClassLoader类。

### 字段

​	在垃圾回收过程中常见的任务是遍历一个对象的所有字段。

```c++
inline void G1FullGCMarker::follow_object(oop obj) {
  assert(_bitmap->is_marked(obj), "should be marked");
  if (obj->is_objArray()) { // 如果是数组，则标记每个数组成员
    // Handle object arrays explicitly to allow  them to
    // be split into chunks if needed.
    follow_array((objArrayOop)obj);
  } else { // 否则标记对象的每个非静态数据成员
    obj->oop_iterate(mark_closure());
    if (VerifyDuringGC) {
      if (obj->is_instance() && InstanceKlass::cast(obj->klass())->is_reference_instance_klass()) {
        return;
      }
      _verify_closure.set_containing_obj(obj);
      obj->oop_iterate(&_verify_closure);
      if (_verify_closure.failures()) {
        log_warning(gc, verify)("Failed after %d", _verify_closure._cc);
        assert(false, "Failed");
      }
    }
  }
}
// 该方法由上面的obj->oop_iterate调用
ALWAYSINLINE void InstanceKlass::oop_oop_iterate_oop_maps(...) {
    OopMapBlock* map             = start_of_nonstatic_oop_maps();
    OopMapBlock* const end_map = map + nonstatic_oop_map_count();
    for (; map < end_map; ++map) {     // 遍历每个OopMapBlock
        oop_oop_iterate_oop_map<T>(map, obj, closure); 
    }
}
```

​	调用obj->oop_iterate后经过一个较长的调用链，会执行oop_oop_iterate_oop_maps，根据代码不难看出它的行为是获取开始OopMapBlock和结束OopMapBlock，然后遍历这些OopMapBlock。OopMapBlock存储了该对象的字段偏移和个数，分别用offset和count表示。offset表示第一个字段相对于对象头的偏移，count表示对象有多少个字段。另外，如果有父类，则再用一个OopMapBlock表示父类，因此通过遍历对象的所有OopMapBlock就能访问对象的全部字段。

### 虚表

​	每个Java对象即oop都有对象头，对象头里面有一个`_klass`指向对象的正确的InstanceKlass类型，而InstanceKlass包含了类的所有方法以及父类信息，当执行n.print()时，JVM可以（但是并没有）从对象n的对象头里取出_klass，找到描述AddNode类的InstanceKlass，再在其中寻找print方法。这一过程并不需要虚表参与。正如上面讨论的，虚表是Java动态派发的优化而不是必要组件，就像native入口之于Method，Java的虚表也是位于InstanceKlass之外。

![](https://pic.imgdb.cn/item/608f9f41d1a9ae528f4f1641.jpg)

```c++
void klassVtable::initialize_vtable(bool checkconstraints, TRAPS) {
    ...
    // 处理当前类的所有方法
    for(int i = 0; i < len; i++) {
        methodHandle mh(THREAD, methods->at(i));
        // 该方法是否为虚方法
        bool needs_new_entry = update_inherited_vtable(...);
        // 如果是，则需要更新当前类的虚表索引
        if (needs_new_entry) {
        put_method_at(mh(), initialized);
        mh()->set_vtable_index(initialized); 
        initialized++;
        }
    }
    ...// 和前面类似，处理default方法
}
```

​	update_inherited_vtable会经过一系列检查来确定一个方法是否为虚方法以及是否需要加入类的虚表。上述例子的Node与AddNode经过虚表初始化后的vtable如图所示。

![](https://pic.imgdb.cn/item/608f9f41d1a9ae528f4f1641.jpg)

​	也可以开启VM参数-Xlog:vtables=trace查看所有类的虚表的创建过程。在调用虚方法时虚拟机会在运行时常量池中查找n的静态类型Node的print方法，获取它在Node虚表中的index，接着用index定位动态类型AddNode虚表中的虚方法进行调用。第一步的运行时常量池在HotSpot VM中的表示是oops/ConstantPoolCache，它也是对象和类模型的一部分。

​	gcc使用-fdump-class-hierarchy输出虚表，clang使用-Xclang -fdump-vtable-layouts输出虚表，msvc使用/d1reportAllClassLayout输出虚表。

# 运行时

## 线程创生纪

​	![](https://pic.imgdb.cn/item/608fa897d1a9ae528fba318a.jpg)

* Thread：线程基类，定义所有线程都具有的功能。
* JavaThread：Java线程在虚拟机层的实现。
* NonJavaThread：相比Thread只多了一个可以遍历所有NonJavaThread的能力。
* ServiceThread：服务线程，会处理一些杂项任务，如检查内存过低、JVMTI事件发生。
* JvmtiAgentThread：JVMTI的RunAgentThread()方法启动的线程。
* CompilerThread：JIT编译器线程。
* CodeCacheSweeperThread：清理Code Cache的线程。
* WatcherThread：计时器（Timer）线程。
* JfrThreadSampler：JFR数据采样线程。
* VMThread：虚拟机线程，会创建其他线程的线程，也会执行GC、退优化等。
* ConcurrentGCThread：与WorkerThread及其子类一样，都是为GC服务的线程。





​	当使用命令行工具java启动应用程序时，操作系统会定位到java启动器的main函数，java启动器调用JavaMain完成一个程序的生命周期。

```c++
int JNICALL JavaMain(void * _args){
    ...
    // 初始化Java虚拟机
    if (!InitializeJVM(&vm, &env, &ifn)) {
        JLI_ReportErrorMessage(JVM_ERROR1);
        exit(1);
    }
    ...
    // 加载main函数所在的类
    mainClass = LoadMainClass(env, mode, what);
    CHECK_EXCEPTION_NULL_LEAVE(mainClass);
    // 对GUI程序的支持
    appClass = GetApplicationClass(env);
    mainArgs = CreateApplicationArgs(env, argv, argc);
    if (dryRun) {
ret = 0; LEAVE();
    }
    PostJVMInit(env, appClass, vm);
    // 获取main方法id
    mainID = (*env)->GetStaticMethodID(env, mainClass, "main",
                                       "([Ljava/lang/String;)V");
    // main方法调用
    (*env)->CallStaticVoidMethod(env, mainClass, mainID, mainArgs);
    // 启动器的返回值（非System.exit退出）
    ret = (*env)->ExceptionOccurred(env) == NULL ? 0 : 1;
    LEAVE();
}
```

​	InitializeJVM会调用JNI函数JNI_CreateJavaVM初始化虚拟机。JNI_CreateJavaVM又会将初始化虚拟机的任务委派给Threads::create_vm。Threads::create_vm是虚拟机的创生纪，几乎所有HotSpot VM组件都会在这一步初始化和创建，有关初始化的问题大部分都能在这找到答案。

### 容器化支持

​	近几年容器技术越来越流行，作为云原生的技术基石，得到很多应用和服务的广泛应用。容器使用	cgroup限制CPU、内存资源，但是Java 8早期并没有对容器提供支持（Java 10提供了对Linux容器的支持，并backport[1]到Java 8，所以最新的Java 8也支持容器），所以当在容器中运行JVM时，它会忽略cgroup施加的限制，错误地“看到”宿主机的所有CPU和内存资源，可能造成一些问题。
Java 10提供了对容器的支持，使用-XX:+UseContainerSupport开启容器支持后，由Threads::create_vm调用OSContainer::init()检查虚拟机是否运行在容器中，如果是则读取容器所施加的资源限制，并据此设置默认的GC线程数、堆大小等。

### Java线程

​	JVM会创建一个JavaThread数据结构，然后由record_stack_base_and_size()将当前操作系统线程（即执行Threads::create_vm代码的线程）的栈顶地址和栈大小保存到JavaThread中，由set_as_starting_thread()将当前操作系统线程的id保存到JavaThread中，这样一来JavaThread就可以代表当前操作系统线程了。

​	注意，由于当前操作系统线程后续会解释字节码，而Java main方法会通过字节码解释执行的，因此执行Java main的线程是Java主线程，这里创建的JavaThread数据结构也就是常说的Java主线程。

```c++
jint Threads::create_vm(JavaVMInitArgs* args, bool* canTryAgain) {
    ...
    // 创建Java主线程，附加到当前线程
    JavaThread* main_thread = new JavaThread();
    main_thread->set_thread_state(_thread_in_vm);
    main_thread->initialize_thread_current();
    main_thread->record_stack_base_and_size();
    main_thread->register_thread_stack_with_NMT();
    main_thread->set_active_handles(JNIHandleBlock::allocate_block()
    if (!main_thread->set_as_starting_thread()) {
     ... // 主线程启动失败，虚拟器退出
    }
    // 栈保护页创建
    main_thread->create_stack_guard_pages();
    { // 将Java主线程加入全局线程链表，供后续使用
        MutexLocker mu(Threads_lock);
        Threads::add(main_thread);
    }
    ...
}
```

​	除了要防止栈溢出破坏栈之外的数据结构，语言运行时还会保留最大栈上限所在的一片区域，即保护页（Guard Page），又叫哨兵值（Sentry）、金丝雀（Canary）。当函数返回时检查保护页的值，如果被修改，说明已经到达最大栈上限，此时要终止程序并输出错误。

​	Java也有栈溢出，发生时会抛出StackOverflowError，输出调用栈和代码行数。这些过程都需要额外执行很多方法，但是发生栈溢出就意味着不能继续执行方法了（因为方法执行需要栈空间）。为了解决这个问题，HotSpot VM在C++语言运行时提供的保护页（Linux的JavaThread没有）之外会使用create_stack_guard_pages()创建额外的保护页来支持栈溢出错误处理。

![](https://pic.imgdb.cn/item/608fbb04d1a9ae528f69dbe7.jpg)

​	线程栈的最大上限处会保留三块保护页（Guard Page）支持栈溢出，分别是Reserved Page、Yellow Page、Red Page。

1）Reserved Page：Reserved Page源于JEP 270[2]，旨在为一些关键段（Critical Section）方法保存外栈空间，让有@jdk.internal.vm.annotation.ReservedStackAccess注解的方法能完成执行（如lock与unlock之间的代码），防止关键段方法中的对象出现不一致的状态。当执行关键段方法时分配的栈顶触及Reserved Page，则虚拟机会将Reserved Page标记为正常栈空间，供关键段方法完成执行，然后再抛出StackOVerflowError。Reserved Page的大小由`-XX:+StackReservedPages=<val>`指定。

2）Yellow Page：如果执行Java代码时分配的栈顶触及Yellow Page，则虚拟机会抛出StackOverflowError，然后将Yellow Page标为正常栈空间，让抛异常的代码有栈可用。Yellow Page的数量由参数`-XX:StackYellowPages=<val>`指定，最后Yellow Page占用的空间是page数量*page大小（page的大小一般是4KB，如果开启-XX:+UseLargePages且操作系统支持large page特性，page的大小可达到4MB)。

3）Red Page：如果执行Java代码时分配的栈顶触及Red Page，则虚拟机会创建错误日志`hs_err_pid<pid>.log`然后关闭虚拟机。同样，为了让创建日志的代码执行，虚拟机会将Red Page标为正常栈空间。Red Page的大小由`-XX:StackRedPages=<val>`指定。

4）Shadow Page：前面区域都是执行Java代码出现栈溢出的错误处理。虚拟机还可能执行native方法或者虚拟机本身需要执行的方法，这些方法的栈大小不像Java代码一样能确定（编译器能确定但是虚拟机不能），如果开启虚拟机参数-XX:+UseStackBanging，JVM会分配一块足够大的Shadow Page执行，如果RSP（栈顶指针）超出Shadow Page区则抛出StackOverflowError。

### 虚拟机线程

​	紧接着使用VMThread::create()创建VMThread数据结构以及它对应的VMOperation队列，VMThread即虚拟机线程，它是一个相当重要的线程。与前面的JavaThread一样，VMThread只是一个数据结构，要想发挥“可运行”的线程的能力，需要关联一个真正的线程，这个线程就是操作系统线程（OSThread）。os::create_thread会创建一个新的OSThread然后关联到VMThread。在创建了新的OSThread后，主线程会将它设置为ALLOCATED状态然后阻塞，直到新创建的OSThread完成初始化操作并设置为INITIALIZED，

```c++
jint Threads::create_vm(JavaVMInitArgs* args, bool* canTryAgain) {
    ...
{ // 创建虚VMThread
        VMThread::create();
        Thread* vmthread = VMThread::vm_thread();
        // 创建VMThread对应的OSThread线程，OSThread会调用
        // pthread_create创建真正的内核线程
        if (!os::create_thread(vmthread, os::vm_thread)) {
            ... // 创建失败
        }
        { // 等待VMThread准备就绪，然后运行VMThread
            MutexLocker ml(Notify_lock);
            os::start_thread(vmthread);
            while (vmthread->active_handles() == NULL) {
                Notify_lock->wait();
            }
        }
    }
}
```

​	当完成这一切后OSThread会阻塞，直到主线程执行os::start_thread。这时情况再次反转，主线程阻塞在vmthread->active_handles()，OSThread继续执行，设置active_handle()并最终阻塞在VMOperation队列上，等待VMOperation任务。

![](https://pic.imgdb.cn/item/608fc04cd1a9ae528f95e5ef.jpg)

```c__
class VM_Operation: public CHeapObj<mtInternal> {
private:
    Thread*         _calling_thread;   // 发起VMOperation的线程
    ThreadPriority  _priority;         // VMOPeration优先级
    long            _timestamp;        // 创建时间戳
    VM_Operation*   _next;             // 下一个VMOperation
    VM_Operation*   _prev;             // 上一个VMOperation
    static const char* _names[];       // VMOperation名字
public:
     virtual void doit() = 0;          // 具体功能实现
    virtual Mode evaluation_mode(){ return _safepoint; }  // 执行模式
    ...
};
```

​	evaluation_mode()会返回当前VM_Operation的执行模式，即虚拟机线程以何种方式执行该操作。目前虚拟机支持四种执行模式，具体如下所示。

* Safepoint：虚拟机线程需要等其他线程和发起操作的线程都进入安全点才能执行操作。
* No Safepoint：虚拟机线程无须等待其他线程和发起操作的线程进入安全点就能执行该操作。
* Concurrent：线程发起操作后可继续执行，虚拟机线程执行操作无须等待发起操作的线程和其他线程进入安全点。
* Asynchronous Safepoint：线程发起操作后可继续执行，但是当虚拟机线程执行该操作时发起操作的线程和其他线程都会进入安全点。



​	理解四种状态的关键是想象JVM只有三种线程：发起操作的线程、虚拟机线程、其他线程。

![](https://pic.imgdb.cn/item/608fc0a7d1a9ae528f99442d.jpg)

### 编译器线程

​	Threads::create_vm()会在中后期调用CompileBroker::compilation_init_phase1创建JIT编译器线程CompilerThread。与VMThread类似，JIT编译器线程会阻塞在各自的CompileQueue队列，当有编译任务发起时，其他线程会向CompileQueue投递一个CompileTask，然后编译线程启动并进行编译。`-XX:CICompilerCount=<val>`可以限制JIT编译器线程的数量，这个参数在早期Java 8及以前是有意义的，因为该参数基于CPU数调整，如果虚拟机运行在容器中无视容器的内存限制和CPU数限制，可以通过手动设置该参数解决这个问题。

### 服务线程

​	Threads::create_vm()后期会创建服务线程（ServiceThread），而服务线程会检查一系列事件是否发生，如果发生则唤醒执行，否则阻塞等待。

1）低内存探测：检查堆内存和堆外内存（Non-heap memory）的内存分配是否达到阈值。
2）JVMTI deferred事件：只有Java线程能投递JVMTI事件，如果非Java线程想要投递JVMTI事件，如CompiledMethodLoad（方法被编译并载入内存），CompiledMethodUnload（方法从内存中移除），DynamicCodeGenerated（虚拟机自身组件的，如模板解释器，动态代码生成）事件，只能先投到JvmtiDeferredQueue然后等待服务线程拉取处理。
3）GC通知：GC完成后会向服务线程投递通知。
4）jcmd命令：当使用jcmd执行一些命令时会向服务线程投递通知。
5）Table改变：当一些表发生重哈希行为时会设置标记，而服务线程能发现该标记。

6）Oop区域清理：服务线程会检查一些Oop区域是否有可清理的无效引用。

### 计时器线程

​	计时器线程（Watcher Thread）是JVM内部唯一一个具有最高优先级的线程，它可以模拟计时中断来定时执行某个周期任务（Periodic Task）。计时器线程的具体实现比较简单，首先线程如果没有周期任务就阻塞，如果有周期任务则先睡眠指定时长，然后立刻唤醒执行周期任务。周期任务是PeriodicTask及其子类，比较常见的是更新性能计数数据（`-XX:+UsePerfData`·，`-XX:PerfDataSamplingInterval=50`）的任务和更新内存Profiling任务（`-XX:+MemProfiling`，`-XX:MemProfilingInterval=500`）。

## Java线程

​	JavaThread持有一个指向java.lang.Thread对象的指针，即oop（JavaThread::_threadObj），java.lang.Thread也持有一个指向JavaThread的指针（java.lang.Thread中的eetop字段），只是这个指针是以整数的形式表示。

```c++
JavaThread* java_lang_Thread::thread(oop java_thread) {
    // 通过线程对象获取JavaThread（返回long值，强制类型转换为JavaThread*）
    return (JavaThread*)java_thread->address_field(_eetop_offset);
}
class JavaThread: public Thread {
    oop            _threadObj; 
    // 通过JavaThread获取线程对象
    oop threadObj() const { return _threadObj; }
  ...
};
```

​	JavaThread还持有指向OSThread的指针，OSThread即操作系统线程。线程可以看作执行指令序列的一个实体，指令的执行依赖指令指针寄存器和栈指针寄存器等，它们放到一起就称为线程上下文。如果线程上下文是由硬件提供，那么该线程称为硬件线程；如果线程上下文是由软件提供，那么该线程称为软件线程。硬件线程是指令执行的最终使能对象，一般一个处理器至少提供一个硬件线程，在现实中，一个处理器通常提供两个硬件线程。硬件线程数量对于现代操作系统是远远不够的，通常操作系统会在硬件线程之上构造出操作系统线程（内核线程），然后将操作系统线程映射到硬件线程上。不同的操作系统可能选择不同的映射方式，例如在Linux中，操作系统线程以M:N映射到硬件线程，而JavaThread以1:1映射到操作系统线程，此时JavaThread调度问题实际转化为操作系统调度内核线程的问题。

​	线程调度会不可避免地涉及线程状态的转换。在用户看来，Java线程有NEW（线程未启动）、RUNNABLE（线程运行中）、BLOCKED（线程阻塞在monitor上加锁）、WAITING（线程阻塞等待，直到等待条件被打破）、TIME_WAITING（同WAITING，等待条件新增超时一项）、TERMINATED（线程结束执行）6种状态。而虚拟机则对Java线程了解得更深刻，它不但知道线程正在执行，还知道线程正在执行哪部分代码：_thread_new表示正在初始化；_thread_in_Java表示线程在执行Java代码；`_thread_in_vm`线程在执行虚拟机代码；_thread_blocked表示线程阻塞。

### 线程启动

​	Java层的Thread.start()可以启动新的Java线程，该方法在JVM层调用prims/jvm的JVM_StartThread函数启动线程，这个函数会先确保java.lang.Thread类已经被虚拟机可用，然后创建一个JavaThread对象。

```c++
// Thread.start()对应JVM_StartThread
JVM_ENTRY(void, JVM_StartThread(...))
    ...
    // 虚拟机创建JavaThread，该类内部会创建操作系统线程，然后关联Java线程
    native_thread = new JavaThread(&thread_entry, sz);
    ...
    // 设置线程状态为RUNNABLE
    Thread::start(native_thread);
JVM_END
// JVM_StartThread创建操作系统线程，执行thread_entry函数
static void thread_entry(JavaThread* thread, TRAPS) {
    HandleMark hm(THREAD);
    Handle obj(THREAD, thread->threadObj());
    JavaValue result(T_VOID);
    // Thread.start()调用java.lang.Thread类的run方法
   JavaCalls::call_virtual(&result,obj, SystemDictionary::Thread_klass(), vmSymbols::run_method_name(), vmSymbols::void_method_signature(),THREAD);
}
// thread_native使用JavaCalls调用Java方法Thread.run()
public class java.lang.Thread {
    private Runnable target;
    public void run() {
        if (target != null) {
            target.run();  // Thread.run()又调用Runnable.run()
        }
    }
    ...
}
```

​	Thread.start()先用JNI进入JVM层，创建对应的JavaThread，再由JavaThread创建操作系统线程，然后用JavaCalls进入Java层，让新线程执行Runnable.run代码。

![](https://pic.imgdb.cn/item/608fc931d1a9ae528fdb01c4.jpg)

### 线程停止

​	线程停止的机制比较特别。在Java层面，JDK会创建一个ThreadDeath对象，该类继承自Error，然后传给JVM_StopThread停止线程。

```c++
JVM_ENTRY(void, JVM_StopThread(...))
    // 获取JDK传入的ThreadDeath对象，确保不为空
    oop java_throwable = JNIHandles::resolve(throwable);
    if(java_throwable == NULL) {
        THROW(vmSymbols::java_lang_NullPointerException());
    }
    ...
    // 如果要待停止的线程还活着
    if (is_alive) {
        // 如果停止当前线程
        if (thread == receiver) {
            // 抛出ThreadDeath（Error）停止
            THROW_OOP(java_throwable);
        } else {
            // 否则停止其他线程，向虚拟机线程投递VM_ThreadStop
            Thread::send_async_exception(java_thread, java_throwable);
        }
    } else {
        // 否则复活它（停止没有启动的线程是java.lang.Thread允许的行为）
        java_lang_Thread::set_stillborn(java_thread);
    }
JVM_END
```

​	如果要停止的线程是当前线程，那么JVM_StopThread只是让它抛出ThreadDeathError，这意味着如果捕获Error那么线程是不会停止的

```c++
public class ThreadTest {
    public static void main(String[] args) {
        new Thread(()->{
            try{
                Thread.currentThread().stop();
            }catch (Error ignored){ }
            System.out.println("still alive");
        }).start();
    }
}
```

​	如果停止的不是当前线程，则情况会复杂一些。JVM_ThreadStop向虚拟机线程投递一个VM_ThreadStop的操作，由虚拟机线程负责停止它。

​	VM_ThreadStop是一个VM_Operation，它的执行模式是asnyc_safepoint，即发起操作的线程在向虚拟机线程队列投递VM_ThreadStop后可继续执行，仅当虚拟机线程执行VM_ThreadStop时才需要除了虚拟机线程外的所有线程都到达安全点。

```c++
class VM_ThreadStop: public VM_Operation {
    private:
    oop     _thread;        // 要停止的线程
    oop     _throwable;     // ThreadDeath对象
    public:
    ...
    // 停止线程操作需要异步安全点
    Mode evaluation_mode() const { return _async_safepoint; }
    void doit() {
        // 位于全局停顿的安全点
        ThreadsListHandle tlh;
        JavaThread* target = java_lang_Thread::thread(target_thread());
        if(target != NULL && ...) {
            // 发送线程停止命令
            target->send_thread_stop(throwable());
        }
    }
};
```

​	VM_ThreadStop::doit()中的“发送”二字可能有些迷惑性，毕竟位于安全点的除了虚拟机线程外的其他应用线程都停顿了，发送给停顿线程数据意义不大，因此它们无法被观测到。实际上，send_thread_stop()只是将JDK创建的ThreadDeath对象设置到目标线程JavaThread中的`_pending_async_exception`字段。紧接着目标线程执行每条字节码时会检查是否设置了`_pending_async_exception`字段，如果设置了则转化为`_pending_exception`，最后线程退出时会检查是否设置了该字段并根据情况调用Thread::dispatchUncaughtException()。

​	与Thread.resume()配套的Thread.suspend()的实现也使用了类似Thread.stop()的机制，前者可让一个线程恢复执行，后者可暂停线程的执行。Thread.suspend()会向VMThread的VMOperation队列投递一个执行模式为safepoint的VM_ThreadSuspend操作，然后等待VMThread执行该操作。

​	这种实现方式导致Thread.stop等接口具有潜在的不安全性。因为当ThreadDeath异常传播到上层栈帧时，上层栈帧中的monitor将会被解锁，如果受这些monitor保护的对象正处于不一致状态（如对象正在初始化中），其他线程也会看到对象的不一致状态。换句话说，这些对象结构已经损坏。使用损坏的对象造成任何错误结果并不奇怪，更糟糕的是这些错误可能在很久后才会出现，导致调试困难。基于这些原因，Thread.stop/resume/suspend接口被标记为废弃，不应该使用。结束线程的正确方式是让线程完成任务后自然消亡。

### 睡眠与中断

​	Thread.sleep()可以让一个线程进入睡眠状态，它在底层调用JVM_Sleep方法。

```c++
JVM_ENTRY(void, JVM_Sleep(...))
    JVMWrapper("JVM_Sleep");
    // 如果睡眠时间<0，则抛出参数错误异常
    if (millis < 0) {
        THROW_MSG(...);
    }
    // 如果待睡眠的线程已经处于中断状态
    if (Thread::is_interrupted (...) && !HAS_PENDING_EXCEPTION) {
        THROW_MSG(...);
    }

    // 保存当前线程状态
    JavaThreadSleepState jtss(thread);
    // 如果睡眠时间为0，Thread.sleep()退化为Thread.yield()
    if (millis == 0) {
        os::naked_yield();
    } else {
        ThreadState old_state = thread->osthread()->get_state();
        thread->osthread()->set_state(SLEEPING);
        if (os::sleep(thread, millis, true) == OS_INTRPT) {
            if (!HAS_PENDING_EXCEPTION) {
                THROW_MSG(...);// 如果睡眠的时候有异步异常发生 
            }
        }
        // 恢复之前保存的线程状态
        thread->osthread()->set_state(old_state);
    }
JVM_END
```

​	Thread.sleep()首先确保线程睡眠时间大于等于零。接着还需要防止睡眠已经中断的线程，这种情况少见但也会发生。

```c++
public class ThreadTest {
    public static void main(String[] args) {
        Thread t = new Thread(()->{
            synchronized (ThreadTest.class){ }
            try { 
                Thread.sleep(1000);
            } catch (InterruptedException e) { 
                e.printStackTrace();
            }
        });
        synchronized (ThreadTest.class){
            t.start();
            t.interrupt();
        }
    }
}
```

​	防止了异常情况后，如果Thread.sleep()检查睡眠时间为0则会退化为Thread.yield()，调用操作系统提供的线程让出函数。如果睡眠时间正常，会调用如代码清单4-12所示的os::sleep()。

```c++
int os::sleep(Thread* thread, jlong millis, bool interruptible) {
    ParkEvent * const slp = thread->_SleepEvent ;
    slp->reset() ;
    OrderAccess::fence() ;
    if (interruptible) {
        jlong prevtime = javaTimeNanos();
        for (;;) {
            // 检查是否中断
            if (os::is_interrupted(thread, true)) {
                return OS_INTRPT;
            }
            // 更精确的睡眠时间
            jlong newtime = javaTimeNanos();
            if (newtime - prevtime < 0) {
            } else {
                millis -= (newtime - prevtime)/NANOSECS_PER_MILLISEC;
            }
            if (millis <= 0) {
   				return OS_OK;
            }
            prevtime = newtime;
            ...
            // 进行睡眠
            slp->park(millis);
        }
    } else {
        ... // 类似上面的可中断逻辑，只是少了中断检查
    }
}
```

​	为了支持可中断的睡眠，HotSpot VM实际上是使用ParkEvent实现的[2]。同样地，HotSpot VM的线程中断也是使用ParkEvent实现的。

```c++
void os::interrupt(Thread* thread) {
    OSThread* osthread = thread->osthread();
    // 如果线程没有处于中断状态，调用ParkEvent::unpark()通知睡眠线程中断
    if (!osthread->interrupted()) {
        osthread->set_interrupted(true);
        OrderAccess::fence();
        ParkEvent * const slp = thread->_SleepEvent ;
        if (slp != NULL) slp->unpark() ;
 	}
    if (thread->is_Java_thread())
        ((JavaThread*)thread)->parker()->unpark();
    ParkEvent * ev = thread->_ParkEvent ;
    if (ev != NULL) ev->unpark() ;
}
```

​	ParkEvent是Java层的对象监控器（Object Monitor）语意的底层实现，也是虚拟机内部使用的同步设施的基础依赖。在虚拟机运行时随便打个断点，会看到大多数线程最后一层栈帧都是调用ParkEvent::park()随后阻塞。

​	ParkEvent还有个孪生兄弟Parker，用于在底层支持java.util.concurrent.*中的各种组件。关于这两者将会在第6章中详细讨论。现在可以简单认为ParkEvent::park()让线程阻塞等待，ParkEvent::unpark()唤醒线程执行。

​	多次用到OrderAccess，该组件用于保证内存操作的连续性与一致性，它是Java内存模型（Java Memory Model，JMM）的基础设施，有助于虚拟机消除编译器重排序和CPU重排序，实现JMM中的Happens-Before关系等。

## 栈帧

​	线程是程序执行的代名词，而程序执行过程中一个至关重要的东西是栈帧。它可以存放局部变量、方法参数等数据。

```c++
class frame {
    private:
        intptr_t* _sp;             // 栈顶指针
        address   _pc;             // 指向下一条指令的指针（RIP）
        CodeBlob* _cb;             // 持有pc的代码块
        deopt_state _deopt_state;  // 退优化状态（未退优化、退优化、未知）
    public:
    ...
    #include CPU_HEADER(frame)     // 巧妙地用#include包含CPU架构相关的代码
};
```

​	HotSpot VM将CPU无关的代码放置于runtime/frame中，然后用头文件包含指令#include巧妙地根据不同的CPU架构包含不同的代码，CPU_HEADER将会在编译的时候被替换为指定的架构，如x86是hotspot/cpu/x86/frame_x86.hpp。

​	虚拟机没有创造frame，它只是借用frame这个数据结构来描述栈帧。除了frame外还有vframe，如代码清单4-15所示，vframe是对frame的进一步封装，表示Java层面的虚拟栈帧。除了frame提供的信息外还能通过vframe访问到栈帧所属线程和Callee-saved寄存器。

```c++
class vframe: public ResourceObj {
    protected:
        frame        _fr;      // 物理栈帧
        RegisterMap  _reg_map; // callee-saved寄存器
        JavaThread*  _thread;  // 栈帧所属线程
	public:
    ...
};
```

​	Callee-saved表示被调用者保存的寄存器，如果方法调用者希望调用了方法后某些寄存器还能保持原来的值，就需要被调用者在使用它们前提前保存。与之类似的概念是Caller-saved，即调用者在调用前自行保存这些寄存器的值，而被调用者可以自由使用它。两者的区别就是如果某个寄存器需要在一个调用后使用，那么是调用者保存它的值（Caller-saved）还是被调用者保存它的值（Callee-saved）。

​	vframe的Callee-saved寄存器保存了调用者的栈底指针（RBP/EBP）。

​	虚拟机会在vframe上抽象出javaVFrame用于表示Java栈帧。javaVFrame栈帧还可以细分为解释器栈帧（interpretedVFrame）和编译代码栈帧（compiledVFrame）。这些新的栈帧几乎没有增加数据结构，只是相比frame而言更方便。

​		很多地方都属于GC Root，其中之一便是线程栈。垃圾回收器遍历线程上的每个frame（所有frame构成一个线程栈），然后调用frame::oops_do()寻找frame中的所有对象引用，并以它们为起进行对象标记。

```c++
void frmae::oops_do(...) { oops_do_internal(...); }
void frame::oops_do_internal(...) {
    if (is_interpreted_frame()) {           // 解释器栈帧
        oops_interpreted_do(f, map, use_interpreter_oop_map_cache);
    } else if (is_entry_frame()) {          // call_stub调用起始栈帧
        oops_entry_do(f, map);
    } else if (CodeCache::contains(pc())) { // 编译后代码栈帧
        oops_code_blob_do(f, cf, map);
    } else {
        ShouldNotReachHere();
   }
}
```

## Java/JVM沟通

![](https://pic.imgdb.cn/item/608fe68cd1a9ae528fd6139f.jpg)

### JNI

​	开发者通常使用`Class<?>.getDeclaredFields()`获取某类的所有（父类除外）字段。在具体实现中，它调用native修饰的方法getDeclaredFields0，该方法又通过JNI调用虚拟机内部的JNI函数JVM_GetClassDeclaredFields。那么虚拟机如何知道native方法getDeclaredFields0对应的JNI函数JVM_GetClassDeclaredFields呢？答案是使用`Class<?>.registerNatives`。

```c++
static JNINativeMethod methods[] = {
    {"getDeclaredFields0",   
    "(Z)[" FLD,      
    (void *)&JVM_GetClassDeclaredFields}, ...
};
JNIEXPORT void JNICALL
Java_java_lang_Class_registerNatives(JNIEnv *env, jclass cls){
    methods[1].fnPtr = (void *)(*env)->GetSuperclass;
    (*env)->RegisterNatives(env, cls, methods,
                          sizeof(methods)/sizeof(JNINativeMethod));
}
```

​	HotSpot VM将一些JNI函数放入一个数组（methods），然后用registerNatives统一注册。在JDK源码中，有很多类（如java.lang.Class, java.lang.Object, java.lang.System）都有这个注册函数，它们都是在一个类的静态代码块里面调用registerNatives。

```c++
void Method::set_native_function(...) {
    // native_function_addr会返回Method之后的位置
    address* native_function = native_function_addr();
    address current = *native_function;
    // 如果已经注册过就返回
    if (current == function) return;
    // 否则将native方法入口地址写到Method之后的位置
    *native_function = function;
}
```

​	set_native_function会将JNI函数地址写到Method后的native_function_addr。

![](https://pic.imgdb.cn/item/608fe820d1a9ae528fea58a1.jpg)

​                                                                                                                                

​	如果是普通Java方法，Method就存放一切需要的入口地址，比如解释器入口地址、JIT编译后的入口地址，此时Method后面没有附加内容。但是如果Java方法是native，其对应的JNI函数地址会放到Method后面的第一个附加槽（不属于Method数据结构的部分），这个“将JNI函数地址放入第一个槽”就是registerNative()要完成的。这样之后registerNative注册的native方法就能在类初始化后被调用了。

​	在日常开发中可以用javah生成C++函数，即JNI函数，它和Java的native方法“自动”对应，无须用到registerNative。

1）解释器遇到native方法，调用InterpreterRuntime::prepare_native_call准备。
2）prepare_native_call检查Method是否存在附加槽（是否已经有native入口），如果存在直接返回；如果不存在则用NativeLookup::lookup继续后面的查找过程。

3）NativeLookup::lookup调用Java代码ClassLoader.findNative。
4）findNative在synchronized块内寻找所有动态链接库，然后又调用一个native方法回到JVM层，这个native方法最终调用JVM_FindLibraryEntry。

5）JVM_FindLibraryEntry代理操作系统相关的动态链接API os::dll_lookup。
6）os::dll_lookup平台相关，在Linux/OS X上调用dlsym()，在Windows上调用GetProcAddress。

![](https://pic.imgdb.cn/item/608fea36d1a9ae528f053e31.jpg)

​	如果使用registerNative提前注册，类初始化阶段会完成这些准备工作，否则上述开销将会推迟到运行时。

​	参数-XX:+UseFastSignatureHandlers（默认开启）还会启用一个优化：对于不超过13个参数的native方法，signature handler会走快速路径。所谓快速路径是指JVM计算方法签名字符串得到一个64位整数方法指纹（Method Fingerprint）值，后续signature handler将不需要每次都解析签名字符串得到参数个数和类型，而是直接用方法指纹值。同时快速路径也不会将参数放到C栈再取一些放入寄存器而是一步到位，直接放入寄存器或者C栈。

### JavaCalls

​	在虚拟机中，Java代码通过JNI调用JVM方法，而JVM反过来通过JavaCalls模块调用Java方法。

​	JavaCalls模块可细分为call_virtual调用Java虚方法、call_static调用Java静态方法等。虚方法调用会根据对象类型进行方法决议，所以需要获取对象引用再查找实际要调用的方法，而静态方法调用直接查找要调用的方法即可。无论如何，这些方法都是先找到要调用的方法的methodHandle。

```c++
void JavaCalls::call_helper(...) {
    ...
    // 调用函数指针_call_stub_entry，把实际的函数调用工作转交给它
    { JavaCallWrapper link(method, receiver, result, CHECK);
        { HandleMark hm(thread);
            StubRoutines::call_stub()(
                (address)&link,
                result_val_address,      
                result_type,
                method(),
                entry_point,
                args->parameters(),
                args->size_of_parameters(),
                CHECK
            );
     ...
        }
    } 
}
```

​	call_helper还没有做方法调用，它只是检查方法是否需要编译，验证参数是否正确等，最终它会跳转到函数指针_call_stub_entry处，把方法调用这件事又转交给_call_stub_entry。`_call_stub_entry`由generate_call_stub()生成，这是一个运行时代码生成的过程。

​	`_call_stub_entry`会调用Java方法，而调用Java方法前需要建立栈帧，所以它也会负责栈帧的创建。

​	调用Java方法更确切来说是跳转到解释器入口entry_point处。，实际上entry_point到解释器真正解释Java代码还有一小段距离，这里为了便于理解可以将它看作解释器入口。

![](https://pic.imgdb.cn/item/608fecd4d1a9ae528f2fe7ee.jpg)



## Unsafe类

​	JEP 193引入的VarHandle，它可以替代JUC.atomic和Unsafe类的部分操作，并提供了标准的内存屏障方法。JEP 370提供了外部内存访问API（JDK15二次孵化）可以安全且高效地访问堆外内存。

### 堆外内存

​	Java堆又叫堆内内存，它交由垃圾回收器全权负责，垃圾回收器在其上分配内存、储存对象、释放内存。与之相对的概念是堆外内存（Off-heap），这部分内存不受垃圾回收器控制，由开发者自行负责。调用java.nio.ByteBuffer.allocateDirect()可以分配堆外内存，allocateDiret()实际上是借助Unsafe.AllocateMemory实现分配堆外内存分配的。

```c++
UNSAFE_ENTRY(jlong, Unsafe_AllocateMemory0(...)) {
    size_t sz = (size_t)size;
	sz = align_up(sz, HeapWordSize);
    void* x = os::malloc(sz, mtOther);
    return addr_to_java(x);
} UNSAFE_END

UNSAFE_ENTRY(void, Unsafe_FreeMemory0(...)) {
    void* p = addr_from_java(addr);
    os::free(p);
} UNSAFE_END
```

### 内存屏障

​	JEP 171在Unsafe类中增加了内存屏障方法loadFence/storeFence/fullFence方法。

```c++
UNSAFE_LEAF(void, Unsafe_LoadFence(JNIEnv *env, jobject unsafe)) {
    OrderAccess::acquire();
} UNSAFE_END

UNSAFE_LEAF(void, Unsafe_StoreFence(JNIEnv *env, jobject unsafe)) {
    OrderAccess::release();
} UNSAFE_END

UNSAFE_LEAF(void, Unsafe_FullFence(JNIEnv *env, jobject unsafe)) {
    OrderAccess::fence();
} UNSAFE_END
```

### 阻塞和唤醒

​	java.util.concurrent（简称JUC）包含很多并发组件，这些并发组件可以阻塞线程的执行，唤醒线程。这些行为的背后都依赖java.util.concurrent.locks.LockSupport的park/unpark方法，而LockSupport.park/unpark又最终调用Unsafe.park/unpark实现其功能。

```c++
UNSAFE_ENTRY(void, Unsafe_Park(...)) {
    ...
    thread->parker()->park(isAbsolute != 0, time);
} UNSAFE_END

UNSAFE_ENTRY(void, Unsafe_Unpark(...)) {
    Parker* p = NULL;
    ...
    if (p != NULL) {
        HOTSPOT_THREAD_UNPARK((uintptr_t) p);
        p->unpark();
    }
} UNSAFE_END
```

​	JUC背后与虚拟机交互的接口就是Unsafe.park/unpark，可以说Unsafe.park/unpark是整个JUC的基石。

### 对象数据修改

​	Java提供了java.io.Serializable，配合ObjectOutputStream/ObjectInputStream可以实现对象序列化和反序列化，但这是一个很慢的操作，还限制类必须提供无参的public构造方法。一些高性能第三方库会使用Unsafe类完成序列化和反序列化操作，它们调用Unsafe.getInt(Object o, long offset)等获取对象o所在偏移offset处的字段进行序列化。用Unsafe.allocateInstance调用对象的构造方法生成对象，再用Unsafe.putLong(Object o, long offset, long x)等将对象o所在偏移offset的字段设置为x值，以此来反序列化。

# 模板解释器

## 解释器

​	HotSpot VM默认使用解释和编译混合（-Xmixed）的方式执行代码。首先它使用模板解释器对字节码进行解释，当发现一段代码是热点时，就使用C1或C2即时编译器优化编译后再执行，这也是它的名字——“热点”的由来。解释器的代码位于hotspot/share/interpreter。

![](https://pic.imgdb.cn/item/608ff335d1a9ae528f8f0523.jpg)

### C++解释器行为

​	对于Java字节码istore_0和iadd来说，如果是C++字节码解释器。

```c++
void cppInterpreter::work(){
    for(int i=0;i<bytecode.length();i++){
        switch(bytecode[i]){
            case ISTORE_0:
                int value = operandStack.pop();
                localVar[0] = value;
                break;
            case IADD:
                int v1 = operandStack.pop();
                int v2 = operandStack.pop();
                int res = v1+v2;
                operandStack.push(res);
                break;
            ....
        }
    }
}
```

### 模板解释器行为

​	模板解释器是一堆机器代码的例程，会在虚拟机创建时初始化好，换句话说，模板解释器在虚拟机初始化的时候为iadd和istore_0申请两片内存，并设置为可读、可写、可执行，然后向内存写入模拟iadd和istore_0执行的机器代码。在解释执行时遇到iadd，跳转到相应内存，并将该片内存的数据视作代码直接执行。

```c++
class TemplateInterpreter: public AbstractInterpreter {
protected:
    static address       _throw_ArrayIndexOutOfBoundsException_entry;
    static address       _throw_ArrayStoreException_entry;
    static address       _throw_ArithmeticException_entry;
    static address       _throw_ClassCastException_entry;
    static address       _throw_NullPointerException_entry;
    static address       _throw_exception_entry;
    static address       _throw_StackOverflowError_entry;
    static address       _remove_activation_entry;                
    static address       _remove_activation_preserving_args_entry; 
    static EntryPoint    _return_entry[number_of_return_entries];     
    static EntryPoint    _earlyret_entry;                             
    static EntryPoint    _deopt_entry[number_of_deopt_entries];        
    static address       _deopt_reexecute_return_entry;
    static EntryPoint    _safept_entry;
    static DispatchTable _active_table;                        
    static DispatchTable _normal_table;                         
    static DispatchTable _safept_table;            
    static address       _wentry_point[DispatchTable::length];
    ... 
};
```

**抽象解释器**

```c++
class AbstractInterpreter: AllStatic {
protected:
    static StubQueue* _code;                              
    static bool       _notice_safepoints;                        
    static address    _native_entry_begin;                      
    static address    _native_entry_end;
    // 方法入口点
    static address    _entry_table[number_of_method_entries];    
    static address    _cds_entry_table[number_of_method_entries]; 
    static address    _slow_signature_handler;          
    static address    _rethrow_exception_entry;      
    ...
};
```

## 机器代码片段

​	机器代码片段的生成是由TemplateInterpreterGenerator完成的，它是解释器本体的生成器。

```c++
class StubQueue: public CHeapObj<mtCode> {
    private:
        StubInterface* _stub_interface;    // 沟通Stub和StubQueue的接口
        address       _stub_buffer;        // 存放机器的地方（buffer）
        int            _buffer_size;       // buffer大小
        int            _buffer_limit;      // buffer大小限制
        int            _queue_begin;       // 队列开始
        int            _queue_end;         // 队列结束
        int            _number_of_stubs;   // 机器代码片段个数
        Mutex* const   _mutex;
    public:
	StubQueue::StubQueue(...) : _mutex(lock) {
        intptr_t size = align_up(buffer_size, 2*BytesPerWord);
        BufferBlob* blob = BufferBlob::create(name, size);
        if( blob == NULL) {
            vm_exit_out_of_memory(...);
        }
        _stub_interface  = stub_interface;
        _buffer_size     = blob->content_size();
        _buffer_limit    = blob->content_size();
        _stub_buffer     = blob->content_begin();
        _queue_begin     = 0;
        _queue_end       = 0;
        _number_of_stubs = 0;
    }
};
```

## CodeCache

​	为了统一管理这些运行时生成的机器代码，HotSpot VM抽象出一个CodeBlob体系，由CodeBlob作为基类表示所有运行时生成的机器代码，并衍生出五花八门的子类:

* CompiledMethod：编译后的Java方法。

  * nmethod：JIT编译后的Java方法。

  * AOTCompiledMethod：AOT编译的方法。
* RuntimeBlob：非编译后的代码片段。
  * BufferBlob：解释器等使用的代码片段。
    * AdapterBlob：C2I/I2C适配器代码片段。
    * VtableBlob：虚表代码片段。
    * MethodHandleAdapterBlob：MethodHandle代码片段。
  * RuntimeStub：调用运行时方法的代码片段。
  * SingletonBlob：单例代码片段。
    * DeoptimizationBlob：退优化代码片段。
    * ExceptionBlob：异常处理代码片段。
    * SafepointBlob：错误指令异常处理代码片段。
    * UncommonTrapBlob：打破编译器假设的稀有情况代码片段。



​	第4章曾提到Threads::create_vm会初始化线程和组件，CodeCache便是这里所说的组件之一，它在Threads::create_vm初始化主线程后初始化

**CodeCache初始化**

```c++
void CodeCache::initialize() {
    ... // 开启分段CodeCache，将运行时生成的代码片段按类别放到三个区域
    if (SegmentedCodeCache) {
        initialize_heaps();
    } else {
        ... // 不开启分段CodeCache，所有运行时生成的代码片段都放到一个区域
        add_heap(rs, "CodeCache", CodeBlobType::All);
    }
    // 初始化指令缓存刷新模块（ICache Flush）
    icache_init();
 	// * Windows上为CodeCache中的运行时生成的代码注册结构化异常处理（SEH）
    os::register_code_area((char*)low_bound(), (char*)high_bound());
}
```

​	CodeCache区域的最大空间可以用`-XX:ReservedCodeCacheSize=<val>`指定。Java 9在JEP 197中引入了CodeCache分段。如果没有开启CodeCache分段，JVM会用一个区域存放所有运行时生成的代码片段。如果使用`-XX:+SegmentedCodeCache`开启分段，JVM会将CodeCache内部拆分为三个区域，分别用于存放非nmethod代码片段（如解释器、C2I/I2C适配器等）、处于分层编译的2和3级别带Profiling信息的nmethod、处于分层编译的1和4级别不带Profiling信息的nmethod。CodeCache分段有很多好处，比如：

* 分隔非nmethod方法，例如带Profiling的nmethod与不带Profiling的nmethod，可以根据需要访问不同的区域，无须每次遍历整个CodeCache。
* 提升程序运行时尤其是GC的性能。在开启分段堆后GC扫描根只需要遍历一个区域。
* 提升代码局部性，因为相同类型的代码很有可能在最近一段时间被频繁访问。

## 指令缓存刷新

​	模板解释器和JIT编译器都重度依赖运行时代码生成技术，它们在运行时向内存写入数据，这些数据可以被当作指令执行。CPU和主存间一般有L1、L2、L3三级高速缓存，L1级高速缓存又可以分为指令缓存（Instruction Cache）和数据缓存（Data Cache），这样划分后CPU可以同时获取指令和数据，进而提升性能，但是也带来了一致性问题。

![](https://pic.imgdb.cn/item/6090b8e3d1a9ae528fa7d142.jpg)

​	处理器未来会自动将数据缓存的数据写回内存，然后指令缓存重新读取位于内存的指令，但是没有办法知道处理器何时这样做。举个例子，如果虚拟机运行时生成了新的代码想要立即执行它们，处理器可能会忽略它们执行旧的代码，因为旧的代码仍然位于指令缓存中。

![](https://pic.imgdb.cn/item/6090b90cd1a9ae528fa967ff.jpg)

​	可以强制刷新指令缓存的数据，使缓存的指令无效化，这时处理器会主动将数据缓存中的数据写入内存，然后读取内存的新指令到指令缓存。HotSpot VM中无效化指令缓存的操作由runtime/icache模块完成，CodeCache区域初始化后会调用icache_init()初始化指令缓存刷新模块。

```c++
void ICacheStubGenerator::generate_icache_flush(...) {
    ...
    // 如果待清理的内存地址为0，则跳过清理
    __ testl(lines, lines);
    __ jcc(Assembler::zero, done);
    // 禁止CPU指令重排序（只能使用mfence屏障）
    __ mfence();
    // 否则清理[0，ICache::line_size]内存地址范围内的缓存行
    __ bind(flush_line);
    __ clflush(Address(addr, 0)); // 底层是x86的clflush实现
    __ addptr(addr, ICache::line_size);
    __ decrementl(lines);
    __ jcc(Assembler::notZero, flush_line);

    // 禁止CPU指令重排序
  	__ mfence();
    // 清理完成
    __ bind(done);
    __ ret(0);
    *flush_icache_stub = (ICache::flush_icache_stub_t)start;
}
```

​	x86上指令缓存刷新是由clflush指令实现的，该指令是唯一一个必须配合使用mfence的指令。


## 解释器生成

​	由于运行时生成的机器代码是人类不可读的二进制形式，要想阅读它们，可以下载hsdis-amd64插件，并将该插件放到编译后的JDK中的/lib/server目录下面，然后开启虚拟机参数-XX:+PrintAssembly和-XX:+PrintInterpreter，然后便可输出解释器各个例程的机器代码的汇编表示形式了。也可以开启-XX:+TraceBytecodes跟踪解释器正在执行的字节码和对应方法。

### 普通方法入口

​	JavaCalls::call_helper进入`_call_stub_entry`会创建Java栈帧（见图4-10），然后进入entry_point执行方法。entry_point即第2章中类链接阶段设置的解释器入口。实际上，解释器入口和`_call_stub_entry`一样，也是一段机器代码，也是在虚拟机初始化时动态生成的，只是生成它的是generate_normal_entry。

​	解释器入口entry_point的（生成）代码如下:

```c++
address TemplateInterpreterGenerator::generate_normal_entry(...) {
    ... // ebx存放了指向Method的指针
    // 获取参数个数，放入rcx
    __ movptr(rdx, constMethod);
    __ load_unsigned_short(rcx, size_of_parameters);
    // 获取局部变量个数
    __ load_unsigned_short(rdx, size_of_locals); 
    __ subl(rdx, rcx); 
    // 检查栈上是否可以容纳即将分配的局部变量槽
    generate_stack_overflow_check();
    // 获取返回地址
    __ pop(rax);
    // 计算第一个参数的地址
    __ lea(rlocals, ...);
    {// 分配局部变量槽，然后初始化这些槽
        Label exit, loop;
        __ testl(rdx, rdx);
        __ jcc(Assembler::lessEqual, exit); 
        __ bind(loop);
        __ push((int) NULL_WORD);// 用0初始化，局部变量有默认值便是因为它 
        __ decrementl(rdx); 
        __ jcc(Assembler::greater, loop);
        __ bind(exit);
    }
    // 创建解释器栈帧（有别于Java栈帧）
	generate_fixed_frame(false);
    // 从当前位置到对方法加锁还有一段距离，万一中间发生异常且方法又是同步方法，则异常处理器
    // 会unlock一个未lock的方法，所以这里需要告诉线程不能unlock
    NOT_LP64(__ get_thread(thread));
    const Address do_not_unlock_if_synchronized(..);
    __ movbool(do_not_unlock_if_synchronized, true);
    __ profile_parameters_type(rax, rcx, rdx);
    // 执行字节码前增加方法调用计数，如果计数到达一定值即跳到后面通知编译器
    ...
    if (inc_counter) {
        generate_counter_incr(...);
    }
    // 设置编译后继续执行的地方
    Label continue_after_compile;
    __ bind(continue_after_compile);
    // 检查geneate_fixed_frame分配之后是否栈溢出
    bang_stack_shadow_pages(false);
    // 对方法加锁，并通知线程可以unlock方法了（因为可能抛出异常的区域已经没了）
    NOT_LP64(__ get_thread(thread));
    __ movbool(do_not_unlock_if_synchronized, false);
    // 如果方法是同步方法，调用lock_method()锁住方法
    if (synchronized) {
        lock_method();
    }
    // 将字节码指针设置到第一个字节码，然后开始执行字节码（！）
    __ dispatch_next(vtos);
    if (inc_counter) {
        ...
	// 如果之前增加调用计数达到一定值，则跳转到此处通知编译器判断是否编译
        __ bind(invocation_counter_overflow);
        generate_counter_overflow(continue_after_compile);
    }
    return entry_point;
}
```

​	HotSpot VM中有两个地方可能发生“解释器认为Java方法（循环）是热点并通知编译器判断是否编译”这一行为：字节码中的回边（Backedge）分支计数通知（本章后面讨论）与方法调用计数通知。	

​	generate_counter_inc()检查当前方法调用频率是否超过一个通知值（`-XX:TierXInvokeNotifyFreq-Log=<val>`，X表示分层编译的层数，该值可根据情况进行调整），generate_counter_overflow通知编译器方法调用频率达到了一定的程度。两者检查在一定时间范围内某个方法调用是否到达了一定的频率，实际上是否编译是根据编译器策略（Tiered Threshold Policy）进行抉择的。

### 方法加锁

​	普通方法入口中一个重要的内容是方法加锁。

```c++
void TemplateInterpreterGenerator::lock_method() {
    ...
    { Label done;
        // 检查方法是否为static
        __ movl(rax, access_flags);
        __ testl(rax, JVM_ACC_STATIC);
        // 获取局部变量槽的第一个元素（即receiver）
        __ movptr(rax, ...);
        __ jcc(Assembler::zero, done);
        // 如果是static，加载方法所在类的Class对象
        __ load_mirror(rax, rbx);
        __ bind(done);
        // 保持使用receiver
        __ resolve(IS_NOT_NULL, rax);
    }
    // 基本对象锁（BasicObjectLock）包含一个待锁对象和Displaced Header
    // 下面会将receiver放入基本对象锁的待锁对象字段
	__ subptr(rsp, entry_size); 
    __ movptr(monitor_block_top, rsp);
    __ movptr(Address(rsp, BasicObjectLock),...);
    const Register lockreg = NOT_LP64(rdx) LP64_ONLY(c_rarg1);
    __ movptr(lockreg, rsp); 
    // 加锁
    __ lock_object(lockreg);
}
```

![](https://pic.imgdb.cn/item/6090d18bd1a9ae528f8de48e.jpg)

​	monitor区存放了若干个基本对象锁（Basic Object Lock）。基本对象锁又叫轻量级锁、瘦锁，它包含一个加锁对象和Displaced Header。在获取到加锁对象后，会将加锁对象放入基本对象锁，然后调用lock_object()。lock_object()并不是简单锁住对象，它还会应用一些锁的优化措施：最开始尝试偏向锁，如果加锁失败则尝试基本对象锁，如果仍然失败则会使用重量级锁。

### 本地方法入口

​	本地方法入口由generate_native_entry()生成，它和普通方法最大的不同是普通方法通过dispatch_next()执行每一条字节码并达到解释的效果，而本地方法本身就是机器代码，可以直接执行。

```c++
address TemplateInterpreterGenerator::generate_native_entry(...) {
    ...
    // 获取native方法入口，第3章提到过native方法入口在Method后面的第一个槽
    {
    Label L;
	// 获取native入口
    __ movptr(rax, ...Method::native_function_offset());
    __ cmpptr(rax, unsatisfied.addr());
    // 获取失败会调用prepare_native_call以找到native入口和signature handler。
    // 查找native入口的过程请参见第3章
    __ jcc(Assembler::notEqual, L);
    __ call_VM(...InterpreterRuntime::prepare_native_call);
    __ get_method(method);
    __ movptr(rax, ...Method::native_function_offset());
    __ bind(L);
    }
    // 从线程上拿出JNIEnv并作为第一个参数传入
    __ lea(c_rarg0, ...r15_thread);
    __ set_last_Java_frame(rsp, rbp, (address) __ pc());

    // 设置线程状态位_thread_in_native（即执行native方法）
    __ movl(Address(thread, JavaThread::thread_state_offset()),
            _thread_in_native);
    // 调用native方法
    __ call(rax);
    ...
}
```

​	本地方法和普通方法的另一个不同之处是对同步方法的处理。generate_normal_entry()中只使用lock_method()对方法加锁而没有对应的解锁代码，因为dispatch_next()执行字节码时（见5.5.4节），一些字节码（如return、athrow）在移除栈帧的时候会解锁同步方法，所以无须在generate_normal_entry()中解锁。但是generate_native_entry()没有执行字节码，它必须在执行完native方法之后检查是否需要解锁同步方法。

### 标准字节码

```c++
void InterpreterMacroAssembler::dispatch_next(...) {
    // 加载一条字节码
    load_unsigned_byte(rbx, Address(_bcp_register, step));
    // 字节码指针（bcp）前进
    increment(_bcp_register, step);
    // dispatch_next的详细实现
    dispatch_base(...);
}
```

```c++
void InterpreterMacroAssembler::dispatch_base(...) {
    // 获取安全点表
    address* const safepoint_table = Interpreter::safept_table(state);
    Label no_safepoint, dispatch;

    // 如果需要生成安全点
    if (SafepointMechanism::uses_thread_local_poll() && ...) {
        testb(Address(r15_thread,Thread::polling_page_offset()),
            SafepointMechanism::poll_bit());
    jccb(Assembler::zero, no_safepoint);
  	lea(rscratch1, ExternalAddress((address)safepoint_table));
    jmpb(dispatch);
    }

    // 如果不需要安全点
    bind(no_safepoint);

    // 获取模板表
    lea(rscratch1, ExternalAddress((address)table));
    bind(dispatch);
    // 跳转到模板表中指定字节码处的机器代码，然后执行
    jmp(Address(rscratch1, rbx, Address::times_8));
}
```

​	dispatch_next相当于一个“发动机”，它能推进字节码指针，从一个模板表（即字节码表）中找到与当前字节码对应的机器代码片段，并跳到该片段执行字节码片段。字节码执行完后还有一段“结尾曲”代码，会再次调用dispatch_next()。整个解释过程就像由dispatch_next()串联起来的链，只要调用并启动dispatch_next()一次，就能执行方法中的所有字节码。

![](https://pic.imgdb.cn/item/6090f3e3d1a9ae528f5b6ef3.jpg)

​	HotSpot VM的解释器名为模板解释器，它为所有字节码生成对应的机器代码片段，然后全部放入一个表。当VMThread收到一些请求（如垃圾回收任务）并需要安全点时，它会调用TemplateInterpreter::notice_safepoints()通知模板解释器将普通的模板表切换为安全点表，由HotSpot VM在安全点表中寻找与当前字节码对应的逻辑然后跳到该位置执行字节码指令。如果不需要安全点，VMThread会关闭安全点，并调用TemplateInterpreter::ignore_safepoints()，再从安全点表切换回模板表，然后正常执行相关指令。实际上安全点表相比于模板表只多了一次InterpreterRuntime::at_safepoint调用，这个调用用于处理安全点的逻辑。

#### **iconst**

​	iconst向栈压入一个整型常量值

```c++
void TemplateTable::iconst(int value) {
    transition(vtos, itos);
    if (value == 0) {
        __ xorl(rax, rax);
    } else {
        __ movl(rax, value);
    }
}
```

​	iconst的代码并没有压栈操作，它把值放入了rax寄存器，因为相比于寄存器，压栈操作开销较大，模板解释器把rax和xmm0当作栈顶缓存（Top of Stack，ToS），凡是能用ToS解决的绝不用栈。

​	代码最开始有一个transition(vtos,itos)约束条件，它表示iconst字节码执行前栈顶无缓存，执行后栈顶缓存是int类型。约束条件除了vtos（无缓存）和itos（int），还有btos（byte）、ztos（bool）、ctos（char）、stos（short）、ltos（long）、ftos（float，使用xmm0寄存器）、dtos（double，使用xmm0寄存器）和atos（object）。

#### **add、sub、mul、and、or、shl、shr、ushr**

```c++
void TemplateTable::iop2(Operation op) {
    transition(itos, itos);
    switch (op) {
    case add : __ pop_i(rdx); __ addl (rax, rdx); break;
	case sub  :__ movl(rdx, rax);__ pop_i(rax);__ subl (rax, rdx); break;
    case mul : __ pop_i(rdx); __ imull(rax, rdx); break;
    case _and: __ pop_i(rdx); __ andl (rax, rdx); break;
    case _or : __ pop_i(rdx); __ orl  (rax, rdx); break;
    case _xor: __ pop_i(rdx); __ xorl (rax, rdx); break;
    case shl :__ movl(rcx, rax); __ pop_i(rax); __ shll (rax);break;
    case shr :__ movl(rcx, rax); __ pop_i(rax); __ sarl (rax);break;
    case ushr:__ movl(rcx, rax);__ pop_i(rax); __ shrl (rax);break;
    default   : ShouldNotReachHere();
    }
}
```

ToS只能缓存一个数据，而加法需要两个操作数，所以第二个参数必须使用栈操作弹出。

#### **new**

​	new会真实地在堆上分配对象，然后返回对象引用并压入栈中。

```c++
void TemplateTable::_new() {
    // new会在创建对象后向栈顶压入对象引用，所以ToS从无变成atos
    transition(vtos, atos);
    ...
	// 如果开启-XX:+UseTLAB，则在TLAB中分配对象，否则在Eden区中分配对象，
    // 如果分配成功则将对象引用放入rax
    if (UseTLAB) {
        __ tlab_allocate(thread, rax, rdx, 0, rcx, rbx, slow_case);
        // 如果开启-XX:+ZeroTLAB
        if (ZeroTLAB) {
            // 初始化对象头
            __ jmp(initialize_header);
        } else {
            // 初始化对象头和字段
            __ jmp(initialize_object);
        }
    } else {
        __ eden_allocate(thread, rax, rdx, 0, rbx, slow_case);
    }
    // 用0填充对象头和对象字段
    ...
    // 慢速分配路径：调用InterpreterRuntime::_new
    __ bind(slow_case);
    ...
    call_VM(rax, ...InterpreterRuntime::_new);
    // 分配完成
    __ bind(done);
}`	
```

​	new和方法锁类似，在分配前要尽可能使用轻量级操作。它不会直接在堆中分配对象，如如果启用-XX:+UseTLAB，它会尝试在线程的TLAB区中分配对象。TLAB（Thread Local Allocation Buffer）是Thread数据结构的一部分。由于TLAB是每个线程私有的存储区域，因此对象分配无须加锁同步。

​	如果启用-XX:+UseTLAB，它会尝试在线程的TLAB区中分配对象。TLAB（Thread Local Allocation Buffer）是Thread数据结构的一部分。由于TLAB是每个线程私有的存储区域，因此对象分配无须加锁同步。

**iinc**

​	iinc会将某个局部变量的值增加到指定大小。它的字节码占用三个字节，分别对应iinc、index、const。第一个字节表示它是iinc，第二个字节表示局部变量在局部变量表中的索引，第三个字节表示递增大小

**arraylength**

​	arraylength获取数组的长度，然后压栈。

```c++
void TemplateTable::arraylength() {
    transition(atos, itos);  // 执行前栈顶是数组引用，执行后栈顶是数组大小
    // 空值检查，确保数组有length字段 
    __null_check(rax, arrayOopDesc::length_offset_in_bytes());
    // 获取数组（arrayOop）的length字段，然后放入ToS
    __movl(rax,Address(rax,arrayOopDesc::length_offset_in_bytes()));
}
```

**montiorenter**

​	monitorenter是Java关键字synchronized的底层实现，它获取栈顶对象，然后对其加锁

```c++
void TemplateTable::monitorenter() {
    transition(atos, vtos);
    // 在栈的monitor区中分配一个槽，用来存放基本对象锁
..
    __ bind(allocated);
    // 字节码指针递增
    __ increment(rbcp);
    // 将待加锁的对象放入基本对象锁中
    __ movptr(Address(rmon, BasicObjectLock::obj_offset_in_bytes()), rax);
    // 加锁
    __ lock_object(rmon);
    // 确保不会因为刚刚栈上分配的槽造成栈溢出
    __ save_bcp(); 
    __ generate_stack_overflow_check(0);
    // 执行下一条字节码
    __ dispatch_next(vtos);
}
```

​	monitorenter会在栈的monitor区中分配一个基本对象锁。

**athrow**

​	Java语言的throw关键字反映到JVM上是一个athrow字节码。

​	athrow弹出栈顶的对象引用作为异常对象，然后在当前方法寻找该匹配对象类型的异常处理器。如果找到它则清空当前栈，重新压入异常对象，最后跳转到异常处理器，就像没有发生异常一样；如果没有找到异常处理器，则弹出当前栈帧并恢复到调用者栈帧，然后在调用者栈帧上继续抛出异常，这样相当于将异常继续传播到调用者。

```c++
void TemplateTable::athrow() {
    transition(atos, vtos);
    __ null_check(rax);
    __ jump(ExternalAddress(Interpreter::throw_exception_entry()));
}
```

​	athrow会跳到抛异常的机器代码片段，该片段由generate_throw_exception()生成。

```c++
void TemplateInterpreterGenerator::generate_throw_exception() {
	...
    // 抛异常入口
    Interpreter::_throw_exception_entry = __ pc();
    ...  
    // 清空当前栈的一部分
    __ empty_expression_stack();
    // 寻找异常处理器入口地址，如果找到则返回异常处理器入口，否则返回弹出当前栈帧的代码入口
    __ call_VM(rdx,
        CAST_FROM_FN_PTR(address,
        InterpreterRuntime::exception_handler_for_exception),rarg);
    // 不管是哪个入口，结果都会放到rax寄存器
    __ push_ptr(rdx); 
    __ jmp(rax); // 跳到入口执行
    ...
}
```

#### **`if_icmp<cond> & branch`**

​	if_icmp根据条件（大于/等于/小于等于）跳转到指定字节码处。

```c++
void TemplateTable::if_icmp(Condition cc) {
    transition(itos, vtos);
    Label not_taken;
    __ pop_i(rdx);    
    __ cmpl(rdx, rax);               // 栈顶两个整数比较
    __ jcc(j_not(cc), not_taken);    // 根据条件进行跳转
    branch(false, false);            // 跳转
    __ bind(not_taken);              // 不需要跳转
    __ profile_not_taken_branch(rax);
}
```

​	branch()实现了各种跳转行为，如if_icmp、goto等。前面提到的字节码的回边分支是两个可能触发编译行为的地方之一，而回边的实现就位于branch()。回边是指从当前字节码出发到前面的一条路径。

```c++
0: iconst_0
1: istore_1
2: iload_1
3: iconst_2
4: if_icmpeq     13
7: iinc          1, 1
10: goto         2
13: return
```

​	goto字节码会向上跳转到第2条字节码，这样向上跳跃的字节码与目标字节码之间形成的就是一条回边。HotSpot VM不但会对热点方法进行性能计数，还会对回边进行性能计数。通过回边可以探测到热点循环。

​	在运行main函数的过程中（而非运行main函数后）使用编译后的方法替代当前执行，这样的机制被称为OSR。OSR用于方法从低层次（解释器）执行向高层次（JIT编译器）执行变换。

​	发生OSR的时机是遇到回边字节码，而回边又是在branch中体现的。

```c++
void TemplateTable::branch(bool is_jsr, bool is_wide) {
    ...
    if (UseLoopCounter) {
		// / rax: MethodData
        // rbx: MDO bumped taken-count
        // rcx: method
        // rdx: target offset
        // r13: target bcp
        // r14: locals pointer
        // 检查跳转相对于当前字节码是前向还是后向
        __ testl(rdx, rdx);
        // 如果是回边跳转就执行下面的计数操作，否则直接跳到dispatch处
        __ jcc(Assembler::positive, dispatch); 
        ...
        if (TieredCompilation) {
        ...
        // 增加回边计数
        __movptr(rcx,Address(rcx, Method::method_counters_offset()));
        // 检查回边计数是否到达一定的值，如果是就跳转
        __ increment_mask_and_jump(Address(rcx, be_offset), increment,
            mask,rax, false, Assembler::zero,
        UseOnStackReplacement ? &backedge_counter_overflow : NULL);
        } else {
        ... 
        }
        __ bind(dispatch); // [!]正常跳转到目标字节码，然后执行
    }
  	// 加载跳转目标
    __ load_unsigned_byte(rbx, Address(rbcp, 0));
    // 从目标字节码处开始执行
	__ dispatch_only(vtos, true);
   
    if (UseLoopCounter) {
        ...
        // 如果开启栈上替换机制
        if (UseOnStackReplacement) {
            // 回边计数到达一定值，调用frequency_counter_overflow以通知编译器
            __ bind(backedge_counter_overflow);
            __ negptr(rdx);
            __ addptr(rdx, rbcp); 
            __ call_VM(..InterpreterRuntime::frequency_counter_overflow);
            // rax存放编译结果，为NULL则表示没有编译，否则表示进行了编译，且这次编译的
            // 方法即osr nmethod
            // 如果进行了编译，则继续执行，否则跳转到dispatch处
            __ testptr(rax, rax);
            __ jcc(Assembler::zero, dispatch);
            // 确保osr nmethod是执行的方法（in_use表示正常方法， not_used表示不可重
            //入但可以复活的方法，not_installed表示还在编译，not_entrant表示即将退
            // 优化，zombie表示可以被gc，unloaded表示即将转换为zomibe）
            __cmpb(Address(rax,nmethod::state_offset()),nmethod::in_use);
            __ jcc(Assembler::notEqual, dispatch);
            // 到这里说明osr nmethod存在且可以使用，即将进行栈上替换（OSR）
            __ mov(rbx, rax);
            NOT_LP64(__ get_thread(rcx));
            // [!]调用OSR_migration_begin将当前解释器栈的数据打包成OSR buffer        
            call_VM(...SharedRuntime::OSR_migration_begin);
            // 将OSR buffer地址放入rax寄存器
            LP64_ONLY(__ mov(j_rarg0, rax));
			const Register retaddr   = LP64_ONLY(j_rarg2) NOT_LP64(rdi);
            const Register sender_sp = LP64_ONLY(j_rarg1) NOT_LP64(rdx);
            // [!]弹出当前解释器栈
            __movptr(sender_sp,Address(rbp, frame::interpreter_frame_sender_sp_offset * wordSize));
            __ leave();    
            __ pop(retaddr);   
            __ mov(rsp, sender_sp);  
            __ andptr(rsp, -(StackAlignmentInBytes));
            __ push(retaddr);
            // [!] 跳到OSR入口，执行编译后的方法。这样一来就成功完成了从低层次执行
            // 到高层次执行的转换
            __ jmp(Address(rbx, nmethod::osr_entry_point_offset()));
        }
    }
}
```

​	如果目标字节码位于当前字节码前面（回边跳转），情况就复杂很多了。假设main方法里面有一个执行10000次的循环，0～5000次时解释执行，5001～10000次时执行编译后的代码，每隔1000次会调用1次InterpreterRuntime::frequency_counter_overflow。解释器每次执行循环（branch）时都会递增回边计数器，当执行4000次时，回边计数器到达阈值，调用frequency_counter_overflow方法，该方法通知编译器决定是否编译。

代码，每隔1000次会调用1次InterpreterRuntime::frequency_counter_overflow。解释器每次执行循环（branch）时都会递增回边计数器，当执行4000次时，回边计数器到达阈值，调用frequency_counter_overflow方法，该方法通知编译器决定是否编译。

#### **return**

return终止方法执行并返回调用者栈帧，

```c++
void TemplateTable::_return(TosState state) {
    transition(state, state);
    // 如果方法重写了Object.finalize()，会额外调用register_finalizer
    if (_desc->bytecode() == Bytecodes::_return_register_finalizer) {
    ...
        __ call_VM(...InterpreterRuntime::register_finalizer);
        __ bind(skip_register_finalizer);
    }
    // 检查是否需要进入安全点
	if(SafepointMechanism::uses_thread_local_poll()&& _desc->bytecode() != Bytecodes::_return_register_finalizer) {
        ...
        __ call_VM(...InterpreterRuntime::at_safepoint);
        __ pop(state);
        __ bind(no_safepoint);
    }
    ...
    // 弹出当前栈帧
    __ remove_activation(state, rbcp);
    __ jmp(rbcp);
}
```

​	如果重写了Object.finalize()方法，return字节码也会重写成非标准字节码`_return_register_finalizer`，当解释器发现它是重写版本后，会调用InterpreterRuntime::register_finalizer，将对象加入一个链表，等待后面垃圾回收器调用链表中的对象的finalize()方法。处理完finalize重写后，return还会检查是否允许线程局部轮询（区别于全局安全点轮询，详见第10章），并调用InterpreterRuntime::at_safepoint检查是否允许进入安全点。

#### putstatic/putfield

​	putfield将一个值存放到对象成员字段，putstatic将一个值存放到类的static字段。Java有一个volatile关键字，它具有如下三个特性。

* 原子性：读写volatile修饰的变量都是原子性的。相对地，对于非volatile修饰的变量，如long、double类型，是否为原子性由实现定义（Implementation-specific）。
* 可见性：多线程访问变量时，一个线程如果修改了它的值，其他线程能立刻看到最新值。
* 有序性：volatile写操作不能和volatile写操作/读操作发生重排序，但是可以和普通变量读写发生重排序；volatile读操作不能与任何操作发生重排序。

```c++
void TemplateTable::putfield_or_static(...) {
    ...
    // 检查成员是否有volatile关键字
    __ movl(rdx, flags);
    __ shrl(rdx, ConstantPoolCacheEntry::is_volatile_shift);
    __ andl(rdx, 0x1);
    __ testl(rdx, rdx);
    __ jcc(Assembler::zero, notVolatile);
    // 如果是volatile变量，先正常赋值给成员变量再插入内存屏障
    putfield_or_static_helper(...);
    volatile_barrier(...Assembler::StoreLoad|Assembler::StoreStore);
    __ jmp(Done);
    // 如果不是volatile成员变量，简单赋值即可
    __ bind(notVolatile);
    putfield_or_static_helper(...);
    __ bind(Done);
}
```

原子性来自于putfield_or_static_helper()，该函数会判断成员变量类型，然后调用access_store_at()，将值写入成员变量。access_store_at会进一步调用BarrierSetAssembler::store_at

```c++
void BarrierSetAssembler::store_at(...) {
    ...
    switch (type) {
    case T_LONG:
#ifdef _LP64
        __ movq(dst, rax);
#else
        if (atomic) {
            __ push(rdx);
            __ push(rax);                 // 必须用FIST进行原子性更新
            __ fild_d(Address(rsp,0));    // 先加载进FPU寄存器
            __ fistp_d(dst);              // 放入内存
            __ addptr(rsp, 2*wordSize);
        } else {
            __ movptr(dst, rax);
            __ movptr(dst.plus_disp(wordSize), rdx);
        }
#endif
        break;
	}
}
```

​	以上代码均为x64架构，如果BarrierSetAssembler发现变量类型为long且是64位CPU，它会直接使用原生具有原子性的mov操作，如果是32位CPU则使用fild将值放入FPU栈，然后使用fistp读取FPU栈顶元素（ST(0)）并存放到目标位置。volatile的原子性是指写64位值的原子性（fistp），而不是赋值这一过程的原子性（五条指令）。

​	volatile的可见性和一致性是通过volatile屏障实现的。普通变量和volatile变量写之间的唯一区别是volatile写完会插入一个volatile_barrier，由volatile_barrier实际执行membar内存屏障指令。该内存屏障可以保证volatile写完后，后续的读写指令都可以看到volatile的最新值。虽说是内存屏障，但是虚拟机未必会真的使用指令集中的内存屏障指令。

​	x86上lock指令前缀具有内存屏障的效果同时又比内存屏障指令（mfence、sfence、lfence）速度更快，因此在x86上，HotSpot VM是使用代码清单5-27所示的lock addl $0, 0($rsp)指令实现内存屏障的。

```c++
void membar(Membar_mask_bits order_constraint) {
    // x86只会发生StoreLoad重排序，因此只需要处理它
    if (order_constraint & StoreLoad) {
        ...
        // 用lock add 实现内存屏障
        lock();addl(Address(rsp, offset), 0);
    }
}
```

#### invokestatic

​	JVM有五条字节码用于方法调用：invokedynamic调用动态计算的调用点；invokeinterface调用接口方法；invokespecial调用实例方法和类构造函数；invokestatic调用静态方法；invokevirtual调用虚函数。

```c++
void TemplateTable::invokestatic(int byte_no) {
    transition(vtos, vtos);
    prepare_invoke(byte_no, rbx);       // 准备调用
    __ profile_call(rax);
    __ profile_arguments_type(rax, rbx, rbcp, false);
    __ jump_from_interpreted(rbx, rax); // 进行调用
}
```

### 非标准字节码

​	除了实现Java虚拟机规范规定的字节码外，模板解释器还实现了一些非标准字节码，它们都在interpterer/templateTable.cpp中定义。

​	非标准字节码多是对标准字节码的特化，以加速程序运行。

#### fast_iload2

​	有时候代码会连续使用iload从局部变量表将变量加载到操作数栈，最典型的例子为定义两个变量，然后相加

```c++
int a = 23;              // iload 23
int b = 34;              // iload 34
a + b                    // iadd
```

​	上面的字节码将派发三次，每次都要执行前奏曲代码、字节码实现代码、尾曲代码，所以HotSpot VM根据这样的模式，使用非标准字节码fast_iload一次完成两个局部变量的相加。

```c++
void TemplateTable::fast_iload2() {
    transition(vtos, itos);
    locals_index(rbx);
    __ movl(rax, iaddress(rbx));
    __ push(itos);
    locals_index(rbx, 3);
    __ movl(rax, iaddress(rbx));
}
```

#### fast_iputfield

​	前面讨论的putfield字节码的实现较为复杂，因为需要解析字段，检查字段的修饰符。当第一次完成上述操作后，虚拟机会将标准字节码putfield重写为fast版本，这样在第二次执行时将不需要任何额外的操作。

```c++
void TemplateTable::fast_storefield_helper(...) {
    switch (bytecode()) {
    case Bytecodes::_fast_iputfield:
        __ access_store_at(T_INT, IN_HEAP, field, rax, noreg, noreg);
        break;
    ...
}
```

# 并发设施

## 指令重排序

​	编译器可能进行指令调度，可能消除内存访问；CPU为了流水线饱，可能乱序执行指令，可能执行分支预测；Cache可以预取指令或者存储一些程序的执行状态。所有系统组合到一起的效果是程序顺序（代码顺序）与硬件执行指令的执行顺序大相径庭，这个现象即指令重排序。指令重排序会导致多线程环境下程序的行为与开发者预期的不一样，甚至出现严重问题。

### 编译器重排序

​	CPU执行寄存器读写的速度比主存读写快一个或多个数量级。读写操作如果命中L1、L2缓存，那么比从主存中读写快，比从寄存器中读写慢。现代处理器通常使用流水线将不同指令的不同部分放到一起执行，而指令重排序正是为了避免因流水线造成的操作等待。

​	指令重排序有且只有一条规则，即指令重排序不会改变单线程程序的语意，除此之外没有任何限制。如果编译器发现将一个写操作放到读操作后面可能会提升性能，同时这样做不会改变单线程程序的语意，那么编译器就会对代码进行重排序。

​	对于编译器重排序，可以使用编译器提供的编译器屏障（Compiler Barrier）阻止，如GCC使用代码:

```c++
__asm__ volatile ("" : : : "memory");
```

```c++
int v1, v2;
void foo(){
    v1 = v2 + 1;
    __asm__ volatile ("" : : : "memory");
    v2 = 0;
}
```

### 处理器重排序

​	编译器屏障解决了编译器重排序问题，但是并不能完全解决问题，即使消除了编译器重排序，CPU也可能对指令进行重排序，出现类似编译器重排序后的代码序列。CPU级的指令重排序又与CPU架构相关。

​	![](https://pic.imgdb.cn/item/60912206d1a9ae528f3113d1.jpg)

​	如果把指令抽象为读和写两类，那么两者组合后共有四种重排序规则。注意，x86只允许一种重排序规则，即Store操作被重排序到Load后面，而原来的StoreLoad操作变成LoadStore操作，对于CPU级别的指令重排序，我们需要同样由CPU指令集提供的内存屏障（Memory Barrier）指令来阻止。在HotSpot VM中，指令内存屏障的实现位于OrderAccess模块，以x86为例，它的各种内存屏障实现如代码清单6-6所示

```c++
static inline void compiler_barrier() {
    __asm__ volatile ("" : : : "memory");
}
inline void OrderAccess::loadload()   { compiler_barrier(); }
inline void OrderAccess::storestore() { compiler_barrier(); }
inline void OrderAccess::loadstore()  { compiler_barrier(); }
inline void OrderAccess::storeload()  { fence();              }
inline void OrderAccess::fence() {
#ifdef AMD64
    __asm__ volatile ("lock; addl $0,0(%%rsp)" : : : "cc", "memory");
#else
    __asm__ volatile ("lock; addl $0,0(%%esp)" : : : "cc", "memory");
#endif
  	compiler_barrier();
}
```

​	上面的代码是GCC的扩展内联汇编形式，这里的关键字volatile表示禁止编译器优化汇编代码。memory告知编译器汇编代码执行内存读取和写入操作，编译器可能需要在执行汇编前将一些指定的寄存器刷入内存。

​	由于x86只支持StoreLoad重排序，所以x86上的OrderAccess只实现了storeload()，对于其他重排序类型，可以使用编译器屏障简单代替。虽然x86指令集有专门的内存屏障指令，如lfence、sfence、mfence，但是OrderAccess::storeload()使用了指令加上lock前缀来当作内存屏障指令，因为lock指令前缀具有内存屏障的语意且有时候比mfence等指令的开销小。

​	LoadLoad、LoadStore、StoreStore、StoreLoad这四种基本内存屏障外，HotSpot VM还定义了特殊的acquire和release内存屏障：acquire防止它后面的读写操作重排序到acquire的前面；release防止它前面的读写操作重排序到release后面。acquire和release两者放在一起就像一个“栅栏”，可禁止“栅栏”内的事务跑到“栅栏”外，但是它不阻止“栅栏”外的事务跑到“栅栏”内部。之所以说acquire和release特殊是因为它们两个可以通过基本内存屏障组合而成：acquire可由LoadLoad和LoadStore组合而成，release可由StoreStore和LoadStore组合而成。另一个值得注意的地方是acquire和release都没有使用StoreLoad屏障，这意味着x86架构原生就具有acquire和release语意。

​	在Java层面操作内存屏障的办法是Unsafe.loadFence()、Unsafe.storeFence()和Unsafe.fullFence()，它们分别对应OrderAccess::acquire()、OrderAccess::release()、OrderAccess::fence()。

​	四种基本内存屏障是无法在Java层直接使用的。如何放置内存屏障是极具挑战的，它们通常出现在高级并发编程中，是专家级并发开发者的任务，在大多数情况下缺少它们不会产生影响，但是在高并发场景下缺少它们通常是致命的。HotSpot VM内部使用了大量的内存屏障。

```c++
oid Method::set_code(...) {
    ...
    OrderAccess::storestore();
    mh->_from_compiled_entry = code->verified_entry_point();
    OrderAccess::storestore();
    if (!mh->is_method_handle_intrinsic())
        mh->_from_interpreted_entry = mh->get_i2c_entry();
}
```

​	Java 9及以后的版本使用VarHandle代替Unsafe类的一些方法，包括内存屏障操作。

## 内存模型

​	对于Java语言来说，内存模型还可以这样理解：在一些规则的约束下，检查代码执行顺序中的写操作能否被读操作观察到。这些规则被统称为内存模型，在这个模型下，可以确定任意程序点P能否读取到变量V的值。

### happens-before内存模型

​	最严格的内存模型是顺序一致性内存模型（Sequential Consistency Memory Model），顺序一致性内存是指程序顺序和执行顺序完全一致，假设有变量v、读操作r、写操作w，那么顺序一致性内存模型还规定：

* 在执行顺序中，w发生在r前；
* 在w和r之间，没有其他写操作可以发生。



​	换句话说，顺序一致性内存模型要求CPU严格按照代码的顺序执行。它禁止了所有编译器优化和处理器重排序，这对于大多数应用来说都是不可接受的。

​	一种更为宽松的内存模型是happens-before内存模型。不同于顺序一致性内存模型完全禁止优化和重排序，happens-before内存模型只有几条合情合理的约束条件：

1）所有同步动作（加锁、解锁、读写volatile变量、线程启动、线程完成）的代码顺序与执行顺序一致，同步动作的代码顺序也叫作同步顺序。

* 同步动作中对于同一个monitor，解锁发生在加锁前面。
* 同一个volatile变量写操作发生在读操作前面。
* 线程启动操作是该线程的第一个操作，不能有先于它的操作发生。
* 当T2线程发现T1线程已经完成或者连接到T1，T1的最后一个操作要先于T2所有操作。
* 对变量写入默认值的操作要先于线程的第一个操作；对象初始化完成操作要先于finalize()方法的第一个操作。

2）如果a先于b发生，b先于c发生，那么可以确定a先于c发生。
3）volatile的写操作先于volatile的读操作。

```c++
public class HappensBefore {
    static int x = 0;
    static int y = 42;
    public static void main(String[] args) {
        x = 1;
        Thread t = new Thread() {
            public void run() {
                y = x;
                System.out.println(y);
            };
        };
        t.start();
        x = y + 1;
    }
}
```

### Java内存模型

​	happens-before内存模型是Java内存模型的必要不充分条件，它提到的条件是必要的，但是还不能满足Java内存模型的要求。happens-before最致命的问题是它允许值“无中生有”

![](https://pic.imgdb.cn/item/60913195d1a9ae528fb5deb2.jpg)

​	Java内存模型扩展了happens-before内存模型，并规定了哪些因果关系是可以接受的，哪些是不可接受的。

​	新的Java内存模型相当于开发者和硬件系统通过HotSpot VM这个代理人签订的“契约”，该“契约”保证了在一些特定的地方，程序中的代码顺序与硬件的执行顺序一致，而这个一致性的保证通常是由前面提到的内存屏障实现的。

![](https://pic.imgdb.cn/item/60913243d1a9ae528fbc2b0b.jpg)

​	当volatile字段和普通字段读写混合时会需要一些内存屏障的支持，总结以上操作可以得到

![](https://pic.imgdb.cn/item/60913263d1a9ae528fbd419b.jpg)

* 普通读：读取非volatile的普通字段、静态字段，以及读取数组指定索引的元素。
*  普通写：写入非volatile的普通字段、静态字段，以及写入数组指定索引的元素。
* volatile读：和普通读一样，只是字段使用volatile修饰。
* volatile写：和普通写一样，只是字段使用volatile修饰。
* 进入monitor：进入synchronized区域。
* 退出monitor：退出synchronized区域。

## 基础设施

​	HotSpot VM并发的基础设施主要是原子操作、ParkEvent和Parker，后面两个功能的重合度很高，未来可能合并为一个ParkEvent。

### 原子操作

​	原子操作即普通意义上的不可打断的操作。HotSpot VM的原子模块位于runtime/atomic，它实现了原子性的递增值、交换值、比较并交换等操作，其底层实现依赖于CPU指令。

​	x86提供lock指令前缀，以保证一个CPU在执行被修饰的指令期间互斥地拥有对应的Cache Line的所有权。这个保证是并发的基础，并发离不开线程，线程离不开锁，如果多个线程在同一时刻抢锁（互斥量/同步量），锁内部就必须有一条只能互斥执行的代码，这便是原子指令。

### ParkEvent

​	使用ParkEvent可以使线程睡眠与唤醒。一个ParkEvent与一个线程的生命周期绑定，当线程结束时，ParkEvent会移到一个EventFreeList链表，而新创建的线程会在EventFreeList中查找ParkEvent，如果没有就分配新的ParkEvent。ParkEvent本身只有分配和释放接口，但是它继承了平台相关的PlaformEvent，因此它就有了PlatformEvent提供的park、unpark接口。

```c++
void os::PlatformEvent::park() {
    // CAS递减_event的值
    int v;
    for (;;) {
        v = _event;
        if (Atomic::cmpxchg(v - 1, &_event, v) == v) break;
    }
	if (v == 0) {
        int status = pthread_mutex_lock(_mutex);
        // 阻塞线程加一
        ++_nParked;
        // 如果递减之后event为-1则阻塞，否则立刻返回
        while (_event < 0) {
            status = pthread_cond_wait(_cond, _mutex);
        }
            // 阻塞线程减一
            --_nParked;
            _event = 0;

            status = pthread_mutex_unlock(_mutex);
            OrderAccess::fence();
    }
}
void os::PlatformEvent::unpark() {
    // 如果_event大于等于0，则设置为1并返回
    if (Atomic::xchg(1, &_event) >= 0) return;
  
    // 仅当_event为-1时，即另一个线程阻塞住时才执行后面的唤醒另一个线程的操作
    int status = pthread_mutex_lock(_mutex);
    int anyWaiters = _nParked;
    status = pthread_mutex_unlock(_mutex);
    if (anyWaiters != 0) {
        status = pthread_cond_signal(_cond);
  }
}
```

​	ParkEvent依赖的假设是它只被当前绑定的线程park，但是允许多个线程unpark。了解Windows的读者都知道，Windows内核有可以作为同步工具的Event内核对象。当一个操作完成时，可以将Event对象设置为触发状态，此时等待Event事件的线程将得到通知。ParkEvent在Windows上是通过Event内核对象实现的，由于内核的原生支持，其实现也比POSIX简单不少.

**Windows PlatformEvent的实现**

```c++
void os::PlatformEvent::park() {
    int v;
    for (;;) {
        v = _Event;
        if (Atomic::cmpxchg(v-1, &_Event, v) == v) break;
    }
    if (v != 0) return;
    while (_Event < 0) {
        DWORD rv = ::WaitForSingleObject(_ParkHandle, INFINITE);
	}
    _Event = 0;
    OrderAccess::fence();
}

void os::PlatformEvent::unpark() {
    if (Atomic::xchg(1, &_Event) >= 0) return;

    ::SetEvent(_ParkHandle);
}
```

### Parker

​	除了ParkEvent，HotSpot VM还有个与之功能重合的Parker。

```c++
void Parker::park(bool isAbsolute, jlong time) {
    // 如果count为1，当前park调用直接返回
    if (Atomic::xchg(0, &_counter) > 0) return;
.
    if (time == 0) {
        _cur_index = REL_INDEX;
        status = pthread_cond_wait(&_cond[_cur_index], _mutex);
    }
    else {
        // 如果时间不为零，根据isAbsolute来选择毫秒还是微秒
        _cur_index = isAbsolute ? ABS_INDEX : REL_INDEX;
        status = pthread_cond_timedwait(...);
    }
    _cur_index = -1;
    _counter = 0;
    status = pthread_mutex_unlock(_mutex);
    OrderAccess::fence();
    ...
}
void Parker::unpark() {
    int status = pthread_mutex_lock(_mutex);
    const int s = _counter;
    _counter = 1;
    int index = _cur_index;
    status = pthread_mutex_unlock(_mutex);
    // 线程肯定是park的，唤醒它；对于没有park的线程，调用unpark是安全的，因为此时unpark
    // 只会把counter设置为可获得然后返回。
    if (s < 1 && index != -1) {
        status = pthread_cond_signal(&_cond[index]);
    }
}
```

​	Parker的核心是_counter值的变化，_coutner也叫permit。如果permit可获得（为1），那么调用park的线程立刻返回，否则可能阻塞。调用unpark使permit可获得；调用park使permit不可获得。与之不同的是，信号量（Semaphore）的permit可以累加，而Parker只有可获得、不可获得两种状态，它可以被看作受限的信号量。

​	前提到过JDK有个Unsafe类，该类允许Java层做一些底层的工作，如插入内存屏障，Parker也是通过Unsafe类暴露API的。

```c++
package java.util.concurrent.locks;

public class LockSupport {
    private static final Unsafe U = Unsafe.getUnsafe();
    public static void park(Object blocker) {
        Thread t = Thread.currentThread();
        setBlocker(t, blocker);
        U.park(false, 0L);
        setBlocker(t, null);
    }
    public static void unpark(Thread thread) {
        if (thread != null)
            U.unpark(thread);
    }
    ...
}
```



### Moniter

```c++
class Monitor : public CHeapObj<mtInternal> {
    protected:
    SplitWord _LockWord ;                 // 竞争队列（cxq）
    Thread * volatile _owner;             // 锁的owner
    ParkEvent * volatile _EntryList ;     // 等待线程列表
    ParkEvent * volatile _OnDeck ;        // 假定继承锁的线程
    volatile intptr_t _WaitLock [1] ;     // 保护_WaitSet
    ParkEvent * volatile  _WaitSet ;      // 等待集合
    volatile bool     _snuck;             // 用于sneaky locking 
    char _name[MONITOR_NAME_LEN];         // Monitor名称
    ...
```

​	为了追求性能和可扩展性的平衡，Monitor实现了fast/slow惯例。在fast路径时，线程会原子性地修改Lock_Word中的Lock_Byte，如果没有线程竞争Monitor，则线程成功加锁，否则进入slow路径。

​	slow路径的设计思想围绕扩展性。Monitor为锁竞争的线程准备了cxq和EntryList两个队列，并包含了OnDeck和owner两种锁状态。最近达到的线程会进入cxq，owner表示持有当前锁的线程，OnDeck是由owner选择的、作为继承者将要获取到锁的线程。同时，owner也负责将EntryList的线程移动到OnDeck，如果EntryList为空，那么owner会将cxq的所有线程移动到EntryList。

​	就效率来说，slow路径也是同样优秀的。它会限制并发获取锁的线程数目。比如把GC线程放入WaitSet，当条件成立后GC线程不会立即唤醒竞争锁，因为Monitor会将WaitSet转移到cxq中。另外，位于cxq和EntryList中的阻塞态线程也不允许竞争锁。因此，任何时候只有三类线程可以竞争锁：OnDeck、刚释放锁的owner和刚到达但是未被放入cxq的线程。

​	LockWord是一个机器字，它可以表示cxq，存放线程指针作为一个竞争队列，也可以作为一个整型变量扮演锁的角色。线程加锁、解锁实际上是将cxq最低有效字节（LSB，Least Significant Byte）分别置0、置1。Monitor的加锁、解锁过程比较复杂。

```c++
void Monitor::ILock(Thread * Self) {
    // 尝试获取cxq锁，如果没加锁（没竞争）则快速加锁并结束
    if (TryFast()) {
    Exeunt:
        return;
    }
    // 否则有竞争
    ParkEvent * const ESelf = Self->_MutexEvent;
    if (TrySpin(Self)) goto Exeunt;
    ESelf->reset();
    OrderAccess::fence();
    // 尝试获取cxq锁，如果成功则结束，否则将Self线程放入cxq
    if (AcquireOrPush(ESelf)) goto Exeunt;

    // 任何时刻只有一个线程位于OnDeck，如果Self线程没有位于OnDeck，那么阻塞等待
    while (OrderAccess::load_acquire(&_OnDeck) != ESelf) {
        ParkCommon(ESelf, 0);
    }
	// 此时Self位于OnDeck，直到获取到锁，否则它一直待在OnDeck
    for (;;) {
        if (TrySpin(Self)) break;
        ParkCommon(ESelf,0);
    }
    _OnDeck = NULL;
    goto Exeunt;
}
```

​	假设有多条线程同时调用Monitor::ILock，只有一条线程成功执行CAS，将cxq锁的LSB置为1，当其他线程发现竞争后，CAS失败表示发现锁存在竞争，进入竞争逻辑。

​	竞争逻辑是一个复杂且漫长的过程。线程先入cxq，然后阻塞，当它们被唤醒后会先检查自己是否在OnDeck，如果没有则再次阻塞。如果在OnDeck中，除非抢到锁才能退出去，否则一直待在OnDeck中。解锁更为复杂，不仅需要解除锁定，还需要寻找下一个待解锁的线程。

Monitor::IUnlock

```c++
void Monitor::IUnlock(bool RelaxAssert) {
    // 将LSB置为0，释放cxq锁
    OrderAccess::release_store(&_LockWord.Bytes[_LSBINDEX], 0);
    OrderAccess::storeload();
    // 如果OnDeck不为空，唤醒OnDeck线程
    ParkEvent * const w = _OnDeck;
    if (w != NULL) {
        if ((UNS(w) & _LBIT) == 0) w->unpark();
        return;
    }
    // OnDeck为空，如果cxq和EntryList都为空，没有Unlock的线程，直接退出
    intptr_t cxq = _LockWord.FullWord;
    if (((cxq & ~_LBIT)|UNS(_EntryList)) == 0) {
        return;
    }
    // 如果在第一行代码（释放cxq）到此处期间有线程获取到锁，那么当前Unlock返回
    // 寻找Succession的任务并交给那个线程
    if (cxq & _LBIT) {
        return;
    }
    // 寻找Succession（寻找下一个放到OnDeck做Unlock的线程）
    Succession:
    if (!Atomic::replace_if_null((ParkEvent*)_LBIT, &_OnDeck)) {
        return;
    }

    // 如果EntryList不为空，则将第一个元素从EntryList放入OnDeck
    ParkEvent * List = _EntryList;
	if (List != NULL) {
        WakeOne:
        ParkEvent * const w = List;
        _EntryList = w->ListNext;
        OrderAccess::release_store(&_OnDeck, w);
        OrderAccess::storeload();
        cxq = _LockWord.FullWord;
        if (cxq & _LBIT) return;
        w->unpark();
        return;
    }
    // 否则EntryList为空
    cxq = _LockWord.FullWord;
    if ((cxq & ~_LBIT) != 0) {
        // 从cxq获取元素，放到EntryList
        for (;;) {
            // 如果从第一行解释代码到此处有其他线程加锁了，
            // 则寻找Succession的任务并交给那个线程
            if (cxq & _LBIT) goto Punt;
            intptr_t vfy = Atomic::cmpxchg(...);
            if (vfy == cxq) break;
            cxq = vfy;
        }
        // 否则从cxq拿出线程放入EntryList，然后把那个线程从EntryList拿出
        // 放到OnDeck，并唤醒
        _EntryList = List = (ParkEvent *)(cxq & ~_LBIT);
        goto WakeOne;
    }
Punt:
    _OnDeck = NULL; 
    OrderAccess::storeload();
    cxq = _LockWord.FullWord;
    if ((cxq & ~_LBIT) != 0 && (cxq & _LBIT) == 0) {
        goto Succession;
    }
    return;
}
```

​	首先线程抢锁，抢到锁后线程进入等待队列然后释放锁，接着立刻阻塞等待。线程可能因为被其他线程通知，或者等待超时，或者伪唤醒而解除等待状态，此时如果要接着执行wait()后面的代码，需要线程能再次抢到外部的锁。Monitor::IWait()的实现与上述场景大致相同，只是多了些细节。

```c++
int Monitor::IWait(Thread * Self, jlong timo) {
    // 执行wait的线程必须已经获得了锁
    assert(ILocked(), "invariant");

    ParkEvent * const ESelf = Self->_MutexEvent;
    ESelf->Notified = 0;
    ESelf->reset();
    OrderAccess::fence();

    // 将Self线程加入等待集合
    Thread::muxAcquire(_WaitLock, "wait:WaitLock:Add");
    ESelf->ListNext = _WaitSet;
    _WaitSet = ESelf;
    Thread::muxRelease(_WaitLock);

    // 释放外部的锁（即执行wait前加的锁）
    IUnlock(true);
	// 线程阻塞等待，直到收到另一个线程的通知，或者超时唤醒，或者伪唤醒
    for (;;) {
        if (ESelf->Notified) break;
        int err = ParkCommon(ESelf, timo);
        if (err == OS_TIMEOUT) break;
    }
    OrderAccess::fence();

    // 现在线程从wait状态唤醒了，需要将它移出等待集合WaitSet
    int WasOnWaitSet = 0;
    if (ESelf->Notified == 0) {
        Thread::muxAcquire(_WaitLock, "wait:WaitLock:remove");
        if (ESelf->Notified == 0) {
            ParkEvent * p = _WaitSet;
            ParkEvent * q = NULL;
            while (p != NULL && p != ESelf) {
                q = p;
                p = p->ListNext;
            }
            if (p == _WaitSet) {
                _WaitSet = p->ListNext;
            } else { 
                q->ListNext = p->ListNext;
            }
            WasOnWaitSet = 1;
        }
        Thread::muxRelease(_WaitLock);
    }
	// 尝试重新获取锁
    if (WasOnWaitSet) {
        // 如果Self线程是因为wait超时被唤醒，那么它还在等待集合里面，可直接获得锁
        ILock(Self);
    } else {
        // 否则Self线程是因为其他线程通知而被唤醒，尝试抢锁，抢不到就阻塞等待
        for (;;) {
            if (OrderAccess::load_acquire(&_OnDeck) == ESelf 
                && TrySpin(Self)) break;
            ParkCommon(ESelf, 0);
        }
        _OnDeck = NULL;
    }

    // 醒来后可继续执行的线程必须抢到了外部锁
    assert(ILocked(), "invariant");
    return WasOnWaitSet != 0;
}
```

## 锁优化

```c++
void InterpreterMacroAssembler::lock_object(Register lock_reg) {
    // 如果强制使用重量级锁，lock_object()就不做优化了
    if (UseHeavyMonitors) { ... } else {
        ...
        // 将加锁对象放入obj_reg寄存器
        movptr(obj_reg, Address(lock_reg, obj_offset));
        // 如果开启偏向锁优化且偏向加锁成功，跳转到done，
        // 否则跳到slow_case使用重量级锁
        if (UseBiasedLocking) { biased_locking_enter(...);  }
        // 加载1到swap_reg
        movl(swap_reg, (int32_t)1);
		// 获取加锁对象的对象头，与1做位或运算，结果放入swap_reg
        orptr(swap_reg, 
            Address(obj_reg, oopDesc::mark_offset_in_bytes()));
        // 再将swap_reg保存到Displaced Header
        movptr(Address(lock_reg, mark_offset), swap_reg);
        // 使用对象头和swap_reg做比较，如果相等，将对象头替换为指向栈顶基本对象锁的指针，
        // 加锁完成跳到done。否则将swap_reg设置为基本对象指针
        lock();
        cmpxchgptr(lock_reg, 
            Address(obj_reg, oopDesc::mark_offset_in_bytes()));
        jcc(Assembler::zero, done);
        // 加锁失败，再看看当前对象头是否已经是指向栈顶基本对象锁
        const int zero_bits = LP64_ONLY(7) NOT_LP64(3);
        subptr(swap_reg, rsp);
        andptr(swap_reg, zero_bits - os::vm_page_size());
        movptr(Address(lock_reg, mark_offset), swap_reg);
        // 如果成功表示已经加过锁，跳到done完成。否则lock_object各种优化均失败，进入slow_
        // case执行重量级锁
        jcc(Assembler::zero, done);
        // 重量级锁
        bind(slow_case);
        call_VM(...InterpreterRuntime::monitorenter);
        bind(done);
    }
}
```

​	如果用户强制使用重量级锁（-XX:+UseHeavyMonitors）那么使用lock_object()也无济于事。但默认情况下lock_object()会应用一系列优化措施：最开始尝试偏向锁，如果加锁失败则尝试基本对象锁，如果仍然失败再使用重量级锁。具体过程大致如下：

​	**不加锁→偏向锁→基本对象锁→重量级锁**

### 偏向锁

​	锁优化的第一个尝试是偏向锁。如果开启-XX:+UseBiasedLocking偏向锁优化标志，虚拟机将尝试用偏向锁操作免除加锁同步带来的性能惩罚。偏向锁会记录第一次获取该锁对象的线程的指针，然后将它记录在对象头中，并修改对应的位。此时偏向锁偏向于该线程。接下来如果同一个线程在同一个对象上执行同步操作，那么这些操作无须任何原子指令，完全消除了后续加锁、解锁的开销。但是只要有其他线程尝试获取这个锁，偏向模式就会立即结束，虚拟机会撤销偏向，后续加锁、解锁则使用基本对象锁。

​	在多个线程上使用对应的某个对象并进行大量同步操作时，与普通锁相比，偏向锁的性能有明显提升，但是在今天，这些性能提升变得不那么明显。现代处理器的原子操作比以前开销小，另外，由于偏向锁优化针对的应用程序一般都是那些老的、过时的应用程序，它们均使用Java早期的Collection API如Vector、Hashtable，这些类的每个操作都需要同步，而现在的应用程序，在单线程中一般使用非同步的HashMap、ArrayList，在多线程中使用更高效的并发数据结构，所以偏向锁对于现在的应用程序起到的优化效果甚微。除此之外，偏向锁的实现也相当复杂，阻碍了HotSpot VM开发者对代码各个部分的理解，也阻碍了HotSpot VM同步模块的设计变更。因此JEP 374提议在JDK15之后默认关闭偏向锁，并逐渐移除它。

### 基本对象锁

​	如果偏向锁获取失败，虚拟机将尝试基本对象锁。前面提到在lock_object()调用前，栈上monitor区存在一个基本对象锁，包含锁住的对象和BasicLock，BasicLock又包含Displaced Header。虚拟机会尝试获取锁住的对象的对象头然后与1做位或操作（lock_object->mark() | 1），并将获得的结果放入rax寄存器和栈顶Displaced Header。接下来使用原子CAS指令比较rax寄存器和对象头，如果相等，说明对象没有加锁，可以将对象头替换为指向栈顶基本对象锁的指针和00轻量级锁模式。如果不相等，此时CAS操作会将对象头放入rax寄存器，然后查看对象头是否已经指向栈顶指针，即是否已经加过锁。若两次判断都失败，lock_object()膨胀为重量级锁ObjectMonitor。

![](https://pic.imgdb.cn/item/60915777d1a9ae528f825bb6.jpg)

​	lock_object()的代码逻辑。对象头与1位或操作其实就是判断对象尾部2位以确认是否加锁。第3章曾提到32位和64位的对象头，它们的尾部有2位的锁模式。当锁模式为01时表示未被锁定，此时lock_obj->mark() == (lock_obj->mark()|1)，对象头被替换为指向栈上基本对象锁的指针。基本对象锁总是机器位对齐，它的最后两位是00，而锁模式为00时表示已上锁。

### 重量级锁

​	

​	与Object.wait/notify等方法相同，重量级锁会调用runtime/synchronizer的ObjectSynchronizer，它封装了一些逻辑，如对象锁的分配和释放、对象头的改变等，然后由这些函数代理ObjectMonitor执行wait/notify等底层操作。ObjectMonitor即重量级锁底层实现，与Monitor类似，ObjectMonitor也有cxq和EntryList的概念，不过ObjectMonitor的实现相对来说更为复杂。

```c++
void ObjectMonitor::enter(TRAPS) {
    // CAS抢锁，如果当前线程抢到锁则直接返回
    Thread * const Self = THREAD;
    void * cur = Atomic::cmpxchg(Self, &_owner, (void*)NULL);
    if (cur == NULL) {
        return;
    }
	// 否则CAS返回_owner给cur，_owner的值可能是线程指针，也可能是基本对象锁
    // 检查_owner是否为当前线程指针，如果是则当前线程再次加锁（递归计数加一）
    if (cur == Self) {
        _recursions++;
        return;
    }

    // 检查_owner是否为位于当前线程栈上的基本对象锁，如果是则递归计数加一以加锁
    if (Self->is_lock_owned ((address)cur)) {
        _recursions = 1;
        _owner = Self;
        return;
    }

    // 否则当前对象锁的_owner是其他线程或者位于其他线程栈上的基本对象锁
    // 尝试自旋来和其他线程竞争该锁
    Self->_Stalled = intptr_t(this);
    if (TrySpin(Self) > 0) {
        Self->_Stalled = 0;
        return;
    }
    // 如果自旋竞争失败
    JavaThread * jt = (JavaThread *) Self;
    Atomic::inc(&_count);
    { 
        // 改变当前线程状态，使其阻塞在对象锁上
        ...
            EnterI(THREAD);
.
        // 阻塞结束，线程继续执行
            exit(false, Self);
        ...
    }
    Atomic::dec(&_count);
    Self->_Stalled = 0;
}

void ObjectMonitor::exit(bool not_suspended, TRAPS) {
    Thread * const Self = THREAD;
    // 如果_owner不是当前线程
    if (THREAD != _owner) {
        ...
    }
    // 否则_owner是当前线程，或者当前线程栈上的基本对象锁
    // 如果已经加过锁，递归计数减一即可
    if (_recursions != 0) {
        _recursions--;
        return;
    }
    _Responsible = NULL;
    // 由于当前线程没有递归加锁，同时又是对象锁的持有者，这意味着当前线程执行对象锁的exit，
    // 同时还需要找到下一个待唤醒的线程，因为如果当前线程结束了同步执行又没有唤醒其他线程，
    // 那么其他线程会无限等待下去
    for (;;) {
        // 将对象锁持有者置空
        OrderAccess::release_store(&_owner, (void*)NULL);
 OrderAccess::storeload();
        // 如果没有其他线程竞争对象锁，直接返回
        if ((intptr_t(_EntryList)|intptr_t(_cxq)) == 0 || _succ != NULL){
            return;
        }
        if (!Atomic::replace_if_null(THREAD, &_owner)) {
            return;
        }
        // 如果EntryList中存在等待对象锁的线程
        ObjectWaiter * w = NULL;
        w = _EntryList;
        if (w != NULL) {
            ExitEpilog(Self, w);
            return;
        }
        // cxq中存在等待对象锁的线程，将线程从cxq转移到EntryList
        // ---- 1. 保存cxq
        w = _cxq;
        if (w == NULL) continue;
        // ---- 2. 将cxq置空
        for (;;) {
            ObjectWaiter * u = Atomic::cmpxchg(NULL, &_cxq, w);
            if (u == w) break;
            w = u;
        }
        // ---- 3.将cxq转移到EntryList
        _EntryList = w;
		// 将EntryList中的所有线程设置为TS_ENTER
        ObjectWaiter * q = NULL;
        ObjectWaiter * p;
        for (p = w; p != NULL; p = p->_next) {
            p->TState = ObjectWaiter::TS_ENTER;
            p->_prev = q;
            q = p;
        }

        if (_succ != NULL) continue;
        // 唤醒EntryList的第一个线程
        w = _EntryList;
        if (w != NULL) {
            ExitEpilog(Self, w);
            return;
        }
    }
}
```

​	获取对象锁的核心逻辑是首先尝试使用CAS获取锁（设置_owner），如果失败再和其他线程正常竞争对象锁，并在竞争失败的情况下阻塞。

​	释放对象锁只需要检查当前线程是否持锁，如果持锁（且没有多次获取过，即递归计数为0）则释放锁（设置_owner为NULL），同时如果对象锁已经存在其他等待获取的线程，挑选一个等待对象锁的线程唤醒即可。

### RTM锁

​	从因特尔微架构Haswell开始，增加了事务同步扩展指令集，该指令集包括硬件锁消除和受限事务内存（Restricted Transactional Memory，RTM）。下面详细介绍RTM如何从硬件上支持程序执行事务代码。
RTM使用硬件指令实现。xbegin和xend限定了事务代码块的范围，两者结合相当于monitorenter和monitorexit。如果在事务代码块执行过程中没有异常发生，寄存器和内存的修改都会在xend执行时提交。xabort可以用于显式地终止事务的执行，xtest检查EIP/RIP是否位于事务代码块。前文提到过锁的膨胀过程大致如下：

​	**不加锁→偏向锁→基本对象锁→重量级锁**

​	如果开启-XX:+UseRTMLocking，经过C2编译后的代码的加锁过程会多一个RTM加锁代码：

​	**无锁→基本对象锁→重量级锁的RTM加锁→重量级锁**

​	如果同时开启-XX:+UseRTMLocking和-XX:+UseRTMForStackLocks，加锁过程会增加两步：

​	**无锁→基本对象锁的RTM加锁→基本对象锁→重量级锁的RTM加锁→重量级锁**

​	RTM的关键是无数据竞争。当没有数据竞争时，只要多个线程访问xbegin和xend限定事务代码中的同一个内存位置且没有写操作，那么硬件允许多个线程同时并行执行完事务，即使monitor代码段的语义是互斥执行。但是当发生数据竞争时，事务执行会失败，且事务终止的开销和事务重试的开销不容忽视。RTM从实现到工业应用还有很长的一段路要走。

# 垃圾回收

## 垃圾回收概述

### GC Root

​	GC Root又叫根集，它是垃圾回收器扫描存活对象的起始地点。

除了线程栈外，HotSpot VM还有一些地方也可以作为GC Root

1）所有已加载的类的对象引用（ClassLoaderDataGraph::roots_cld_do）；
2）所有线程栈上的对象引用（Threads::possibly_parallel_oops_do）；
3）虚拟机内部使用的Java对象引用（Universe::oops_do,SystemDictionary::oops_do）；
4）JNI Handle（JNIHandles::oops_do）；
5）被synchronized锁住的对象引用（ObjectSynchronizer::oops_do）；

6）Java工具用到的对象引用（Management::oops）；
7）JVMTI导出对象引用（JvmtiExport::oops_do）；
8）AOT堆对象引用（AOTLoader::oops）；
9）CodeCache代码引用（CodeCache::blobs_do）；
10）String常量池对象引用（StringTable::oops_do）。

### 安全点

​	在垃圾回收器的眼中只有垃圾回收线程和修改对象的线程，后者被称为Mutator线程。由于垃圾回收线程也需要修改对象，尤其是在垃圾回收过程中可能有移动对象的情况，如果Mutator线程在移动对象的同时修改对象，势必会造成错误，因此在垃圾回收时一般需要全过程，或者部分过程暂停Mutator线程，这种暂停Mutator线程的现象又叫作世界停顿（Stop The World，STW）。一般来说，Mutator线程可以主动或者被动达到STW，在HotSpot VM中，使用安全点（Safepoint）作为主动STW机制。安全点本质上是一页内存。

```c++
void SafepointMechanism::default_initialize() {
    if (ThreadLocalHandshakes) {
    ...
    // 分配两页内存，一页用于bad_page，一页用于good_page
    char* polling_page = os::reserve_memory(...);
    char* bad_page  = polling_page;
    char* good_page = polling_page + page_size;
    // bad_page表示这片内存不可读不可写，good_page表示可读
    os::protect_memory(bad_page,  page_size, os::MEM_PROT_NONE);
    os::protect_memory(good_page, page_size, os::MEM_PROT_READ);
    os::set_polling_page((address)(bad_page));
    ...
} else {
    // 分配一页内存
    char* polling_page = os::reserve_memory(...);
    os::commit_memory_or_exit(...);
    // 将它设置为可读
    os::protect_memory(polling_page, page_size, os::MEM_PROT_READ);
	os::set_polling_page((address)(polling_page));
    }
}
```

​	虚拟机将“读取安全点内存页”的操作安插在一些合适的地方。当程序没有请求垃圾回收时（实际上除了垃圾回收外，还有其他操作可能会请求安全点），安全点内存页可读，Mutator线程对安全点的访问不会引发任何问题。当需要垃圾回收时，VMThread将安全点设置为不可读不可写，然后等待所有Mutator线程走到安全点。由于Mutator线程访问不可读不可写的内存时会引发异常信号，虚拟机可通过内部的信号处理器捕获并停止Mutator线程的执行，这样一来相当于让所有Mutator线程主动停止。

​	在具体实现中，SafepointSynchronize::begin()和SafepointSynchronize::end()分别表示安全点的开启和关闭，两者之间构成一个安全区域，它们只能被VMThread调用。

```c++
void SafepointSynchronize::begin() {
    ...
    // 设置状态为安全点开启中
    _state            = _synchronizing;
    // 如果使用全局安全点，修改安全点内存页，将其设置为不可读不可写
    // （对应还有如果使用线程握手的处理，这里已省略）
    if (SafepointMechanism::uses_global_page_poll()) {
        Interpreter::notice_safepoints();
        PageArmed = 1 ;
        os::make_polling_page_unreadable();
    }
    ...
    while(still_running > 0) {
        jtiwh.rewind();
        // 对于当前所有运行的线程
        for (; JavaThread *cur = jtiwh.next(); ) {
            // 获取线程状态
            ThreadSafepointState *cur_state = cur->safepoint_state();
            // 如果还在运行
            if (cur_state->is_running()) {
                // 检查线程是不是suspend或者其他情况，并处理它
                cur_state->examine_state_of_thread();
                // 再次检查线程是否还在运行
                if (!cur_state->is_running()) {
                    // 如果没有运行，计数减一（still_running表示当前还在运行的线程）
					still_running--;
                }
            }
        }
    ... // 如果循环太多次，可能会使当前线程暂停
    }
    // VMThread等待所有线程停下来
    while (_waiting_to_block > 0) {
        ... Safepoint_lock->wait(true, remaining_time / MICROUNITS);
    }
    // 安全点开启成功，设置状态，计数增加
    _safepoint_counter ++;
    _state = _synchronized;
    OrderAccess::fence();
    ... // 日志记录等
}
```

1）解释器线程：第5章提到过，VMThread调用TemplateInterpreter::notice_safepoints通知模板解释器将模板表切换为安全点表（这意味着执行完一条字节码后遇到一个安全点时，可以进入安全点），安全点表除了执行字节码代码外还负责安全点处理，其中就包括进入安全点。

2）执行native代码的线程：VMThread不会暂停执行native代码的线程，但是当线程从native代码返回到Java代码时，需要检查_state，如果发现是_synchronizing则线程停止。

3）执行编译后的代码的线程：开启安全点后，执行编译后的代码的线程使用test指令访问安全点，此时安全点不可读，所以引发异常信号，异常信号会被虚拟机的信号处理器（在Linux平台上是handle_linux_signal）捕获，然后阻塞线程。

4）已经阻塞的线程：对于已经阻塞的线程，继续保持阻塞状态即可，在安全点操作没有结束前不允许醒来。

5）执行虚拟机内部代码或者正在状态转换的线程：Java线程大部分时间在执行字节码，有时也会执行虚拟机自身的一些代码，这些线程会在状态转换时阻塞自身。

### 线程局部握手	

​	线程局部握手（ThreadLocal Handshakes）标志，它是JEP 312引入的特性。根据上面的描述，安全点是一个全局的内存页，一旦VMThread开启安全点（将内存设置为不可读不可写）后，所有Mutator线程都会继续运行直到遇到附近的安全点读取，再通过异常处理机制主动停止。但是有时并不需要停止所有Mutator线程，如偏向锁撤销，或者打印某个线程的线程栈，在这些情况下，VMThread只需要停止某个指定的线程并打印线程栈即可。基于这些考虑，HotSpot VM引入了线程局部握手机制，使VMThread可以有选择性地针对某个线程开启或者关闭线程局部的安全点。

### GC屏障

​	GC屏障即后缀为BarrierSet的一系列类，它们的作用是在字段读操作或者写操作前后插入一段代码，执行某些垃圾回收必要的逻辑

```c++
public void barrier(Struct obj){
    // Write_barreir_pre();
    obj.field = new Object();
    // Write_barreir_post();
}
```

​	虚拟机在字段写操作前后可以分别插入前置写屏障、后置写屏障（读屏障同理），这些写屏障会执行一些GC必要的逻辑，如检测到对象引用关系的修改并记录到记忆集中。GC屏障对性能有较大影响，因为字段读写操作是程序最常见的行为，所以不应该在GC屏障中放置“重量级”代码。

## Epsilon GC

```
├── epsilonArguments.cpp
├── epsilonArguments.hpp     # GC参数，如是否使用TLAB、是否开启EpsilonGC等
├── epsilonBarrierSet.cpp
├── epsilonBarrierSet.hpp         # GC barrier
├── epsilonCollectorPolicy.hpp    # 垃圾回收策略，如堆初始大小、最大最小值等
├── epsilonHeap.cpp
├── epsilonHeap.hpp               # Epsilon GC的堆
├── epsilonMemoryPool.cpp
├── epsilonMemoryPool.hpp         # 堆内存的使用情况、GC次数、上次GC时间等
├── epsilonMonitoringSupport.cpp
├── epsilonMonitoringSupport.hpp  # PerfData支持
├── epsilonThreadLocalData.hpp    # 对象在TLAB分配
├── epsilon_globals.hpp           # Epsilon GC特定虚拟机参数
└── vmStructs_epsilon.hpp         # Serviceability Agent支持
```

## EpsilonHeap

​	每个垃圾回收器都抽象出了自己的Java堆结构，包含最重要的对象分配和回收垃圾接口，如Epsilon GC使用EpsilonHeap；Serial GC使用SerialHeap；CMS GC使用CMSHeap。

```c++
class EpsilonHeap : public CollectedHeap {
private:
    EpsilonCollectorPolicy* _policy;           // 回收器策略
    SoftRefPolicy _soft_ref_policy;            // 软引用清除策略
    EpsilonMonitoringSupport* _monitoring_support; // perfdata支持
    MemoryPool* _pool;                         // 感知内存池使用情况
    GCMemoryManager _memory_manager;           // 内存管理器
    ContiguousSpace* _space;                   // 实际堆空间
    VirtualSpace _virtual_space;               // 虚拟内存
	size_t _max_tlab_size;                     // 最大TLAB
    size_t _step_counter_update;               // perfdata更新频率
    size_t _step_heap_print;                   // 输出堆信息频率
    int64_t _decay_time_ns;                    // TLAB大小衰减时间
    volatile size_t _last_counter_update;      // 最后一次perdata更新
    volatile size_t _last_heap_print;          // 最后一次输出堆信息
public:
    virtual HeapWord* mem_allocate(...);       // 内存分配
    virtual HeapWord* allocate_new_tlab(...);  // TLAB分配
    virtual void collect(...);                 // System.gc触发
    virtual void do_full_collection(...);      // 普通垃圾回收
    ...
};
```

​	EpsilonHeap继承自CollectedHeap，表示可用于垃圾回收的Java堆，它实现了垃圾回收的部分常用操作，剩下的工作留给纯虚函数[2]，需要子类具体实现。代码清单10-5所示的EpsilonHeap实现了CollectedHeap的所有纯虚函数，换句话说，EpsilonHeap实现了一个最小化的Java堆所必须实现的功能。出于这个原因，用户如果想为HotSpot VM定制或实现一个新的垃圾回收器，可以仿照Epsilon GC实现自己的功能。

### 对象分配

​	当虚拟机想在Java堆上分配对象时，它会找到oop对应的klass，并调用InstanceKlass::allocate_instance()，由该函数调用Java堆的内存分配接口，即mem_allocate()，分配新的内存以容纳对象。

```c++
HeapWord* EpsilonHeap::mem_allocate(...) {
    *gc_overhead_limit_was_exceeded = false;
    return allocate_work(size);
}
HeapWord* EpsilonHeap::allocate_work(size_t size) {
    // 分配内存（Lock free）
    HeapWord* res = _space->par_allocate(size);
    // 当分配失败时，尝试扩容，然后再次尝试分配
    while (res == NULL) {
        MutexLockerEx ml(Heap_lock);
		size_t space_left = max_capacity() - capacity();
        size_t want_space = MAX2(size, EpsilonMinHeapExpand);
        // 如果剩余空间大于请求扩容空间，那么可以扩容
        if (want_space < space_left) {
            bool expand = _virtual_space.expand_by(want_space);
        } else if (size < space_left) {
            // 如果剩余空间不能完成扩容，但还是可能完成这次对象分配
            bool expand = _virtual_space.expand_by(space_left);
        } else {
            return NULL;// 没有剩余空间，分配失败
        }
        // 修改堆结束位置，即扩容
        _space->set_end((HeapWord *) _virtual_space.high());
        // 再次尝试内存分配
        res = _space->par_allocate(size);
    }
    size_t used = _space->used();
    ... // 分配成功，输出log信息
    return res;
}
```

### 回收垃圾

​	EpsilonHeap::collect()只是简单记录垃圾回收请求并更新计数。

## Serial GC

​	它的Java堆符合弱分代假说（Weak Generational Hypothesis）。弱分代假说的含义是大多数对象都在年轻时死亡，这个假说已经在各种不同编程范式或者编程语言中得到证实。与之相对的是强分代假说，它的含义是越老的对象越不容易死亡，但是支持该假说的证据稍显不足。

​	虚拟机实现了分代堆模型，它将Java堆分为空间较大的老年代（Old Generation）和空间较小的新生代（Young Generation）。其中新生代容纳朝生夕死的新对象，在此区域发生垃圾回收较为频繁，老年代容纳生命周期较长的对象，可以简单认为多次垃圾回收后仍然存活的对象生命周期较长。老年代增长缓慢，因此发生垃圾回收的频率较低，这样的堆被称为分代堆。在分代堆模型中，GC的工作不再是面向整个堆，而是“专代专收”，Young GC（以下简称YGC）只回收新生代，Full GC（以下简称FGC）回收整个堆。YGC的出现使得GC无须遍历整个堆寻找存活对象，同时降低了老年代回收的频率。

​	分代堆受益于对象生命周期的区分，但是也受桎于它。之前只需要遍历整个堆即可找出所有存活对象，分代后却不能简单遍历单个分代，因为可能存在老年代指向新生代的引用，即跨代引用。如果只遍历新生代可能会错误标记一些本来存在引用的对象，继而杀死，而垃圾回收的原则是“宁可漏过不可错杀”，错误地清理存活对象是绝对不可以的。现在的问题是分代后新生代对象除了被GC Root引用外还会被老年代跨代引用，如果要遍历空间较大的老年代和GC Root才能找出新生代的存活对象，那么就失去了分代的优势，得不偿失。

​	跨代引用是所有分代式垃圾回收器必须面对的问题，为了处理跨代引用问题，需要一种名为记忆集（Remember Set，RSet）的数据结构来记录老年代指向新生代的引用。

​	记忆集暗示着GC拥有发现每个写对象操作的能力，每当对象写操作发生时，GC会检查被写入对象是否位于不同分代并据此决定是否放入记忆集。赋予GC这种“发现所有写对象操作”能力的组件是GC屏障，具体到上下文中是写屏障。写对象操作属于代码执行系统的一部分，由GC屏障与JIT编译器、模板解释器合力完成。

​	在Serial GC中，FGC遍历整个堆，不需要考虑跨代引用，YGC只发生在新生代，需要处理跨代引用问题。Serial GC使用的是一种名为卡表的粗粒度的记忆集，下面将展开具体介绍。

### 卡表

​	卡表（Card Table）是一种可以存储跨代引用的粗粒度的记忆集，它没有精确记录老年代中指向新生代的对象和引用，而是将老年代划分为2次幂大小的一些内存页，记录它们所在的内存页。用卡表来映射这些页，减少了记忆集本身的内存开销，同时也尽量避免了整个老年代的遍历。标准的卡表实现通常为一个bitmap，它的每个bit对应一片内存页。

![](https://pic.imgdb.cn/item/6092148ad1a9ae528f9c2250.jpg)

​	当Mutator线程执行类成员变量赋值操作时，虚拟机会检查是否将一个老年代对象或引用赋值给新生代成员，如果是，则对成员变量所在内存页对应的卡表中的bit进行标记，后续只需要遍历卡表中标记过的bit对应的内存页，而无须遍历整个老年代。

​	不过使用bitmap可能会相当慢，因为对bitmap其中一个bit标记时，需要读取整个机器字，更新，然后写回，另外在RISC处理器上执行bit操作也需要数条指令。一个有效的性能改进是使用byte数组代替bitmap，虽然byte数组使用的内存是bitmap的8倍，但是总的内存占比仍然小于堆的1%。HotSpot VM的卡表由CardTable实现，它使用byte数组而非bitmap，CardTable::byte_for函数负责内存地址到卡表byte数组的映射。

```c++
jbyte* CardTable::byte_for(const void* p) const {
    jbyte* result = &_byte_map_base[uintptr_t(p) >> card_shift];
    return result; 
}
```



​	其中card_shift为9。从实现中不难看出虚拟机将一片内存页定义为512字节，每当某个内存页存在跨代引用时就将byte_map_base数组对应的项标记为dirty。

### Young GC

​	Serial GC将新生代命名为DefNewGeneration，将老年代命名为TenuredGeneration。DefNewGeneration又将新生代划分为Eden空间和Survivor空间，而Survivor空间又可进一步划分为From、To空间。

![](https://pic.imgdb.cn/item/6092165fd1a9ae528fad3e33.jpg)

​	YGC使用复制算法清理新生代空间。关于YGC的一个常见场景是起初在Eden空间分配小对象，当Eden空间不足时发生YGC，此时Eden空间和From空间的存活对象被标记，接着虚拟机将两个空间的存活对象转移到To空间，如果To空间不能容纳对象，那么会转移到老年代。如果To空间能够容纳对象，Eden空间和From空间清空，From空间和To空间交换角色，此时存在一个空的Eden空间、存在部分存活对象的From空间以及空的To空间，当下次YGC发生时，重复上述步骤。

​	当一些对象在多次YGC后仍然存活时，可以认为该对象生命周期较长，不属于朝生夕死的对象，所以GC会晋升该对象，将其从新生代的对象晋升到老年代。

```c++
void DefNewGeneration::collect(...) {
    ...
    if (!collection_attempt_is_safe()) {// 检查老年代是否能容纳晋升对象
        heap->set_incremental_collection_failed();
        return;
    }
    FastScanClosure fsc_with_no_gc_barrier(...);
    FastScanClosure fsc_with_gc_barrier(...);
    CLDScanClosure cld_scan_closure(...);
    FastEvacuateFollowersClosure evacuate_followers(...);
    { // 从GC Root出发扫描存活对象
        StrongRootsScope srs(0);
        heap->young_process_roots(&srs, &fsc_with_no_gc_barrier,
            &fsc_with_gc_barrier, &cld_scan_closure);
    }
    evacuate_followers.do_void();// 处理非GC Root直达、成员字段可达的对象
    ... // 特殊处理软引用、弱引用、虚引用、final引用
    // 如果可以晋升，则清空Eden、From空间；交换From、To空间；调整老年代晋升阈值
    if (!_promotion_failed) {
        eden()->clear(SpaceDecorator::Mangle);
        from()->clear(SpaceDecorator::Mangle);
        swap_spaces();
    } else {
    // 否则通知老年代晋升失败，仍然交换From和To空间
        swap_spaces();
        _old_gen->promotion_failure_occurred();
    }
    ...
}
```

​	在做YGC之前需检查此次垃圾回收是否安全（collection_attempt_is_safe）。所谓是否安全是要判断在新生代全是需要晋升的存活对象的最坏情况下，老年代能否安全容纳这些新生代。如果可以再继续做YGC。

​	young_process_roots()会扫描所有类型的GC Root，并扫描卡表记忆集找出老年代指向新生代的引用，然后使用快速扫描闭包将它们复制到To空间。快速扫描闭包即FastScanClosure，它将针对一个对象（线程、对象、klass等）的操作抽象成闭包操作，然后传递到处理连续对象的逻辑代码中。由于HotSpot VM使用的C++ 98语言标准没有lambda表达式，所以只能使用类模拟出闭包[1]。

```c++
template <class T> inline void FastScanClosure::do_oop_work(T* p) {
    // 从地址p处获取对象
    T heap_oop = RawAccess<>::oop_load(p); 
 	if(!CompressedOops::is_null(heap_oop)) {
        oop obj = CompressedOops::decode_not_null(heap_oop);
        // 如果对象位于新生代
        if ((HeapWord*)obj < _boundary) {
            // 如果对象有转发指针，相当于已复制过，那么可以直接使用已经复制后的对象，否则
            // 需要复制
            oop new_obj = obj->is_forwarded() 
                ?obj->forwardee(): _g->copy_to_survivor_space(obj);
            RawAccess<IS_NOT_NULL>::oop_store(p, new_obj);
            if (is_scanning_a_cld()) { // 根据情况设置gc_barrier
                do_cld_barrier();
            } else if (_gc_barrier) {
            do_barrier(p);
            }
        }
    }
}
```

​	从GC Root和老年代出发，所有能达到的对象都是活对象，FastScanClosure会应用到每个活对象上。如果遇到已经设置了转发指针的对象，即已经复制过的，则直接返回复制后的对象。，否则使用如代码清单10-10所示的copy_to_survivor_space进行复制。

```c++
oop DefNewGeneration::copy_to_survivor_space(oop old) {
    size_t s = old->size();
    oop obj = NULL;

    // 在To空间分配对象
    if (old->age() < tenuring_threshold()) {
        obj = (oop) to()->allocate_aligned(s);
    }
    // To空间分配失败，在老年代分配
    if (obj == NULL) {
        obj = _old_gen->promote(old, s);
        if (obj == NULL) {
            handle_promotion_failure(old);
            return old;
        }
    } else {
        // To空间分配成功
        const intx interval = PrefetchCopyIntervalInBytes;
        Prefetch::write(obj, interval); // 预取到缓存
        // 将对象复制到To空间
        Copy::aligned_disjoint_words((HeapWord*)old,(HeapWord*)obj,s);
        // 对象年龄增加
        obj->incr_age();
        age_table()->add(obj, s);
    }

    // 在对象头插入转发指针（使用新对象地址代替之前的对象地址，并设置对象头GC bit）
    old->forward_to(obj);
 	return obj;
}
```

​	copy_to_survivor_space()视情况将对象复制到To空间或者晋升到老年代，然后为老对象设置新对象地址，即可转发指针（Forwarding Pointer）。设置转发指针的意义在于GC Root可能存在两个指向相同对象的槽位，如果简单移动对象，并将槽位修改为新的对象地址，第二个GC Root槽位就会访问到错误的老对象地址，而设置转发指针后，后续对老对象的访问将转发到正确的新对象上。

​	上述过程会触碰到GC Root和老年代出发直接可达的对象，并将它们移动到To空间（或者晋升老年代），这些移动后的对象可能包含引用字段，即可能间接可达其他对象。Serial GC维护一个save_mark指针和已分配空间顶部（to()->top()）指针，To空间底部到save_mark的区域中的对象表示自身和自身字段都扫描完成的对象，save_mark到空间顶部的区域中的对象表示自身扫描完成但是自身字段未完成的对象。FastEvacuateFollowersClosure的任务就是扫描save_mark到空间顶部的对象遍历它们的字段，并将这些能达到的对象移动到空间底部到save_mark的区域，然后向前推进save_mark，直到save_mark等于空间顶部，扫描完成。
​	由于新生代对象可能移动到To空间，也可能晋升到老年代，所以上述逻辑对于老年代也同样适用。

### Full GC

​	由于历史原因，FGC的实现位于serial/genMarkSweep。虽然从名字上看SerialGC的FGC的实现似乎是基于标记清除算法，但是实际上FGC是基于标记压缩算法实现。

![](https://pic.imgdb.cn/item/60921f0ed1a9ae528ffd878c.jpg)

​	FGC使用的标记整理算法是基于Donald E. Knuth提出的Lisp2算法：首先标记（Mark）存活对象，然后把所有存活对象移动（Compact）到空间的一端。FGC始于TenuredGeneration::collect，它会在GC前后记录一些日志，可以使用-Xlog:gc*输出这些日志。

```c++
GC(1) Phase 1: Mark live objects
GC(1) Phase 2: Compute new object addresses
GC(1) Phase 3: Adjust pointers
GC(1) Phase 4: Move objects
```

![](https://pic.imgdb.cn/item/60922083d1a9ae528f0c0e36.jpg)

#### 标记存活对象（Mark Live Object）

​	然后使用XX::oops_do(root_closure)从该GC Root出发标记所有存活对象。XX表示GC Root类型，root_closure表示标记存活对象的闭包。root_closure即MarkSweep::FollowRootClosure闭包，给它一个对象，就能标记这个对象、标记迭代标记对象的成员，以及标记对象所在的栈的所有对象及其成员。

```c++
template <class T> inline void MarkSweep::follow_root(T* p) {
    // 如果引用指向的对象不为空且未标记
    T heap_oop = RawAccess<>::oop_load(p);
    if (!CompressedOops::is_null(heap_oop)) {
        oop obj = CompressedOops::decode_not_null(heap_oop);
        if (!obj->mark_raw()->is_marked()) {
            mark_object(obj);    // 标记对象
            follow_object(obj);  // 标记对象的成员 
        }
    }
    follow_stack();              // 标记引用所在栈
}
// 如果对象是数组对象则标记数组，否则标记对象的成员
inline void MarkSweep::follow_object(oop obj) {
    if (obj->is_objArray()) {
        MarkSweep::follow_array((objArrayOop)obj);
    } else {
        obj->oop_iterate(&mark_and_push_closure);
   }
}
void MarkSweep::follow_stack() {    // 标记引用所在的整个栈
    do {
        // 如果待标记栈不为空则逐个标记
        while (!_marking_stack.is_empty()) {
            oop obj = _marking_stack.pop();
            follow_object(obj);
    }
    // 如果对象数组栈不为空则逐个标记
    if (!_objarray_stack.is_empty()) {
        ObjArrayTask task = _objarray_stack.pop();
        follow_array_chunk(objArrayOop(task.obj()), task.index());
        }
    }while(!_marking_stack.is_empty()||!_objarray_stack.is_empty());
}
// 标记数组的类型的Class和数组成员，比如String[] p = new String[2]
// 对p标记会同时标记java.lang.Class，p[1],p[2]
inline void MarkSweep::follow_array(objArrayOop array) {
    MarkSweep::follow_klass(array->klass());
    if (array->length() > 0) {
        MarkSweep::push_objarray(array, 0);
    }
}
```

#### 计算对象新地址（Compute New Object Address）

​	标记完所有存活对象后，Serial GC会为存活对象计算出新的地址，然后存放在对象头中，为接下来的对象整理（Compact）做准备。计算对象新地址的思想是先设置cur_obj和compact_top指向空间底部，然后从空间底部开始扫描，如果cur_obj扫描到存活对象，则将该对象的新地址设置为compact_top，然后继续扫描，重复上述操作，直至cur_obj到达空间顶部。

#### 调整对象指针（Adjust Pointer）

​	虽然计算出了对象新地址，但是GC Root指向的仍然是老对象，同时对象成员引用的也是老的对象地址，此时通过调整对象指针可以修改这些指向关系，让GC Root指向新的对象地址，然后对象成员的引用也会相应调整为引用新的对象地址。

#### 移动对象（Move object）

​	当一切准备就绪后，就在新地址为对象分配了内存，且引用关系已经修改，但是新地址的对象并不包含有效数据，所以要从老对象地址处将对象数据逐一复制到新对象地址处，至此FGC完成。Serial GC将重置GC相关数据结构，并用日志记录GC信息。

### 世界停顿

​	世界停顿（Stop The World，STW）即所有Mutator线程暂停的现象。Serial GC的YGC和FGC均使用单线程进行，所以GC工作时所有Mutator线程必须暂停，Java堆越大，STW越明显，且长时间的STW对于GUI程序或者其他要求伪实时、快速响应的程序是不可接受的，所以STW是垃圾回收技术中最让人诟病的地方之一：一方面所有Mutator线程走到安全点需要时间，另一方面STW后垃圾回收工作本身也需要大量时间。那么，能否利用现代处理器多核，并行化STW后垃圾回收中的部分工作呢？关于这一点，Parallel GC给出了一份满意的答案。

## Parallel GC

### 多线程垃圾回收

​	Parallel GC即并行垃圾回收器，它是面向吞吐量的垃圾回收器，使用-XX:+UseParallelGC开启。Parallel GC是基于分代堆模型的垃圾回收器，其YGC和FGC的逻辑与Serial GC基本一致，只是在垃圾回收过程中不再是单线程扫描、复制对象等，而是用GCTaskManager创建GCTask并放入GCTaskQueue，然后由多个GC线程从队列中获取GCTask并行执行。相比单线程的Serial GC，它的显著优势是当处理器是多核时，多个GC线程使得STW时间大幅减少。

```c++
bool PSScavenge::invoke_no_policy() {
    ...
    {
        // GC任务队列
	GCTaskQueue* q = GCTaskQueue::create();
        // 扫描跨代引用
        if (!old_gen->object_space()->is_empty()) {
            uint stripe_total = active_workers;
            for(uint i=0; i < stripe_total; i++) {
                q->enqueue(new OldToYoungRootsTask(...));
            }
        }
        // 扫描各种GC Root
        q->enqueue(new ScavengeRootsTask(universe));
        q->enqueue(new ScavengeRootsTask(jni_handles));
        PSAddThreadRootsTaskClosure cl(q);
        Threads::java_threads_and_vm_thread_do(&cl);
        q->enqueue(new ScavengeRootsTask(object_synchronizer));
        q->enqueue(new ScavengeRootsTask(management));
        q->enqueue(new ScavengeRootsTask(system_dictionary));
        q->enqueue(new ScavengeRootsTask(class_loader_data));
        q->enqueue(new ScavengeRootsTask(jvmti));
        q->enqueue(new ScavengeRootsTask(code_cache));

        TaskTerminator terminator(...);
        // 如果active_workers大于1，添加一个StealTask
        if (gc_task_manager()->workers() > 1) {
            for (uint j = 0; j < active_workers; j++) {
                q->enqueue(new StealTask(terminator.terminator()));
            }
        }
        // 停止继续执行，直到上述Task执行完成
	gc_task_manager()->execute_and_wait(q);
    }
    // 处理非GC Root直达、成员字段可达的对象
    PSKeepAliveClosure keep_alive(promotion_manager);
    PSEvacuateFollowersClosure evac_followers(promotion_manager);
    ...
    // YGC结束，交换From和To空间
    if (!promotion_failure_occurred) {
        young_gen->eden_space()->clear(SpaceDecorator::Mangle);
        young_gen->from_space()->clear(SpaceDecorator::Mangle);
        young_gen->swap_spaces();
        ...
    }
    return !promotion_failure_occurred;
}
```

​	Serial GC使用young_process_roots()扫描GC Root，而Parallel GC是将GC Root扫描工作包装成一个个GC任务，放入GC任务队列等待GC任务管理器一起处理；Serial GC使用FastEvacuateFollowersClosure处理对象成员字段可达对象，而Parallel GC使用PSEvacuateFollowersClosure多线程处理；不过，YGC完成后Serial GC和Parllel GC都会交换From和To空间。从算法上看，两个垃圾回收器并无太大区别，只是Parallel GC充分利用了多核处理器。

### GC任务管理器

​	Parallel GC使用ScavengeRootsTask表示GC Root扫描任务。ScavengeRootsTask实际上继承自GCTask，它会被放入GCTaskQueue，然后由GCTaskManager统一执行。

![](https://pic.imgdb.cn/item/609235bad1a9ae528f01a62e.jpg)

​	垃圾回收器会向GCTaskQueue投递OldToYoungRootTask、ScavengeRootsTask、ThreadRootsTask和StealTask，然后execute_and_wait()会阻塞垃圾回收过程，直到所有GC Task被GC线程执行完毕，这也是并发垃圾回收器和并行垃圾回收器的显著区别：并发垃圾回收器（几乎）不会阻塞垃圾回收过程，而并行垃圾回收器会阻塞整个GC过程。

​	实际上execute_and_wait()也创建了一个GC Task。

```c++
GCTaskManager::execute_and_wait(GCTaskQueue* list) {
    WaitForBarrierGCTask* fin = WaitForBarrierGCTask::create();
    list->enqueue(fin);
    OrderAccess::storestore();
    add_list(list);
    fin->wait_for(true /* reset */);
    WaitForBarrierGCTask::destroy(fin);
}
```

​	GCTaskManager相当于一个任务调度中心，实际执行任务的是GCTaskThread，即GC线程。当投递了一个WaitForBarrierGCTask任务后，当前垃圾回收线程一直阻塞，直到GC任务管理器发现没有工作线程在执行GCTask。

​	每个GCTask的工作量各不相同，如果一个GC线程快速完成了任务，另一个GC线程仍然在执行需要消耗大量算力的任务，此时虽然其他线程空闲，但垃圾回收STW时间并不会减少，因为在执行下一步操作前必须保证所有GCTask都已经执行完成。这是任务调度的一个常见问题。

​	为了负载均衡，GC线程可以将GCTask分割为更细粒度的GCTask然后放入队列，比如一个指定GC Root类型扫描任务可以使用BFS（Breadth First Searching，广度优先搜索）算法，将GC Root可达的对象放入BFS队列，搜索BFS队列中对象及其成员字段以构成一个更细粒度的GCTask，这些细粒度任务可被其他空闲GC线程窃取，这种方法也叫作工作窃取（Work Stealing）。

​	工作窃取是Parallel GC性能优化的关键，它实现了动态任务负载（Dynamic Load Balancing，DLB），可以确保其他线程IDLE时任务线程不会过度负载。

```c++
template<class T, MEMFLAGS F> bool
GenericTaskQueueSet<T, F>::steal_best_of_2(...) {
    // 如果任务队列多于2个
    if (_n > 2) {
        T* const local_queue = _queues[queue_num];
        // 随机选择两个队列
        uint k1 = ...;
        uint k2 = ...;
        uint sz1 = _queues[k1]->size();
        uint sz2 = _queues[k2]->size();
        uint sel_k = 0;
        bool suc = false;
        // 在随机选择的k1和k2队列中选择GCTask个数较多的那个，窃取一个GCTask
        if (sz2 > sz1) {
            sel_k = k2;
            suc = _queues[k2]->pop_global(t);
        } else if (sz1 > 0) {
            sel_k = k1;
 			suc = _queues[k1]->pop_global(t);
        }
        ...// 窃取成功
        return suc;
    } else if (_n == 2) { 
        // 如果任务队列只有两个，那么随机窃取一个任务队列的GCTask
        uint k = (queue_num + 1) % 2;
        return _queues[k]->pop_global(t);
    } else {
        return false;
    }
}
```

​	垃圾回收器将随机选择两个任务队列（如果有的话），再在其中选择一个更长的队列，并从中窃取一个任务。任务窃取不总是成功的，如果一个GC线程尝试窃取但是失败了2`*`N次，N等于(ncpus<=8)?ncpus:3+((ncpus`*`5)/8))，那么当前GC线程将会终止运行。

​	工作窃取使用GenericTaskQueue，这是一个ABP（Aurora-Blumofe-Plaxton）风格的双端队列，队列的操作无须阻塞，持有队列的线程会在队列的一端指向push或pop_local，其他线程可以对队列使用。

​	有了动态任务负载后，GC线程的终止机制也需要对应改变。具体来说，GC线程执行完GCTask后不会简单停止，而是查看能否从其他线程任务队列中窃取一个任务队列，如果所有线程的任务队列都没有任务，再进入终结模式。终结模式包含三个阶段，首先指定次数的自旋，接着GC线程调用操作系统的yield让出CPU时间，最后睡眠1ms。如果GC线程这三个小阶段期间发现有可窃取的任务，则立即退出终结模式，继续窃取任务并执行。

### 并行与并发

​	在垃圾回收领域中，并发（Concurrent）和并行（Parallel）有区别[1]于通用编程概念中的并发和并行：并发意味着垃圾回收过程中（绝大部分时间）Mutator线程可以和多个GC线程一起工作，几乎可以认为在垃圾回收进行时，Mutator也可以继续执行而无须暂停；并行是指垃圾回收过程中允许多个GC线程一同工作来完成某些任务，但是Mutator线程仍然需要暂停，即垃圾回收过程中应用程序需要一直暂停。

​	Parallel GC为减少STW时间付出了努力，它的解决方式是暂停Mutator线程，使用多线程进行垃圾回收，最后唤醒所有Mutator。

![](https://pic.imgdb.cn/item/60923b02d1a9ae528f3cf8be.jpg)

​	并发垃圾回收线程数目由`-XX:ConcGCThreads=<val>`控制，并行垃圾回收线程数目由`-XX:ParallelGCThreads=<val>`控制。但多线程并行化垃圾回收工作过程中Mutator线程仍然需要暂停。

​	并发垃圾回收器CMS GC可以解决这个问题（虽然它的解决方案并不完美）。

## CMS GC

​	CMS GC的全称是最大并发标记清除垃圾回收器（Mostly Mark and Sweep Garbage Collector），可以使用-XX:+UseConcMarkSweepGC开启。CMS GC的新生代清理仍然使用与Parallel GC类似的方式，即开启多个线程一起清理，且在这个过程中，Mutator线程不能工作。

​	从逻辑上来说，该过程与Parallel GC的Young GC几乎一致，所以这里不再赘述。不同点是CMS GC多了个专门针对老年代的Old GC。

![](https://pic.imgdb.cn/item/60923c89d1a9ae528f4e2bec.jpg)

​	Minor GC与Young GC等价，都表示只清理新生代，Old GC表示只清理老年代，Mixed GC表示清理整个新生代和部分老年代，它们都属于Partial GC。Full GC表示清理整个堆，通常它等价于Major GC。

​	CMS GC除了有负责清理新生代的YGC、特殊情况下的FGC外，还有只回收老年代的垃圾回收策略，即Old GC。Old GC大部分过程允许Mutator线程和GC线程一起进行，此时Mutator线程无须停止，这种方式称为并发垃圾回收，所使用的算法称为并发标记清除算法。

### 对象丢失问题

​	传统的标记清除算法分为标记、清除两个阶段。为了将它改造为并发算法，CMS GC将标记清除算法细分为初始标记、并发标记、预清理、可中断预清理、重新标记、并发清理，重置几个阶段，其中只有初始标记和重新标记需要STW，其他最耗时的阶段允许GC线程和Mutator线程一起进行。正是因为它有两个阶段需要STW，所以CMS GC的名字是最大程度（Mostly）的并发而非完全（Completely）并发。 Mutator线程和GC线程一起工作会造成一些问题

![](https://pic.imgdb.cn/item/60923f3bd1a9ae528f6c403c.jpg)

​	三色抽象（Tricolor Abstraction）可以简洁地描述回收过程中对象状态的变化，所以本节将使用三色抽象描述对象标记过程：图10-9中黑色表示对象及成员都被处理，浅色网格表示对象本身已处理，白色表示未处理对象。

​	起初垃圾回收器已经处理了A、B、C对象，并正在处理E对象成员。由于Mutator线程可以与GC线程一起工作，所以Mutator线程可以更新B对象的引用，使其指向D对象，并删除G对象对D对象的引用。由于B对象已经被标记为黑色对象，不会再做扫描，所以GC只会继续处理E对象，并清扫未被标记的D对象。更进一步，研究表明，只要同时满足以下两条要求就会造成存活对象丢失：

* Mutator线程插入了从黑色对象指向白色对象的新引用；
* Mutator线程删除了从灰色对象指向该白色对象的所有可能路径。



​	“垃圾回收器只能清理垃圾”是垃圾回收器最重要的原则，如果只是简单地引入并发算法，则会违背该原则，因此，并发垃圾回收器必须处理对象丢失问题。

​	常用的解决对象丢失的方法有增量更新（Incremental Update）和SATB（Snapshot At The Beginning，起始快照）技术。

​	增量更新的原理是打破第一个条件，通过写屏障记录下Mutator线程对黑色对象的增量修改，然后重新扫描这些黑色对象，以图10-9为例，当删除G到D的引用，并添加B到D的引用时，增量更新的写屏障会记录对象G并将它标记为灰色以等待二次处理。

​	SATB的原理是打破第二个条件，同样的例子，SATB写屏障会将D放入标记栈等待后续处理。

​	CMS GC使用增量更新技术，具体实现方式是复用其他分代GC处理跨代引用的卡表和写屏障代码，只要黑色对象写入白色对象的引用，就记录在卡表中以等待后续重新标记阶段再次扫描。这样做的问题是由于卡表本来用于处理跨代引用，每次YGC后都会重置，导致CMS GC需要的数据可能被重置掉，因此CMS GC引入了mod-union表，当CMS GC的Old GC进行并发标记时，每发生一次YGC，就会在重置卡表前更新mod-union表的对应数据。

### Old GC周期

​	CMS GC在Old GC中实现了并发标记清除算法，在创建CMSCollector时，虚拟机会同时创建ConcurrentMarkSweepThread（以下简称CMS GC线程），用于负责Old GC的实际工作。

```c++
void ConcurrentMarkSweepThread::run_service() {
    ...
    while (!should_terminate()) {
        sleepBeforeNextCycle();        // 阻塞一段时间，直到下一次Old GC发生
        if (should_terminate()) break; // 如果请求退出则终止CMS GC线程
        GCIdMark gc_id_mark;
        GCCause::Cause cause = _collector->_full_gc_requested ?
            _collector->_full_gc_cause : GCCause::_cms_concurrent_mark;
        _collector->collect_in_background(cause); // 清理老年代
    }
}
```

​	CMS GC线程会进入一个循环，每次它调用sleepBeforeNextCycle()时会阻塞一段时间，唤醒后使用CMSCollector::collect_in_background()清理老年代。

```c++
void CMSCollector::collect_in_background(GCCause::Cause cause) {
    ...
    switch (_collectorState) {
        // 初始标记（STW）
        case InitialMarking:{
            ReleaseForegroundGC x(this);
            stats().record_cms_begin();
            VM_CMS_Initial_Mark initial_mark_op(this);
            VMThread::execute(&initial_mark_op);
        }
        break;
        // 并发标记
        case Marking: markFromRoots();break;
        // 预清理
        case Precleaning: preclean();break;
        // 可中断预清理
        case AbortablePreclean: abortable_preclean();break;
        // 重新标记（STW）
        case FinalMarking:{
            ReleaseForegroundGC x(this);
            VM_CMS_Final_Remark final_remark_op(this);
            VMThread::execute(&final_remark_op);
		}
        break;
        // 并发清理
        case Sweeping: sweep(); // fallthrough
        case Resizing: {
            ReleaseForegroundGC x(this); 
            MutexLockerEx       y(...);
            CMSTokenSync        z(true);
            if (_collectorState == Resizing) {
                compute_new_size();
                save_heap_summary();
                _collectorState = Resetting;
            }
            break;
        }
        // 重置垃圾回收器的各种数据结构
        case Resetting: ... break;
        case Idling: 
        default: ShouldNotReachHere();break;
    }
    ...
}
```

​	collect_in_background实现了一个完整的Old GC，代码使用状态机模式，通过_collectorState状态转换来切换到不同的垃圾回收周期，简化了代码逻辑。

#### 初始标记

​		初始标记（InitiaMarking）是Old GC的第一个周期，它需要Mutator线程暂停，这一步通过安全点来保障，而虚拟机中能开启安全点的操作只能是VMThread，所以InitialMarking阶段会创建一个VM_CMS_Initial_Mark的VMOperation，当VMThread执行该VMOperation并协调所有线程进入安全点后，会调用checkpointRootsInitialWork()进行初始标记。

```c++
void CMSCollector::checkpointRootsInitialWork() {
    // 确保位于安全点，并且处于InitialMarking阶段
    assert(SafepointSynchronize::is_at_safepoint(), ...);
    assert(_collectorState == InitialMarking, "just checking");
    ...
    // 新生代指向老年代的引用
    MarkRefsIntoClosure notOlder(_span, &_markBitMap);
    ...
    if (CMSParallelInitialMarkEnabled) {
	... // 使用多线程进行初始标记
        CMSParInitialMarkTask tsk(this, &srs, n_workers);
        if (workers->total_workers() > 1) {
            workers->run_task(&tsk);
        } else {
            tsk.work(0);
        }
    } else {
        ... // 使用单线程进行初始标记
        heap->cms_process_roots(...,&notOlder, &cld_closure);
    }
    ...
}
```

​	并发标记清除把整个标记清除细分为几个阶段，然后以STW的方式执行其中两个阶段，其他阶段允许Mutator线程和GC一起工作，在STW的两个阶段，垃圾回收器还可以充分发挥多核处理器的优势，使用多个线程进行回收工作，减少STW时间。

​	为了进一步减少STW时间，初始标记只会扫描并标记GC Root指向老年代的直接引用以及新生代指向老年代的直接引用，而所有间接引用都由后面的并发标记处理。

#### 并发标记

​	初始标记是从GC Root和新生代指向老年代记忆集出发，寻找直接可达的对象，接下来并发标记（Marking）是从这些对象出发，寻找间接可达的对象。

​	这一步由markFromRoots()完成，该函数内部会创建CMSConcMarkingTask并发标记。CMSConcMarkingTask包括标记逻辑和工作窃取逻辑，前者由do_scan_and_mark完成，后者由do_work_stealing完成。

​	标记的逻辑是每当发现初始标记的存活对象cur，就将它放入_markStack，然后进入循环。每次从_markStack中弹出一个对象，扫描cur的成员引用，直到`_markStack`为空，这是一个典型的广度优先搜索过程，只是CMS GC在扫描cur成员引用时稍有改变，它不会将扫描到的cur的成员全部放入`_markStack`，而是选择性地放入

![](https://pic.imgdb.cn/item/6092468bd1a9ae528fc25502.jpg)

​	扫描策略是找到存活对象cur，如果它的成员对象地址位于cur前面，则标记并继续扫描成员对象，如果它的成员对象地址位于cur后面，则只标记不扫描成员对象。这样做实际上结合了广度优先搜索和深度优先搜索，好处是减小了`_markStack`的大小，在该例中`_markStack`最大仅包含一个元素，若直接使用广度优先搜索会导致`_markStack`快速膨胀，虚拟机内存空间不足的情况。

#### 预清理

​	并发预清理和并发可中断预清理（Precleaning && AbortablePreclean）是可选步骤，如果关闭`-XX:-CMSPrecleaningEnabled`，虚拟机会跳过它直接执行下一阶段的重新标记。

​	如果上一阶段并发标记过程中Mutator线程修改了对象引用关系，比如创建了新生代指向老年代的引用，那么预清理可以发现这些修改，并标记老年代的对象图。可中断预清理与之类似，它会尝试若干次预清理过程，直到次数到达GC允许的上限，或者超过指定时间。两个阶段的意义在于做尽可能多的标记工作，减少下一阶段重新标记的STW时间。

#### 重新标记

​	重新标记（FinalMarking）过程会再次停止全部Mutator线程（STW），只允许垃圾回收线程。
因为初始标记到重新标记的间隔允许Mutaor线程和GC线程一起进行，所以可能产生大量从新生代指向老年代的引用，即新生代记忆集大增，也可能之前新生代已经存活的很多对象变成了死亡对象，但是GC不知道这个事实，仍然从GC Root和新生代记忆集出发标记存活对象，使本该死亡的对象被标记为存活对象，产生浮动垃圾。这是分代垃圾回收器面临的常见问题，如果开启`-XX:+CMSScavengeBeforeRemark`，在重新标记前GC会先对新生代进行垃圾回收，这样可以有效减少新生代记忆集大小，继而减少重新标记造成的STW时间。注意，以上讨论仅在两次STW标记期间新生代记忆集大增，或者大量新生代记忆集的对象从存活转变为死亡时才成立，如果随意开启该选项可能适得其反。

​	除了重新标记新增可选的新生代回收步骤外，重新标记过程与初始标记过程大致一样，两者都是向VMThread投递VMOperation，区别在于前者的VMOperation调用checkpointRootsInitialWork，后者调用checkpointRootsFinalWork。

```c++
void CMSCollector::checkpointRootsFinalWork() {
    ...
    // 根据虚拟机参数使用多线程重新标记或者使用单线程重新标记
    if (CMSParallelRemarkEnabled) {
        do_remark_parallel();
    } else {
        do_remark_non_parallel();
    }

    ...// 处理虚引用、弱引用等特殊引用
    refProcessingWork();
    _collectorState = Sweeping; // 修改状态，下一步是并发清理
}
```

#### 并发清理

​	并发清理（Sweeping）是指通过寻找卡表中标记为未被标记的页，找到对应的老年代空间，然后使用SweepClosure清理这些空间的无用对象。

### 并发模式失败

​	在CMS GC已经处于Old GC过程中时，如果垃圾回收器再被请求FGC，可能意味着Old GC的回收速度跟不上分配速度，此时CMS GC将会报告并发模式失败（如果此次FGC是用户请求的，如System.gc()调用或Heap dump等，那么会报告并发模式中断，只有GC自主发起的才被称为并发模式失败），并启用备用方案，使用单线程[1]标记整理算法的FGC。单线程的FGC会造成应用程序长时间停顿，严重影响程序响应时间。

### 堆碎片化

​	CMS GC的Old GC使用（并发的）标记清除算法而不是像Serial GC、Parallel GC一样使用标记整理算法，原因在于如果使用标记整理算法，GC只能在标记阶段（大部分时间）并发：整理阶段由于需要移动对象，整个阶段需要STW，这对致力于减少STW时间的CMS GC来说是不可接受的。

## G1 GC

​	可以使用-XX:+UseG1GC开启。

![](https://pic.imgdb.cn/item/60924f52d1a9ae528f22dbe7.jpg)

​	G1没有抛弃弱分代假说，如图10-11所示，每个Region仍然包含代纪，YGC和Mixed GC（混合回收）会选择合适的Region，然后只回收这一部分Region。

### 混合回收

​	Mixed GC是G1独有的回收策略，分为全局并发标记和对象复制两个部分：全局并发标记使用G1ConcurrentMarkThread在后台不定期运行，试图标记存活对象并找出收益较高的Region，接下来由YGC选择这些收益较高的Region并对它们使用复制算法，将其中的存活对象复制到Survivor Region，然后清空原本的Region。

​	复制算法可以有效地解决类似CMS GC老年代的碎片化问题，同时由于全局并发标记选择一部分Region，这使得用户可以指定一个GC最大暂停时间作为目标，由G1根据历史数据和选择的Region回收垃圾，努力达到用户设置的目标，也即让用户在一定程度上控制STW时间。

![](https://pic.imgdb.cn/item/60925423d1a9ae528f5eef92.jpg)

​	除了Young GC和Mixed GC，G1也有Full GC。回收速度跟不上老年代回收速度，或者无法容纳晋升对象等都可能导致Full GC。G1的Full GC与其他垃圾回收器的Full GC一样都使用标记整理算法，整个Full GC是一个完全STW的过程。

​	在混合回收中，复制阶段是全局STW的，它是一个相当耗时的过程，如果G1跟不上用户设置的目标，反而容易引发Full GC。对于这些问题，新一代低停顿并发垃圾回收器Shenandoah GC和ZGC交出了新的答卷。

## Shenandoah GC

​	在Shenandoah GC之前的所有垃圾回收器都必须主动或者被动地整理老年代或者新生代，因此会导致长时间的STW，对于大型的堆，比如超过100GB，所有现存的垃圾回收器几乎都表现得很差。为了解决这些问题，Red Hat开发了一个低停顿的并发垃圾回收器，并于JEP 189贡献给了OpenJDK社区，目前Shenandoah GC仍然属于实验性特性，需要使用参数`-XX:+UnlockExperimentalVMOptions` `-XX:+UseShenandoahGC`开启。

​	Shenandoah GC的STW时间不会随堆的增大而线性增长，所以回收200GB的堆和2GB的堆的STW时间相差无几。Shenandoah GC的大部分阶段都是并发的

![![](https://pic.imgdb.cn/item/60925a94d1a9ae528fa46794.jpg)](https://pic.imgdb.cn/item/60925560d1a9ae528f6fe7ca.jpg)

​	Shenandoah GC类似于G1，也是基于Region的堆设计，但是它没有采用弱分代假设。一个特别的地方是它在移动对象时允许Mutator线程运行，即并发整理阶段（Concurrent Compact），这是它减少停顿时间的关键，也是低延时的秘诀所在。因为没有分代设计，在并发整理阶段，Shenandoah GC会清理所有可能Region。并发整理的关键技术是Brooks指针。

​	G1仅在STW期间才能移动对象，而Shenandoah GC可以在Mutator线程运行期间并发地重定位对象位置，它通过名为Brooks指针的技术来实现这个特性。在堆中的每个对象都有一个额外的Brooks指针字段（fwdptr），它指向这个对象，当对象移动后，它指向移动后的对象，同时将对对象的修改转发到移动后的对象上。

![](https://pic.imgdb.cn/item/60925a54d1a9ae528fa24329.jpg)

![](https://pic.imgdb.cn/item/60925a94d1a9ae528fa46794.jpg)

​	通过转发指针，对象移动期间无须全局STW，只需要CAS交换指针即可。除了对象修改外，对象访问也需要借助fwdptr转发，因为如果不转发就可能读取到未更新的x、y、z值。转发对象读取和对象访问请求需要通过读屏障和写屏障来完成。

```c++
oop ShenandoahBarrierSet::read_barrier(oop src) {
    if (ShenandoahReadBarrier && _heap->has_forwarded_objects()) {
        // 最终对(*brooks_ptr_addr(src))解引用
        return ShenandoahBarrierSet::resolve_forwarded(src); 
    }
    return src;
}
HeapWord** ShenandoahBrooksPointer::brooks_ptr_addr(oop obj) {
    // 读取obj的fwdptr，然后解引用，得到地址
    return (HeapWord**)((HeapWord*) obj + word_offset());
}
```

​	Shenandoah GC要求每个对象附加一个额外的转发指针字段，这会浪费一些内存，同时读屏障也会造成比较严重的开销，因为每次对象读取都会额外执行其他指令。

## ZGC

​	ZGC是由Oracle开发的一个低停顿的并发垃圾回收器，并于JEP 333贡献给OpenJDK社区。ZGC的目标与Shenandoah GC的目标非常相似：控制STW时间，目标10ms以内；STW时间不会随着堆的增大而变长。

​	ZGC使用基于Region的堆设计，同样在移动对象过程中允许GC线程和Mutator线程一同运行。Shenandoah GC给出的解决方案是Brooks指针，而ZGC使用染色指针。

​	x64的硬件限制使得处理器只能使用48条地址线访问256TB的内存，ZGC为对象地址保留42位，这导致目前ZGC最大只支持4TB的内存，因为着色指针的设计，ZGC不支持32位指针也不支持压缩指针。剩下的位用于存放finalizable，remapped，marked1和marked0几个标志 。

![](https://pic.imgdb.cn/item/60925fa5d1a9ae528fcdd0a0.jpg)

​	这些标志可以指明对象是否已经被移动、是否被标记、是否只能通过finalizer可达。除了并发移动对象外，ZGC还支持基于NUMA的CPU架构，并且能归还未使用的内存给操作系统等。目前ZGC也处于实验阶段，需要`-XX:+UnlockExperimentalVMOptions` ` -XX:UseZGC`开启。

​	各式各样垃圾回收器的出现说明一个事实：GC没有“银弹”，换句话说，所有GC都不能兼具低停顿时间和低运行时开销的特性。

​	Epsilon GC可能算一个，但是它不回收垃圾，不能用于常见应用环境。也许在遥远的未来会出现类似-XX:+SelectOptimumGC的参数，可以根据用户描述的应用程序特性和环境来自动选择最合适的垃圾回收器，但是目前，开发者仍然需要根据自己的应用程序特性和运行环境手动选择最合适的GC，并适度调整GC参数，使GC与应用程序相契合。没有最好的垃圾回收器，只有最合适的选择。

![](https://pic.imgdb.cn/item/6092639ad1a9ae528ff03cd4.jpg)

# G1 GC

## 简介

### 基于Region的堆

​	G1 GC全称是Garbage-First Garbage Collector，即垃圾优先的垃圾回收器，可以使用-XX:+UseG1GC开启。G1 GC（以下简称G1）抛弃了既有堆模型，它将整个堆划分为一些大小固定的内存块（Region），通过`-XX:G1HeapRegionSize=<val>`控制Region大小（注意每个Region的大小只能是1MB、2MB、4MB、8MB、16MB和32MB

​	每个Region仍然包含代纪类型，一个特别的类型是巨型Region（Humongous Region），如果用户分配的对象超过了单个Region的大小，那么将使用连续多个Region存放对象，并将这些Region都标记为巨型Region。

​	G1还有一个Archive类型的Region，它包含的是不可变的数据，该类型用于支持AppCDS。

### 记忆集Rset

​	G1包含YGC、FGC和Mixed GC三种垃圾回收策略，其中，YGC和FGC与其他垃圾回收器类似：YGC只回收新生代Region，而FGC回收整个堆。独有的Mixed GC是一种Partial GC策略，它会回收所有新生代Region和部分老年代Region。

​	既然Mixed GC属于Partial GC，那么它也会面临跨代引用问题，因为它回收整个新生代和部分老年代Region，所以一个老年代Region的根集包括GC Root和从老年代Region指向老年代Region的引用（old->old），新生代Region根集包括GC Root和老年代Region指向新生代Region的引用（old->young）。

​	G1使用RSet记忆集记录这些跨代引用。在记忆集设计中一般包含两种方式：一种是points-into记忆集，它表示“哪些对象引用了我”；另一种是points-out记忆集，它记录的是“我引用了哪些对象”。G1同时使用两种方式。

![](https://pic.imgdb.cn/item/609268ffd1a9ae528f191e18.jpg)

​	假设有a.field = b，如果使用points-into记忆集，那么b拥有记忆集，它记录a的位置。如果使用points-out记忆集，那么a拥有记忆集，它记录b的位置。G1的记忆集RSet同时使用两种设计，首先使用points-into结构来记忆有哪些其他Region引用自身（即对象b所在Region记录引用自身的对象a所在Region），然后每个Region包含一个points-out的卡表结构，记录指向当前对象的对象的具体位置（即对象b所在Region的卡表的索引）。

​	在G1堆中，每个Region会关联一个RSet，后置写屏障（g1_write_barrier_post）捕获Mutator线程向对象写入的每个值。如果发现写入操作导致两个对象产生old->old或者old->young关系，那么可以更新RSet，并将对象写入线程局部的DirtyCardQueue（DCQ），当线程局部的DCQ已满后，再将DCQ放入全局的DirtyCardQueueSet（DCQS）。

​	出于性能考虑，写屏障内的代码应该尽可能简单和高效，g1_write_barrier_post只负责发现那些产生old->old或者old->young关系的修改，并将对象加入DCQ。后续处理DCQ中的对象及更新RSet的操作则由专门的Refine线程负责。Refine线程取出DCQS中的DCQ的对象，找到被该对象引用的对象，然后更新被引用对象所在的Region的RSet。

```c++
void G1ConcurrentRefineOopClosure::do_oop_work(T* p) {
    T o = RawAccess<MO_VOLATILE>::oop_load(p);
	if (CompressedOops::is_null(o)){ return; }
    oop obj = CompressedOops::decode_not_null(o);
    if (HeapRegion::is_in_same_region(p, obj)) {
        return; // 如果对象和被引用对象在同一个Region中，则不需要处理
    }
    // 如果在不同Region中，则需找到被引用者所在Region的RSet
    HeapRegionRemSet* to_rem_set = _g1h->heap_region_containing(obj)->rem_set();
    // 在被引用者的RSet中添加关系
    if (to_rem_set->is_tracked()) {
        to_rem_set->add_reference(p, _worker_i);
    }
}
```

### 停顿预测模型

​	`-XX:MaxGCPauseMillis`

## Young GC

### 选择CSet

​	YGC的回收过程位于G1CollectedHeap::do_collection_pause_at_safepoint()，在进行垃圾回收前它会创建一个清理集CSet（Collection Set），存放需要被清理的Region。选择合适的Region放入CSet是为了让G1达到用户期望的合理的停顿时间。

```c++
void G1Policy::finalize_collection_set(...) {
    // 先选择新生代Region，用户期望的最大停顿时间是target_pause_time_ms
    // G1计算出清理新生代Region的可能用时后，会将剩下的时间（time_remaining_ms）给老年代
        double time_remaining_ms = 
            _collection_set->finalize_young_part(...);
    _collection_set->finalize_old_part(time_remaining_ms);
}
```

​	G1的YGC只负责清理新生代Region，因此finalize_old_part()不会选择任何Region，所以只需要关注finalize_young_part()。finalize_young_part会在将所有Eden和Survivor Region加入CSet后准备垃圾回收。

​	G1在evacuate_collect_set()中创建G1ParTask，然后阻塞，直到G1ParTask执行完成，这意味着整个YGC期间应用程序是STW的。类似Parallel GC的YGC，G1ParTask的执行由线程组GangWorker完成，以尽量减少STW时间。不难看出，YGC的实际工作位于G1ParTask，它主要分为三个阶段：

1）清理根集（G1RootProcessor::evacuate_roots）；

2）处理RSet（G1RemSet::oops_into_collection_set_do）；

3）对象复制（G1ParEvacuateFollowersClosure::do_void）。

### 清理根集

​	第一阶段是清理根集。HotSpot VM很多地方都属于GC Root，G1ParTask的evacuate_roots()会从这些GC Root出发寻找存活对象。以线程栈为例，G1会扫描虚拟机所有JavaThread和VMThread的线程栈中的每一个栈帧，找到其中的对象引用，并对它们应用G1ParCopyClosure。

```c++
void G1ParCopyClosure<barrier, do_mark_object>::do_oop_work(T* p) {
    ...
    oop obj = CompressedOops::decode_not_null(heap_oop);
    const InCSetState state = _g1h->in_cset_state(obj);
    // 如果对象属于CSet
    if (state.is_in_cset()) {
        oop forwardee;
        markOop m = obj->mark_raw();
        if (m->is_marked()) {   // 如果已经复制过则直接返回复制后的新地址
            forwardee = (oop) m->decode_pointer();
        } else {                  // 将它复制到Survivor Region，返回新地址
            forwardee = _par_scan_state->copy_to_survivor_space(...);
        }
        // 修改根集中指向该对象的引用，指向Survivor中复制后的对象
        RawAccess<IS_NOT_NULL>::oop_store(p, forwardee);
        ...
    } else {
    ...
    }
}
```

​	清理根集的核心代码是copy_to_survivor_space，它将Eden Region中年龄小于15的对象移动到Survivor Region，年龄大于等于15的对象移动到Old Region。之前根集中的引用指向Eden Region对象，对这些引用应用G1ParCopyClosure之后，Eden Region的对象会被复制到Survivor Region，所以根集的引用也需要相应改变指向。

![](https://pic.imgdb.cn/item/6092810fd1a9ae528fb69375.jpg)

​	copy_to_survivor_space在移动对象后还会用G1ScanEvacuatedObjClosure处理对象的成员，如果成员也属于CSet，则将它们放入一个G1ParScanThreadState队列，等待第三阶段将它们复制到Survivor Region。总结来说，第一阶段会将根集直接可达的对象复制到Survivor Region，并将这些对象的成员放入队列，然后更新根集指向。

### 处理RSet

​	第一阶段标记了从GC Root到Eden Region的对象，对于从Old Region到Eden Region的对象，则需要借助RSet，这一步由G1ParTask的G1RemSet::oops_into_collection_set_do完成，它包括更新RSet（update_rem_set）和扫描RSet（scan_rem_set）两个过程。scan_rem_set遍历CSet中的所有Region，找到引用者并将其作为起点开始标记存活对象。

### 对象复制

​	经过前面的步骤后，YGC发现的所有存活对象都会位于G1ParScanThreadState队列。对象复制负责将队列中的所有存活对象复制到Survivor Region或者晋升到Old Region

```c++
template <class T> void G1ParScanThreadState::do_oop_evac(T* p) {
    // 只复制位于CSet的存活对象
    oop obj = RawAccess<IS_NOT_NULL>::oop_load(p);
    const InCSetState in_cset_state = _g1h->in_cset_state(obj);
    if (!in_cset_state.is_in_cset()) {
        return;
    }
    // 将对象复制到Survivor Region（或晋升到Old Region）
    markOop m = obj->mark_raw();
    if (m->is_marked()) {
        obj = (oop) m->decode_pointer();
    } else {
        obj = copy_to_survivor_space(in_cset_state, obj, m);
    }
    RawAccess<IS_NOT_NULL>::oop_store(p, obj);
    // 如果复制后的Region和复制前的Region相同，直接返回
    if (HeapRegion::is_in_same_region(p, obj)) {
        return;
    }
    // 如果复制前Region是老年代，现在复制到Survivor/Old Region，
    // 则会产生跨代引用，需要更新RSet
    HeapRegion* from = _g1h->heap_region_containing(p);
    if (!from->is_young()) {
        enqueue_card_if_tracked(p, obj);
    }
}
```

​	对象复制是YGC的最后一步，在这之后新生代所有存活对象都被移动到Survivor Region或者晋升到Old Region，之前的Eden空间可以被回收（Reclaim）。另外，YGC复制算法相当于做了一次堆碎片的清理工作，如整理Eden Region可能存在的碎片。

## Mixed GC

​	Mixed GC（混合回收）是G1独有的回收策略，它与YGC的回收策略的区别如下：

* YGC：选定所有Eden Region放入CSet，使用多线程复制算法将CSet的存活对象复制到Survivor Region或者晋升到Old Region。
*  Mixed GC：选定所有Eden Region和全局并发标记计算得到的收益较高的部分Old Region放入CSet，使用多线程复制算法将CSet的存活对象复制到Survivor Region或者晋升到Old Region。



​	Mixed GC增加了全局并发标记过程，它能够回收部分Old Region，不会再出现在CMS GC中老年代出现的碎片化问题，因为当老年代被加入CSet后，G1会使用和YGC一样的复制算法整理空间。

![](https://pic.imgdb.cn/item/60928567d1a9ae528fd11538.jpg)

### SATB

​	在并发标记过程中，只要满足两个条件就会造成对象丢失，解决方案通常有增量更新技术和SATB。CMS GC使用增量更新技术破坏第一个条件来解决对象丢失问题，而G1使用SATB破坏第二个条件来解决对象丢失问题。

​	SATB会在GC开始时为对象关系打下一个快照，除了快照中存活的对象和GC过程中分配的新对象外，其他都是死亡对象。SATB的快照和GC期间新分配的对象由TAMS实现。TAMS（Top At Mark Start）是一个指针，每个Region包括Bottom、End、Top、PrevTAMS和NextTAMS指针。

![](https://pic.imgdb.cn/item/609285a7d1a9ae528fd2d10b.jpg)

​	当第N-1次并发标记开始时将Region的Top指针赋值给NextTAMS，标记期间如果Mutator线程分配新对象则移动Top指针，最终标记期间所有新分配的对象都位于NextTAMS与Top之间，SATB会将这些对象都标记为存活对象。并发标记完成后会将PrevTAMS指针移动到NextTAMS，重置NextTAMS与Bottom。第N次并发标记开始时将Bottom到PrevTAMS之间的存活对象当作快照，此时快照中的存活对象在第N次并发标记结束后也应该保持存活。
TAMS为并发标记创建了快照，便于后续进行新对象分配，但是它不能解决并发标记期间Mutator线程造成的对象丢失问题（见10.5.2节），这样会破坏快照的完整性，为此G1使用SATB写屏障捕获所有对象修改，

​	TAMS为并发标记创建了快照，便于后续进行新对象分配，但是它不能解决并发标记期间Mutator线程造成的对象丢失问题（见10.5.2节），这样会破坏快照的完整性，为此G1使用SATB写屏障捕获所有对象修改

```c++
JRT_LEAF(void,G1BarrierSetRuntime::write_ref_field_pre_entry(..))
    G1ThreadLocalData::satb_mark_queue(thread).enqueue(orig);
JRT_END

class STABMarkQueue: public PrtQueue {
    void enqueue(void* ptr) {
        // 如果不是并发标记阶段，SATB写屏障不会将对象记录到SATBMarkQueue
        if (!_active) return; // _active只在并发标记阶段才为true
 		else enqueue_known_active(ptr);
    }
    ...
};
```

### 全局并发标记

#### 初始标记

​	初始标记是全局并发标记的第一阶段，是一个STW的过程，这一步会扫描GC Root直接可达的对象，并把它们复制到Survivor Region，该过程与YGC高度一致，所以G1完全复用了YGC的代码。也就是说，do_collection_pause_at_safepoint()可以同时完成YGC和Mixed GC全局并发标记第一阶段这两件事情。
​	do_collection_pause_at_safepoint()如果发现这次回收策略是Mixed GC，则会在G1ParTask的执行前后分别调用G1ConcurrentMark::pre_initial_mark和G1ConcurrentMark::post_initial_mark。前者将NextTAMS设置为top指针，为并发标记快照做好准备；后者通知根Region扫描准备就绪。do_collection_pause_at_safepoint()会在最后将全局并发标记设置为started。

#### 根Region扫描

​	全局并发标记的大部分过程由G1ConcurrentMarkThread在后台完成

```c++
void G1ConcurrentMarkThread::run_service() {
..
    while (!should_terminate()) {
        // 睡眠，直到条件满足，醒来后进行全局并发标记
        sleep_before_next_cycle();
        if (should_terminate()) {
            break;
        }
        // 根Region扫描
        _cm->scan_root_regions();
        for (uint iter = 1; !_cm->has_aborted(); ++iter) {
            // 并发标记
            _cm->mark_from_roots();
            if (_cm->has_aborted()) break;
            ...
            if (_cm->has_aborted()) break;
            // STW重新标记
            CMRemark cl(_cm);
            VM_G1Concurrent op(&cl, "Pause Remark");
            VMThread::execute(&op);
            if (_cm->has_aborted()) break;
            ...
        }
        ...
        // STW清理
        if (!_cm->has_aborted()) {
            CMCleanup cl_cl(_cm);
            VM_G1Concurrent op(&cl_cl, "Pause Cleanup");
            VMThread::execute(&op);
        }
        ...
    }
}
```

​	sleep_before_next_cycle()只会在条件满足时醒来，而条件就是started标志，该标记会在第一阶段初始标记结束时设置，所以初始标记后紧接着G1ConcurrentMark线程就会醒来进行全局并发标记。

​	作为全局并发标记的第二阶段，根Region扫描（scan_root_regions()）是一个并发过程，它将G1CMRootRegionScanTask投递给线程执行。第一阶段初始标记完成后，Eden Region存活对象已经被晋升或者复制到Survivor Region。根Region扫描将会以Survivor Region中的存活对象作为根，扫描被它们引用的对象，并在NextBitmap标记。

#### 并发标记

​	全局并发标记的第三阶段是并发标记，并发标记期间可以发生很多次YGC。G1投递G1CMConcurrentMarkingTask任务等待线程组执行，该任务又会进一步调用G1CMTask::do_marking_step，这是并发标记的实际实现。
​	在并发标记期间，Mutator线程可以修改对象引用，这些被修改的对象会被放入SATBMarkQueue，do_marking_step找到位于SATBMarkQueue的对象后将它们标记为灰色，这样可以让这些对象被进一步扫描，从而解决引用修改导致的对象丢失问题。
​	另外，由于根Region扫描阶段只是将Survivor Region的存活对象当作根处理，这些对象的成员仍然是白色对象，所以do_marking_step也会处理它们。

#### 重新标记

​	重新标记与并发标记非常相似，事实上它们共用代码：重新标记投递G1CMRemarkTask给线程组执行。G1CMRemarkTask也是调用G1CMTask::do_marking_step完成的。

​	do_marking_step有一个参数time_target_ms，表示目标清理时间，如果超过这个时间，do_marking_step会终止执行。初始标记传递给do_maring_step的目标清理时间是10ms（由参数-XX:G1ConcMarkStepDurationMillis指定），表示并发标记在10ms内完成，超过时间会中断，而重新标记传递的目标清理时间约11天（1000000000.0ms），表示无论如何，重新标记都会完成。

​	GC如果想要结束标记阶段，需要满足两个条件：第一个是处理完Survivor Region的所有存活对象和它们的成员，第二个是处理完位于SATBMarkQueue中的所有引用更改产生的灰色对象及其成员。由于标记前打过快照，Survivor Region的存活对象不会增长，所以第一个条件很容易满足，但是如果没有一个STW步骤，第二个条件将永远无法满足，因为在Mutator线程和GC线程并发期间，Mutator线程可能会不断修改对象引用，所以需要重新标记：重新标记是一个STW过程，在这个过程中Mutator线程停止，GC线程处理SATBMarkQueue中剩余的对象。

#### 清理

​	全局并发标记的最后一步是清理，这也是一个STW过程。G1投递VM_G1Concurrent给线程组，这是一个VM_Operation，最终会被VMThread消费并促使所有线程到达安全点，然后G1投递CMCleanup任务给线程组，该任务最终调用G1ConcurrentMark::cleanup进行清理工作。cleanup会重置一些数据结构，如果发现Humongous Region有很大的RSet则会将其清理掉。
​	至此，并发标记的工作就完成了，cleanup调用record_concurrent_mark_cleanup_end告知下一次YGC需要清理新生代和部分老年代，还会设置CollectionSetChooser，选择老年代中存活对象较少的Region，为后期YGC的CSet选择做好准备。

### 对象复制

​	对象复制过程又叫作Evacuation，是一个STW过程。这一步会复用YGC的代码，只是正常YGC的CSet只选择Young Region，而Mixed GC复用YGC代码，在创建CSet时会选择所有Young Region和部分收益较高的Old Region，将CSet中的存活对象复制到Survivor Region，然后回收原来的Region空间。

## Full GC

​	在设计G1时会极力避免Full GC（以下简称FGC），但是总有一些特殊情况，如果当前并发回收的速度跟不上对象分配的速度，那么需要G1启动后备方案进行FGC。早期G1的FGC使用单线程的标记整理算法，后来为了充分发挥多核处理器的优势，JEP 307提案为G1的FGC设计了多线程标记整理算法，此时多线程的FGC的线程数量可以由`-XX:ParallelGCThreads`控制。

​	G1的多线程FGC与Parallel GC的FGC类似，是一个全局STW的过程，G1使用线程组完成垃圾回收工作，整个阶段都不允许Mutator线程运行。FGC的实现位于G1FullCollector::collect()

```c++
void G1FullCollector::collect() {
    phase1_mark_live_objects();
    phase2_prepare_compaction();
    phase3_adjust_pointers();
    phase4_do_compaction();
}
```

正如之前所说，FGC是一个标准的标记整理算法，每个步骤提交任务给线程池，使用多线程完成，尽量减少STW时间。触发FGC的场景有很多，举例如下：

* Mixed GC中如果老年代回收的速度小于对象分配或晋升的速度，会触发FGC；
* YGC最后会移动存活对象到其他分区，如果此时发现没有能容纳存活对象的Region，会触发FGC；
* 如果没有足够的Region容纳下Humongous对象，会触发FGC；
* 应用程序调用System.gc()也会触发FGC。

由于FGC的全局STW性，如果频繁发生FGC是比较糟糕的信号，它暗示应用程序的特性与当前的G1参数配置不能良好契合，需要开发者找到问题并进一步调优处理。

## 字符串去重

​	如果读者对虚拟机进行过Heap Dump（-XX:+HeapDumpOnOutOfMemoryError或者jmap触发）操作，会观察到Java堆中占比最大的通常是一些byte[]对象，这些byte[]对象又通常是String的成员，即字符串对象在Java堆中占据极大比重，如果能发现重复的字符串并消除它们，会节省很大一部分内存。可以手动调用String.intern()消除重复的字符串，但这需要开发者了解哪些字符串可能发生重复，也可以使用G1的新特性自动完成字符串去重。

```c++
bool G1StringDedup::is_candidate_from_evacuation(...) {
    // 如果对象在Eden Region，并且类型是java.lang.String
    if (from_young && java_lang_String::is_instance_inlined(obj)) {
        // 如果对象将要复制到Survivor Region，并且年龄小于阈值 
		 if (to_young && obj->age() == StringDeduplicationAgeThreshold) {
            return true; // 作为候选项加入G1StringDedupQueue
        }
        // 如果对象将要晋升到Old Region，并且年龄小于阈值
        if (!to_young && obj->age() < StringDeduplicationAgeThreshold) {
            return true; // 作为候选项加入G1StringDedupQueue
        }
    }
    return false;
}
void G1StringDedup::enqueue_from_evacuation(...) {
    if (is_candidate_from_evacuation(...)) {
        G1StringDedupQueue::push(worker_id, java_string);
    }
}
```

​	G1将所有存活对象从Eden复制到Survivor Region，所有从Eden晋升到Old Region并且年龄小于`-XX:StringDeduplicationAgeThreshold`的对象都会被放入G1StringDedupQueue等待字符串去重线程处理。字符串去重线程即StringDedupThread，它在发现队列中存在去重候选项后会弹出对象，然后调用StringDedupTable::deduplicate

```c++
void StringDedupTable::deduplicate(...) {
    // 如果java.lang.String的value字段为空，那么不处理
    typeArrayOop value = java_lang_String::value(java_string);
	if (value == NULL) {
        stat->inc_skipped();
        return;
    }
    ...
    // 根据新对象的hash查找已有对象
    typeArrayOop existing_value = lookup_or_add(value, latin1, hash);
    // 如果新对象和已有对象是同一个，那么不处理
    if (oopDesc::equals_raw(existing_value, value)) {
        stat->inc_known();
        return;
    }
    ... // 如果是不同对象，但是包含的字符串相同，则处理它
    if (existing_value != NULL) {
        java_lang_String::set_value(java_string, existing_value);
        stat->deduped(value, size_in_bytes);
    }
}
```

# JVM精要笔记

## 参数

* 标准参数
  * -help
  * -version
* -X参数(非标准参数)
  * -Xint
  * -Xcomp
* -XX参数(使用率较高)
  * -XX:newSize
  * -XX:+UseSerialGC

### **标准参数**

​	`java -help`可以检索出所有的标准参数。

​	主要

#### -server和-client参数

​	可以通过-server或-client设置JVM的运行参数。

* 它们的却别是Server VM的初始堆空间会大一点，默认使用的是并行垃圾处理器，启动慢，运行块。
* Client VM相对来讲会保守一些，初始堆会小一些，使用串行的垃圾回收器，它的目标是让JVM的启动速度更快，但运行速度会比Serverm模式慢些。
* JVM在启动的时候回根据硬件和操作系统自动选择使用Server还是Client类型的JVM。
* 32位操作系统
  * 如果是Windows系统，不论硬件配置，默认Client。
  * 其他操作系统，机器配置有2GB以上的内存并且有2个以上的CPU会默认使用server模式，否则使用client模式。
* 64位操作系统
  * 只有server类型，不支持client类型(设置无效)。

```
java -client -showversion TestJVM
java -server -showversion TestJVM
```

#### -X参数

```cmd
PS C:\Users\81929> java -X
    -Xmixed           混合模式执行 (默认)
    -Xint             仅解释模式执行
    -Xbootclasspath:<用 ; 分隔的目录和 zip/jar 文件>
                      设置搜索路径以引导类和资源
    -Xbootclasspath/a:<用 ; 分隔的目录和 zip/jar 文件>
                      附加在引导类路径末尾
    -Xbootclasspath/p:<用 ; 分隔的目录和 zip/jar 文件>
                      置于引导类路径之前
    -Xdiag            显示附加诊断消息
    -Xnoclassgc       禁用类垃圾收集
    -Xincgc           启用增量垃圾收集
    -Xloggc:<file>    将 GC 状态记录在文件中 (带时间戳)
    -Xbatch           禁用后台编译
    -Xms<size>        设置初始 Java 堆大小
    -Xmx<size>        设置最大 Java 堆大小
    -Xss<size>        设置 Java 线程堆栈大小
    -Xprof            输出 cpu 配置文件数据
    -Xfuture          启用最严格的检查, 预期将来的默认值
    -Xrs              减少 Java/VM 对操作系统信号的使用 (请参阅文档)
    -Xcheck:jni       对 JNI 函数执行其他检查
    -Xshare:off       不尝试使用共享类数据
    -Xshare:auto      在可能的情况下使用共享类数据 (默认)
    -Xshare:on        要求使用共享类数据, 否则将失败。
    -XshowSettings    显示所有设置并继续
    -XshowSettings:all
                      显示所有设置并继续
    -XshowSettings:vm 显示所有与 vm 相关的设置并继续
    -XshowSettings:properties
                      显示所有属性设置并继续
    -XshowSettings:locale
                      显示所有与区域设置相关的设置并继续

-X 选项是非标准选项, 如有更改, 恕不另行通知。
```

##### -Xint, -Xcomp，-Xmixed

* 在解释模式(interpreted mode)下，-Xint标记会强制JVM执行所有的字节码，当然这回降低运行速度，通常低10倍会更多。
* -Xcomp与它相反，JVM会在第一次使用时会把所有字节码编译成本地代码，从而带来最大的程序优化。
  * 然而，很多应用在使用-Xcomp也会有一些性能存世，当然比使用-Xint损失肖，原因是-Xcomp没有让JVM启用JIT的全部功能。JIT编译器可以对是否需要编译做判断，如果所有代码都进行编译的话，对那些只执行一次的代码就没有意义了。
* -Xmixed 混合模式，将解释模式和编译模式混合使用。由jvm自己决定，这是JVM默认的模式，也是推荐使用的模式。

```cmd
java -Xint -showversion TestJVM
```

### -XX参数

非标准参数，主要是用于jvm调优和debug操作。

​	有两种类型，一种是boolean类型，一种是非boolean类型:

* boolean类型
  * 格式: `-XX:[+-]<name>`表示启用或禁用name的属性。
  * 如: `-XX:+DisableExplicitC`表示禁用手动调用gc，也就是System.gc()无效
* 非boolean类型
  * 格式:`-XX:<name>=<value>` 表示`name`的属性值为`value`。
  * 如: `-XX:newRatio=1`表示新生代和老年代的比值。

```sh
java -XX:+DisableExplicitGC -showversion TestJVM
```

### -Xms与-Xmx参数

分别是设置堆内存的初始大小值和最大值。

* `-Xmx2048m`: 等价于`-XX:MaxHeapSize=2048m`，最大堆内存是2048M。
* `-Xms512m`: 等价于`-XX:InitialHeapSize=512m`, 设置JVM初始堆内存为512M。

### 运行java命令时打印参数

`-XX:+PrintFlagFinal`

```sh
java -XX:+PrintFlagFinal TestJVM
```

![](https://pic.imgdb.cn/item/6088dd6ad1a9ae528f6ef106.jpg)

= 代表默认值

:= 代表已经被修改过

### 查看正在运行的参数

```sh
jps
jps -l
jinfo -flags 6219
jinfo -flag MaxHeapSize 6219
```

## JVM 内存模型

​	1.7和1.8有较大区别。主要学习1.8，但是也要对1.7的内存模型有所了解。

### jdk1.7的内存模型

![](https://pic.imgdb.cn/item/6088eda6d1a9ae528ffbe1ea.jpg)

* Young区(新生代)

  Eden区和两个相同大小的Survivor区。

* Tenured 老年代

  主要保存声明周期长的对象，一般是一些老、或打的对象。

* Perm 永久代

  主要保存class,method,filed对象。一般不会溢出，除非一次性加载了很多类。有时候热部署的应用服务器会遇到溢出OutOfMemeoryError: PermGen space错误。可能是重新部署，之前的类没有卸载掉，重启应用即可。

* Virtual区

  * 最大内存和初始内存的差值，就是Virtual区

### jdk1.8的堆内存模型

![](https://pic.imgdb.cn/item/6088ef1fd1a9ae528f0d2903.jpg)



​	新生代+老年代。

* 新生代: Eden+ 2 * Survivor。
* 老年代: Olden



​	变化最大的是Perm区，用Metaspace(元数据空间)进行了替换。

​	Metaspace占用内存不再虚拟机内部，而是在本地内存空间中。

![](https://pic.imgdb.cn/item/6088ef97d1a9ae528f13082b.jpg)

### 为什么要废弃1.7中的永久代？

​	http://openjdk.java.net/jeps/122

​	移除永久代是为了融合Hotspot和JRockit而作出的努力，因为JRockit没有永久代，不需要配置永久代。

​	现实使用中，由于永久代内存经常不够用或发生内存泄露，爆出异常java.lang.OutOfMemoryError: PermGen。基于此，将永久代废弃，而改用元空间，改为了使用本地内存空间。

## 通过jstat命令查看堆内存使用情况

​	jstat命令可以查看堆内存的各部分的使用量，以及加载类的数量。命令的格式如下:

```sh
jstat [-命令选项] [vmid] [间隔时间/毫秒] [查询次数]
jstat -help|-options
jstat -<option> [-t] [-h<lines>] <vmid> [<interval> [<count>]]
```

### 查看类信息

```sh
jps
jstat -class 5584
Loaded  Bytes  Unloaded  Bytes     Time
 12602 22166.6        0     0.0      13.11
```

* Loaded: 已加载class。
* Bytes: 所占用空间大小。
* Unloaded: 未加载数量
* Bytes: 未加载占用空间
* Time: 时间

### 查看编译统计

```sh
jstat -compiler 5584
Compiled Failed Invalid   Time   FailedType FailedMethod
    5097      1       0     1.41          1 sun/misc/ProxyGenerator generateClassFile
```

* Compiled: 编译数量。
* Failed: 失败数量
* Invalid: 不可用数量
* Time: 时间
* FailedType: 失败类型。
* FailedMethod: 失败的方法。

### gc统计

```sh
jstat -gc 5584
 S0C    S1C    S0U    S1U      EC       EU        OC         OU       MC     MU    CCSC   CCSU   YGC     YGCT    FGC    FGCT     GCT
20480.0 25088.0  0.0    0.0   369664.0 96474.5   479232.0   30670.0   59160.0 55679.7 8448.0 7776.3      6    0.068   3      0.262    0.330
```

![](https://pic.imgdb.cn/item/6088f7a8d1a9ae528f613a4d.jpg)

## jmap的使用以及内存溢出分析

​	通过jstat可以对jvm的堆进行统计分析，而jmap可以获取到更加详细的信内容。如：内存使用情况的汇总、对内存溢出的定位于分析。

### 查看内存使用情况

```sh
jmap -heap 5584
Attaching to process ID 5584, please wait...
Debugger attached successfully.
Server compiler detected.
JVM version is 25.102-b14

using thread-local object allocation.
Parallel GC with 8 thread(s)

Heap Configuration:
   MinHeapFreeRatio         = 0
   MaxHeapFreeRatio         = 100
   MaxHeapSize              = 10708058112 (10212.0MB)
   NewSize                  = 223346688 (213.0MB)
   MaxNewSize               = 3569352704 (3404.0MB)
   OldSize                  = 447741952 (427.0MB)
   NewRatio                 = 2
   SurvivorRatio            = 8
   MetaspaceSize            = 21807104 (20.796875MB)
   CompressedClassSpaceSize = 1073741824 (1024.0MB)
   MaxMetaspaceSize         = 17592186044415 MB
   G1HeapRegionSize         = 0 (0.0MB)

Heap Usage:
PS Young Generation
Eden Space:
   capacity = 378535936 (361.0MB)
   used     = 109430400 (104.3609619140625MB)
   free     = 269105536 (256.6390380859375MB)
   28.908853715806785% used
From Space:
   capacity = 20971520 (20.0MB)
   used     = 0 (0.0MB)
   free     = 20971520 (20.0MB)
   0.0% used
To Space:
   capacity = 25690112 (24.5MB)
   used     = 0 (0.0MB)
   free     = 25690112 (24.5MB)
   0.0% used
PS Old Generation
   capacity = 490733568 (468.0MB)
   used     = 31406104 (29.951194763183594MB)
   free     = 459327464 (438.0488052368164MB)
   6.399827940851195% used

25858 interned Strings occupying 2443840 bytes.
```

### 查看内存中对象数量及大小

```sh
jmap -histo <pid> | more
# 查看活跃对象
jmap -histo:live <pid> | more
```

![](https://pic.imgdb.cn/item/6088febfd1a9ae528fa0400b.jpg)

**对象说明**

B byte

C char

D double

F float

I int

J long

Z boolean

[ 数组

[L+类名 其他对象

### 将内存使用情况dump到文件中

```sh
# 用法:
jmap -dump:formt=b,file=dumpFileName <pid>
```

### 通过jhat对dump文件进行分析

```sh
jhat -port <port> <file>
# 示例
jhat -port 9999 dump.dat
Reading from dump.dat...
Dump file created Wed Apr 28 14:28:39 CST 2021
Snapshot read, resolving...
Resolving 2030189 objects...
Chasing references, expect 406 dots......................................................................................................................................................................................................................................................................................................................................................................................................................
Eliminating duplicate references......................................................................................................................................................................................................................................................................................................................................................................................................................
Snapshot resolved.
Started HTTP server on port 9999
Server is ready.
```

http://localhost:9999/

![](https://pic.imgdb.cn/item/60890210d1a9ae528fc5a6ea.jpg)

拉到最后可以通过OQL进行查询。

![](https://pic.imgdb.cn/item/6089034bd1a9ae528fd36b21.jpg)

### 通过MAT工具对dumnp文件进行分析

**介绍**

​	MAT(Memory Analyzer Tool), 一个基于Eclipse的内存分析工具，是一个快速、功能丰富的Java heap分析工具。帮我们查找内存泄露和减少内存消耗。使用内存分析工具从众多对象中进行分析，快速计算出内存中对象占用大小，看看是谁阻止了垃圾收集器的回收工作，并且可以通过报表只管的查看到可能造成这种结果的对象。

官网地址: https://www.eclipse.org/mat

**类实例列表**

![](https://pic.imgdb.cn/item/60890a8bd1a9ae528f1aaf77.jpg)

![](https://pic.imgdb.cn/item/60890afdd1a9ae528f1eae0a.jpg)

**对象依赖树**

![](https://pic.imgdb.cn/item/60890c4ad1a9ae528f2a8d29.jpg)

## 实战: 内存溢出的定位与分析

​	内存溢出在实际生产环境中经常会遇到，比如，不断将数据写入到一个集合中，出现了死循环，读取超大文件等等，都有可能造成内存溢出。

​	出现了内存溢出，首先要定位发生内存溢出的环节，然后分析，是正常还是非正常的情况。正常的情况，应该考虑加大内存的设置，如果是非正常的需求，就要对代码进行修改。

### 模拟内存溢出

​	编写代码，向List集合中添加100万个字符串，每个字符串由1000个UUID组成。如果程序能够正常执行，最后打印ok。

```java
import java.util.ArrayList;
import java.util.List;
import java.util.UUID;

public class Test2 {
    public static void main(String[] args) {
        List<String> list = new ArrayList<>();
        for (int i = 0; i < 100000; i++) {
            String str = "";
            for (int j = 0; j < 1000; j++) {
                str += UUID.randomUUID().toString();
            }
            list.add("str");
        }
        System.out.println("ok");
    }
}
```

运行参数

```
-Xms8m -Xmx8m -XX:+HeapDumpOnOutOfMemoryError
```

```log
Connected to the target VM, address: '127.0.0.1:60231', transport: 'socket'
java.lang.OutOfMemoryError: Java heap space
Dumping heap to java_pid19168.hprof ...
Heap dump file created [8074728 bytes in 0.039 secs]
Exception in thread "main" java.lang.OutOfMemoryError: Java heap space
	at java.util.Arrays.copyOf(Arrays.java:3332)
	at java.lang.AbstractStringBuilder.ensureCapacityInternal(AbstractStringBuilder.java:124)
	at java.lang.AbstractStringBuilder.append(AbstractStringBuilder.java:448)
	at java.lang.StringBuilder.append(StringBuilder.java:136)
	at other.Test2.main(Test2.java:13)
```

用mat打开。

![](https://pic.imgdb.cn/item/608a30c3d1a9ae528f3d0494.jpg)

![](https://pic.imgdb.cn/item/608a3138d1a9ae528f411cd2.jpg)

## jstack的使用

​	有时候需要看下jvm中线程的执行情况，比如，发现服务器的CPU的负载突然增高了，出现了死锁、死循环等。我们该如何分析呢？

​	由于程序时正常运行的，没有任何输出，从日志方面也看不出什么问题，所以就需要看下jvm的内部线程的执行情况，然后再进行分析找出原因。

​	这时候，就需要借助于jstack命令，jstack的作用是将正在运行的jvm线程进行快照，并且打印出来:

```sh
# 用法: jsatck <pid>
```

### 线程的状态

![](https://pic.imgdb.cn/item/608a36a9d1a9ae528f6d27e7.jpg)

线程状态一共被分为6种:

* 初始态(NEW)
  * 创建一个Thread对象，但还未调用start()启动线程，线程处于初始状态。
* 运行台(RUNNABLE), 在Java中，运行态包括就绪态和运行态。
  * 就绪态
    * 该状态下的线程已经获得执行所需的所有资源，只要CPU分配执行权就能运行。
    * 所有就绪态的线程存放在就绪队列中。
  * 运行态
    * 获得CPU执行权，正在执行的线程。
    * 由于一个CPU同一时刻只能执行一条线程，因此每个CPU每个时刻只有一条运行态的线程。
* 阻塞态(BLOCKED)
  * 当一条正在执行的线程请求某一资源失败时，就会进入阻塞态。
  * 而在Java中，阻塞态专指请求锁失败时进入的状态。
  * 由一个阻塞队列存放的所有阻塞态的线程。
  * 处于阻塞态的线程会不断请求资源，一旦请求成功，就会进入就绪队列，等待执行。
* 等待态(WAITING)
  * 当前线程中调用wait、join、park函数时，当前线程就会进入等待态。
  * 也有一个等待对垒存放所有等待态的线程。
  * 线程处于等待态表示它需要等待其他线程的指示才能继续运行。
  * 进入等待态的线程会释放CPU的执行权，并释放资源(如: 锁)。
* 超时等待态(TIMED_WAITING)
  * 等运行中的线程调用sleep(time), wait, join, parkNanos, parkUntil时，就会进入该状态。
  * 它和等待态一样，并不是因为请求不到资源，而是主动进入，并且进入后需要其他线程唤醒。
  * 进入该状态后释放CPU执行权和占有的资源。
  * 与等待态区别: 到了超时时间后自动进入阻塞队列，开始竞争锁。
* 终止态(TERMINATED)
  * 线程执行结束后的状态。

### 实战: 死锁问题

#### 构造死锁

​	启动两个线程，Thead1拿到obj1锁，准备去拿obj2锁时obj2已经被Thead2锁定

```java
public class Test2 {
    public static Object obj1 = new Object();
    public static Object obj2= new Object();

    public static void main(String[] args) {
        new Thread(new Thread1()).start();
        new Thread(new Thread2()).start();
    }
    private static class Thread1 implements Runnable {
        @Override
        public void run() {
            synchronized (obj1) {
                System.out.println("thread1 获取到obj1锁");
                try {
                    Thread.sleep(2000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                synchronized (obj2) {
                    System.out.println("thread1 获取到obj2锁");
                }
            }
        }
    }

    private static class Thread2 implements Runnable {
        @Override
        public void run() {
            synchronized (obj2) {
                System.out.println("thread2 获取到obj2锁");
                try {
                    Thread.sleep(2000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                synchronized (obj1) {
                    System.out.println("thread2 获取到obj1锁");
                }
            }
        }
    }
}
```



```
Found one Java-level deadlock:
=============================
"Thread-1":
  waiting to lock monitor 0x0000000013023608 (object 0x00000000ffdf82a8, a java.lang.Object),
  which is held by "Thread-0"
"Thread-0":
  waiting to lock monitor 0x000000001301f568 (object 0x00000000ffdf82b8, a java.lang.Object),
  which is held by "Thread-1"

Java stack information for the threads listed above:
===================================================
"Thread-1":
        at other.Test2$Thread2.run(Test2.java:43)
        - waiting to lock <0x00000000ffdf82a8> (a java.lang.Object)
        - locked <0x00000000ffdf82b8> (a java.lang.Object)
        at java.lang.Thread.run(Thread.java:745)
"Thread-0":
        at other.Test2$Thread1.run(Test2.java:26)
        - waiting to lock <0x00000000ffdf82b8> (a java.lang.Object)
        - locked <0x00000000ffdf82a8> (a java.lang.Object)
        at java.lang.Thread.run(Thread.java:745)

Found 1 deadlock.
```

## VisualVM工具的使用

​	VisualVM，能够监控线程，内存情况，查看方法的CPU时间和内存中的对象，已被GC的对象，反向查看分配的堆栈(如100个String对象分别由哪几个对象分配出来的)。

​	VisualVM使用简单，几乎0配置，功能还是比较丰富的，几乎囊括了其他JDK自带的命令的所有功能。

* 内存信息。
* 线程信息。
* Dump堆(本地进程)。
* Dump线程(本地进程)。
* 打开堆Dump。堆Dump可用jmap来生成。
* 打开线程Dump。
* 生成应用快照(包含内存信息、线程信息等等)。
* 性能分析。CPU分析(各个方法的调用时间, 检查哪些方法耗时多)，内存分析(各类对象占用的内存没检查哪些类占用内存多)。

### 启动

​	jdk安装目录下，找到jvisualvm.exe, 双击打开。

## 监控远程jvm

### 什么是jmx

​	JMX(Java Management Extensions, 即java管理扩展)是一个为应用程序、设备、系统等植入管理功能的框架。JXM可以跨越一系列异构操作系统平台、系统体系结构和网络服传输协议， 灵活的开发无缝集成的系统、网络和服务管理应用。

### 监控远程的tomcat

```sh
# 在tomcat的bin目录下，修改catalina.sh，添加如下参数
JAVA_OPTS="-Dcom.sun.management.jmxremote -Dcom.sun.management.jmxremote.port=9999 -Dcom.sun.management.jmxremote.authenticate=false -Dcom.sun.management.jmxremote.ssl=false"
# 这几个参数的意思是
# -Dcom.sun.management.jmxremote 允许使用JMX远程管理
# -Dcom.sum.management.jmxremote.port=9999 JMX远程连接端口
# -Dcom.sum.management.jmxremote.authenticate=false 不进行身份认证，任何用户都可以连接
# -Dcom.sum.management.jmxremote.ssl=false 不使用ssl
```

​	保存退出，重启tomcat。

```sh
./shutdown.sh
./startup.sh && tail -f ../logs/catalina.out
```

### 连接远程tomcat

* 添加远程主机。
* 添加JMX连接。

## 什么是垃圾回收？

​	程序运行必然需要申请内存资源，无效的对象资源如果不及时处理就会一直占用内存资源，最终导致内存溢出，所以对内存资源的管理是非常重要的。

### C/C++语言的垃圾回收

​	没有自动垃圾回收机制，手动申请内存和释放内存。

### Java语言的垃圾回收

​	Java语言有自动的垃圾回收机制，有了垃圾回收机制，程序要只需要关心内存的申请即可，内存的释放由系统自动识别完成。

​	换句话说，自动的垃圾回收的算法就会变得非常重要，如果因为算法的不合理，导致内存资源一直没有释放，同时也可能导致内存溢出的。

### 常见垃圾回收算法

#### 引用计数法

优点:

* 实时性较高，无需等待到内存不够，才开始回收，运行时根据对象的计数器是否为0，就可以直接回收。
* 在垃圾回收过程中，应用无需挂起，如果申请内存时，内存不足，则立刻报outofMemory错误。
* 区域性，更新对象的计数器，只会影响到该对象，不会扫描全部对象。

缺点：

* 每次对象被引用时，都需要去更新计数器，有一点时间开销。
* 浪费CPU资源，即使内存够用，仍然在运行时进行计数器的统计。
* 无法解决循环引用问题。(最大的缺点)。

#### 标记清除法

![](https://pic.imgdb.cn/item/608a6926d1a9ae528f5e70a9.jpg)

​	这张图是程序运行期间所有对象的状态，它们的标记位全是0，假设这回有效内存空间耗尽了，jvm将会停止应用程序的运行并开启GC线程，然后开始进行标记工作，按照根搜索算法，标记完以后，对象的状态如下图:

![](https://pic.imgdb.cn/item/608a69dfd1a9ae528f64c573.jpg)

​	根据根搜索算法，所有从root对象可达的对象就被标记为了存活对象，此时已经完成了第一阶段的标记。接下来，就要执行第二节点的清除了，那么清除完以后，是剩下的对象以及对象的状态如图所示:

![](https://pic.imgdb.cn/item/608a6a50d1a9ae528f689993.jpg)

**优缺点**

优点：

* 解决了循环引用问题。

缺点：

* 效率较低，标记和清除两个动作都需要遍历所有的对象，并且在GC时，需要停止程序，对于交互性要求比较高的应用而言，这个体验非常差。
* 通过标记清除算法清理出来的内存，碎片化较为严重，因为被回收的对象可能存在于内存的各个角落，所以清理出来的内存是不连续的。

**为什么要暂停**

​	因为要遍历所有对象，对象引用关系会变化就会导致标记不准确。

#### 标记压缩算法

​	标记阶段一样，清除节点不是简单清理未标记对象，而是将存活的对象压缩到内存的一端，然后清理边界以外的垃圾，从而解决了碎片化的问题。

![](https://pic.imgdb.cn/item/608a6befd1a9ae528f75edfc.jpg)

**优缺点**

优点：

* 解决了垃圾碎片问题。

缺点：

* 比标记清除更多一步，效率更低。

## 复制算法

​	复制算法的核心就是，将原有的内存空间一分为二，每次只用其中的一块，在垃圾回收时，将正在使用的对象复制到另一个内存空间中，然后将该内存空间清空，交换两个内存的角色，完成垃圾回收。

​	如果内存中垃圾对象较多，需要复制的对象就较少，这种情况下使用该方式效率比较高。反之，则不适合。

![](https://pic.imgdb.cn/item/608a6cc1d1a9ae528f7c2f4a.jpg)

### JVM中的新生代的内存空间

![](https://pic.imgdb.cn/item/608a6d7cd1a9ae528f81cff8.jpg)

1. 在GC开始的时候，对象只会存在于Eden区和名为"From"的Survivor区，Survivor区的"To"是空的。
2. 紧接着进行GC，Eden区中所有存活对象都会被复制到"To"区，而在"From"区中，仍然粗活的对象会根据他们的年龄值来决定去向。年龄达到一定值(年龄阈值，可以通过-XX:MaxTenuringThreshold来设置)的对象会被移动到老年代中，没有达到阈值的会被复制到"To"区域。
3. 经过这次GC后，Eden去和From区都已经被清空。这个时候，From和To会交换角色。保证名为To的Survivor区是空的。
4. GC会一直重复这样的过程，直到"To"区被填满, "To"区被填满之后，会将所有对象移动到老年代中。

### 优缺点

优点：

* 在垃圾对象多的情况下，效率较高。
* 清理后，内存无碎片。

缺点：

* 在垃圾对象少的情况下，不适用，如: 老年代。
* 分配的两块内存空间，在同一个时刻，只能使用一般，内存使用率较低。

### 分代算法

​	分代算法根据回收对象的特点进行选择，在jvm中，新生代适合使用复制算法，老年代适合使用标记清除或标记压缩算法。

## 垃圾收集器以及内存分配

### 串行垃圾收集器

​	单线程进行垃圾回收，垃圾回收时，只有一个线程在工作，并且java应用中所有线程都要暂停，等待垃圾回收完成。这种现象叫做STW(Stop-The-World)。

​	对于交互性较强的应用，这种垃圾收集器是不能够接收的。

​	一般在javaweb应用中不会采用该收集器。

#### 编写测试代码

```java
public class TestGc {
    public static void main(String[] args) throws Exception{
        List<Object> list = new ArrayList<>();
        while(true) {
            int sleep = new Random().nextInt(100);
            if (System.currentTimeMillis() % 2 == 0) {
                list.clear();
            } else {
                for (int i = 0; i < 10000; i++) {
                    Properties p = new Properties();
                    p.put("key" + i, "value" + i);
                    list.add(p);
                }
            }
            Thread.sleep(sleep);
        }
    }
}
```

#### 设置垃圾回收为串行收集器

```sh
# 指定新生代和老年代都用串行垃圾回收器， 打印垃圾回收的详细信息
-XX:+UseSerialGC -XX:+PrintGCDetails
```

```log
[GC (Allocation Failure) [DefNew: 174784K->5296K(196608K), 0.0116826 secs] 174784K->5296K(633536K), 0.0118010 secs] [Times: user=0.00 sys=0.00, real=0.01 secs] 
[GC (Allocation Failure) [DefNew: 180080K->21823K(196608K), 0.0410587 secs] 180080K->22010K(633536K), 0.0410834 secs] [Times: user=0.05 sys=0.00, real=0.04 secs] 
```

![](https://pic.imgdb.cn/item/608a79a2d1a9ae528fdee507.jpg)

GC日志解读：

* DefNew
  * 表示使用的是串行垃圾收集器。
* 174784K->5296K(196608K)
  * 表示新生代GC前，有174784K内存，GC后，占用5296K内存，总大小196608K。
* 0.0118010 secs
  * GC所用的时间，单位为毫秒
* 174784K->5296K(633536K)
  * 表示，GC前，堆内存占用174784K，GC后，占有5296K，总大小633536K。
* Full GC
  * 内存空间全部进行GC。

### 并行垃圾收集器

#### ParNew垃圾收集器

​	是工作在新生代上的，只是将串行的垃圾收集器改为了并行。

​	通过`-XX:+UseParNewGC`参数设置新生代使用ParNew回收器，老年代使用的依然是串行收集器。

```sh
-XX:+UseParNewGC -XX:+PrintGCDetails -Xms16m -Xmx16m
```

```sh
[GC (Allocation Failure) [ParNew: 4928K->4928K(4928K), 0.0000177 secs][Tenured: 7531K->8171K(10944K), 0.0191096 secs] 12459K->8171K(15872K), [Metaspace: 3141K->3141K(1056768K)], 0.0191802 secs] [Times: user=0.02 sys=0.00, real=0.02 secs]

[GC (Allocation Failure) [ParNew: 4416K->4416K(4928K), 0.0000149 secs][Tenured: 8171K->10944K(10944K), 0.0205250 secs] 12587K->12585K(15872K), [Metaspace: 3141K->3141K(1056768K)], 0.0206098 secs] [Times: user=0.02 sys=0.00, real=0.02 secs]

[Full GC (Allocation Failure) [Tenured: 10944K->2834K(10944K), 0.0082399 secs] 15871K->2834K(15872K), [Metaspace: 3141K->3141K(1056768K)], 0.0082888 secs] [Times: user=0.00 sys=0.00, real=0.01 secs] 
```

#### ParallelGC垃圾收集器

​	ParalleGC收集器工作机制和ParNewGC收集器一样，只是在此基础之上，新增了两个和系统吞吐量相关的参数，使得其使用更加的灵活高效。

`-XX:+UseParallelGC`

* 新生代使用ParallelGC垃圾收集器，老年代使用串行收集器。

`-XX:+UseParallelOldGC`

* 新生代使用ParallelGC垃圾收集器，老年代使用ParallelOldGC收集器。

`-XX:MaxGCPauseMillis`

* 设置最大的垃圾收集的停顿时间，单位为毫秒。
* 需要注意的是，ParallelGC为了达到设置的停顿时间，可能会调整堆大小或其他参数，如果堆大小设置的比较小，就会导致GC工作很频繁，反而会影响到性能。
* 该参数使用需谨慎。

`-XX:GCTimeRatio`

* 设置垃圾回收时间占程序运行时间的百分比，公式为1/(1+n)。
* 它的值为0~100之间的数字，默认为99，也就是垃圾回收时间不能超过1%。

`-XX:UseAdaptiveSizePolicy`

* 自适应GC模式，垃圾收集器将自动调整新生代、老年代等参数，达到吞吐量、堆大小和停顿时间之间的平衡。
* 一般用于，手动调整参数比较困难的场景。让收集器自动进行调整。

测试

```sh
-XX:+UseParallelGC
-XX:+UseParallelOldGC
-XX:MaxGCPauseMillis=100
-XX:+PrintGCDetails 
-Xms16m 
-Xmx16m
```

```
[GC (Allocation Failure) [PSYoungGen: 3584K->1529K(3584K)] 11175K->10480K(14848K), 0.0017106 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
[Full GC (Ergonomics) [PSYoungGen: 1529K->0K(3584K)] [ParOldGen: 8951K->10095K(11264K)] 10480K->10095K(14848K), [Metaspace: 3141K->3141K(1056768K)], 0.0917094 secs] [Times: user=0.31 sys=0.00, real=0.09 secs] 
```

### CMS垃圾收集器

![](https://pic.imgdb.cn/item/608a81d1d1a9ae528f19e167.jpg)

* 初始化标记(CMS-initial-mark)： 标记root， 会导致stw。
* 并发标记(CMS-concurrent-mark): 与用户线程同时运行。
* 预清理(CMS-concurrent-preclean): 与用户线程同时运行。
* 重新标记(CMS-remark): 会导致stw。
* 并发清除(CMS-concurrent-sweep), 与用户线程同时运行。
* 调整堆大小，设置CMS在清理之后进行内存压缩，目的是清理内存中的碎片。
* 并发重置状态等待下次CMS的触发(CMS-concurrent-reset)，与用户线程同时运行。



**测试**

```sh
# 启动参数
-XX:+UseConcMarkSweepGC 
-XX:+PrintGCDetails 
-Xms16m 
-Xmx16m

# 运行日志
[GC (Allocation Failure) [ParNew: 4928K->512K(4928K), 0.0046008 secs] 9669K->8078K(15872K), 0.0046609 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
# 初始标记
[GC (CMS Initial Mark) [1 CMS-initial-mark: 7566K(10944K)] 8166K(15872K), 0.0002612 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
# 并发标记
[CMS-concurrent-mark-start]
[CMS-concurrent-mark: 0.004/0.004 secs] [Times: user=0.00 sys=0.00, real=0.01 secs] 
# 预处理
[CMS-concurrent-preclean-start]
[CMS-concurrent-preclean: 0.000/0.000 secs] [Times: user=0.00 sys=0.00, real=0.00 secs]
# 重新标记
[GC (CMS Final Remark) [YG occupancy: 2086 K (4928 K)][Rescan (parallel) , 0.0008513 secs][weak refs processing, 0.0001479 secs][class unloading, 0.0008767 secs][scrub symbol table, 0.0015791 secs][scrub string table, 0.0002906 secs][1 CMS-remark: 6573K(10944K)] 8660K(15872K), 0.0040221 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
# 并发清理
[CMS-concurrent-sweep-start]
[CMS-concurrent-sweep: 0.002/0.002 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
# 重置
[CMS-concurrent-reset-start]
[CMS-concurrent-reset: 0.000/0.000 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
```

### G1垃圾收集器(重点)

​	G1垃圾收集器是在jdk1.7中正式使用的全新的垃圾收集器，oracle官方计划jdk9将G1变为默认的垃圾收集器，以替代CMS。

​	G1的设计原则就是简化JVM性能调优，开发人员只需要简单的三步即可完成调优。

* 开启G1垃圾收集器。
* 设置堆的最大内存。
* 设置最大的停顿时间。



​	G1中提供了三种垃圾回收模式，Young GC、 Mixed GC和Full GC，在不同条件下被触发。

#### 原理

​	G1垃圾收集器相对比其他收集器而言，最大的区别是取消了新生代、老年代的物理划分，取而代之的是将堆划分为若干个区域(Region), 这些区域中包含了有逻辑上的新生代、老年代。

​	这样就不用单独的空间对每个代进行设置，也不用担心每个代区域的内存是否足够。

​	~~Eden~~

​	~~Survivor~~

​	~~Tenured~~

![](https://pic.imgdb.cn/item/608a8721d1a9ae528f3c0603.jpg)

![](https://pic.imgdb.cn/item/608a88d1d1a9ae528f46550f.jpg)

​	在G1划分的区域中，年轻代的垃圾收集依然采用暂停所有的应用线程的方式，将存活的对象拷贝到老年代或Survivor空间，G1收集器通过将对象从一个区域复制到另一个区域，完成了清理工作。

​	这就意味着，在正常的处理过程中，G1完成了堆的压缩(至少是部分堆的压缩), 这样就不会有cms内存碎片的问题存在了。

​	在G1中，有一种特殊的区域，叫Humongous区域。

* 如果一个对象超过了分区容量50%，G1收集器就认为这是一个巨型对象。
* 这些巨型对象，默认直接会被分配在老年代，但是如果它是一个短期存在的巨型对象，就会对垃圾收集器造成负面影响。
* 为了解决这个问题，G1划分了一个Humongous区，它用来专门存放巨型对象。如果一个H区装不下一个巨型对象，那么G1会寻找连续的H区来存储。为了找到连续的H区，有时候不得不启动Full gc。

#### Young GC

​	主要是对Eden区进行GC，在Eden空间耗尽时会被触发。

* Eden空间的数据移动到Survivor区，如果Survivor区的空间不够，Eden空间部分数据会直接晋升到老年代空间。
* Survivor去的数据移动到新的Survivor区中，也有部分数据晋升到老年代空间中。
* 最终Eden区空间数据为空，GC停止工作，应用线程继续执行。

![](https://pic.imgdb.cn/item/608ab05cd1a9ae528f9ad6e3.jpg)

![](https://pic.imgdb.cn/item/608ab096d1a9ae528f9d25ed.jpg)

**Remember Set**

​	在GC新生代的对象时，何如找到新生代中对象的根对象呢？
​	根对象可能是在新生代中，也可能在老年代中，那么老年代中的所有对象都是根么？

​	如果全量扫描老年代，那么这样扫描下来会耗费大量的时间。

​	于是，G1引进了Rset的概念，全称 Remember Set， 作用是跟踪指向某个堆的对象引用。

![](https://pic.imgdb.cn/item/608ab181d1a9ae528fa6f2d3.jpg)

​	每个Region初始化时，会初始化一个RSet。该集合用来记录并跟踪其他Region指向该Region中的对象的引用，每个Region默认按照512Kb划分成多个Card，所以RSet需要记录的东西应该是 xxRegion的xx Card。

#### **Mixed GC**

​	当越来越多的对象晋升到老年代old region时，为了避免堆内存被耗尽，虚拟机会触发一个混合的垃圾收集器，即 Mixed GC，并不是一个Old GC，除了回收整个Young Region，还会回收一部分的Old Region， 这里需要注意: 是一部分老年代，而不是全部老年代， 可以选择哪些old region进行回收，从而可以对垃圾回收的耗时时间进行控制。也要注意Mixed GC并不是Full GC。

​	Mixed GC什么时候出发？ 由参数`-XX:InitiatingHeapOccupancyPercent=n`决定。默认45%，该参数的意思是：当老年代大小沾整个堆大小百分比达到该阈值时触发。

​	GC分为两步：

	1. 全局并发标记(global concurrent marking)。

   	2. 拷贝存活对象(evacuation)。

**全局并发标记**

​	分为5个步骤：

* 初始标记(Initial mark, STW)
  * 标记从根节点直接可达的对象，这个阶段会执行一次新生代GC，会产生全局停顿。
* 根区域扫描(root region scan)
  * G1 GC在初始标记的存货区扫描对老年代的引用，并标记被引用的对象。
  * 该阶段与应用程序(非STW)同时运行，并且只有完成该阶段后，才能开始下一次STW年轻代垃圾的回收。
* 并发标记(concurrent marking)
  * G1 GC在整个堆中查找可访问的(存活的)对象。该阶段与应用程序同时运行，可以被STW新生代垃圾回收中断。
* 重新标记(Remark STW)
  * 该阶段是STW回收，因为程序在运行，针对上一次的标记进行修正。
* 清理垃圾(Cleanup, STW)
  * 清点和重置标记状态，该阶段会STW，这个阶段并不会实际上去做垃圾的收集，需要evacuation阶段来回收。

**拷贝存活对象**

​	Evacuation阶段是全暂停的。该阶段会把一部分Region里的活对象拷贝到另一部分Region中，从而实现垃圾的回收清理。

#### G1收集器相关参数

* `-XX:+UseG1GC`
  * 使用G1垃圾收集器。
* `-XX:MaxGCPauseMillis`
  * 设置期望达到的最大GC停顿时间的指标(JVM会尽力实现，但不保证达到)，默认值是200毫秒。
* `-XX:G1HeapRegionSize=n`
  * 设置G1区域的大小，值是2的幂，范围是1MB到32MB之间。目标是根据最小的Java堆划分出约2048个区域。
  * 默认是堆内存的1/2000。
* `-XX：ParallelGCThreads=n`
  * 设置STW工作线程数的值。将n的值设置为逻辑处理器的数量。n的值域逻辑处理器的数量相同，最多为8。
* `-XX：ConcGCThreads=n`
  * 设置并行标记的线程数。将n设置为并行垃圾回收线程数(ParallelGCThreads)的1/4左右
* `-XX:InitiatingHeapOccupancyPercent=n`
  * 设置触发标记周期的java堆占用阈值。默认是占用整个Java堆的45%。



**测试**

```sh
-XX:+UseG1GC -XX:MaxGCPauseMillis=100 -XX:+PrintGCDetails -Xmx256m
```

```sh
# 日志
[GC pause (G1 Evacuation Pause) (young), 0.0043929 secs]
   [Parallel Time: 3.8 ms, GC Workers: 8]
      [GC Worker Start (ms): Min: 284.9, Avg: 285.9, Max: 287.5, Diff: 2.6]
      # 扫描根节点
      [Ext Root Scanning (ms): Min: 0.0, Avg: 0.1, Max: 0.3, Diff: 0.3, Sum: 0.8]
      # 更新RS区域锁消耗的时间
      [Update RS (ms): Min: 0.0, Avg: 0.0, Max: 0.0, Diff: 0.0, Sum: 0.0]
         [Processed Buffers: Min: 0, Avg: 0.0, Max: 0, Diff: 0, Sum: 0]
      [Scan RS (ms): Min: 0.0, Avg: 0.0, Max: 0.0, Diff: 0.0, Sum: 0.0]
      [Code Root Scanning (ms): Min: 0.0, Avg: 0.0, Max: 0.0, Diff: 0.0, Sum: 0.0]
      # 对象拷贝
      [Object Copy (ms): Min: 1.1, Avg: 2.6, Max: 3.4, Diff: 2.4, Sum: 20.7]
      [Termination (ms): Min: 0.0, Avg: 0.1, Max: 0.1, Diff: 0.1, Sum: 0.5]
         [Termination Attempts: Min: 1, Avg: 1.0, Max: 1, Diff: 0, Sum: 8]
      [GC Worker Other (ms): Min: 0.0, Avg: 0.0, Max: 0.0, Diff: 0.0, Sum: 0.3]
      [GC Worker Total (ms): Min: 1.1, Avg: 2.8, Max: 3.8, Diff: 2.6, Sum: 22.4]
      [GC Worker End (ms): Min: 288.7, Avg: 288.7, Max: 288.7, Diff: 0.0]
   [Code Root Fixup: 0.0 ms]
   [Code Root Purge: 0.0 ms]
   [Clear CT: 0.1 ms] # 清空卡表
   [Other: 0.4 ms]
      [Choose CSet: 0.0 ms] # 选取CSet
      [Ref Proc: 0.2 ms] # 软引用、弱引用处理耗时
      [Ref Enq: 0.0 ms] # 弱引用、软引用入队耗时
      [Redirty Cards: 0.1 ms]
      [Humongous Register: 0.0 ms] # 大对象区域注册耗时
      [Humongous Reclaim: 0.0 ms] # 大对象区域回收耗时
      [Free CSet: 0.0 ms]
   [Eden: 9216.0K(9216.0K)->0.0B(4096.0K) Survivors: 0.0B->2048.0K Heap: 9216.0K(16.0M)->5117.0K(16.0M)] # 新生代大小统计
 [Times: user=0.00 sys=0.00, real=0.00 secs] 
```

#### **G1优化建议**

* 新生代大小

  * 避免使用-Xmn选项或-XX:NewRatio等相关选项显示设置新生代大小。
  * 固定新生代大小会覆盖暂停时间目标。

* 暂停时间目标不要太严苛

  * G1 GC的吞吐量目标是90的应用时间和10%的垃圾回收时间。
  * 评估G1 GC的吞吐量时，暂停时间不要太严苛。目标太严苛表示愿意接受更多的垃圾回收开销，进而影响到吞吐量。

  

## 可视化GC日志分析工具

​	涉及日志打印输出的参数如下:

```sh
-XX:+PrintGC # 输出GC日志
-XX:+PrintGCDetail # 输出GC详细日志
-XX:+PrintGCTimeStamps # 输出GC的时间戳(以基准时间的形式)
-XX:+PrintGCDateStamps # 输出GC的时间戳(以日期的形式，如 2020-04-05T21:53:59.245+0800)
-XX:+PrintHeapAtGC # 在进行GC的前后打印出堆的信息
-Xloggc:../logs/gc.log # 日志的输出路径
```

测试

```sh
-XX:+UseG1GC -XX:MaxGCPauseMillis=100 -Xmx256m -XX:+PrintGCDetails
-XX:+PrintGCTimeStamps -XX:+PrintGCDateStamps -XX:+PrintHeapAtGC
-Xloggc:F://Temp/gc/gc.log
```

```sh
Java HotSpot(TM) 64-Bit Server VM (25.102-b14) for windows-amd64 JRE (1.8.0_102-b14), built on Jun 22 2016 13:15:21 by "java_re" with MS VC++ 10.0 (VS2010)
Memory: 4k page, physical 41825224k(33993556k free), swap 44446664k(35239472k free)
CommandLine flags: -XX:InitialHeapSize=268435456 -XX:MaxGCPauseMillis=100 -XX:MaxHeapSize=268435456 -XX:+PrintGC -XX:+PrintGCDateStamps -XX:+PrintGCDetails -XX:+PrintGCTimeStamps -XX:+PrintHeapAtGC -XX:+UseCompressedClassPointers -XX:+UseCompressedOops -XX:+UseG1GC -XX:-UseLargePagesIndividualAllocation 
{Heap before GC invocations=0 (full 0):
 garbage-first heap   total 262144K, used 12288K [0x00000000f0000000, 0x00000000f0100800, 0x0000000100000000)
  region size 1024K, 12 young (12288K), 0 survivors (0K)
 Metaspace       used 3135K, capacity 4556K, committed 4864K, reserved 1056768K
  class space    used 333K, capacity 392K, committed 512K, reserved 1048576K
2021-04-29T22:13:08.829+0800: 0.360: [GC pause (G1 Evacuation Pause) (young), 0.0060868 secs]
   [Parallel Time: 5.2 ms, GC Workers: 8]
      [GC Worker Start (ms): Min: 359.7, Avg: 359.8, Max: 359.8, Diff: 0.1]
      [Ext Root Scanning (ms): Min: 0.2, Avg: 0.3, Max: 0.7, Diff: 0.4, Sum: 2.7]
      [Update RS (ms): Min: 0.0, Avg: 0.0, Max: 0.0, Diff: 0.0, Sum: 0.0]
         [Processed Buffers: Min: 0, Avg: 0.0, Max: 0, Diff: 0, Sum: 0]
      [Scan RS (ms): Min: 0.0, Avg: 0.0, Max: 0.0, Diff: 0.0, Sum: 0.0]
      [Code Root Scanning (ms): Min: 0.0, Avg: 0.0, Max: 0.1, Diff: 0.1, Sum: 0.1]
      [Object Copy (ms): Min: 4.3, Avg: 4.6, Max: 4.7, Diff: 0.4, Sum: 36.7]
      [Termination (ms): Min: 0.0, Avg: 0.0, Max: 0.1, Diff: 0.1, Sum: 0.4]
         [Termination Attempts: Min: 1, Avg: 5.0, Max: 10, Diff: 9, Sum: 40]
      [GC Worker Other (ms): Min: 0.0, Avg: 0.0, Max: 0.1, Diff: 0.0, Sum: 0.3]
      [GC Worker Total (ms): Min: 5.0, Avg: 5.0, Max: 5.1, Diff: 0.1, Sum: 40.3]
      [GC Worker End (ms): Min: 364.8, Avg: 364.8, Max: 364.8, Diff: 0.0]
   [Code Root Fixup: 0.0 ms]
   [Code Root Purge: 0.0 ms]
   [Clear CT: 0.1 ms]
   [Other: 0.7 ms]
      [Choose CSet: 0.0 ms]
      [Ref Proc: 0.4 ms]
      [Ref Enq: 0.0 ms]
      [Redirty Cards: 0.1 ms]
      [Humongous Register: 0.0 ms]
      [Humongous Reclaim: 0.0 ms]
      [Free CSet: 0.0 ms]
   [Eden: 12.0M(12.0M)->0.0B(10.0M) Survivors: 0.0B->2048.0K Heap: 12.0M(256.0M)->7024.5K(256.0M)]
Heap after GC invocations=1 (full 0):
 garbage-first heap   total 262144K, used 7024K [0x00000000f0000000, 0x00000000f0100800, 0x0000000100000000)
  region size 1024K, 2 young (2048K), 2 survivors (2048K)
 Metaspace       used 3135K, capacity 4556K, committed 4864K, reserved 1056768K
  class space    used 333K, capacity 392K, committed 512K, reserved 1048576K
}
 [Times: user=0.01 sys=0.00, real=0.01 secs] 
{Heap before GC invocations=1 (full 0):
 garbage-first heap   total 262144K, used 17264K [0x00000000f0000000, 0x00000000f0100800, 0x0000000100000000)
  region size 1024K, 12 young (12288K), 2 survivors (2048K)
 Metaspace       used 3136K, capacity 4556K, committed 4864K, reserved 1056768K
  class space    used 333K, capacity 392K, committed 512K, reserved 1048576K
```

### GC Easy可视化工具

​	是一款在线的可视化工具，易用，功能强大,  网站:

​	http://gceasy.io

## Tomcat8 优化

```sh
# 修改配置文件， 配置tomcat的管理用户
cd conf
vim tomcat-users.xml
# 写入如下内容:
<role rolename="manager"/>
<role rolename="manager-gui"/>
<role rolename="admin"/>
<role rolename="admin-gui"/>
<user username="tomcat" password="tomcat" roles="admin-gui,admin,manager-gui,manager"/>
# 如果是tomcat7，配置了tomcat用户就可以登录系统了，但是tomcat8不行，还需要修改另一个配置文件，否则访问不了，提示403
vim weapps/manager/META-INF/context.xml

# 将<Value的内容注释掉

<Context antiResourceLocking="false" privileged="true" >
  <CookieProcessor className="org.apache.tomcat.util.http.Rfc6265CookieProcessor"
                   sameSiteCookies="strict" />
           <!--  <Valve className="org.apache.catalina.valves.RemoteAddrValve"
                   allow="127\.\d+\.\d+\.\d+|::1|0:0:0:0:0:0:0:1" /> -->
  <Manager sessionAttributeValueClassNameFilter="java\.lang\.(?:Boolean|Integer|Long|Number|String)|org\.apache\.catalina\.filters\.CsrfPreventionFilter\$LruCache(?:\$1)?|java\.util\.(?:Linked)?HashMap"/>
</Context>

# 保存退出即可

# 启动tomcat
cd bin
./startup.sh && tail -f ../logs/catalina.out

# 打开浏览器访问
http://192.168.40.133:8080
```

![](https://pic.imgdb.cn/item/608addb1d1a9ae528f1ef821.jpg)

点击右上角server status，登录。

![](https://pic.imgdb.cn/item/608addf2d1a9ae528f226a61.jpg)



### 禁用AJP服务

​	在服务状态页面中可以看到，默认状态下会启用AJP服务，并且占用8009端口。

![](https://pic.imgdb.cn/item/608ae20ed1a9ae528f5a5c8c.jpg)

​	什么是AJP呢？

​	AJP(Apache JServer Protocol)

​	AJPv13协议是面向包的。Web服务器和Servlet容器通过TCP来交互; 为了节省Socket创建的昂贵代价，Web服务器会尝试维护一个永久TCP链接到servlet，并且在多个请求和响应周期过程会重用连接。

<img src="https://pic.imgdb.cn/item/608ae0e5d1a9ae528f49fc35.jpg" style="zoom:150%;" />

​	一般是使用Nginx+tomcat的架构，所以用不着AJP协议，所以把AJP连接器禁用。

​	修改conf下的server.xml文件，将AJP服务禁用掉即可。

```xml
<Connector protocol="AJP/1.3"
               address="::1"
               port="8009"
               redirectPort="8443" />
```

### 执行器(线程池)

​	在tomcat中每一个用户请求都是一个线程，所以可以使用线程池提高性能。

​	修改server.xml文件。

```xml
<!-- 注释打开-->
<Executor name="tomcatThreadPool" namePrefix="catalina-exec-"
        maxThreads="500" minSpareThreads="50" prestartminSpareThreads="true"
        maxQueueSize="100"/>
<!--
参数说明:
	maxThreads: 最大并发数，默认200，一般建议在500~1000，根据硬件设施和业务来判断
	minSpareThreads: Tomcat 初始化时创建的线程数，默认25
	prestartminSpareThreads: 在Tomcat初始化的时候就初始化 minSpareThreads的参数值，如果不等于true， minSpareThreads就没啥效果了
	maxQueueSize: 最大的等待队列数，超过则拒绝请求
-->

<!--在Connector中设置executor属性指向上面的执行器 注意把之前的Connector注释掉-->
<Connector executor="tomcatThreadPool"
               port="8080" protocol="HTTP/1.1"
               connectionTimeout="20000"
               redirectPort="8443" />

```

​	保存退出，重启Tomcat，查看效果。

​	在页面上看可能是-1，如果配置了Executor，该属性的任何值集将被正确记录，但是将被显示成-1

![](https://pic.imgdb.cn/item/608b6012d1a9ae528fe0cbae.jpg)

### 三种运行模式

**bio**

​	默认模式，性能非常低下，没有经过任何优化处理和支持。

**nio**

​	nio(new I/O)，是Java SE 1.4及后续版本提供的一种新的I/O方式，即(java.nio包及其子包)。是一个基于缓冲区，并能提供非阻塞I/O操作的Java API，因此也被看成是non-blocking I/O的缩写。拥有比传统I/O(bio)更高的并发运行性能。

**apr**

​	安装起来最困难，但是从操作系统级别来解决异步IO问题，大幅度提升性能。



​	推荐使用nio，不过，在tomcat8中有最新的nio2，速度更快，监视使用nio2.

​	设置nio2

```xml
<Connector executor="tomcatThreadPool"
               port="8080" protocol="org.apache.coyote.http11.Http11Nio2Protocol"
               connectionTimeout="20000"
               redirectPort="8443" />

```

### 工程以及JMeter

工程略

### 调整JVM参数

```sh
# 在tomcat的bin目录下，修改catalina.sh，添加如下参数
JAVA_OPTS="-XX:+UseParallelGC
-XX:+UseParallelOldGC
-XX:MaxGCPauseMillis=100
-XX:+PrintGCDetails 
-XX:+PrintGCTimeStamps
-XX:+PrintGCDateStamps
-XX:+PrintHeapAtGC
-Xloggc:../logs/gc.log
-Xms64m 
-Xmx512m"
```

查看GC日志，如果系统所消耗的时间大于用户时间，反映出服务器的性能存在瓶颈，调度CPU等资源所消耗的时间长一些。

![](https://pic.imgdb.cn/item/608b670bd1a9ae528f185bff.jpg)



​	新生代内存不足导致了很多gc。

**调整新生代大小**

```sh
# 在tomcat的bin目录下，修改catalina.sh，添加如下参数
JAVA_OPTS="-XX:+UseParallelGC
-XX:+UseParallelOldGC
-XX:MaxGCPauseMillis=100
-XX:+PrintGCDetails 
-XX:+PrintGCTimeStamps
-XX:+PrintGCDateStamps
-XX:+PrintHeapAtGC
-Xloggc:../logs/gc.log
-Xms128m 
-Xmx1024m
-XX:NewSize=64m
-XX:MaxNewSize=256m"
```

#### **设置G1垃圾收集器**

```sh
# 在tomcat的bin目录下，修改catalina.sh，添加如下参数
JAVA_OPTS="-XX:+UseG1GC
-XX:MaxGCPauseMillis=100
-XX:+PrintGCDetails 
-XX:+PrintGCTimeStamps
-XX:+PrintGCDateStamps
-XX:+PrintHeapAtGC
-Xloggc:../logs/gc.log
-Xms128m 
-Xmx1024m
```

## jvm字节码

​	从可以查看编译好的class文件中的字节码，去检查程序的效率是否高。

### 通过javap命令查看class文件的字节码

```java
package top.sorie.test;

public class JavapTest {
    public static void main(String[] args) {
        int a = 2;
        int b = 5;
        int c = b - a;
        System.out.println(c);
    }
}
```

通过javap命令查看class文件中的字节码内容:

```sh
javap -v JavapTest.class > Test1.txt

用法: javap <options> <classes>
其中, 可能的选项包括:
  -help  --help  -?        输出此用法消息
  -version                 版本信息
  -v  -verbose             输出附加信息
  -l                       输出行号和本地变量表
  -public                  仅显示公共类和成员
  -protected               显示受保护的/公共类和成员
  -package                 显示程序包/受保护的/公共类
                           和成员 (默认)
  -p  -private             显示所有类和成员
  -c                       对代码进行反汇编
  -s                       输出内部类型签名
  -sysinfo                 显示正在处理的类的
                           系统信息 (路径, 大小, 日期, MD5 散列)
  -constants               显示最终常量
  -classpath <path>        指定查找用户类文件的位置
  -cp <path>               指定查找用户类文件的位置
  -bootclasspath <path>    覆盖引导类文件的位置
```

```java
Classfile /root/src/javaDemo/JavapTest.class
  Last modified Apr 29, 2021; size 423 bytes
  MD5 checksum eb1358bdae0421cbcdf8519dc5be5f7d
  Compiled from "JavapTest.java"
public class top.sorie.test.JavapTest
  minor version: 0
  major version: 52
  flags: ACC_PUBLIC, ACC_SUPER
Constant pool:
// 常量池
   #1 = Methodref          #5.#14         // java/lang/Object."<init>":()V
   #2 = Fieldref           #15.#16        // java/lang/System.out:Ljava/io/PrintStream;
   #3 = Methodref          #17.#18        // java/io/PrintStream.println:(I)V
   #4 = Class              #19            // top/sorie/test/JavapTest
   #5 = Class              #20            // java/lang/Object
   #6 = Utf8               <init>
   #7 = Utf8               ()V
   #8 = Utf8               Code
   #9 = Utf8               LineNumberTable
  #10 = Utf8               main
  #11 = Utf8               ([Ljava/lang/String;)V
  #12 = Utf8               SourceFile
  #13 = Utf8               JavapTest.java
  #14 = NameAndType        #6:#7          // "<init>":()V
  #15 = Class              #21            // java/lang/System
  #16 = NameAndType        #22:#23        // out:Ljava/io/PrintStream;
  #17 = Class              #24            // java/io/PrintStream
  #18 = NameAndType        #25:#26        // println:(I)V
  #19 = Utf8               top/sorie/test/JavapTest
  #20 = Utf8               java/lang/Object
  #21 = Utf8               java/lang/System
  #22 = Utf8               out
  #23 = Utf8               Ljava/io/PrintStream;
  #24 = Utf8               java/io/PrintStream
  #25 = Utf8               println
  #26 = Utf8               (I)V
{
  public top.sorie.test.JavapTest();
  	// 无参构造器
    descriptor: ()V
    flags: ACC_PUBLIC
    Code:
      stack=1, locals=1, args_size=1
         0: aload_0
         1: invokespecial #1                  // Method java/lang/Object."<init>":()V
         4: return
      LineNumberTable:
        line 3: 0

  public static void main(java.lang.String[]);
    descriptor: ([Ljava/lang/String;)V // 方法描述，V代表返回值是viod
    flags: ACC_PUBLIC, ACC_STATIC // 方法修饰符 public, static的
    Code:
      // stack=2:操作栈大小为2。local=4:本地变量表大小4,args_size=1:参数的个数
      stack=2, locals=4, args_size=1 
         0: iconst_2 // 将数字2压入操作栈中
         1: istore_1 // 从操作栈中弹出一个元素，放入本地变量表，位于小标为1的位置(下标0是this)
         2: iconst_5 // 将数字5压入操作栈
         3: istore_2 // 从操作栈中弹出一个元素，放入本地变量表，位于小标为2的位置
         4: iload_2 // 加载本地变量表中下标为2位置的元素压入操作栈 5
         5: iload_1 // 加载本地变量表中下标为1位置的元素压入操作栈 2
         6: isub // 操作栈中的两个数字相减
         7: istore_3 // 将相减的数字存入本地变量表
         // 通过#2找到对应的常量，即可找到对应的引用
         8: getstatic     #2                  // Field java/lang/System.out:Ljava/io/PrintStream;
        11: iload_3 // 加载本地变量表中下标为3位置的元素压入操作栈 3
        // 通过#3找到对应的常量，即可找到对应的引用，进行方法调用
        12: invokevirtual #3                  // Method java/io/PrintStream.println:(I)V
        15: return // 返回
      LineNumberTable: // 行号的列表
        line 5: 0
        line 6: 2
        line 7: 4
        line 8: 8
        line 9: 15
}
SourceFile: "JavapTest.java"
```

![](https://pic.imgdb.cn/item/608b6e53d1a9ae528f5ad87c.jpg)

![](https://pic.imgdb.cn/item/608b6bb0d1a9ae528f441912.jpg)

**描述符**

字段描述符

![](https://pic.imgdb.cn/item/608b6bffd1a9ae528f46de31.jpg)

方法描述符

```java
Object m(int i, double d, Thread t) {
    ...
}
is:
(IDLjava/lang/Thread;)Ljava/lang/Object;
```

**图解**

![](https://pic.imgdb.cn/item/608b6ea5d1a9ae528f5d92d7.jpg)

### 研究i++和++i的不同

i++

![](https://pic.imgdb.cn/item/608b6ee7d1a9ae528f5fc464.jpg)

++i

![](https://pic.imgdb.cn/item/608b6f3dd1a9ae528f62a18b.jpg)

区别:

* i++
  * 只是在本地变量中对数字做了相加，没有将数据压入操作栈。
  * 将前面拿到的数字1，再从操作栈拿到，压入本地变量中。
* ++i
  * 在本地变量中对数字做了相加，并且将数据压入操作栈。
  * 将操作栈中的数据再次压入到本地变量中



​	小结: 可以通过查看字节码的方式对代码的底层做研究，探究其原理。

## 代码优化

**尽可能使用局部变量**

​	调用方法时，传递的参数以及再调用中创建的临时变量都保存在栈中，速度较快。

**尽量减少对变量的重复计算**

​	对方法的调用，即使方法中只有一句语句，也是由消耗的。

**尽量采用懒加载的策略, 需要的时候才创建**

​	...

**异常不应该用来控制程序流程**

​	异常对性能不利，抛出异常要创建一个新的对象，Throwable调用fillInStackTrace的本地同步方法，它检查堆栈，收集调用跟踪信息。只要有异常被抛出，虚拟机就必须调用堆栈，因为在处理过程中创建了一个新的对象。异常只能用于错误处理，不应该用来控制程序流程。

**不要创建一些不使用的对象，不要导入一些不使用的类**

​	...

**程序运行过程中避免使用反射**

​	效率不高, 尽量避免频繁使用，特别是Method的invoke方法。

**使用数据库连接池和线程池**

​	...

**容器初始化时尽可能指定长度**

​	避免容器长度不足。

**ArrayList随机遍历块，LinkedList添加删除快**

​	略

**使用Entry遍历map**

​	避免使用KeySet遍历。

**不要手动调用System.gc()**

​	略

**String 尽量少用正则表达式**

​	replace不支持正则。

​	replaceAll支持。

​	仅仅字符替换，建议使用replace。

**日志的输出要注意级别**

​	略

**对资源的close()分开操作**	

![](https://pic.imgdb.cn/item/608b7364d1a9ae528f861399.jpg)

## 参数快速索引

```sh
java -help
java -version
java version "1.8.0_102"
Java(TM) SE Runtime Environment (build 1.8.0_102-b14)
Java HotSpot(TM) 64-Bit Server VM (build 25.102-b14, mixed mode)

# 获取-D参数
java -Dstr=hello TestJVM 
java -client -showversion TestJVM
java -server -showversion TestJVM
java -Xms64m -Xmx128m -showversion TestJVM

java -XX:+PrintFlagFinal TestJVM

jps
jps -l
jinfo -flags 6219
jinfo -flag MaxHeapSize 6219

jps
jstat -class 5584
Loaded  Bytes  Unloaded  Bytes     Time
 12602 22166.6        0     0.0      13.11
 
jstat -compiler 5584
Compiled Failed Invalid   Time   FailedType FailedMethod
    5097      1       0     1.41          1 sun/misc/ProxyGenerator generateClassFile
    

jstat -gc 5584
 S0C    S1C    S0U    S1U      EC       EU        OC         OU       MC     MU    CCSC   CCSU   YGC     YGCT    FGC    FGCT     GCT
20480.0 25088.0  0.0    0.0   369664.0 96474.5   479232.0   30670.0   59160.0 55679.7 8448.0 7776.3      6    0.068   3      0.262    0.330

jmap -heap 5584

jmap -histo <pid> | more
# 查看活跃对象
jmap -histo:live <pid> | more

jmap -dump:formt=b,file=dumpFileName <pid>

# jhat分析dump文件 访问localhost:9999
jhat -port <port> <file>
# 示例
jhat -port 9999 dump.dat

# 指定新生代和老年代都用串行垃圾回收器， 打印垃圾回收的详细信息
-XX:+UseSerialGC -XX:+PrintGCDetails

# 设置新生代使用ParNew回收器，老年代使用的依然是串行收集器 
-XX:+UseParNewGC -XX:+PrintGCDetails -Xms16m -Xmx16m
# 新生代使用ParallelGC垃圾收集器，老年代使用串行收集器
-XX:+UseParallelGC
# 新生代使用ParallelGC垃圾收集器，老年代使用ParallelOldGC收集器。
-XX:+UseParallelOldGC
# 设置最大的垃圾收集的停顿时间，单位为毫秒。
# 需要注意的是，ParallelGC为了达到设置的停顿时间，可能会调整堆大小或其他参数，如果堆大小设置的比较小，就会导致GC工作很频繁，反而会影响到性能。
-XX:MaxGCPauseMillis
# 设置垃圾回收时间占程序运行时间的百分比，公式为1/(1+n)。
# 它的值为0~100之间的数字，默认为99，也就是垃圾回收时间不能超过1%
-XX:GCTimeRatio
# 自适应GC模式，垃圾收集器将自动调整新生代、老年代等参数，达到吞吐量、堆大小和停顿时间之间的平衡。
-XX:UseAdaptiveSizePolicy

# CMS垃圾收集器
-XX:+UseConcMarkSweepGC

# G1收集器
-XX:+UseG1GC
# 设置期望达到的最大GC停顿时间的指标(JVM会尽力实现，但不保证达到)，默认值是200毫秒。
-XX:MaxGCPauseMillis
# 设置G1区域的大小，值是2的幂，范围是1MB到32MB之间。目标是根据最小的Java堆划分出约2048个区域。默认是堆内存的1/2000
-XX:G1HeapRegionSize=n
# 设置并行标记的线程数。将n设置为并行垃圾回收线程数(ParallelGCThreads)的1/4左右
-XX：ConcGCThreads=n
# 设置触发标记周期的java堆占用阈值。默认是占用整个Java堆的45%
-XX:InitiatingHeapOccupancyPercent=n

# log参数

# 类回收相关参数

```

![](https://pic.imgdb.cn/item/60973618d1a9ae528fb322c6.jpg)

![](https://pic.imgdb.cn/item/6097362dd1a9ae528fb40fbe.jpg)



## CMS调优

https://blog.csdn.net/luzhensmart/article/details/105876603

```sh
-XX:+UseConcMarkSweepGC
# CMS的另一个缺点是它需要更大的堆空间。因为CMS标记阶段应用程序的线程还是在执行的，那么就会有堆空间继续分配的情况，为了保证在CMS回 收完堆之前还有空间分配给正在运行的应用程序，必须预留一部分空间。也就是说，CMS不会在老年代满的时候才开始收集。相反，它会尝试更早的开始收集，已 避免上面提到的情况：在回收完成之前，堆没有足够空间分配！默认当老年代使用68%的时候，CMS就开始行动了。 – XX:CMSInitiatingOccupancyFraction =n 来设置这个阀值。
–XX:CMSInitiatingOccupancyFraction=n
# old GC触发条件
# 1、如果没有设置-XX:+UseCMSInitiatingOccupancyOnly，虚拟机会根据收集的数据决定是否触发（建议线上环境带上这个参数，不然会加大问题排查的难度）。
# 2、老年代使用率达到阈值 CMSInitiatingOccupancyFraction，默认92%。
# 3、永久代的使用率达到阈值 CMSInitiatingPermOccupancyFraction，默认92%，前提是开启 CMSClassUnloadingEnabled。
# 4、新生代的晋升担保失败。

# CMSFullGCsBeforeCompaction 说的是，在上一次CMS并发GC执行过后，到底还要再执行多少次full GC才会做压缩

# 这两个设置一般配合使用,一般用于『降低CMS GC频率或者增加频率、减少GC时长』的需求
# XX:+UseCMSInitiatingOccupancyOnly 只是用设定的回收阈值(上面指定的70%),如果不指定,JVM仅在第一次使用设定值,后续则自动调整.
-XX:CMSInitiatingOccupancyFraction=70 
-XX:+UseCMSInitiatingOccupancyOnly
# 在CMS GC前启动一次ygc，目的在于减少old gen对ygc gen的引用，降低remark时的开销-----一般CMS的GC耗时 80%都在remark阶段
-XX:+CMSScavengeBeforeRemark
```

### CMS优化方向

- cms的的优势就是低延迟，但是如果出现了长时间的stw，则对应用程序有很大的影响
- 如果出现了concurrent mode failure和promotion failed，代价都非常昂贵，我们调优应该尽量避免这些情况

**针对concurrent mode failure的优化**

* 主要是调低CMSInitiatingOccupancyFraction的值
* 但是不能太低，太低会导致过于频繁的gc，会消耗更多的的cpu和停顿
* 需要先计算老年代常驻内存大小，如占用60%，那么这个阈值则可以设置为约70%，否则会比较频繁gc
* 可以考虑担保机制，只要老年代预留剩余空间大于年轻代大小，比如新生代和老年代的比例是1 : 4，即新生代占用老年代的25%，那么这个阈值可以设置为70，即老年代还预留出来30%的空间
* 注意如果浮动垃圾很多的话，也无法解决该问题，即cms并发回收期间，浮动垃圾越来越多，占用预留空间，多次的ygc的话，会有填满预留空间的可能，虽然概率较低
* 两个条件综合考虑，如果设置了阈值70，但是老年代常驻内存很大，甚至超过70，那么此时的建议要提高堆内存，增加老年代的大小或者减少新生代的大小。

**针对promotion failed的优化**

* 这个是cms最为严重的’碎片问题‘，我们要尽量避免这个发生后引起的fgc
* 所以优化这个问题，也可以描述为'如何解决碎片问题'.
* 增大堆内存，增加老年代大小，但要注意不要超过32g(the HotSpot JVM uses a trick to compress object pointers when heaps are less than around 32 GB)
* 尽早执行cms gc，合理设置CMSInitiatingOccupancyFraction，会合并老生代中相邻的free空间，可分配给较大的对象
* 和上面一样，也可以做一个老年代预留空间大于年轻代。
* 到了阈值后，就会触发cms gc，但还是和上面说的，会产生浮动垃圾 + 碎片，还是会出现
* 另外一个比较“挫”的办法，是在每天凌晨访问量低的时候，主动执行一下fgc，执行一下'碎片压缩'
* 如System.gc，但是要注意是否开启了-XX:+ExplicitGCInvokesConcurrent
* 所以建议办法是用jmap -histo:live
* 另外晋升还包括to space空间小，可以根据情况尝试提高Survivor

```sh
基础参数：
-Xloggc:gc_%t.log -XX:+PrintGCDetails -XX:+PrintGCDateStamps
 
可选调试参数:
 -XX:+PrintGCApplicationStoppedTime
 -XX:+PrintTenuringDistribution
 -XX:+PrintPromotionFailure
 -XX:+PrintHeapAtGC
 -XX:PrintFLSStatistics=1
 
1. 物理机内存：16G
2. 预估老年代常驻对象如Player 3000，一个Player平均2M，大约6G，所以老年代比如建议10G
3. -Xms12G -Xmx12G
4. 设置新生代2G，老年代10G
5. 设置CMSInitiatingOccupancyFraction为70，则老年代剩余空间为3G，大于新生代大小
6. 可选：-XX:+CMSScavengeBeforeRemark
 
简单算法：
-XX:NewRatio=4，即新生代和老年代1:4
然后设置CMSInitiatingOccupancyFraction为70，即老年代剩余空间稍大新生代
但要保证这个70基本上要大于老年代常驻内存，否则可能会频繁cms gc
 
另外建议增加脚本，尝试手动执行fgc，整理碎片
如每天凌晨3点
jstat -gccause pid >> cms.log
jmap -histo pid >> cms.log
jstat -gccause pid >> cms.log
jmap -histo:live pid >> cms.log
```



```
设置 -XX:MetaspaceSize=512m -XX:MaxMetaspaceSize=512m
注意如果设置的过小，则会引起fgc甚至metaspace oom
```

# 《深入理解Java虚拟机》

## 内存区域

* 方法区	
  * 类型信息、常量、静态变量、codeCache
  * 1.7永久代 
  * 1.8 Metaspace
  * OutOfMemory
  * 运行时常量池
    * 编译器生成的各种字面量与符号引用
* 虚拟机栈
  * 局部变量表
    * 基本数据类型
    * 对象引用
    * returnAddress
    * 以局部变量槽表示
  * 操作数栈
  * 动态链接
  * 方法出口
  * StackOverflowError + OutOfMemeory(可动态扩展，但内存不够)
* 本地方法栈
  * StackOverflowError + OutOfMemeory
* 堆
  * 几乎所有对象实例分配
  * -Xmx -Xms
* 程序计数器
  * 字节码指令地址
  * 如果本地方法，数字为空
  * 未规定OutofMemory
* 执行引擎
* 本地库接口
* 本地方法库



额外

* 直接内存
  * NIO
  * 更好的性能，不受堆大小限制。
  * OutOfMemory

## 对象

### **创建**

* 去常量池定位类符号引用(如果没有加载该类，就加载、解析、初始化)。
* 分配内存(如果规整 指针碰撞，否则 通过空闲列表)。
  * 修改指针存在同步问题，虚拟机采用CAS分配
  * TLAB(为本地线程分配缓存`-XX:+/-UseTLAB`)
* 初始化零值(不包括对象头)
* 必要的设置。
* 调用构造函数。

### 内存布局

标志位

![](https://pic.imgdb.cn/item/60961410d1a9ae528f2fd0ac.jpg)

​	实例数据部分是对象真正存储的有效信息，即我们在程序代码里面所定义的各种类型的字段内容，无论是从父类继承下来的，还是在子类中定义的字段都必须记录起来。这部分的存储顺序会受到虚拟机分配策略参数（-XX：FieldsAllocationStyle参数）和字段在Java源码中定义顺序的影响。HotSpot虚拟机默认的分配顺序为longs/doubles、ints、shorts/chars、bytes/booleans、oops（Ordinary Object Pointers，OOPs。

​	+XX：CompactFields参数值为true（默认就为true），那子类之中较窄的变量也允许插入父类变量的空隙之中，以节省出一点点空间。

​	**对齐填充**，这并不是必然存在的，也没有特别的含义，它仅仅起着占位符的作用。由于HotSpot虚拟机的自动内存管理系统要求对象起始地址必须是8字节的整数倍，换句话说就是任何对象的大小都必须是8字节的整数倍。对象头部分已经被精心设计成正好是8字节的倍数（1倍或者2倍），因此，如果对象实例数据部分没有对齐的话，就需要通过对齐填充来补全。

### 对象访问定位

* 句柄访问。
* 直接访问。
  * hotspot的方式。

![](https://pic.imgdb.cn/item/60961518d1a9ae528f3a7925.jpg)

![](https://pic.imgdb.cn/item/60961526d1a9ae528f3b0b1a.jpg)

## 垃圾回收

​	垃圾收集器需要完成的三件事

* 哪些内存需要回收？
* 什么时候回收？
* 如何回收?

### 哪些需要回收

* 引用计数
  * 相互依赖无法回收。
* 可发性分析
  * GCRoots
    * 栈帧对象(栈帧中的本地变量表引用的对象)。
    * 静态属性引用的对象，Java类的引用类型静态变量。
    * 方法区常量引用的对象
    * 本地方法栈中JNI引用的对象
    * 虚拟机内部引用，基本类型对应的Class对象、常驻的异常对象、系统
    * 同步锁持久的对象。
    * 反应虚拟机内部情况的JMXBean，JVMTI中注册的回调、本地方法缓存等。



### 引用类型

* 强引用。
* 软引用
  * 还有用，但非必须的对象。
  * 在系统将要发生内存溢出异常前，会把这些对象列进回收范围之中进行第二次回收，如果这次回收还没有足够的内存，才会抛出内存溢出异常。
* 弱引用
  * 非必须对象，但是它的强度比软引用更弱一些，被弱引用关联的对象只能生存到下一次垃圾收集发生为止。
  * 当垃圾收集器开始工作，无论当前内存是否足够，都会回收掉只被弱引用关联的对象。
* 虚引用
  * “幽灵引用”或者“幻影引用”，它是最弱的一种引用关系。
  * 它是最弱的一种引用关系。
  * 一个对象是否有虚引用的存在，完全不会对其生存时间构成影响，也无法通过虚引用来取得一个对象实例。为一个对象设置虚引用关联的唯一目的只是为了能在这个对象被收集器回收时收到一个系统通知。
  * PhantomReference类

### 方法区回收

* 废弃的常量和不再使用的类型。
  * 类型回收调教
  * 所有类实例都被回收
  * 加载该类的实例被回收。

### 分代理论

* 弱分代假说
  * 绝大多数对象朝生夕灭。
* 强分代假说
  * 熬过越多次垃圾回收过程的对象就越难消亡。
* 跨代引用假说
  * 跨代引用相对于同代引用来说仅占极少数。
  * 记忆集
    * 这个结构把老年代划分成若干小块，标识出老年代的哪一块内存会存在跨代引用。此后当发生Minor GC时，只有包含了跨代引用的小块内存里的对象才会被加入到GC Roots进行扫描。虽然这种方法需要在对象改变引用关系（如将自己或者某个属性赋值）时维护记录数据的正确性，会增加一些运行时的开销，但比起收集时扫描整个老年代来说仍然是划算的。



部分收集（Partial GC）：指目标不是完整收集整个Java堆的垃圾收集，其中又分为：

* 新生代收集（Minor GC/Young GC）：指目标只是新生代的垃圾收集。
* 老年代收集（Major GC/Old GC）：指目标只是老年代的垃圾收集。目前只有CMS收集器会有单独收集老年代的行为。另外请注意“Major GC”这个说法现在有点混淆，在不同资料上常有不同所指，读者需按上下文区分到底是指老年代的收集还是整堆收集。
* 混合收集（Mixed GC）：指目标是收集整个新生代以及部分老年代的垃圾收集。目前只有G1收集器会有这种行为。
* 整堆收集（Full GC）：收集整个Java堆和方法区的垃圾收集。

​	

### 回收算法

* 标记清除

  * 执行效率不稳定

    碎片化严重

* 标记整理

  * STW时间更长

* 标记复制

  * 浪费空间





**内存分配担保**

​	当Survivor空间不足以容纳一次Minor GC之后存活的对象时，就需要依赖其他内存区域（实际上大多就是老年代）进行分配担保（Handle Promotion）。

## 算法细节

### 根节点枚举

* OopMap

  * 直接得到哪些地方存放着对象引用。

  

### 安全点

​	HotSpot也的确没有为每条指令都生成OopMap，前面已经提到，只是在“特定的位置”记录了这些信息，这些位置被称为安全点（Safepoint）。

* 抢先式中断
  * 不需要线程的执行代码主动去配合，在垃圾收集发生时，系统首先把所有用户线程全部中断，如果发现有用户线程中断的地方不在安全点上，就恢复这条线程执行，让它一会再重新中断，直到跑到安全点上。
  * 几乎没有虚拟机实现采用抢先式中断。
* 主动式中断
  * 不直接对线程操作，仅仅简单地设置一个标志位，各个线程执行过程时会不停地主动去轮询这个标志，一旦发现中断标志为真时就自己在最近的安全点上主动中断挂起。
  * HotSpot使用内存保护陷阱
    * 一条指令。
    * 当需要暂停用户线程时，虚拟机把0x160100的内存页设置为不可读，那线程执行到test指令时就会产生一个自陷异常信号，然后在预先注册的异常处理器中挂起线程实现等待，这样仅通过一条汇编指令便完成安全点轮询和触发线程中断了。

### 安全区域

​	安全区域是指能够确保在某一段代码片段之中，引用关系不会发生变化，因此，在这个区域中任意地方开始垃圾收集都是安全的。

​	当线程要离开安全区域时，它要检查虚拟机是否已经完成了根节点枚举（或者垃圾收集过程中其他需要暂停用户线程的阶段），如果完成了，那线程就当作没事发生过，继续执行；否则它就必须一直等待，直到收到可以离开安全区域的信号为止。

### 记忆集和卡表

​	记忆集实现方式:

* **字长精度**：每个记录精确到一个机器字长（就是处理器的寻址位数，如常见的32位或64位，这个精度决定了机器访问物理内存地址的指针长度），该字包含跨代指针。
* **对象精度**: 每个记录精确到一个对象，该对象里有字段含有跨代指针。
* **卡精度**: 每个记录精确到一块内存区域，该区域内有对象含有跨代指针。

### 写屏障

​	在HotSpot虚拟机里是通过写屏障（Write Barrier）技术维护卡表状态的。

​	在赋值前的部分的写屏障叫作写前屏障（Pre-Write Barrier），在赋值后的则叫作写后屏障（Post-Write Barrier）。HotSpot虚拟机的许多收集器中都有使用到写屏障，但直至G1收集器出现之前，其他收集器都只用到了写后屏障。



**伪共享**

​	现代中央处理器的缓存系统中是以缓存行（Cache Line）为单位存储的，当多线程修改互相独立的变量时，如果这些变量恰好共享同一个缓存行，就会彼此影响（写回、无效化或者同步）而导致性能降低，这就是伪共享问题。

​	假设处理器的缓存行大小为64字节，由于一个卡表元素占1个字节，64个卡表元素将共享同一个缓存行。这64个卡表元素对应的卡页总的内存为32KB（64×512字节），也就是说如果不同线程更新的对象正好处于这32KB的内存区域内，就会导致更新卡表时正好写入同一个缓存行而影响性能。为了避免伪共享问题，一种简单的解决方案是不采用无条件的写屏障，而是先检查卡表标记，只有当该卡表元素未被标记过时才将其标记为变脏:

```c++
if (CARD_TABLE[this address >> 9] != 0)
    CARD_TABLE[this address >> 9] = 0;
```

​	HotSpot虚拟机增加了一个新的参数-XX：+UseCondCardMark，用来决定是否开启卡表更新的条件判断。

### 并发的可发性分析

**三色标记**

* 白色: 表示对象尚未被垃圾收集器访问过。显然在可达性分析刚刚开始的阶段，所有的对象都是白色的，若在分析结束的阶段，仍然是白色的对象，即代表不可达
* 黑色: 表示对象已经被垃圾收集器访问过，且这个对象的所有引用都已经扫描过。黑色的对象代表已经扫描过，它是安全存活的，如果有其他对象引用指向了黑色对象，无须重新扫描一遍。黑色对象不可能直接（不经过灰色对象）指向某个白色对象。
* 灰色: 表示对象已经被垃圾收集器访问过，但这个对象上至少存在一个引用还没有被扫描过。

**对象消失**

* 赋值器插入了一条或多条从黑色对象到白色对象的新引用；
* 赋值器删除了全部从灰色对象到该白色对象的直接或间接引用。

**增量更新**

* 增量更新要破坏的是第一个条件，当黑色对象插入新的指向白色对象的引用关系时，就将这个新插入的引用记录下来，等并发扫描结束之后，再将这些记录过的引用关系中的黑色对象为根，重新扫描一次。这可以简化理解为，黑色对象一旦新插入了指向白色对象的引用之后，它就变回灰色对象了。

**原始快照**

* 原始快照要破坏的是第二个条件，当灰色对象要删除指向白色对象的引用关系时，就将这个要删除的引用记录下来，在并发扫描结束之后，再将这些记录过的引用关系中的灰色对象为根，重新扫描一次。这也可以简化理解为，无论引用关系删除与否，都会按照刚刚开始扫描那一刻的对象图快照来进行搜索。

## 收集器

### **权衡**

​	如果你接手的是遗留系统，软硬件基础设施和JDK版本都比较落后，那就根据内存规模衡量一下，对于大概4GB到6GB以下的堆内存，CMS一般能处理得比较好，而对于更大的堆内存，可重点考察一下G1。

### 日志

​	jdk9以后

`-Xlog[:[selector][:[output][:[decorators][:output-options]]]]`

距离

| 说明                                                         | JDK9之前                        | JDK9                  |
| ------------------------------------------------------------ | ------------------------------- | --------------------- |
| GC基本信息                                                   | -XX:+PrintGC                    | -Xlog:gc:             |
| GC详细信息                                                   | -XX:+PrintGCDetails，           | -X-log:gc*            |
| GC前后的堆、方法区可用容量变化                               | -XX:+PrintHeapAtGC              | -Xlog:gc+heap=debug： |
| 收集器Ergonomics机制（自动设置堆空间各分代区域大小、收集目标等内容，从Parallel收集器开始支持）自动调节的相关信息。 | -XX:+PrintAdaptive-SizePolicy   | -Xlog:gc+ergo*=trace: |
| 查看熬过收集后剩余对象的年龄分布信息                         | -XX:+PrintTenuring-Distribution | -Xlog:gc+age=trace:   |

![](https://pic.imgdb.cn/item/60973579d1a9ae528fac5651.jpg)

![](https://pic.imgdb.cn/item/60973591d1a9ae528fad63c9.jpg)

### 收集器部分行为

* 大对象优先在Eden区分配。
* 大对象直接进入老年代。
* 长期存活的对象进入老年代。
* 动态年龄判断
  * 如果在Survivor空间中相同年龄所有对象大小的总和大于Survivor空间的一半，年龄大于或等于该年龄的对象就可以直接进入老年代，无须等到-XX：MaxTenuringThreshold中要求的年龄。
* 空间分配担保
  * 虚拟机必须先检查老年代最大可用的连续空间是否大于新生代所有对象总空间，如果这个条件成立，那这一次Minor GC可以确保是安全的。如果不成立，则虚拟机会先查看-XX：HandlePromotionFailure参数的设置值是否允许担保失败（Handle Promotion Failure）；如果允许，那会继续检查老年代最大可用的连续空间是否大于历次晋升到老年代对象的平均大小，如果大于，将尝试进行一次Minor GC，尽管这次Minor GC是有风险的；如果小于，或者-XX：HandlePromotionFailure设置不允许冒险，那这时就要改为进行一次Full GC。

## 调优案例

### 大内存

* 大内存必须有64位虚拟机的支持，但由于压缩指针、处理器缓存行容量（Cache Line）等因素，64位虚拟机的性能测试结果普遍略低于相同版本的32位虚拟机。
* 相同的程序在64位虚拟机中消耗的内存一般比32位虚拟机要大，这是由于指针膨胀，以及数据类型对齐补白等因素导致的，可以开启（默认即开启）压



### 集群同步导致内存溢出

略

### 不恰当的数据结构导致内存占用过大

​	100万个HashMap<Long，Long>Entry

​	如果不修改程序，仅从GC调优的角度去解决这个问题，可以考虑直接将Survivor空间去掉（加入参数-XX：SurvivorRatio=65536、-XX：MaxTenuringThreshold=0或者-XX：+Always-Tenure），让新生代中存活的对象在第一次Minor GC后立即进入老年代，等到Major GC的时候再去清理它们。这种措施可以治标，但也有很大副作用；治本的方案必须要修改程序，因为这里产生问题的根本原因是用HashMap<Long，Long>结构来存储数据文件空间效率太低了。

### 安全点导致的长时间停顿

-XX：+PrintSafepointStatistics和-XX：PrintSafepointStatisticsCount=1去查看安全点日志

-XX：+SafepointTimeout和-XX：SafepointTimeoutDelay=2000两个参数，让虚拟机在等到线程进入安全点的时间超过2000毫秒时就认定为超时，这样就会输出导致问题的线程名称

## 类文件结构

* 魔数
* 文件版本
  * 次版本号
  * 朱版本号
* 常量池
  * 字面量(文本字符串、final常量值)
  * 符号引用
    * 导出或开放的包。
    * 类、接口的全限定名。
    * 字段的名称和描述符。
    * 方法的名称和描述符。
    * 方法句柄和方法类型。
    * 动态调用点和动态常量。
* 访问标志
* 类索引、父类索引和接口索引
* 字段表集合
* 方法表集合
* 属性表集合。



大部分指令都没有支持整数类型byte、char和short，甚至没有任何指令支持boolean类型。

### 常见指令

* 将一个局部变量加载到操作栈：`iload、iload_<n>、lload、lload_<n>、fload、fload_<n>、dload、dload_<n>、aload、aload_<n>`

* ·将一个数值从操作数栈存储到局部变量表: `istore、istore_<n>、lstore、lstore_<n>、fstore、fstore_<n>、dstore、dstore_<n>、astore、astore_<n>`

* 将一个常量加载到操作数栈: `bipush、sipush、ldc、ldc_w、ldc2_w、aconst_null、iconst_m1、iconst_<i>、lconst_<l>、fconst_<f>、dconst_<d>`

* 扩充局部变量表的访问索引的指令：wide

  

**运算指令**

* 加法指令：iadd、ladd、fadd、dadd
* 减法指令：isub、lsub、fsub、dsub
* 乘法指令：imul、lmul、fmul、dmul
* 除法指令：idiv、ldiv、fdiv、ddiv
* 求余指令：irem、lrem、frem、drem
* 取反指令：ineg、lneg、fneg、dneg
* 位移指令：ishl、ishr、iushr、lshl、lshr、lushr
* 按位或指令：ior、lor
* 按位与指令：iand、land
* 按位异或指令：ixor、lxor
* 局部变量自增指令：iinc
* 比较指令：dcmpg、dcmpl、fcmpg、fcmpl、lcmp



​	仅规定了在处理整型数据时，只有除法指令（idiv和ldiv）以及求余指令（irem和lrem）中当出现除数为零时会导致虚拟机抛出ArithmeticException异常，其余任何整型数运算场景都不应该抛出运行时异常。

**转换指令**

​	小到大无需。

​	大到小：

​	i2b、i2c、i2s、l2i、f2i、f2l、d2i、d2l和d2f

**创建对象和访问**

* 创建类实例的指令：new
* 创建数组的指令：newarray、anewarray、multianewarray
* 访问类字段（static字段，或者称为类变量）和实例字段（非static字段，或者称为实例变量）的指令：getfield、putfield、getstatic、putstatic
* 把一个数组元素加载到操作数栈的指令：baload、caload、saload、iaload、laload、faload、daload、aaload
* 将一个操作数栈的值储存到数组元素中的指令：bastore、castore、sastore、iastore、fastore、dastore、aastore
* 取数组长度的指令：arraylength
* 检查类实例类型的指令：instanceof、checkcast

**操作栈管理**

* 将操作数栈的栈顶一个或两个元素出栈：pop、pop2
* 复制栈顶一个或两个数值并将复制值或双份的复制值重新压入栈顶：dup、dup2、dup_x1、dup2_x1、dup_x2、dup2_x2
* 将栈最顶端的两个数值互换：swap

**控制转移**

* 条件分支：ifeq、iflt、ifle、ifne、ifgt、ifge、ifnull、ifnonnull、if_icmpeq、if_icmpne、if_icmplt、if_icmpgt、if_icmple、if_icmpge、if_acmpeq和if_acmpne
* 复合条件分支：tableswitch、lookupswitch
* 无条件分支：goto、goto_w、jsr、jsr_w、ret

**方法调用和返回指令**

* invokevirtual指令：用于调用对象的实例方法，根据对象的实际类型进行分派（虚方法分派），这也是Java语言中最常见的方法分派方式。
* invokeinterface指令：用于调用接口方法，它会在运行时搜索一个实现了这个接口方法的对象，找出适合的方法进行调用。
* invokespecial指令：用于调用一些需要特殊处理的实例方法，包括实例初始化方法、私有方法和父类方法。
* invokestatic指令：用于调用类静态方法（static方法）。
* invokedynamic指令：用于在运行时动态解析出调用点限定符所引用的方法。并执行该方法。前面四条调用指令的分派逻辑都固化在Java虚拟机内部，用户无法改变，而invokedynamic指令的分派逻辑是由用户所设定的引导方法决定的

**异常处理指令**

​	在Java程序中显式抛出异常的操作（throw语句）都由athrow指令来实现，除了用throw语句显式抛出异常的情况之外，《Java虚拟机规范》还规定了许多运行时异常会在其他Java虚拟机指令检测到异常状况时自动抛出。例如前面介绍整数运算中，当除数为零时，虚拟机会在idiv或ldiv指令中抛出ArithmeticException异常。
​	而在Java虚拟机中，处理异常（catch语句）不是由字节码指令来实现的（很久之前曾经使用jsr和ret指令来实现，现在已经不用了），而是采用异常表来完成。

**同步指令**

​	方法级的同步是隐式的，无须通过字节码指令来控制，它实现在方法调用和返回操作之中。虚拟机可以从方法常量池中的方法表结构中的ACC_SYNCHRONIZED访问标志得知一个方法是否被声明为同步方法。当方法调用时，调用指令将会检查方法的ACC_SYNCHRONIZED访问标志是否被设置，如果设置了，执行线程就要求先成功持有管程，然后才能执行方法，最后当方法完成（无论是正常完成还是非正常完成）时释放管程。

​	同步一段指令集序列通常是由Java语言中的synchronized语句块来表示的，Java虚拟机的指令集中有monitorenter和monitorexit两条指令来支持synchronized关键字的语义，正确实现synchronized关键字需要Javac编译器与Java虚拟机两者共同协作支持。

## 虚拟机类加载机制

类的生命周期

* 加载

  * 加载时机
    * new、getstatic、putstatic、invokestatic
    * java.lang.reflect包的方法对类型进行反射调用。
    * 初始化类时、父类没初始化、需要初始化父类
    * 启动时的主类。
    * JDK 7新加入的动态语言支持时，如果一个java.lang.invoke.MethodHandle实例最后的解析结果为REF_getStatic、REF_putStatic、REF_invokeStatic、REF_newInvokeSpecial四种类型的方法句柄，并且这个方法句柄对应的类没有进行过初始化。
    * 一个接口中定义了JDK 8新加入的默认方法（被default关键字修饰的接口方法）时，如果有这个接口的实现类发生了初始化，那该接口要在其之前被初始化。
  * 加载过程
    * 通过一个类的全限定名来获取定义此类的二进制字节流。
    * 将这个字节流所代表的静态存储结构转化为方法区的运行时数据结构。
    * 在内存中生成一个代表这个类的java.lang.Class对象，作为方法区这个类的各种数据的访问入口。

* 连接

  * 验证
    * 确保Class文件的字节流中包含的信息符合《Java虚拟机规范》的全部约束要求
    * 文化格式验证
      * 魔数
      * 主次版本号
      * 常量池的常量类型
      * 索引值是否指向不存在的或不符合类型的常量。
      * CONSTANT_Utf8_info型的常量中是否有不符合UTF-8编码的数据。
      * Class文件中各个部分及文件本身是否有被删除的或附加的其他信息。
    * 元数据验证
      * 是否有父类
      * 父类是否继承了不允许被继承的类。
      * 类不是抽象，是否实现了其父类或接口中要求实现的方法。
      * 类的字段和方法是否和父类产生矛盾。
    * 字节码验证
      * 目的是通过数据流分析和控制流分析，确定程序语义是合法的、符合逻辑的。
    * 符号引用验证
      * 该类是否缺少或者被禁止访问它依赖的某些外部类、方法、字段等资源。
      * 符号引用中通过字符串描述的全限定名是否能找到对应的类。
      * 在指定类中是否存在符合方法的字段描述符及简单名称所描述的方法和字段。
      * 符号引用中的类、字段、方法的可访问性（private、protected、public、`<package>`）是否可被当前类访问。

  * 准备
    * 类中定义的变量（即静态变量，被static修饰的变量）分配内存并设置类变量初始值的阶段
  * 解析
    * 将常量池内的符号引用替换为直接引用的过程。
    * 符号引用（Symbolic References）：符号引用以一组符号来描述所引用的目标，符号可以是任何形式的字面量，只要使用时能无歧义地定位到目标即可。
    * 直接引用是可以直接指向目标的指针、相对偏移量或者是一个能间接定位到目标的句柄。
    * 类或接口解析
    * 字段解析
    * 方法解析。

* 初始化

  * 进行准备阶段时，变量已经赋过一次系统要求的初始零值，而在初始化阶段，则会根据程序员通过程序编码制定的主观计划去初始化类变量和其他资源。我们也可以从另外一种更直接的形式来表达：初始化阶段就是执行类构造器`<clinit>()`方法的过程。
  * `<clinit>()`方法是由编译器自动收集类中的所有类变量的赋值动作和静态语句块（static{}块）中的语句合并产生的，编译器收集的顺序是由语句在源文件中出现的顺序决定的，静态语句块中只能访问到定义在静态语句块之前的变量，定义在它之后的变量，在前面的静态语句块可以赋值，但是不能访问。

* 使用卸载

### 类加载器

* 比较两个类是否“相等”，只有在这两个类是由同一个类加载器加载的前提下才有意义，否则，即使这两个类来源于同一个Class文件，被同一个Java虚拟机加载，只要加载它们的类加载器不同，那这两个类就必定不相等。

### 双亲委派模型

​	从虚拟机角度，分类:

* 启动类加载器
  * C++实现
* 其他加载器
  * Java实现



​	从开发人员角度，其他加载器可以分为三层。

* 启动类加载器

* 扩展类加载器
  * 这个类加载器是在类sun.misc.Launcher$ExtClassLoader中以Java代码的形式实现的。
  * 负责加载<JAVA_HOME>\lib\ext目录中，或者被java.ext.dirs系统变量所指定的路径中所有的类库。
* 应用程序类加载器
  * sun.misc.Launcher$AppClassLoader。
  * 负责加载用户类路径（ClassPath）上所有的类库，开发者同样可以直接在代码中使用这个类加载器。



* 双亲委派

  * 如果一个类加载器收到了类加载的请求，它首先不会自己去尝试加载这个类，而是把这个请求委派给父类加载器去完成，每一个层次的类加载器都是如此，因此所有的加载请求最终都应该传送到最顶层的启动类加载器中，只有当父加载器反馈自己无法完成这个加载请求（它的搜索范围中没有找到所需的类）时，子加载器才会尝试自己去完成加载。

  * 三次破坏双亲委派

    * JDK1.2才出现双亲委派，为了兼容问题，添加了findClass()方法，用户编写类加载逻辑尽量去重写这个方法。如果父类加载失败，会自动调用自己的findClass()方法来完成加载，这样既不影响用户按照自己的意愿去加载类，又可以保证新写出来的类加载器是符合双亲委派规则的。

    * 有基础类型又要调用回用户的代码，那该怎么办呢？

      * 线程上下文类加载器
      * 这个类加载器可以通过java.lang.Thread类的setContext-ClassLoader()方法进行设置，如果创建线程时还未设置，它将会从父线程中继承一个，如果在应用程序的全局范围内都没有设置过的话，那这个类加载器默认就是应用程序类加载器。
      * 在JDK 6时，JDK提供了java.util.ServiceLoader类，以META-INF/services中的配置信息，辅以责任链模式，来消除硬编码。

    * 用户对程序动态性的追求

      * 热替换

      * 模块热部署

        * OSGI搜索顺序

          * 将以java.*开头的类，委派给父类加载器加载。
          * 否则，将委派列表名单内的类，委派给父类加载器加载。
          * 否则，查找当前Bundle的ClassPath，使用自己的类加载器加载。
          * 否则，查找类是否在自己的Fragment Bundle中，如果在，则委派给Fragment Bundle的类加载器加载。
          * 否则，查找Dynamic Import列表的Bundle，委派给对应Bundle的类加载器加载。
          * 否则，类查找失败。

          

## 虚拟机字节码执行引擎

* 运行时栈帧结构

  * 局部变量表
    * 用于放方法参数和方法内部定义的局部变量。
    * 变量槽为单位，变量槽可以32位、也可以64位。
      * 即使在64位虚拟机中使用了64位的物理内存空间去实现一个变量槽，虚拟机仍要使用对齐和补白的手段让变量槽在外观上看起来与32位虚拟机中的一致。
    * 通过索引定位的方式使用局部变量表，索引值的范围是从0开始至局部变量表最大的变量槽数量。如果访问的是32位数据类型的变量，索引N就代表了使用第N个变量槽，如果访问的是64位数据类型的变量，则说明会同时使用第N和N+1两个变量槽。不允许采用任何方式单独访问其中的某一个。
    * 是实例方法（没有被static修饰的方法），那局部变量表中第0位索引的变量槽默认是用于传递方法所属对象实例的引用，在方法中可以通过关键字“this”来访问到这个隐含的参数。
    * 变量槽可以重用。
    * 某些情况下变量槽的复用会直接影响到系统的垃圾收集行为

  * 操作数栈
    * 写入到Code属性的max_stacks数据项之中。
    * 32位数据类型所占的栈容量为1，64位数据类型所占的栈容量为2。
    * 操作数栈的深度都不会超过在max_stacks数据项中设定的最大值。
    * 不同栈帧作为不同方法的虚拟机栈的元素，是完全相互独立的。但是在大多虚拟机的实现里都会进行一些优化处理，令两个栈帧出现一部分重叠。让下面栈帧的部分操作数栈与上面栈帧的部分局部变量表重叠在一起，这样做不仅节约了一些空间，更重要的是在进行方法调用时就可以直接共用一部分数据，无须进行额外的参数复制传递了。

  ![](https://pic.imgdb.cn/item/6097e9d5d1a9ae528f0b438d.jpg)

  * 动态链接
    * 每个栈帧都包含一个指向运行时常量池[1]中该栈帧所属方法的引用，持有这个引用是为了支持方法调用过程中的动态连接（Dynamic Linking）。
    * 符号引用一部分会在类加载阶段或者第一次使用的时候就被转化为直接引用，这种转化被称为静态解析。另外一部分将在每一次运行期间都转化为直接引用，这部分就称为动态连接。
  * 方法返回地址
    * 只有两种方式退出这个方法。
    * 第一种方式是执行引擎遇到任意一个方法返回的字节码指令，这时候可能会有返回值传递给上层的方法调用者（调用当前方法的方法称为调用者或者主调方法），方法是否有返回值以及返回值的类型将根据遇到何种方法返回指令来决定，这种退出方法的方式称为“正常调用完成”（Normal Method Invocation Completion）
    * 另外一种退出方式是在方法执行的过程中遇到了异常，并且这个异常没有在方法体内得到妥善处理。无论是Java虚拟机内部产生的异常，还是代码中使用athrow字节码指令产生的异常，只要在本方法的异常表中没有搜索到匹配的异常处理器，就会导致方法退出，这种退出方法的方式称为“异常调用完成（Abrupt Method Invocation Completion）”。一个方法使用异常完成出口的方式退出，是不会给它的上层调用者提供任何返回值的。
    * 在方法退出之后，都必须返回到最初方法被调用时的位置，程序才能继续执行，方法返回时可能需要在栈帧中保存一些信息，用来帮助恢复它的上层主调方法的执行状态。一般来说，方法正常退出时，主调方法的PC计数器的值就可以作为返回地址，栈帧中很可能会保存这个计数器值。而方法异常退出时，返回地址是要通过异常处理器表来确定的，栈帧中就一般不会保存这部分信息。
    * 方法退出的过程实际上等同于把当前栈帧出栈，因此退出时可能执行的操作有：恢复上层方法的局部变量表和操作数栈，把返回值（如果有的话）压入调用者栈帧的操作数栈中，调整PC计数器的值以指向方法调用指令后面的一条指令等。
  * 在编译Java程序源码的时候，栈帧中需要多大的局部变量表，需要多深的操作数栈就已经被分析计算出来，并且写入到方法表的Code属性之中
  * 一个线程中的方法调用链可能会很长，以Java程序的角度来看，同一时刻、同一条线程里面，在调用堆栈的所有方法都同时处于执行状态。而对于执行引擎来讲，在活动线程中，只有位于栈顶的方法才是在运行的，只有位于栈顶的栈帧才是生效的，其被称为“当前栈帧”（Current Stack Frame），与这个栈帧所关联的方法被称为“当前方法”（Current Method）。执行引擎所运行的所有字节码指令都只针对当前栈帧进行操作

### 方法调用

* 解析

  * 调用目标在程序代码写好、编译器进行编译那一刻就已经确定下来。这类方法的调用被称为解析（Resolution）。
  * 只要能被invokestatic和invokespecial指令调用的方法，都可以在解析阶段中确定唯一的调用版本，Java语言里符合这个条件的方法共有静态方法、私有方法、实例构造器、父类方法4种，再加上被final修饰的方法（尽管它使用invokevirtual指令调用），这5种方法调用会在类加载的时候就可以把符号引用解析为该方法的直接引用。这些方法统称为“非虚方法”（Non-Virtual Method）。
  * Java中的非虚方法除了使用invokestatic、invokespecial调用的方法之外还有一种，就是被final修饰的实例方法。
  * 由于历史设计的原因，final方法是使用invokevirtual指令来调用的，但是因为它也无法被覆盖，没有其他版本的可能。
  
* 分派

  * 静态分派

    * 所有依赖静态类型来决定方法执行版本的分派动作，都称为静态分派。静态分派的最典型应用表现就是方法重载。静态分派发生在编译阶段，因此确定静态分派的动作实际上不是由虚拟机来执行的，这点也是为何一些资料选择把它归入“解析”而不是“分派”的原因。
  * 动态分派

    * 它与Java语言多态性的另外一个重要体现[3]——重写（Override）有着很密切的关联。
    * invokevirtual指令的运行时解析过程[4]大致分为以下几步：
      * 找到操作数栈顶的第一个元素所指向的对象的实际类型，记作C。
      * ）如果在类型C中找到与常量中的描述符和简单名称都相符的方法，则进行访问权限校验，如果通过则返回这个方法的直接引用，查找过程结束；不通过则返回java.lang.IllegalAccessError异常。
      * 否则，按照继承关系从下往上依次对C的各个父类进行第二步的搜索和验证过程。
      * 子类的字段会遮蔽父类的同名字段。
  * 单分派和多分派
    * Java语言的静态分派属于多分派类型。需要考虑两个宗量。
    * 因为只有一个宗量作为选择依据，所以Java语言的动态分派属于单分派类型。
  * 动态分派实现
    * 虚方法表
    * 虚方法表中存放着各个方法的实际入口地址。如果某个方法在子类中没有被重写，那子类的虚方法表中的地址入口和父类相同方法的地址入口是一致的，都指向父类的实现入口。如果子类中重写了这个方法，子类虚方法表中的地址也会被替换为指向子类实现版本的入口地址。
    * 为了进一步提高性能，还会使用类型继承关系分析（Class Hierarchy Analysis，CHA）、守护内联（Guarded Inlining）、内联缓存（Inline Cache）等多种非稳定的激进优化来争取更大的性能空间。  

    

  方法句柄。

  ```c++
  public class MethodHandleTest {
  
      static class ClassA {
          public void println(String s) {
              System.out.println(s);
          }
      }
  
      public static void main(String[] args) throws Throwable {
          Object obj = System.currentTimeMillis() % 2 == 0 ? System.out : new ClassA();
          // 无论obj最终是哪个实现类，下面这句都能正确调用到println方法。
          getPrintlnMH(obj).invokeExact("icyfenix");
      }
  
      private static MethodHandle getPrintlnMH(Object reveiver) throws Throwable {
          // MethodType：代表“方法类型”，包含了方法的返回值（methodType()的第一个参数）和具体参数（methodType()第二个及以后的参数）。
          MethodType mt = MethodType.methodType(void.class, String.class);
          // lookup()方法来自于MethodHandles.lookup，这句的作用是在指定类中查找符合给定的方法
             名称、方法类型，并且符合调用权限的方法句柄。
          // 因为这里调用的是一个虚方法，按照Java语言的规则，方法第一个参数是隐式的，代表该方法的接收者，也即this指向的对象，这个参数以前是放在参数列表中进行传递，现在提供了bindTo()方法来完成这件事情。
          return lookup().findVirtual(reveiver.getClass(), "println", mt).bindTo(reveiver);
      }
  }
  ```

  和反射的区别

* Reflection和MethodHandle机制本质上都是在模拟方法调用，但是Reflection是在模拟Java代码层次的方法调用，而MethodHandle是在模拟字节码层次的方法调用。在MethodHandles.Lookup上的3个方法findStatic()、findVirtual()、findSpecial()正是为了对应于invokestatic、invokevirtual（以及invokeinterface）和invokespecial这几条字节码指令的执行权限校验行为，而这些底层细节在使用Reflection API时是不需要关心的。

* Reflection中的java.lang.reflect.Method对象远比MethodHandle机制中的java.lang.invoke.MethodHandle对象所包含的信息来得多。前者是方法在Java端的全面映像，包含了方法的签名、描述符以及方法属性表中各种属性的Java端表示方式，还包含执行权限等的运行期信息。而后者仅包含执行该方法的相关信息。用开发人员通俗的话来讲，Reflection是重量级，而MethodHandle是轻量级。

* 由于MethodHandle是对字节码的方法指令调用的模拟，那理论上虚拟机在这方面做的各种优化（如方法内联），在MethodHandle上也应当可以采用类似思路去支持（但目前实现还在继续完善中），而通过反射去调用方法则几乎不可能直接去实施各类调用点优化措施。



* invokedynamic
  * 每一处含有invokedynamic指令的位置都被称作“动态调用点（Dynamically-Computed Call Site）”
  * 变为JDK 7时新加入的CONSTANT_InvokeDynamic_info常量，从这个新常量中可以得到3项信息：引导方法（Bootstrap Method，该方法存放在新增的BootstrapMethods属性中）、方法类型（MethodType）和名称。
    * 引导方法是有固定的参数，并且返回值规定是java.lang.invoke.CallSite对象，这个对象代表了真正要执行的目标方法调用。

```java
public class InvokeDynamicTest {

    public static void main(String[] args) throws Throwable {
        INDY_BootstrapMethod().invokeExact("icyfenix");
    }

    public static void testMethod(String s) {
        System.out.println("hello String:" + s);
    }

    public static CallSite BootstrapMethod(MethodHandles.Lookup lookup, String name, MethodType mt) throws Throwable {
        return new ConstantCallSite(lookup.findStatic(InvokeDynamicTest.class, name, mt));
    }

    private static MethodType MT_BootstrapMethod() {
        return MethodType
                .fromMethodDescriptorString(
                        "(Ljava/lang/invoke/MethodHandles$Lookup;Ljava/lang/String; Ljava/lang/invoke/MethodType;)Ljava/lang/invoke/CallSite;", null);
    }

    private static MethodHandle MH_BootstrapMethod() throws Throwable {
        return lookup().findStatic(InvokeDynamicTest.class, "BootstrapMethod", MT_BootstrapMethod());
    }

    private static MethodHandle INDY_BootstrapMethod() throws Throwable {
        CallSite cs = (CallSite) MH_BootstrapMethod().invokeWithArguments(lookup(), "testMethod",
                MethodType.fromMethodDescriptorString("(Ljava/lang/String;)V", null));
		return cs.dynamicInvoker();
    }
}
```



###  实战

​	在Java程序中，可以通过“super”关键字很方便地调用到父类中的方法，但如果要访问祖类的方法呢？

```java
class Test {

class GrandFather {
    void thinking() {
        System.out.println("i am grandfather");
    }
}

class Father extends GrandFather {
    void thinking() {
        System.out.println("i am father");
    }
}

class Son extends Father {
    void thinking() {
        try {
                MethodType mt = MethodType.methodType(void.class);
MethodHandle mh = lookup().findSpecial(GrandFather.class,
"thinking", mt, getClass());
                mh.invoke(this);
            } catch (Throwable e) {
            }
        }
    }

    public static void main(String[] args) {
        (new Test().new Son()).thinking();
    }
}
```

​	使用JDK 7 Update 9之前的HotSpot虚拟机运行，会得到如下运行结果：
i am grandfather
​	但是这个逻辑在JDK 7 Update 9之后被视作一个潜在的安全性缺陷修正了，原因是必须保证findSpecial()查找方法版本时受到的访问约束（譬如对访问控制的限制、对参数类型的限制）应与使用invokespecial指令一样，两者必须保持精确对等，包括在上面的场景中它只能访问到其直接父类中的方法版本。

​	所以在JDK 7 Update 10修正之后，运行以上代码只能得到如下结果：
i am father

​	那在新版本的JDK中，上面的问题是否能够得到解决呢？答案是可以

​	访问保护是通过一个allowedModes的参数来控制，而且这个参数可以被设置成“TRUSTED”来绕开所有的保护措施。尽管这个参数只是在Java类库本身使用，没有开放给外部设置，但我们通过反射可以轻易打破这种限制。

```java
void thinking() {
    try {
        MethodType mt = MethodType.methodType(void.class);
        Field lookupImpl = MethodHandles.Lookup.class.getDeclaredField("IMPL_LOOKUP");
        lookupImpl.setAccessible(true);
        MethodHandle mh = ((MethodHandles.Lookup) lookupImpl.get(null)).findSpecial(GrandFather.class,"thinking", mt, GrandFather.class);
        mh.invoke(this);
    } catch (Throwable e) {
    }
}
```

运行以上代码，在目前所有JDK版本中均可获得如下结果：
i am grandfather



* 基于栈的执行引擎
* 基于栈的指令集和基于寄存器的指令值
  * Javac编译器输出的字节码指令流，基本上[1]是一种基于栈的指令集架构（Instruction Set Architecture，ISA），字节码指令流里面的指令大部分都是零地址指令，它们依赖操作数栈进行工作。
  * 可移植。
  * 性能较差。

## 类加载及执行子系统的案例与实战

### Tomcat

Tomcat 类库

* 放置在/common目录中。类库可被Tomcat和所有的Web应用程序共同使用。
* 放置在/server目录中。类库可被Tomcat使用，对所有的Web应用程序都不可见。
* 放置在/shared目录中。类库可被所有的Web应用程序共同使用，但对Tomcat自己不可见。
* 放置在/WebApp/WEB-INF目录中。类库仅仅可以被该Web应用程序使用，对Tomcat和其他Web应用程序都不可见。

自定义了多个类加载器

![](https://pic.imgdb.cn/item/6098b036d1a9ae528f37821f.jpg)



### OSGi

​	某个Bundle声明了一个它依赖的Package，如果有其他Bundle声明了发布这个Package后，那么所有对这个Package的类加载动作都会委派给发布它的Bundle类加载器去完成。不涉及某个具体的Package时，各个Bundle加载器都是平级的关系，只有具体使用到某个Package和Class的时候，才会根据Package导入导出定义来构造Bundle间的委派和依赖

### 字节码生成技术



```java
public class DynamicProxyTest {
	interface IHello {
    	void sayHello();
    }

    static class Hello implements IHello {
        @Override
        public void sayHello() {
            System.out.println("hello world");
        }
    }

    static class DynamicProxy implements InvocationHandler {

        Object originalObj;

        Object bind(Object originalObj) {
            this.originalObj = originalObj;
            return Proxy.newProxyInstance(originalObj.getClass().getClassLoader(), originalObj.getClass().getInterfaces(), this);
        }

        @Override
        public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
            System.out.println("welcome");
            return method.invoke(originalObj, args);
        }
    }

    public static void main(String[] args) {
        IHello hello = (IHello) new DynamicProxy().bind(new Hello());
        hello.sayHello();
    }
}
```
## 前端编译

* 前端编译器：JDK的Javac、Eclipse JDT中的增量式编译器（ECJ）
* 即时编译器：HotSpot虚拟机的C1、C2编译器，Graal编译器。
* 提前编译器：JDK的Jaotc、GNU Compiler for the Java（GCJ）[2]、Excelsior JET。



* 词法、语法分析
  * 词法分析
  * com.sun.tools.javac.parser.Scanner
  * 语法分析
    * com.sun.tools.javac.parser.Parser
  * 抽象语法树
  * com.sun.tools.javac.tree.JCTree
* 填充符号表
  * 一组符号地址和符号信息构成的数据结构。
  * com.sun.tools.javac.comp.Enter
* 注解处理器
  * 插入式注解处理器的初始化过程是在initPorcessAnnotations()方法中完成的，而它的执行过程则是在processAnnotations()方法中完成。
* 语义分析和字节码生成
  * 标注检查
    * 标注检查步骤要检查的内容包括诸如变量使用前是否已被声明、变量与赋值之间的数据类型是否能够匹配，等等。
  * 数据及控制流分析
    * 数据流分析和控制流分析是对程序上下文逻辑更进一步的验证，它可以检查出诸如程序局部变量在使用前是否有赋值、方法的每条路径是否都有返回值、是否所有的受查异常都被正确处理了等问题。
  * 解语法糖
    * 泛型。
    * 变长参数
    * 自动装箱拆箱。
  * 字节码生成
    * com.sun.tools.javac.jvm.Gen
    * 不仅仅是把前面各个步骤所生成的信息（语法树、符号表）转化成字节码指令写到磁盘中，编译器还进行了少量的代码添加和转换工作。
    * 实例构造器`<init>()`方法和类构造器`<clinit>()`方法就是在这个阶段被添加到语法树之中的。
    * 会把填充了所有所需信息的符号表交到com.sun.tools.javac.jvm.ClassWriter。



* 语法糖
  * 泛型
    * 伪泛型
    * 对于运行期的Java语言来说，`ArrayList<int>`与`ArrayList<String>`其实是同一个类型。
    * 无论在使用效果上还是运行效率上，几乎是全面落后于C#的具现化式泛型，而它的唯一优势是在于实现这种泛型的影响范围上：擦除式泛型的实现几乎只需要在Javac编译器上做出改进即可，不需要改动字节码、不需要改动Java虚拟机，也保证了以前没有使用泛型的库可以直接运行在Java 5.0之上。
    * 运行期无法取到泛型类型信息，会让一些代码变得相当啰嗦。
  * 装箱拆箱
  * 条件编译
    * 直接用if，会把用不到的去掉。

## 后端编译与优化

解释器与编译器两者各有优势：

* 当程序需要迅速启动和执行的时候，解释器可以首先发挥作用，省去编译的时间，立即运行。
* 当程序启动后，随着时间的推移，编译器逐渐发挥作用，把越来越多的代码编译成本地代码，这样可以减少解释器的中间损耗，获得更高的执行效率。

![](https://pic.imgdb.cn/item/6098ccdad1a9ae528f3eea7c.jpg)

* 分层编译
  * 第0层。程序纯解释执行，并且解释器不开启性能监控功能。
  * 第1层。使用客户端编译器将字节码编译为本地代码来运行，进行简单可靠的稳定优化，不开启性能监控功能。
  * 第2层。仍然使用客户端编译器执行，仅开启方法及回边次数统计等有限的性能监控功能。
  * 第3层。仍然使用客户端编译器执行，开启全部性能监控，除了第2层的统计信息外，还会收集如分支跳转、虚方法调用版本等全部的统计信息。
  * 第4层。使用服务端编译器将字节码编译为本地代码，相比起客户端编译器，服务端编译器会启用更多编译耗时更长的优化，还会根据性能监控信息进行一些不可靠的激进优化。

* 编译对象与处罚条件
  * 多次调用的方法。
  * 多次执行的循环体。
    * 热点探测
    * 基于采样的热点探测（Sample Based Hot Spot Code Detection）。
    * 基于计数器的热点探测（Counter Based Hot Spot Code Detection）。



* 方法内联
* 逃逸分析

  * 分析对象动态作用域，当一个对象在方法里面被定义后，它可能被外部方法所引用，例如作为调用参数传递到其他方法中，这种称为方法逃逸；甚至还有可能被外部线程访问到，譬如赋值给可以在其他线程中访问的实例变量，这种称为线程逃逸；从不逃逸、方法逃逸到线程逃逸，称为对象由低到高的不同逃逸程度。
* 栈上分配

  * 可以支持方法逃逸，但不能支持线程逃逸。
* 标量替换（Scalar Replacement）

  * 若一个数据已经无法再分解成更小的数据来表示了，Java虚拟机中的原始数据类型（int、long等数值类型及reference类型等）都不能再进一步分解了，那么这些数据就可以被称为标量。
  * 如果把一个Java对象拆散，根据程序访问的情况，将其用到的成员变量恢复为原始类型来访问，这个过程就称为标量替换。
  * 用户可以使用参数-XX：+EliminateAllocations来开启标量替换，使用参数-XX：+PrintEliminateAllocations查看标量的替换情况。
* 同步消除（Synchronization Elimination）

  * 使用+XX：+EliminateLocks来开启同步消除
  * 线程同步本身是一个相对耗时的过程，如果逃逸分析能够确定一个变量不会逃逸出线程，无法被其他线程访问，那么这个变量的读写肯定就不会有竞争，对这个变量实施的同步措施也就可以安全地消除掉。
* 逃逸分析

  * 可以使用参数-XX：+DoEscapeAnalysis来手动开启逃逸分析，开启之后可以通过参数-XX：+PrintEscapeAnalysis来查看分析结果。
* 公共子表达式消除
  * 如果一个表达式E之前已经被计算过了，并且从先前的计算到现在E中所有变量的值都没有发生变化，那么E的这次出现就称为公共子表达式。对于这种表达式，没有必要花时间再对它重新进行计算，只需要直接用前面计算过的表达式结果代替E。
  * 代数化简。
* 数组边界检查消除。

## Java内存模型与线程

* 缓存一致性。
* 主内存
  * 共享
* 工作内存
  * 线程私有

![image-20210510143452256](C:\Users\81929\AppData\Roaming\Typora\typora-user-images\image-20210510143452256.png)

**内存间交互操作**

* lock（锁定）：作用于主内存的变量，它把一个变量标识为一条线程独占的状态。
* unlock（解锁）：作用于主内存的变量，它把一个处于锁定状态的变量释放出来，释放后的变量才可以被其他线程锁定。
* read（读取）：作用于主内存的变量，它把一个变量的值从主内存传输到线程的工作内存中，以便随后的load动作使用。
* load（载入）：作用于工作内存的变量，它把read操作从主内存中得到的变量值放入工作内存的变量副本中。
* use（使用）：作用于工作内存的变量，它把工作内存中一个变量的值传递给执行引擎，每当虚拟机遇到一个需要使用变量的值的字节码指令时将会执行这个操作。
* assign（赋值）：作用于工作内存的变量，它把一个从执行引擎接收的值赋给工作内存的变量，每当虚拟机遇到一个给变量赋值的字节码指令时执行这个操作。
* store（存储）：作用于工作内存的变量，它把工作内存中一个变量的值传送到主内存中，以便随后的write操作使用。
* write（写入）：作用于主内存的变量，它把store操作从工作内存中得到的变量的值放入主内存的变量中。



规则

* 不允许read和load、store和write操作之一单独出现，即不允许一个变量从主内存读取了但工作内存不接受，或者工作内存发起回写了但主内存不接受的情况出现。
* 不允许一个线程丢弃它最近的assign操作，即变量在工作内存中改变了之后必须把该变化同步回主内存。
* 不允许一个线程无原因地（没有发生过任何assign操作）把数据从线程的工作内存同步回主内存中。
* 一个新的变量只能在主内存中“诞生”，不允许在工作内存中直接使用一个未被初始化（load或assign）的变量，换句话说就是对一个变量实施use、store操作之前，必须先执行assign和load操作。
* 一个新的变量只能在主内存中“诞生”，不允许在工作内存中直接使用一个未被初始化（load或assign）的变量，换句话说就是对一个变量实施use、store操作之前，必须先执行assign和load操作。
* 一个变量在同一个时刻只允许一条线程对其进行lock操作，但lock操作可以被同一条线程重复执行多次，多次执行lock后，只有执行相同次数的unlock操作，变量才会被解锁。
* 如果对一个变量执行lock操作，那将会清空工作内存中此变量的值，在执行引擎使用这个变量前，需要重新执行load或assign操作以初始化变量的值。
* 如果一个变量事先没有被lock操作锁定，那就不允许对它执行unlock操作，也不允许去unlock一个被其他线程锁定的变量。

**对于volatile型变量的特殊规则**

* 保证此变量对所有线程的可见性，这里的“可见性”是指当一条线程修改了这个变量的值，新值对于其他线程来说是可以立即得知的。
* 在不符合以下两条规则的运算场景中，我们仍然要通过加锁（使用synchronized、java.util.concurrent中的锁或原子类）来保证原子性：
  * 运算结果并不依赖变量的当前值，或者能够确保只有单一的线程修改变量的值。
  * 变量不需要与其他的状态变量共同参与不变约束。
* 禁止指令重排序

**双重锁校验**

https://www.cnblogs.com/nicori/p/11698240.html

**volatile特殊规则**

* 只有当线程T对变量V执行的前一个动作是load的时候，线程T才能对变量V执行use动作；并且，只有当线程T对变量V执行的后一个动作是use的时候，线程T才能对变量V执行load动作。线程T对变量V的use动作可以认为是和线程T对变量V的load、read动作相关联的，必须连续且一起出现。
  * 这条规则要求在工作内存中，每次使用V前都必须先从主内存刷新最新的值，用于保证能看见其他线程对变量V所做的修改。
* 只有当线程T对变量V执行的前一个动作是assign的时候，线程T才能对变量V执行store动作；并且，只有当线程T对变量V执行的后一个动作是store的时候，线程T才能对变量V执行assign动作。线程T对变量V的assign动作可以认为是和线程T对变量V的store、write动作相关联的，必须连续且一起出现。
  * 这条规则要求volatile修饰的变量不会被指令重排序优化，从而保证代码的执行顺序与程序的顺序相同。



**long和double**

* volatile不能保证原子性。
* 从JDK 9起，HotSpot增加了一个实验性的参数-XX：+AlwaysAtomicAccesses（这是JEP 188对Java内存模型更新的一部分内容）来约束虚拟机对所有数据类型进行原子性的访问。



* 原子性
  * 我们大致可以认为，基本数据类型的访问、读写都是具备原子性的（例外就是long和double的非原子性协定)。
* 可见性
  * 可见性就是指当一个线程修改了共享变量的值时，其他线程能够立即得知这个修改。
* 有序性
  * 如果在本线程内观察，所有的操作都是有序的；如果在一个线程中观察另一个线程，所有的操作都是无序的。



先行发生原则

​	先行发生是Java内存模型中定义的两项操作之间的偏序关系，比如说操作A先行发生于操作B，其实就是说在发生操作B之前，操作A产生的影响能被操作B观察到，“影响”包括修改了内存中共享变量的值、发送了消息、调用了方法等。

* 程序次序规则（Program Order Rule）在一个线程内，按照控制流顺序，书写在前面的操作先行发生于书写在后面的操作。注意，这里说的是控制流顺序而不是程序代码顺序，因为要考虑分支、循环等结构。
* 管程锁定规则（Monitor Lock Rule）：一个unlock操作先行发生于后面对同一个锁的lock操作。这里必须强调的是“同一个锁”，而“后面”是指时间上的先后。
* volatile变量规则（Volatile Variable Rule）：对一个volatile变量的写操作先行发生于后面对这个变量的读操作，这里的“后面”同样是指时间上的先后。
* 线程启动规则（Thread Start Rule）：Thread对象的start()方法先行发生于此线程的每一个动作。
* 线程终止规则（Thread Termination Rule）：线程中的所有操作都先行发生于对此线程的终止检测，我们可以通过Thread::join()方法是否结束、Thread::isAlive()的返回值等手段检测
* 线程中断规则（Thread Interruption Rule）：对线程interrupt()方法的调用先行发生于被中断线程的代码检测到中断事件的发生，可以通过Thread::interrupted()方法检测到是否有中断发生。
* 对象终结规则（Finalizer Rule）：一个对象的初始化完成（构造函数执行结束）先行发生于它的finalize()方法的开始。
* 传递性（Transitivity）：如果操作A先行发生于操作B，操作B先行发生于操作C，那就可以得出操作A先行发生于操作C的结论。



### 线程

实现线程方式

* 使用内核线程实现（1：1实现）
* 使用用户线程实现（1：N实现）
* 使用用户线程加轻量级进程混合实现（N：M实现）

![](https://pic.imgdb.cn/item/6098d94cd1a9ae528fa9ebf6.jpg)

​	从JDK 1.3起，“主流”平台上的“主流”商用Java虚拟机的线程模型普遍都被替换为基于操作系统原生线程模型来实现，即采用1：1的线程模型。

​	在Solaris平台的HotSpot虚拟机，由于操作系统的线程特性本来就可以同时支持1：1（通过Bound Threads或Alternate Libthread实现）及N：M（通过LWP/Thread Based Synchronization实现）的线程模型，因此Solaris版的HotSpot也对应提供了两个平台专有的虚拟机参数，即-XX：+UseLWPSynchronization（默认值）和-XX：+UseBoundThreads来明确指定虚拟机使用哪种线程模型。

### 线程调度

* 写协同式（Cooperative Threads-Scheduling）线程调度
* 抢占式（Preemptive Threads-Scheduling）线程调度。
* Java使用的线程调度方式就是抢占式调度。

![](https://pic.imgdb.cn/item/6098db76d1a9ae528fbc769b.jpg)

**状态**

* 新建（New）：创建后尚未启动的线程处于这种状态。
* 运行（Runnable）：包括操作系统线程状态中的Running和Ready，也就是处于此状态的线程有可能正在执行，也有可能正在等待着操作系统为它分配执行时间。
* 无限期等待（Waiting）：处于这种状态的线程不会被分配处理器执行时间，它们要等待被其他线程显式唤醒。以下方法会让线程陷入无限期的等待状态：
  * 没有设置Timeout参数的Object::wait()方法；
  * 没有设置Timeout参数的Thread::join()方法；
  * LockSupport::park()方法。
* 限期等待（Timed Waiting）：处于这种状态的线程也不会被分配处理器执行时间，不过无须等待被其他线程显式唤醒，在一定时间之后它们会由系统自动唤醒。以下方法会让线程进入限期等待状态：
  * Thread::sleep()方法；
  * 设置了Timeout参数的Object::wait()方法；
  * 设置了Timeout参数的Thread::join()方法；
  * LockSupport::parkNanos()方法；
  * LockSupport::parkUntil()方法。
* 阻塞（Blocked）：线程被阻塞了，“阻塞状态”与“等待状态”的区别是“阻塞状态”在等待着获取到一个排它锁，这个事件将在另外一个线程放弃这个锁的时候发生；而“等待状态”则是在等待一段时间，或者唤醒动作的发生。在程序等待进入同步区域的时候，线程将进入这种状态。
* 结束（Terminated）：已终止线程的线程状态，线程已经结束执行。

![](https://pic.imgdb.cn/item/6098dbe6d1a9ae528fc03779.jpg)



​	内核线程的调度成本主要来自于用户态与核心态之间的状态转换，而这两种状态转换的开销主要来自于响应中断、保护和恢复执行现场的成本。

​	由于最初多数的用户线程是被设计成协同式调度（Cooperative Scheduling）的，所以它有了一个别名——“协程”（Coroutine）。又由于这时候的协程会完整地做调用栈的保护、恢复工作，所以今天也被称为“有栈协程”（Stackfull Coroutine），起这样的名字是为了便于跟后来的“无栈协程”（Stackless Coroutine）区分开。

​	协程的主要优势是轻量，无论是有栈协程还是无栈协程，都要比传统内核线程要轻量得多。如果进行量化的话，那么如果不显式设置-Xss或-XX：ThreadStackSize，则在64位Linux上HotSpot的线程栈容量默认是1MB，此外内核数据结构（Kernel Data Structures）还会额外消耗16KB内存。与之相对的，一个协程的栈通常在几百个字节到几KB之间，所以Java虚拟机里线程池容量达到两百就已经不算小了，而很多支持协程的应用中，同时并存的协程数量可数以十万计。

​	协程当然也有它的局限，需要在应用层面实现的内容（调用栈、调度器这些）特别多。

**Java解决方案**

​	对于有栈协程，有一种特例实现名为纤程（Fiber），这个词最早是来自微软公司，后来微软还推出过系统层面的纤程包来方便应用做现场保存、恢复和纤程调度。

​	在新并发模型下，一段使用纤程并发的代码会被分为两部分——执行过程（Continuation）和调度器（Scheduler）。执行过程主要用于维护执行现场，保护、恢复上下文状态，而调度器则负责编排所有要执行的代码的顺序。将调度程序与执行过程分离的好处是，用户可以选择自行控制其中的一个或者多个，而且Java中现有的调度器也可以被直接重用。事实上，Loom中默认的调度器就是原来已存在的用于任务分解的Fork/Join池（JDK 7中加入的ForkJoinPool）。

## 线程安全与锁优化

可以将Java语言中各种操作共享的数据分为以下五类

* 不可变
  * final
* 绝对线程安全
  * 不管运行时环境如何，调用者都不需要任何额外的同步措施
* 相对线程安全
  * 保证对这个对象单次的操作是线程安全的，我们在调用的时候不需要进行额外的保障措施，但是对于一些特定顺序的连续调用，就可能需要在调用端使用额外的同步手段来保证调用的正确性。
* 线程兼容
  * 线程兼容是指对象本身并不是线程安全的，但是可以通过在调用端正确地使用同步手段来保证对象在并发环境中可以安全地使用。
* 线程对立
  * 不管调用端是否采取了同步措施，都无法在多线程环境中并发使用代码。

**线程安全实现**

* 互斥同步
  * synchronized
  * 重入锁
    * 等待可中断
      * 是指当持有锁的线程长期不释放锁的时候，正在等待的线程可以选择放弃等待，改为处理其他事情。可中断特性对处理执行时间非常长的同步块很有帮助。
    * 公平锁
      * 是指多个线程在等待同一个锁时，必须按照申请锁的时间顺序来依次获得锁；而非公平锁则不保证这一点，在锁被释放时，任何一个等待锁的线程都有机会获得锁。synchronized中的锁是非公平的，ReentrantLock在默认情况下也是非公平的，但可以通过带布尔值的构造函数要求使用公平锁。不过一旦使用了公平锁，将会导致ReentrantLock的性能急剧下降，会明显影响吞吐量。
    * 锁绑定多个条件
      * 指一个ReentrantLock对象可以同时绑定多个Condition对象。在synchronized中，锁对象的wait()跟它的notify()或者notifyAll()方法配合可以实现一个隐含的条件，如果要和多于一个的条件关联的时候，就不得不额外添加一个锁；而ReentrantLock则无须这样做，多次调用newCondition()方法即可。



* synchronized是在Java语法层面的同步，足够清晰，也足够简单。每个Java程序员都熟悉synchronized，但J.U.C中的Lock接口则并非如此。因此在只需要基础的同步功能时，更推荐synchronized。
* Lock应该确保在finally块中释放锁，否则一旦受同步保护的代码块中抛出异常，则有可能永远不会释放持有的锁。这一点必须由程序员自己来保证，而使用synchronized的话则可以由Java虚拟机来确保即使出现异常，锁也能被自动释放。
* 尽管在JDK 5时代ReentrantLock曾经在性能上领先过synchronized，但这已经是十多年之前的胜利了。从长远来看，Java虚拟机更容易针对synchronized来进行优化，因为Java虚拟机可以在线程和对象的元数据中记录synchronized中锁的相关信息，而使用J.U.C中的Lock的话，Java虚拟机是很难得知具体哪些锁对象是由特定线程锁持有的。

**非阻塞同步**

​	互斥同步面临的主要问题是进行线程阻塞和唤醒所带来的性能开销，因此这种同步也被称为阻塞同步（Blocking Synchronization）。

​	将会导致用户态到核心态转换、维护锁计数器和检查是否有被阻塞的线程需要被唤醒等开销。

​	基于冲突检测的乐观并发策略，通俗地说就是不管风险，先进行操作，如果没有其他线程争用共享数据，那操作就直接成功了；如果共享的数据的确被争用，产生了冲突，那再进行其他的补偿措施，最常用的补偿措施是不断地重试，直到出现没有竞争的共享数据为止。这种乐观并发策略的实现不再需要把线程阻塞挂起，因此这种同步操作被称为非阻塞同步（Non-Blocking Synchronization），使用这种措施的代码也常被称为无锁（Lock-Free）编程。

* 测试并设置（Test-and-Set）；
* 获取并增加（Fetch-and-Increment）；
* 交换（Swap）；
* 比较并交换（Compare-and-Swap，下文称CAS）；
* 加载链接/条件储存（Load-Linked/Store-Conditional，下文称LL/SC）。



​	存在ABA问题，加一个version即可。AtomicStampedReference。

**无同步方案**

​	略

### 锁优化

* 适应性自旋。

  * 默认10次
  * -XX：PreBlockSpin来自行更改。

* 锁消除。

  * 指虚拟机即时编译器在运行时，对一些代码要求同步，但是对被检测到不可能存在共享数据竞争的锁进行消除。

* 锁粗化。

  * 反复加锁的情况，粗化到循环外面。

* 轻量级锁。

  * 拟机将使用CAS操作尝试把对象的Mark Word更新为指向Lock Record的指针。如果这个更新动作成功了，即代表该线程拥有了这个对象的锁，并且对象Mark Word的锁标志位（Mark Word的最后两个比特）将转变为“00”，表示此对象处于轻量级锁定状态。

* 偏向锁

  * 目的是消除数据在无竞争情况下的同步原语，进一步提高程序的运行性能。
  * 在无竞争的情况下把整个同步都消除掉，连CAS操作都不去做了。
  * 那么当锁对象第一次被线程获取的时候，虚拟机将会把对象头中的标志位设置为“01”、把偏向模式设置为“1”，表示进入偏向模式。同时使用CAS操作把获取到这个锁的线程的ID记录在对象的Mark Word之中。如果CAS操作成功，持有偏向锁的线程以后每次进入这个锁相关的同步块时，虚拟机都可以不再进行任何同步操作（例如加锁、解锁及对Mark Word的更新操作等）。
  * 一旦出现另外一个线程去尝试获取这个锁的情况，偏向模式就马上宣告结束。根据锁对象目前是否处于被锁定的状态决定是否撤销偏向（偏向模式设置为“0”），撤销后标志位恢复到未锁定（标志位为“01”）或轻量级锁定（标志位为“00”）的状态，后续的同步操作就按照上面介绍的轻量级锁那样去执行。
  * 因此，当一个对象已经计算过一致性哈希码后，它就再也无法进入偏向锁状态了；而当一个对象当前正处于偏向锁状态，又收到需要计算其一致性哈希码请求[1]时，它的偏向状态会被立即撤销，并且锁会膨胀为重量级锁。
  * 有时候使用参数-XX：-UseBiasedLocking来禁止偏向锁优化反而可以提升性能。

  

