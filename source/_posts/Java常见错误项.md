# 用Short接收强转结果

## 削减拆箱

![](https://pic.imgdb.cn/item/61b4ab092ab3f51d91184486.jpg)

## 削减indexOf

![](https://pic.imgdb.cn/item/61b4ab962ab3f51d91189b01.jpg)

## 尽量使用ComputeIfAbsent

​	HashMap会做优化，而不是get后null再push。

* 清晰
* 不会查找两次位置。

## 内部类

​	内部类会带有一个对外部的引用，尽量带static，少用一个槽。



