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

- ```
  csp // 工程名称
  ├── common //通用库
  │   ├── configx
  │   ├── errorx
  │   └── httpx
  └── service // 服务存放目录
      └── data
          ├── cmd // 业务层
          │   ├── api // HTTP访问服务，业务需求实现
          │   │   └── data.api
          │   └── cronjob // 定时任务，定时数据更新业务
          |	└── rmq // 消息处理系统：mq和dq，处理一些高并发和延时业务
          └── model // model层，直接操作数据库，服务访问持久化数据层的桥梁
              └── orderdatas
                  ├── orderdatasmodel.go
                  └── vars.go
  ```

#### model 生成

- ddl 生成

- `goctl model mysql ddl -src user.sql -dir . -c`

- datacource 生成

- `goctl model mysql datasource -url="$username:$password@tcp($host)" -table="$tablename" -c -dir .`

- 注意事项

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

- 生成 api 文件：`goctl api -o data.api`
- 编写 api 文件，[api语法介绍](https://go-zero.dev/cn/api-grammar.html)

#### api 文件生成 api 服务

- ```shell
  cd csp/service/data/cmd/api
  goctl api go -api data.api -dir .
  ```

### 编写业务代码

#### 添加 Mysql 配置

- `vim service/data/cmd/api/internal/config/config.go`

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

- `vim service/data/cmd/api/etc/data-api.yaml`

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

- `vim service/data/cmd/api/internal/svc/servicecontext.go`

- ```go
  package svc
  
  import (
  	"csp/service/data/cmd/api/internal/config"
  	"csp/service/data/model/orderdatas"
  	"github.com/tal-tech/go-zero/core/stores/sqlx"
  )
  
  type ServiceContext struct {
  	Config config.Config
  	OrderDatasModel orderdatas.OrderDatasModel
  }
  
  func NewServiceContext(c config.Config) *ServiceContext {
  	conn := sqlx.NewMysql(c.Mysql.DataSource)
  	return &ServiceContext{
  		Config: c,
  		OrderDatasModel: orderdatas.NewOrderDatasModel(conn, c.CacheRedis),
  	}
  }
  
  ```

#### 填充业务逻辑

- `vim service/data/cmd/api/internal/logic/orders/createorderLogic.go`

### 错误处理

##### 自定义错误

- 首先在common中添加一个error.go文件

- ```shell
  mkdir -p common/errorx
  touch error.go
  ```

- ```go
  package errorx
  
  import (
  	"github.com/tal-tech/go-zero/core/jsonx"
  	"net/http"
  )
  
  const (
  	defaultCode = 35000
  
  )
  
  var (
  	//ErrorDataNotFound = errors.New("data not found")
  	//ErrorUserNotFound = errors.New("用户不存在")
  	//ErrorNoRequiredParameters = errors.New("必要参数不能为空")
  	//ErrorUserOperation = errors.New("用户正在操作中，请稍后重试")
  	//ErrorServer = NewCodeError("系统错误，请联系管理员")
  	//ErrorMysqlOperation = NewCodeError(350000, "数据库操作错误")
  )
  
  type CodeError struct {
  	Status int `json:"status"`
  	Code int `json:"code"`
  	Message string `json:"message"`
  }
  
  type CodeErrorResponse struct {
  	Code int `json:"code"`
  	Message string `json:"message"`
  }
  
  //ErrorHandler 处理自定义错误
  func ErrorHandler(err error) (int, interface{}) {
  	switch e := err.(type) {
  	case *CodeError:
  		return e.Status, e.Data()
  	default:
  		return http.StatusInternalServerError, CodeErrorResponse{
  			Code: defaultCode,
  			Message: err.Error(),
  		}
  	}
  }
  
  
  func NewCodeError(status, code int, message string) error {
  	return &CodeError{Status: status, Code: code, Message: message}
  }
  
  func NewDefaultError(message string) error {
  	return NewCodeError(http.StatusInternalServerError, defaultCode, message)
  }
  
  // NotFoundHandler 处理404错误
  func NotFoundHandler() http.Handler {
  	return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
  		w.WriteHeader(http.StatusNotFound)
  		w.Header().Set("Content-Type", "text/json; charset=utf-8")
  		msg, _ := jsonx.Marshal(CodeError{Code: http.StatusNotFound, Message: "not found"})
  		w.Write(msg)
  	})
  }
  
  func (e *CodeError) Error() string {
  	return e.Message
  }
  
  func (e *CodeError) Data() *CodeErrorResponse {
  	return &CodeErrorResponse{
  		Code: e.Status * 1000 + e.Code,
  		Message: e.Message,
  	}
  }
  ```

- 开启自定义错误

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
  	//
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

### 日志配置

#### api配置

- `vim etc/data-api.yaml`

- ```yaml
  Name: data-api
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
    ServiceName: csp
    Mode: file
    Path: logs
    Level: info
    Compress: false
  ```

- 
