# Go Zore简明教程


<!--more-->

## 开发环境

- golang v1.15.1

- go module 开启
  - `go env GO111MODULE on`
- protoc v3.14
  - https://github.com/protocolbuffers/protobuf/releases
- protoc-gen-go 1.3.2
  -  ` $ go get -u github.com/golang/protobuf/protoc-gen-go@v1.3.2`

- etcd
- redis
- mysql

## 文档

- [简介 · go-zero使用文档 (zeromicro.github.io)](https://zeromicro.github.io/go-zero/)

- [最简单的 Go Dockerfile 编写姿势，没有之一！ · GoCN社区](https://gocn.vip/topics/11370)
- [从代码到部署微服务实战（一）](https://mp.weixin.qq.com/s/jMsL464stSRctj2OLjOZBg)


## 项目开发流程

目录拆分

model生成

api文件编写

业务编码

jwt鉴权

中间件使用

rpc文件编写

错误处理

## 开发规范

#### 编码规范

##### import

- 单行 import 不建议用圆括号包括

- 按照 `官方包`空行  `当前工程包` 空行  `第三方依赖包`

  ```go
  import (
  	"context"
      "string"
      
      "greet/user/internal/config"
      
       "google.golang.org/grpc"
  )
  ```

##### 函数返回

- 对象避免非指针返回
- 遵循有正常值返回则一定无 error，有 error 则一定无正常值返回的原则

##### 错误处理

- 有error必须处理，如果不能处理就必须抛出
- 避免下划线(_)忽略 error

## 构建docker运行

```bash
docker build -t user:v1 -f service/user/cmd/api/Dockerfile .
```

删除原本的有问题的镜像，`-f` 是强制删除及其关联状态

若不执行 `-f`，你需要执行 `docker ps -a` 查到所关联的容器，将其 `rm` 解除两者依赖关系

```bash
docker run --rm -it -p 8888:8888 user:v1
```

## 注意事项

```bash
nohup ./etcd >/tmp/etcd.log 2>&1 & # 日志文件输出到/tmp/etcd.log目录 
```

## git

> [Go Modules 入门教程 | Go 技术论坛 (learnku.com)](https://learnku.com/go/t/38809)



为了标识身份，建议先完成git全局配置

```shell
git config --global user.name "username"
git config --global user.email "user@mail.com"
```

方式1：克隆仓库

```shell
git clone git@codeup.aliyun.com:5f61bcb1df9df74e36b01a89/testmod.git
cd testmod
touch README.md
git add README.md
git commit -m "add README"
git push -u origin master
```

方式2：已有文件夹或仓库

```shell
cd existing_folder
git init
git remote add origin git@codeup.aliyun.com:5f61bcb1df9df74e36b01a89/testmod.git
git add .
git commit
git push -u origin master
```

方式3：导入代码库

```shell
git clone --bare https://git.example.com/your/project.git your_path
cd your_path
git remote set-url origin git@codeup.aliyun.com:5f61bcb1df9df74e36b01a89/testmod.git
git push --mirror
```


