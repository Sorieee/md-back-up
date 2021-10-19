# 源码方法论

* 不要忽略源码的注释
* 不要一来就深入细节，死扣一个方法，先梳理脉络，再看细节。
* 大胆猜测、大胆验证。
* 见名知意。



# IOC



![](https://pic.imgdb.cn/item/616d8ecd2ab3f51d910d8fd0.jpg)



## BeanFactory

![](https://pic.imgdb.cn/item/616d909b2ab3f51d910f72e7.jpg)

## Aware接口

​	方便通过SpringBean对象来获取容器中的相关属性值。

![](https://pic.imgdb.cn/item/616eca832ab3f51d91f61c06.jpg)

# 重要方法

AbsctractApplicationContext::refresh();

# 循环依赖

![](https://pic.imgdb.cn/item/616ed3ea2ab3f51d91005e6b.jpg)



Bean1级缓存

2级缓存

3级缓存。

DeafaultSingletonBeanFactory

