# 解决GitHub clone repo太慢的问题


### 引言

今天拉取自己GitHub上的repo，发现特别特别的慢，搜索了各种方法，例如：修改Host文件，感觉明显不靠谱。自己一直在使用代理，感觉还是得通过代理来解决，最后功夫不负有心人，通过了代理的方式解决了这个一直以来的问题。

> 注：前提要有梯子，我是使用了池大推荐的一个梯子：[Ageneo](https://ageneo.org/)，一直感觉挺好用的，

### 解决办法

我的代理客户端使用的是：clash for windows，开始想着切换到全局模式，git 应该就快了，但 clone 依然很慢，搜索后发现，git 命令并不会直接走全局代理，而是走自己默认的配置，修改这个配置的方法如下：

```bash
git config --global http.proxy <http://127.0.0.1:7890>
git config --global https.proxy <https://127.0.0.1:7890>
```

端口号获取的方式如下：

Windows中打开”网络和 Internet“→"代理"→"手动设置代理"，查看代理端口号，上面的命令修改为自己的端口号即可。

![windows_config](https://raw.githubusercontent.com/wlight/cdn-images/main/blog-images/windows_config.png)

设置好以后就可以clone了，这次速度上来了，特别快。

```bash
PS D:\\study\\go\\hugo\\sites\\blog> git submodule add <https://github.com/dillonzq/LoveIt.git> .\\themes\\LoveIt
Cloning into 'D:/study/go/hugo/sites/blog/.\\themes\\LoveIt'...
remote: Enumerating objects: 9952, done.
remote: Total 9952 (delta 0), reused 0 (delta 0), pack-reused 9952R
Receiving objects: 100% (9952/9952), 36.96 MiB | 1.83 MiB/s, done.
Resolving deltas: 100% (4953/4953), done.
warning: LF will be replaced by CRLF in .gitmodules.
The file will have its original line endings in your working directory
```
