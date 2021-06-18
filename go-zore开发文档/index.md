# Go Zore开发文档


### 开发环境

* golang 1.16
* mysql 5.7
* redis
* etcd (多服务时使用，服务发现)
* go-zore 框架 [官方文档](https://go-zero.dev/cn/)

### 项目开发

#### 创建数据库和数据表

####  框架初始化

- 目录拆分


- ```shell
  csp // 工程名称
  ├── common //通用库
  │   ├── errorx
  │   │   ├── base_error.go // 基础错误处理
  │   │   └── error_handler.go // 错误处理
  │   └── utils
  │       └── tools.go
  ├── go.mod // mod文件
  ├── go.sum
  └── service // 服务存放目录
      ├── material 
      │   ├── api
      │   ├── model
      │   └── rpc
      └── order // 
          ├── api // // HTTP访问服务，业务需求实现
          │   ├── cmd
          │   │   ├── etc  // 配置文件
          │   │   │   └── order-api.yaml
          │   │   ├── internal
          │   │   │   ├── config
          │   │   │   │   └── config.go  // 配置声明type
          │   │   │   ├── handler // 路由及handler转发
          │   │   │   │   ├── routes.go
          │   │   │   │   └── v1
          │   │   │   │       ├── create_order_handler.go
          │   │   │   │       ├── del_orders_handler.go
          │   │   │   │       ├── get_orders_detail_handler.go
          │   │   │   │       ├── get_orders_handler.go
          │   │   │   │       └── update_orders_handler.go
          │   │   │   ├── logic // 业务逻辑
          │   │   │   │   └── v1
          │   │   │   │       ├── create_order_logic.go
          │   │   │   │       ├── del_orders_logic.go
          │   │   │   │       ├── get_orders_detail_logic.go
          │   │   │   │       ├── get_orders_logic.go
          │   │   │   │       └── update_orders_logic.go
          │   │   │   ├── middleware // 中间件文件
          │   │   │   │   └── checkauth_middleware.go
          │   │   │   ├── svc // logic所依赖的资源池
          │   │   │   │   └── service_context.go
          │   │   │   └── types // request、response的struct，根据api自动生成，不建议编辑
          │   │   │       └── types.go
          │   │   └── order.go // main函数入口
          │   └── order.api // api描述文件
          ├── model  // model层，直接操作数据库，服务访问持久化数据层的桥梁
          │   ├── order_datas_model.go
          │   ├── order_details_model.go
          │   └── vars.go
          └── rpc // rpc服务，给其他子系统提供基础数据访问
  ```


#### model 生成

##### **修改 goctl 生成模板**

  - goctl 只能默认用户在建表时会创建createTime、updateTime字段(忽略大小写、下划线命名风格)且默认值均为`CURRENT_TIMESTAMP`，而updateTime支持`ON UPDATE CURRENT_TIMESTAMP`，对于这两个字段生成`insert`、`update`时会被移除，不在赋值范畴内，当然，如果你不需要这两个字段那也无大碍。但是我们数据库表里使用的是`created_at` 和 `updated_at`，所以我们要修改`goctl`的生成模板。

    - 查找`goctl`的源代码的位置 `$GOPATH/pkg/mod/github.com/tal-tech/go-zero@$VERSION/tools/goctl`
    - 查找 `createTime`，然后替换成 `CreatedAt` 和 `UpdatedAt`，修改以下文件 :
      - `$GOPATH/pkg/mod/github.com/tal-tech/go-zero@v1.1.7/tools/goctl/model/sql/template/vars.go`
      - `$GOPATH/pkg/mod/github.com/tal-tech/go-zero@v1.1.7/tools/goctl/model/sql/gen/update.go`
      - `$GOPATH/pkg/mod/github.com/tal-tech/go-zero@v1.1.7/tools/goctl/model/sql/gen/insert.go`
      - `$GOPATH/pkg/mod/github.com/tal-tech/go-zero@v1.1.7/tools/goctl/model/sql/parser/parser.go`
    - 重新编译成运行工具：
      - `cd  $GOPATH/pkg/mod/github.com/tal-tech/go-zero@v1.1.7/tools/goctl/`
      - `go build`
      - `mv goctl $GOPATH/bin/`

Model 生成方式有以下两种方式：

切换到model文件夹中：

`cd csp/service/order/model`

##### 1、ddl 生成

- `goctl model mysql ddl -src user.sql -dir . -c`

##### 2、**datacource 生成**

- `goctl model mysql datasource -url="$username:$password@tcp($host)" -table="$tablename" -c -dir . -style go_zero`
- 格式化符有`go`,`zero`组成，如常见的三种格式化风格你可以这样编写：
  - lower: `gozero`
  - camel: `goZero`
  - snake: `go_zero`

##### **注意事项**

- -c 生成带缓存，目前仅支持 redis 缓存，如果选择带缓存模式，即生成的FindOne(ByXxx)&Delete代码会生成带缓存逻辑的代码，目前仅支持单索引字段（除全文索引外），对于联合索引我们默认认为不需要带缓存，且不属于通用型代码，因此没有放在代码生成行列，如example中user表中的id、name、mobile字段均属于单字段索引。


- model事务（**TODO：只能在单个model里写事务，感觉很难受啊，暂时没有找到跨model的办法**）

  - ```go
    var insertsql = `insert into User(uid, username, mobilephone) values (?, ?, ?)`
    err := usermodel.conn.Transact(func(session sqlx.Session) error {
        stmt, err := session.Prepare(insertsql)
        if err != nil {
            return err
        }
        defer stmt.Close()
    
        // 返回任何错误都会回滚事务
        if _, err := stmt.Exec(uid, username, mobilephone); err != nil {
            logx.Errorf("insert userinfo stmt exec: %s", err)
            return err
        }
    
        // 还可以继续执行 insert/update/delete 相关操作
        return nil
    })
    ```

  - **事务** 中的操作都包装在一个函数 `func(session sqlx.Session) error {}` 中即可，如果事务中的操作返回任何错误， `Transact()` 都会自动回滚事务。

####  api 文件编写

- 生成 api 文件：
- `cd csp/service/order/api`
- `goctl api -o data.api`
- 编写 api 文件，[api语法介绍](https://go-zero.dev/cn/api-grammar.html)

#### api 文件生成 api 服务

- ```shell
  cd csp/service/data/api
  // -style 编程代码文件风格，有三种风格，也就是有三种值：gozero(全小写，默认)；goZero/Gozero（驼峰）；go_zero（下划线）
  goctl api go -api data.api -dir . -style go_zero
  ```

### 初始化配置

#### 添加 Mysql 和 Redis 配置

- `vim csp/service/order/api/cmd/internal/config/config.go`

- ```go
  package config
  
  import (
  	"github.com/tal-tech/go-zero/core/stores/cache"
  	"github.com/tal-tech/go-zero/rest"
  )
  
  type Config struct {
  	rest.RestConf
  	Mysql struct{
  		DataSource string
  	}
  	CacheRedis cache.CacheConf
  }
  ```

#### 完善 yaml 配置

- `vim csp/service/order/api/cmd/etc/data-api.yaml`

- ```yaml
  Name: data-api
  Host: 0.0.0.0
  Port: 8888
  Mysql:
    DataSource: root:123456@tcp(127.0.0.1:3306)/ostar?charset=utf8mb4&parseTime=true&loc=Asia%2FShanghai
  CacheRedis:
    - Host: 127.0.0.1:6379
      Type: node
  
  ```

#### 完善服务依赖

- `vim csp/service/order/api/cmd/internal/svc/service_context.go`

- ```go
  package svc
  
  import (
  	"csp/service/order/api/cmd/internal/config"
  	"csp/service/order/api/cmd/internal/middleware"
  	"csp/service/order/model"
  	"github.com/tal-tech/go-zero/core/stores/sqlx"
  	"github.com/tal-tech/go-zero/rest"
  )
  
  type ServiceContext struct {
  	Config    config.Config
  	CheckAuth rest.Middleware
  	OrderDatasModel model.OrderDatasModel
  	OrderDetailsModel model.OrderDetailsModel
  }
  
  func NewServiceContext(c config.Config) *ServiceContext {
  	conn := sqlx.NewMysql(c.Mysql.DataSource)
  	return &ServiceContext{
  		Config:    c,
  		CheckAuth: middleware.NewCheckAuthMiddleware().Handle,
  		OrderDatasModel: model.NewOrderDatasModel(conn, c.CacheRedis),
  		OrderDetailsModel: model.NewOrderDetailsModel(conn, c.CacheRedis),
  	}
  }
  
  
  ```

#### 错误处理

##### 自定义错误

###### 全局错误处理

- 首先在common中添加一个全局错误处理error_handler.go文件

- ```shell
  mkdir -p common/errorx
  touch error_handler.go
  ```

  ```go
  package errorx
  
  import (
  	"github.com/tal-tech/go-zero/core/jsonx"
  	"net/http"
  )
  
  // ErrorHandler 处理自定义错误
  func ErrorHandler(err error) (int, interface{}) {
  	switch e := err.(type) {
  	case *CodeError:
  		return e.statusCode, e.Data()
  	default:
  		return http.StatusInternalServerError, CodeErrorResponse{
  			Code: defaultCode,
  			Message: err.Error(),
  		}
  	}
  }
  
  // NotFoundHandler 处理 404 错误
  func NotFoundHandler() http.Handler {
  	return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
  		w.WriteHeader(http.StatusNotFound)
  		w.Header().Set("Content-Type", "text/json; charset=utf-8")
  		msg, _ := jsonx.Marshal(CodeErrorResponse{Code: http.StatusNotFound, Message: "not found"})
  		w.Write(msg)
  	})
  }
  
  ```

###### 基础错误文件

- ```sheel
  touch base_error.go
  ```

- ```go
  package errorx
  
  import (
  	"net/http"
  )
  
  const (
  	defaultCode = 000
  	// OrdersModule 数据中心功能模块(编号区间 301 ~ 350)
  	OrdersModule = 301 + iota // 订单模块编号
  	MaterialsModule
  )
  
  var (
  //ErrorDataNotFound = errors.New("data not found")
  //ErrorUserNotFound = errors.New("用户不存在")
  //ErrorNoRequiredParameters = errors.New("必要参数不能为空")
  //ErrorUserOperation = errors.New("用户正在操作中，请稍后重试")
  //ErrorServer = NewCodeError("系统错误，请联系管理员")
  //ErrorMysqlOperation = NewCodeError(350000, "数据库操作错误")
  )
  
  /*
  HTTP Status状态码要求与实际的状态码一致
  code：6位（前3位为功能模块编号(moduleCode)，后3位为业务错误编号(bizCode)）
  message：返回成功或者失败或者异常信息
  data：返回的数据信息（使用小驼峰命名法作为属性标识符）
   */
  
  type CodeError struct {
  	statusCode int // http status code
  	moduleCode int // 模块号码
  	bizCode int // 业务错误号码
  	message string // 信息
  }
  
  type CodeErrorResponse struct {
  	Code int `json:"code"`
  	Message string `json:"message"`
  }
  
  func NewCodeError(httpStatusCode, moduleCode, bizCode int, message string) error {
  	return &CodeError{statusCode: httpStatusCode,moduleCode: moduleCode, bizCode: bizCode, message: message}
  }
  
  
  // NewDefaultError TODO 默认错误
  func NewDefaultError(message string) error {
  	return NewCodeError(http.StatusInternalServerError, http.StatusInternalServerError * 1000, defaultCode, message)
  }
  
  // NotFindDataError 未查找到数据返回错误
  func NotFindDataError(moduleCode, bizCode int,message string) error {
  	return NewCodeError(http.StatusNotFound, moduleCode, bizCode, message)
  }
  
  func (e *CodeError) Error() string {
  	return e.message
  }
  
  func (e *CodeError) Data() *CodeErrorResponse {
  	return &CodeErrorResponse{
  		Code: e.moduleCode * 1000 + e.bizCode,
  		Message: e.message,
  	}
  }
  ```

###### 开启自定义错误

- ```shell
  vim service/user/cmd/api/user.go
  ```

- ```go
  package main
  
  import (
  	"csp/common/errorx"
  	"flag"
  	"fmt"
  	"github.com/tal-tech/go-zero/rest/httpx"
  
  	"csp/service/data/cmd/api/internal/config"
  	"csp/service/data/cmd/api/internal/handler"
  	"csp/service/data/cmd/api/internal/svc"
  
  	"github.com/tal-tech/go-zero/core/conf"
  	"github.com/tal-tech/go-zero/rest"
  )
  
  var configFile = flag.String("f", "etc/data-api.yaml", "the config file")
  
  func main() {
  	flag.Parse()
  
  	var c config.Config
  	conf.MustLoad(*configFile, &c)
  
  	ctx := svc.NewServiceContext(c)
  	// 自定义 404 错误返回
  	server := rest.MustNewServer(c.RestConf, rest.WithNotFoundHandler(errorx.NotFoundHandler()))
  	defer server.Stop()
  	// 自定义错误返回
  	httpx.SetErrorHandler(errorx.ErrorHandler)
  
  	handler.RegisterHandlers(server, ctx)
  
  	fmt.Printf("Starting server at %s:%d...\n", c.Host, c.Port)
  	server.Start()
  }
  ```

- 然后逻辑中错误使用CodeError自定义错误替换

- ```go
  return nil, errorx.NewDefaultError("用户名密码不正确")
  ```

#### 日志配置

##### api 日志配置

- `vim etc/order-api.yaml`

- ```yaml
  Name: order-api
  Host: 0.0.0.0
  Port: 8888
  // 数据库配置
  Mysql:
    DataSource: root:123456@tcp(127.0.0.1:3306)/ostar?charset=utf8mb4&parseTime=true&loc=Asia%2FShanghai
  // redis 配置
  CacheRedis:
    - Host: 127.0.0.1:6379
      Type: node
  // 日志配置
  Log:
    ServiceName: order
    Mode: file
    Path: logs
    Level: info
    Compress: false
  ```

- `vim internal/config`

- ```go
  type Config struct {
  	rest.RestConf
  	Mysql struct{
  		DataSource string
  	}
  	CacheRedis cache.CacheConf
  	Log logx.LogConf // 日志配置
  }
  ```

### 编写业务逻辑

业务逻辑代码都在 `service/order/api/cmd/internal/logic/`

- `vim service/order/api/cmd/internal/logic/orders/create_orderLogic.go`

### 运行项目

- 直接在项目服务目录下运行
- `go run order.go -f etc/order-api.yaml`

### docker 运行

#### 生成 Dockerfile

- 在 api 业务目录下生成 Dockerfile

- 切换到服务目录下 `cd /csp/service/order/api/cmd`

- 生成 Dokcerfile `goctl docker -go order.go`

- 修改 Dockerfile 文件，适合项目运行

- ```dockerfile
  FROM golang:alpine AS builder
  
  LABEL stage=gobuilder
  
  ENV CGO_ENABLED 0
  ENV GOOS linux
  ENV GOPROXY https://goproxy.cn,direct
  
  WORKDIR /build/zero
  
  ADD go.mod .
  ADD go.sum .
  RUN go mod download
  COPY . .
  # 修改此路径，因为要复制 go.mod go.sum，所以要在go.mod 所在目录下运行 docker build 构建镜像 ，
  # 或者修改上面的复制go.mod 文件的地址也可以 
  COPY service/order/api/cmd/etc /app/etc
  RUN go build -ldflags="-s -w" -o /app/order service/order/api/cmd/order.go
  
  
  FROM alpine
  
  RUN apk update --no-cache && apk add --no-cache ca-certificates tzdata
  ENV TZ Asia/Shanghai
  
  WORKDIR /app
  COPY --from=builder /app/order /app/order
  COPY --from=builder /app/etc /app/etc
  
  EXPOSE 8888
  
  CMD ["./order", "-f", "etc/order-api.yaml"]
  
  ```

#### 构建镜像

- 在 go.mod 所在的目录下构建镜像
- `cd ostar/csp`
- `docker build -t order:v1 -f service/order/api/cmd/Dockerfile .`

- 查看生成的镜像
- `docker image ls`

#### 启动服务

`docker run --rm -it -p 8888:8888 order:v1`

> --rm ：停止后删除容器
>
> --i ：以交互模式运行容器，通常与 -t 同时使用；
>
> --t：为容器重新分配一个伪输入终端，通常与 -i 同时使用；

#### 测试服务

- `curl -i http://localhost:8888/order/1`
- 返回数据，成功

