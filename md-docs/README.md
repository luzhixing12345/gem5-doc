# gem5

## 前言

gem5 官方提供了[文档](https://www.gem5.org/documentation/), 但是笔者个人认为不尽人意, 故留此文以作学习笔记和使用过程记录

如果您遇到相关 gem5 使用上的问题, 建议您优先在 [gem5-user-email](https://www.gem5.org/mailing_lists/) 中询问

因为本文档是关于体系结构的一个较为复杂的软件 gem5 的使用文档, 因此假定读者已经 **具备扎实的体系结构基础和一定程度的C++和python编程能力**, 一些较为基础的知识概念就不再赘述了.鉴于笔者水平有限, 文中难免出现语焉不详之处, 望读者海涵. 欢迎您在 [issue](https://github.com/luzhixing12345/gem5/issues) 中留言指出文中不当之处

## 文档内容

本文档用于记录三部分内容

- gem5的编译和使用
- gem5源码阅读
- cxl的gem5实现

考虑到不同人使用 gem5 的目标不尽相同, 有可能涉及 linux kernel 的修改与测试, 亦或者 arch/mem/clock/cache/device 等等层面的仿真和设计, **本文仅关注 gem5 使用层面的问题**. 

`gem5的编译和使用` 介绍了 gem5 的基本使用方法, 在不大幅修改源码的情况下如何使用已有的 gem5 模块完成常规任务

`gem5源码阅读` 介绍了 gem5 各个模块的相关代码实现, 如果您需要修改源码理解 gem5 整体系统架构请参考阅读

`cxl的gem5实现` 介绍了笔者所在课题组的研究方向 CXL 的相关实现细节, 如果您对此不感兴趣忽略即可

下面的参考链接是一些中文互联网 gem5 相关学习资源, 您可以参考作为扩展补充资料

## 代码参考

- [gem5-cxl实现](https://github.com/fadedzipper/gem5/tree/cxl.mem-dev)
- [gem5-cxl](https://github.com/ferry-hhh/gem5)
- [中科大的gem-cxl实现](https://github.com/zxhero/gem5-CXL)
- [cxl驱动实现](https://github.com/zjthappy/cxl_mem_driver)

## gem5

- [Github gem5-bootcamp-env](https://github.com/gem5bootcamp/gem5-bootcamp-env)
- [gem5/videos](https://www.youtube.com/@gem5/videos)
- [youtube gem5-tutorial](https://www.youtube.com/@JLowePower)
- [gem5 Bootcamp 2022](https://gem5bootcamp.github.io/gem5-bootcamp-env/)
- [CSDN 好啊啊啊啊 Gem5模拟器专栏](https://blog.csdn.net/qq_46675545/category_12120503.html)
- [CSDN ivy_reny gem5专栏](https://blog.csdn.net/ivy_reny/category_6666068.html)
- [CSDN 小张的学习手册 gem5专栏](https://blog.csdn.net/drinks_/category_11071651.html)
- [CSDN wyj7260 gem5专栏](https://blog.csdn.net/wyj7260/category_1301132.html)
- [CSDN 人间之清欢 GEM5专栏](https://blog.csdn.net/qq_43381135/category_9731557.html)
- 国防科大实验室

  - [微信公众号文章 gem5学习笔记一:模拟器安装与使用](https://mp.weixin.qq.com/s/VoPI0jbAERg8VK98oVff4Q)
  - [微信公众号文章 gem5学习笔记二:全系统模拟浅析](https://mp.weixin.qq.com/s/ru61jLFNQ503Z0XMR5NvaQ)
  - [微信公众号文章 gem5学习笔记三:创建模拟系统](https://mp.weixin.qq.com/s/K8BIZb25BGVgwQG2knBhsg)

## CXL

- [CSDN 空空7 CXL扩展内存专栏](https://blog.csdn.net/qq_45726331/category_12220993.html)
- [gem5-user email 有关CXL的讨论](https://www.mail-archive.com/search?q=CXL&l=gem5-users%40gem5.org)

## 其他

- [DMA](https://en.wikipedia.org/wiki/Direct_memory_access)
- [DMA 如何与 PCI Express 设备一起工作?](https://stackoverflow.com/questions/27470885/how-does-dma-work-with-pci-express-devices)