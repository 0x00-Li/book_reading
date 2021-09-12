https://www.cnblogs.com/trgl/p/7353782.html

![](https://upload-images.jianshu.io/upload_images/6912735-51aa162747fcdc3d.png?imageMogr2/auto-orient/strip)

- 第一部分进行SpringApplication的初始化模块，配置一些基本的环境变量、资源、构造器、监听器
- 第二部分实现了应用具体的启动方案，包括启动流程的监听模块、加载配置环境模块、及核心的创建上下文环境模块
- 第三部分是自动化配置模块，该模块作为springboot自动配置核心，在后面的分析中会详细讨论。在下面的启动程序中我们会串联起结构中的主要功能

