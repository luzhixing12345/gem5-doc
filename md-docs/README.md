# gem5

## 前言

gem5 官方提供了[文档](https://www.gem5.org/documentation/), 但是笔者个人认为不尽人意, 故留此文以作学习笔记和使用过程记录

## 文档内容

本文档用于记录三部分内容

- gem5的编译和使用
- gem5源码阅读

考虑到不同人使用 gem5 的目标不尽相同, 有可能涉及 linux kernel 的修改与测试, 亦或者 arch/mem/clock/cache/device 等等层面的仿真和设计, **本文仅关注 gem5 使用层面的问题**. 

`gem5的编译和使用` 介绍了 gem5 的基本使用方法, 在不大幅修改源码的情况下如何使用已有的 gem5 模块完成常规任务

`gem5源码阅读` 介绍了 gem5 各个模块的相关代码实现, 如果您需要修改源码理解 gem5 整体系统架构请参考阅读

下面的参考链接是一些中文互联网 gem5 相关学习资源, 您可以参考作为扩展补充资料

## 参考

- [Github gem5-bootcamp-env](https://github.com/gem5bootcamp/gem5-bootcamp-env)
- [gem5/videos](https://www.youtube.com/@gem5/videos)
- [youtube gem5-tutorial](https://www.youtube.com/@JLowePower)
- [gem5 Bootcamp 2022](https://gem5bootcamp.github.io/gem5-bootcamp-env/)
- [好啊啊啊啊 Gem5模拟器专栏](https://blog.csdn.net/qq_46675545/category_12120503.html)
- [ivy_reny gem5专栏](https://blog.csdn.net/ivy_reny/category_6666068.html)
- [小张的学习手册 gem5专栏](https://blog.csdn.net/drinks_/category_11071651.html)
- [wyj7260 gem5专栏](https://blog.csdn.net/wyj7260/category_1301132.html)
- [人间之清欢 GEM5专栏](https://blog.csdn.net/qq_43381135/category_9731557.html)
- [gem5学习笔记一:模拟器安装与使用](https://mp.weixin.qq.com/s/VoPI0jbAERg8VK98oVff4Q)
- [gem5学习笔记二:全系统模拟浅析](https://mp.weixin.qq.com/s/ru61jLFNQ503Z0XMR5NvaQ)
- [gem5学习笔记三:创建模拟系统](https://mp.weixin.qq.com/s/K8BIZb25BGVgwQG2knBhsg)

## 其他

- [DMA](https://en.wikipedia.org/wiki/Direct_memory_access)
- [DMA 如何与 PCI Express 设备一起工作?](https://stackoverflow.com/questions/27470885/how-does-dma-work-with-pci-express-devices)