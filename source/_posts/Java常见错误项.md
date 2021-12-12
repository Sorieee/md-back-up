# 削减拆箱

![](https://pic.imgdb.cn/item/61b4ab092ab3f51d91184486.jpg)

# 削减indexOf

![](https://pic.imgdb.cn/item/61b4ab962ab3f51d91189b01.jpg)

# 尽量使用ComputeIfAbsent

​	HashMap会做优化，而不是get后null再push。

* 清晰
* 不会查找两次位置。

# 内部类

​	内部类会带有一个对外部的引用，尽量带static，少用一个槽。

# 用isBlank

​	而不是trim().length() == 0

# Iterable的iterator不要返回null

可以返回

![](https://pic.imgdb.cn/item/61b55fa82ab3f51d9160cc65.jpg)

# Logger系接口，如果要传reason，message就不能使用占位

![](https://pic.imgdb.cn/item/61b560302ab3f51d9160fd3a.jpg)

![](https://pic.imgdb.cn/item/61b560422ab3f51d91610288.jpg)

# 优化Map创建

![](https://pic.imgdb.cn/item/61b56bd12ab3f51d91652e14.jpg)

# 使用System.currentTimeMillis

而不是new Date().getTime();

# instanceof 不用判断null

![](https://pic.imgdb.cn/item/61b5760a2ab3f51d9168ce7d.jpg)

2:58

