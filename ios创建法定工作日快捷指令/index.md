# iOS创建法定工作日快捷指令


<!--more-->
- 需求

iOS上设置闹钟没办法按照国家法定节假日来设置，网上搜索了很多办法，都不是很好，最后发现使用快捷指令这个可以做到，就学习了一下。
- 解决过程

先查找了一下官方文档，学习了一下基本的操作流程，发现这个和程序开发的流程特别像，可以学习一下，锻炼自己的编程思维。
[快捷指令官方文档](https://support.apple.com/zh-cn/guide/shortcuts/welcome/ios)

- 操作步骤

1. 设置一个每天的响的闹钟，闹钟标签改为“工作日”

   ![创建闹钟](https://cdn.jsdelivr.net/gh/wlight/cdn-images/blog-images/创建闹钟.jpg)

2. 下载快捷指令这个官方的app，一般手机可能没有这个app

3. 打开快捷指令app，点击右上角加号，添加一个新的快捷指令

   ![创建快捷指令1](https://cdn.jsdelivr.net/gh/wlight/cdn-images/blog-images/创建快捷指令1.jpg)

4. 添加响应的操作即可，大概的操作流程为，获取到当前日期，然后请求是否为节假日的api，根据返回的结果是否为节假日，来决定是否关闭闹钟。

   ![img-1](https://cdn.jsdelivr.net/gh/wlight/cdn-images/blog-images/快捷指令详情.JPEG)

5. 保存为"法定工作日闹钟"名称

6. 让“快捷指令”每天自动化后台运行，可以根据勿扰模式打开来自动化运行，比如我自己设置的每天11：00～7：00自动开关勿扰模式，即可设置自动化开启勿扰模式时静默执行此快捷指令
   1. 打开快捷指令app的第二项“自动化”，点击“创建个人自动化”

      ![自动化1](https://cdn.jsdelivr.net/gh/wlight/cdn-images/blog-images/自动化1.jpg)

   2. 点击“勿扰模式”

      ![自动化2](https://cdn.jsdelivr.net/gh/wlight/cdn-images/blog-images/自动化2.jpg)![自动化3](https://cdn.jsdelivr.net/gh/wlight/cdn-images/blog-images/自动化3.jpg)

   3. 当勿扰模式打开时运行“快捷指令app”里的“法定工作日闹钟”

      ![自动化4](https://cdn.jsdelivr.net/gh/wlight/cdn-images/blog-images/自动化4.jpg)

   4. 关闭运行前询问，这样才可以在后台运行

      ![自动化5](https://cdn.jsdelivr.net/gh/wlight/cdn-images/blog-images/自动化5.jpg)

7. 完成，这样就可以享受节假日闹钟了


> **提示：**
>
> **一定要将需要控制的闹钟名称改为“工作日”，运行此快捷指令即可根据当前日期开关对应闹钟！**
>
> **可配合个性化使用，比如我自己设置的每天11：00～7：00自动开关勿扰模式，即可设置自动化开启勿扰模式时静默执行此快捷指令。**
