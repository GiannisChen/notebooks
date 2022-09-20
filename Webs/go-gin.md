# Go-Gin 学习笔记

## 使用

### HelloWorld

1. 引用 `github.com/gin-gonic/gin` ：

   ```shell
   $ go get -u github.com/gin-gonic/gin
   ```

2. 样例代码 `main.go` ：

   ```go
   package main
   
   import "github.com/gin-gonic/gin"
   
   func main() {
   	r := gin.Default()
   	r.GET("/ping", func(c *gin.Context) {
   		c.JSON(200, gin.H{
   			"message": "pong",
   		})
   	})
   	r.Run() // listen and serve on 0.0.0.0:8080
   }
   ```

3. 运行：

   ```go
   $ go run main.go
   ```



### 高效摸鱼TIPS

#### 优雅的启动和停止服务

- `fvbock/endless` —— https://github.com/fvbock/endless

  - 但是不能在 `Windows` 下用😓

  ```go
  router := gin.Default()
  router.GET("/", handler)
  // [...]
  endless.ListenAndServe(":8080", router)
  ```

- 其他，见：https://gin-gonic.com/docs/examples/graceful-restart-or-stop/



### 替代组件

#### `jsoniter`

- https://github.com/json-iterator/go ；
- 宣称是对 `encoding/json` 的热插拔替换，可以在 `Gin` 里指定替换；

![benchmark](images/jsoniter-benchmark)

- 替换规则：

  ```go
  import "encoding/json"
  json.Marshal(&data)
  
  👇
  
  import jsoniter "github.com/json-iterator/go"
  var json = jsoniter.ConfigCompatibleWithStandardLibrary
  json.Marshal(&data)
  ```

  ```go
  import "encoding/json"
  json.Unmarshal(input, &data)
  
  👇
  
  import jsoniter "github.com/json-iterator/go"
  var json = jsoniter.ConfigCompatibleWithStandardLibrary
  json.Unmarshal(input, &data)
  ```

- 在 `Gin` 里的替换方法：

  ```shell
  $ go build -tags=jsoniter .
  ```

---

### API 例子

#### `HHTP` 方法

- 常见的方法，`GET` `POST` `PUT` `DELETE` `PATCH` `HEAD` `OPTIONS` ：

  ```go
  func main() {
  	// Creates a gin router with default middleware:
  	// logger and recovery (crash-free) middleware
  	router := gin.Default()
  
  	router.GET("/someGet", getting)
  	router.POST("/somePost", posting)
  	router.PUT("/somePut", putting)
  	router.DELETE("/someDelete", deleting)
  	router.PATCH("/somePatch", patching)
  	router.HEAD("/someHead", head)
  	router.OPTIONS("/someOptions", options)
  
  	// By default it serves on :8080 unless a
  	// PORT environment variable was defined.
  	router.Run()
  	// router.Run(":3000") for a hard coded port
  }
  ```



#### `AsciiJSON()`

- 将 `ASCII` 编码而非其他编码的数据传回：

  ```go
  import (
  	"github.com/gin-gonic/gin"
  	"net/http"
  )
  
  func main() {
  	r := gin.Default()
  	r.GET("/json", func(c *gin.Context) {
  		data := map[string]interface{}{
  			"lang": "Go语言",
  			"tag":  "test<br>Test",
  		}
  		// {"lang":"Go语言","tag":"test\u003cbr\u003eTest"}
  		c.JSON(http.StatusOK, data)
  
  		// {"lang":"Go\u8bed\u8a00","tag":"test\u003cbr\u003eTest"}
  		c.AsciiJSON(http.StatusOK, data)
  	})
  	r.Run(":8080")
  }
  ```



#### `JsonP()`

- 使用 `JSONP()` 从不同域中的服务器请求数据。如果查询参数 `callback` 存在，则将 `callback` 添加到响应体中：

  ```go
  package main
  
  import (
      "net/http"
  
      "github.com/gin-gonic/gin"
  )
  
  func main() {
      r := gin.Default()
      r.GET("/JSONP", func(c *gin.Context) {
          data := map[string]interface{}{
              "foo": "bar",
          }
  
          //callback is x
          // Will output: x({"foo":"bar"})
          c.JSONP(http.StatusOK, data)
      })
  
      // Listen and serve on 0.0.0.0:8080
      r.Run(":8080")
  }
  ```

  ![image-20220916151224582](images/image-api-jsonp.png)



#### `SecureJSON()`

- `JSON` 作为简单的数据传输方法，容易被劫持（`JSON Hijacking`），详见 https://zhuanlan.zhihu.com/p/65634755，其中一种域内请求最完备的解决方案是加上 `while(1);` ，使用 `SecureJSON` 防止 `JSON` 被劫持。如果给定的结构是数组值，默认会在响应体前加上 `while(1)` 。

  ```go
  func main() {
  	r := gin.Default()
  
  	// You can also use your own secure json prefix
  	// r.SecureJsonPrefix(")]}',\n")
  
  	r.GET("/someJSON", func(c *gin.Context) {
  		names := []string{"lena", "austin", "foo"}
  
  		// Will output  :   while(1);["lena","austin","foo"]
  		c.SecureJSON(http.StatusOK, names)
  	})
  
  	// Listen and serve on 0.0.0.0:8080
  	r.Run(":8080")
  }
  ```

  



#### 重定向 `Redirect()`

- `HTTP` 层级的重定向，支持内部重定向和外部重定向：

  ```go
  // external
  r.GET("/test", func(c *gin.Context) {
  	c.Redirect(http.StatusMovedPermanently, "http://www.google.com/")
  })
  
  // internal
  r.POST("/test", func(c *gin.Context) {
  	c.Redirect(http.StatusFound, "/foo")
  })
  ```

- `Router` 层级的重定向，使用 `HandleContext` 方法：

  ```go
  r.GET("/test", func(c *gin.Context) {
      c.Request.URL.Path = "/test2"
      r.HandleContext(c)
  })
  r.GET("/test2", func(c *gin.Context) {
      c.JSON(200, gin.H{"hello": "world"})
  })
  ```

  

#### 绑定自定义结构体 `Bind`

- 要将请求主体绑定到类型中，请使用模型绑定。`Gin` 目前支持 `JSON` 、 `XML` 、 `YAML` 和标准表单值的绑定 `(foo=bar&boo=baz)` 

  - 注意，需要在想要绑定的所有字段上设置相应的绑定标记。例如，当从 `JSON` 绑定时，设置 `JSON:"fieldname"` 

- 此外，`Gin` 还提供了两组绑定方法：

  - **Type** —— Must bind
    - **Methods** - `Bind`, `BindJSON`, `BindXML`, `BindQuery`, `BindYAML`
    - **Behavior** - 这些方法在底层使用 `MustBindWith` 。如果存在绑定错误，则使用 `c.AbortWithError(400, err).settype(ErrorTypeBind)` 终止请求。这将把响应状态码设置为 `400` ，并将 `Content-Type` 报头设置为 `text/plain; charset=utf-8` 。**注意**，如果您尝试在此之后设置为其他响应代码，它将导致一个警告 `[GIN-debug] [WARNING] Headers were already written. Wanted to override status code 400 with 422`。如果希望自己更加灵活地控制绑定，可以考虑使用 `ShouldBind` 等效方法。
  - **Type** —— Should bind
    - **Methods** - `ShouldBind`, `ShouldBindJSON`, `ShouldBindXML`, `ShouldBindQuery`, `ShouldBindYAML`
    - **Behavior** - 这些方法在底层使用 `ShouldBindWith` 。如果存在绑定错误，则返回错误，需要手动处理这些错误。

- 当需要绑定时， `Gin` 会试图依照报文头的 `Content-Type` 来判断数据类型。如果能够确定使用的是什么数据格式，可以使用 `MustBindWith` 和 `ShouldBindWith` 。

- 还可以指定需要的特定字段。如果一个字段用 `binding:"required"` 修饰，并且绑定时为空值，则会返回一个错误，具体见 https://github.com/go-playground/validator 。

- `ShouldBindQuery` 只绑定 `query params` 而不是提交的榜单数据。

- **注意**，一般的绑定过程 `ShouldBind` 会“消耗” `c.Request.Body`  ，因为 `c.Request.Body` 事实上是一个读通道，也就意味着为了高效读取，每次遍历过后并不会复位，会引起 `io.EOF` 的错误：

  ![image-20220918135824268](C:/Users/10656/AppData/Roaming/Typora/typora-user-images/image-20220918135824268.png)

  ```go
  type formA struct {
    	Foo string `json:"foo" xml:"foo" binding:"required"`
  }
  
  type formB struct {
    	Bar string `json:"bar" xml:"bar" binding:"required"`
  }
  
  func SomeHandler(c *gin.Context) {
      objA := formA{}
      objB := formB{}
      // This c.ShouldBind consumes c.Request.Body and it cannot be reused.
      if errA := c.ShouldBind(&objA); errA == nil {
          c.String(http.StatusOK, `the body should be formA`)
          // Always an error is occurred by this because c.Request.Body is EOF now.
      } else if errB := c.ShouldBind(&objB); errB == nil {
          c.String(http.StatusOK, `the body should be formB`)
      } else {
          ...
      }
  }
  ```

  为了可以多次绑定，可以采用特殊的方法 `c.ShouldBindBodyWith` ：

  ```go
  func SomeHandler(c *gin.Context) {
    objA := formA{}
    objB := formB{}
    // This reads c.Request.Body and stores the result into the context.
    if errA := c.ShouldBindBodyWith(&objA, binding.JSON); errA == nil {
      c.String(http.StatusOK, `the body should be formA`)
    // At this time, it reuses body stored in the context.
    } else if errB := c.ShouldBindBodyWith(&objB, binding.JSON); errB == nil {
      c.String(http.StatusOK, `the body should be formB JSON`)
    // And it can accepts other formats
    } else if errB2 := c.ShouldBindBodyWith(&objB, binding.XML); errB2 == nil {
      c.String(http.StatusOK, `the body should be formB XML`)
    } else {
      ...
    }
  }
  ```

  - `c.ShouldBindBodyWith` 在绑定之前将 `Body` 存储到上下文中。这对性能有轻微影响，因此如果可以立即调用绑定，则不应该使用此方法。
  - 上述特性只会影响部分数据格式的绑定，例如 `JSON`, `XML`, `MsgPack`, `ProtoBuf`。其他的数据格式诸如 `Query`, `Form`, `FormPost`, `FormMultipart` ，则不受这个特性的影响，可以多次被 `c.ShouldBind()` 绑定而不会造成性能影响或者引起意外的错误。

##### 递归绑定自定义结构体

- 可以快速填充对应的对象：

  ```go
  import (
  	"github.com/gin-gonic/gin"
  	"net/http"
  )
  
  type StructA struct {
  	Code  int    `form:"code" json:"code"`
  	Msg   string `form:"msg" json:"msg"`
  	Field Field  `form:"field"`
  }
  
  type Field struct {
  	FieldA string `form:"a" json:"a"`
  	FieldB string `form:"b" json:"b"`
  }
  
  func main() {
  	r := gin.Default()
  
  	r.POST("/updateA", func(c *gin.Context) {
  		var s StructA
  		c.Bind(&s)
          
          // {"code":10,"field_a":"A","field_b":"B","msg":"check check"}
  		c.JSON(http.StatusOK, gin.H{
  			"code":    s.Code,
  			"msg":     s.Msg,
  			"field_a": s.Field.FieldA,
  			"field_b": s.Field.FieldB,
  		})
  	})
  
  	r.Run(":8080")
  }
  ```

##### 绑定 `checkboxes` 

- 绑定 `HTML` 的 `checkboxes` ：

  ```go
  import (
  	"github.com/gin-gonic/gin"
  	"net/http"
  )
  
  type Form struct {
  	Colors []string `form:"colors[]" json:"colors[]"`
  }
  
  func main() {
  	r := gin.Default()
  
  	// Bind checkboxes
  	r.LoadHTMLGlob("./test/templates/*")
  	r.GET("/color.html", func(c *gin.Context) {
  		c.HTML(http.StatusOK, "color.html", nil)
  	})
  	r.POST("/color", func(c *gin.Context) {
  		var form Form
  		c.Bind(&form)
          // {"color(s)":["red","green"]}
  		c.JSON(http.StatusOK, gin.H{"color(s)": form.Colors})
  	})
  
  	// Run
  	r.Run(":8080")
  }
  ```

  ![image-20220914162441963](images/image-color-checkboxes.png)

##### 绑定 `Uri`

- 在 `Uri` 里的参数，也可以与自定义结构体绑定：

  ```go
  import (
  	"github.com/gin-gonic/gin"
  	"net/http"
  )
  
  type Person struct {
  	ID   string `uri:"id" binding:"required,uuid"`
  	Name string `uri:"name" binding:"required"`
  }
  
  func main() {
  	r := gin.Default()
  
  	// Bind Uri values
  	r.GET("/person/:name/:id", func(c *gin.Context) {
  		var person Person
  		if err := c.ShouldBindUri(&person); err != nil {
  			c.JSON(400, gin.H{"msg": err.Error()})
  			return
  		}
          // curl "localhost:8080/person/giannis/987fbc97-4bed-5078-9f07-9141ba07c9f3"
          // {"name":"giannis","uuid":"987fbc97-4bed-5078-9f07-9141ba07c9f3"}
  		c.JSON(200, gin.H{"name": person.Name, "uuid": person.ID})
  	})
  
  	// Run
  	r.Run(":8080")
  }
  ```



#### 从 `form` 或者 `Uri` 中获取 `Map` `Array`

- 使用 `QueryArray()` 和 `QueryMap()` ：

  ```go
  // TestParseMap parses map querystring or post form parameters
  func TestParseMap(c *gin.Context) {
  	ids := c.QueryArray("ids")
  	cityMap := c.QueryMap("map")
  	cities := make([]string, len(ids))
  	for i := range ids {
  		cities[i] = cityMap[ids[i]]
  	}
  
  	c.JSON(200, gin.H{
  		"ids":     ids,
  		"cityMap": cityMap,
  		"cities":  cities,
  	})
  }
  
  func main() {
  	router := gin.Default()
  	// localhost:8080/map?map[1]=Beijing&map[2]=Nanjing&map[3]=Tokyo&ids=1&ids=3&ids=2
  	router.POST("/map", TestParseMap)
  	router.Run(":8080")
  }
  ```

- `Response` ：

  ```json
  {
      "cities":["Beijing","Tokyo","Nanjing"],
      "cityMap":{"1":"Beijing","2":"Nanjing","3":"Tokyo"},
      "ids":["1","3","2"],
  }
  ```



#### 在 `Uri` 中寻找指定参数

- 对于 `/path/:name` 这样的：

  ```go
  func main() {
  	router := gin.Default()
  
  	router.GET("/user/:name", func(c *gin.Context) {
  		name := c.Param("name")
  		c.String(http.StatusOK, "Hello %s", name)
  	})
  
  	router.Run(":8080")
  }
  
  // Hello Giannis
  ```

- 对于 `/path/:name/*action` 这样的：

  ```go
  func main() {
  	router := gin.Default()
  
  	router.GET("/path/:name/*action", func(c *gin.Context) {
  		name := c.Param("name")
          action := c.Param("action")
          message := name + " is " + action
          c.String(200, message)
  	})
  
  	router.Run(":8080")
  }
  
  // Giannis is /select
  ```

  

---



### 中间件Middleware

- `Gin` 本身就有一系列的中间件，只需要简单的调用即可：

  ```go
  func main() {
  	// Creates a router without any middleware by default
  	r := gin.New()
      
      // Default With the Logger and Recovery middleware already attached
  	// r := gin.Default()
  
  	// Global middleware
  	// Logger middleware will write the logs to gin.DefaultWriter even if you set with GIN_MODE=release.
  	// By default gin.DefaultWriter = os.Stdout
  	r.Use(gin.Logger())
  
  	// Recovery middleware recovers from any panics and writes a 500 if there was one.
  	r.Use(gin.Recovery())
  
  	// Per route middleware, you can add as many as you desire.
  	r.GET("/benchmark", MyBenchLogger(), benchEndpoint)
  
  	// Authorization group
  	// authorized := r.Group("/", AuthRequired())
  	// exactly the same as:
  	authorized := r.Group("/")
  	// per group middleware! in this case we use the custom created
  	// AuthRequired() middleware just in the "authorized" group.
  	authorized.Use(AuthRequired())
  	{
  		authorized.POST("/login", loginEndpoint)
  		authorized.POST("/submit", submitEndpoint)
  		authorized.POST("/read", readEndpoint)
  
  		// nested group
  		testing := authorized.Group("testing")
  		testing.GET("/analytics", analyticsEndpoint)
  	}
  
  	// Listen and serve on 0.0.0.0:8080
  	r.Run(":8080")
  }
  ```

  

#### 日志输出

##### 颜色

- 默认情况下，控制台上的日志输出应该根据检测到的TTY进行着色：

  ![image-20220914174328303](images/image-log-color.png)

  ```go
  func main() {
  	{
  		// Disable log's color
  		gin.DisableConsoleColor()
  	}
  	{
  		// Enable log's color
  		gin.ForceConsoleColor()
  	}
      
      // Creates a gin router with default middleware:
      // logger and recovery (crash-free) middleware
      router := gin.Default()
      
      router.GET("/ping", func(c *gin.Context) {
          c.String(200, "pong")
      })
      
      router.Run(":8080")
  }
  ```

##### 自定义日志样式

- 通过中间件指定输出的样式：

  ```go
  func main() {
  	router := gin.New()
  	// LoggerWithFormatter middleware will write the logs to gin.DefaultWriter
  	// By default gin.DefaultWriter = os.Stdout
  	router.Use(gin.LoggerWithFormatter(func(param gin.LogFormatterParams) string {
  		// your custom format
  		return fmt.Sprintf("%s - [%s] \"%s %s %s %d %s \"%s\" %s\"\n",
  				param.ClientIP,
  				param.TimeStamp.Format(time.RFC1123),
  				param.Method,
  				param.Path,
  				param.Request.Proto,
  				param.StatusCode,
  				param.Latency,
  				param.Request.UserAgent(),
  				param.ErrorMessage,
  		)
  	}))
  	router.Use(gin.Recovery())
  	router.GET("/ping", func(c *gin.Context) {
  		c.String(200, "pong")
  	})
  	router.Run(":8080")
  }
  
  // ::1 - [Wed, 14 Sep 2022 20:02:58 CST] "GET /person/giannis/987fbc97-4bed-5078-9f07-9141ba07c9f3 HTTP/1.1 200 0s "PostmanRuntime/7.29.2" "ng HTTP on :8080
  
  ```

##### 自定义路由注册日志

- 如果你想用给定的格式（例如 `JSON` 、 `KV` 或其他格式）记录这些信息，那么可以用 `gin.DebugPrintRouteFunc` 定义这种格式。在下面的例子中，我们使用标准的日志包记录所有的路由：

  ```
  [GIN-debug] GET    /ping                     --> Gin-Test/test/controller.Ping (6 handlers)
  [GIN-debug] GET    /json                     --> Gin-Test/test/controller.TestJSONGet (6 handlers)        
  [GIN-debug] POST   /updateA                  --> Gin-Test/test/controller.TestBindStructRecursivelyPost (6
   handlers)
  [GIN-debug] GET    /color.html               --> Gin-Test/test/controller.GetCheckboxes (6 handlers)      
  [GIN-debug] POST   /color                    --> Gin-Test/test/controller.TestBindCheckboxesPost (6 handle
  rs)
  [GIN-debug] GET    /person/:name/:id         --> Gin-Test/test/controller.TestBindUriGet (6 handlers)     
  [GIN-debug] POST   /user                     --> Gin-Test/test/controller.TestValidateUser (6 handlers)  
  ```

- 修改前👆，修改后👇：

  ```
  2022/09/15 19:51:49 endpoint GET /ping Gin-Test/test/controller.Ping 6
  2022/09/15 19:51:49 endpoint GET /json Gin-Test/test/controller.TestJSONGet 6
  2022/09/15 19:51:49 endpoint POST /updateA Gin-Test/test/controller.TestBindStructRecursivelyPost 6       
  2022/09/15 19:51:49 endpoint GET /color.html Gin-Test/test/controller.GetCheckboxes 6
  2022/09/15 19:51:49 endpoint POST /color Gin-Test/test/controller.TestBindCheckboxesPost 6
  2022/09/15 19:51:49 endpoint GET /person/:name/:id Gin-Test/test/controller.TestBindUriGet 6
  2022/09/15 19:51:49 endpoint POST /user Gin-Test/test/controller.TestValidateUser 6
  ```

#### 日志文件

- 日志写入文件，持久化：

  ```go
  func main() {
      // Disable Console Color, you don't need console color when writing the logs to file.
      gin.DisableConsoleColor()
  
      // Logging to a file.
      f, _ := os.Create("gin.log")
      gin.DefaultWriter = io.MultiWriter(f)
  
      // Use the following code if you need to write the logs to file and console at the same time.
      // gin.DefaultWriter = io.MultiWriter(f, os.Stdout)
  
      router := gin.Default()
      router.GET("/ping", func(c *gin.Context) {
          c.String(200, "pong")
      })
  
      router.Run(":8080")
  }
  ```



#### HTTP设置

- 设置自定义的 `HTTP` 参数：

  ```go
  // Use http.ListenAndServe() directly
  func main() {
  	router := gin.Default()
  	http.ListenAndServe(":8080", router)
  }
  ```

  ```go
  // Use Server.ListenAndServe()
  func main() {
  	router := gin.Default()
  
  	s := &http.Server{
  		Addr:           ":8080",
  		Handler:        router,
  		ReadTimeout:    10 * time.Second,
  		WriteTimeout:   10 * time.Second,
  		MaxHeaderBytes: 1 << 20,
  	}
  	s.ListenAndServe()
  }
  ```

  

#### 自定义中间件

```go
func MyMiddleware() gin.HandlerFunc {
	return func(c *gin.Context) {
		t := time.Now()
		c.Set("passerby", "giannis")

		s, exisits := c.Get("after")
		status := c.Writer.Status()
		if exisits {
			log.Printf("[status: %d] %v", status, s)
		} else {
			log.Printf("[status: %d] %v", status, "not found")
		}

		c.Next()

		log.Println(time.Now().UnixNano() - t.UnixNano())

		s, exisits = c.Get("after")
		status = c.Writer.Status()
		log.Printf("[status: %d] %v", status, s)
	}
}

func main() {
	r := gin.New()
	r.Use(MyMiddleware())

	r.GET("/ping", func(c *gin.Context) {
		c.Set("after", "get")
		log.Println(c.MustGet("passerby").(string))
	})

	// Listen and serve on 0.0.0.0:8080
	r.Run(":8080")
}

// 2022/09/14 21:53:12 [status: 200] not found
// 2022/09/14 21:53:12 giannis
// 2022/09/14 21:53:12 9531000
// 2022/09/14 21:53:12 [status: 200] get
// [GIN] 2022/09/14 - 21:53:12 | 200 | 10.1156ms | ::1 | GET "/ping"
```



#### 自定义验证

- 用 https://github.com/go-playground/validator 来完成验证，在属性层级（`FieldLevel`）和结构体层级（`StructLevel`）：

  ```go
  type User struct {
  	FirstName string `form:"fname"`
  	LastName  string `form:"lname"`
  	Email     string `form:"email" binding:"required,email"`
  }
  ```

- `form` ：用来指定从 `Uri` 读并绑定相关数据，对应的时**表单**的方式；

- `binding` ：在 `Gin` 里使用 `validator` 进行验证，指定字段和验证类型，常用验证类型：https://github.com/go-playground/validator#baked-in-validations；

  ```go
  package validation
  
  import (
  	"Gin-Test/test/controller"
  	"github.com/go-playground/validator/v10"
  	"regexp"
  )
  
  func UserStructLevelValidation(sl validator.StructLevel) {
  	user := sl.Current().Interface().(controller.User)
  	if len(user.FirstName) == 0 && len(user.LastName) == 0 {
  		sl.ReportError(user.FirstName, "FirstName", "fname", "fnameorlname", "")
  		sl.ReportError(user.LastName, "LastName", "lname", "fnameorlname", "")
  	}
  }
  
  func UserEmailFieldValidation(fl validator.FieldLevel) bool {
  	email, ok := fl.Field().Interface().(string)
  	if ok {
  		re, err := regexp.Compile(`^[a-zA-Z\d_-]+@[a-zA-Z\d_-]+(\.[a-zA-Z\d_-]+)+$`)
  		if err != nil {
  			return false
  		}
  		return re.MatchString(email)
  	}
  	return false
  }
  ```

  ```go
  // TestValidateUser validates the users
  func TestValidateUser(c *gin.Context) {
  	var u User
  	if err := c.ShouldBind(&u); err == nil {
  		c.JSON(200, gin.H{
  			"message": "success",
  			"name":    fmt.Sprintf("%s-%s", u.FirstName, u.LastName),
  			"email":   u.Email,
  		})
  	} else {
  		c.JSON(http.StatusBadRequest, gin.H{
  			"message": "User validation failed!",
  			"error":   err.Error(),
  		})
  	}
  }
  
  func main() {
  	route := gin.Default()
  
  	// Validate users
  	if v, ok := binding.Validator.Engine().(*validator.Validate); ok {
  		v.RegisterValidation("email", validation.UserEmailFieldValidation)
  		v.RegisterStructValidation(validation.UserStructLevelValidation, controller.User{})
  	}
  	r.POST("/user", controller.TestValidateUser)
  
  	// Run
  	r.Run(":8080")
  }
  ```

- 结果：

  ```json
  // fname:Giannis
  // lname:Chen
  // email:111@test.com
  
  {"email": "111@test.com", "message": "success", "name": "Giannis-Chen"}
  ```

  ```go
  // fname:Giannis
  // lname:Chen
  // email:111test.com
  
  {"error": "Key: 'User.Email' Error:Field validation for 'Email' failed on the 'email' tag", "message": "User validation failed!"}
  ```



#### 在中间件内使用线程

- **规范**：在中间件和处理器中使用额外的 `goroutine` 来处理业务时，需要将 `gin.Context` 进行**只读的副本**，而不是使用原来的 `gin.Context` ：

  ```go
  func main() {
  	route := gin.Default()
  
  	// goroutine
  	r.GET("/goroutine", TestGoroutine)
  
  	// Run
  	r.Run(":8080")
  }
  
  // TestGoroutine tests goroutine inside a handler
  func TestGoroutine(c *gin.Context) {
  	ccp := c.Copy()
  	go func() {
  		time.Sleep(time.Second)
  		log.Println("Done in path " + ccp.Request.URL.Path)
  	}()
  }
  ```



### XML/JSON/YAML/ProtoBuf 返回值渲染

```go
func main() {
	r := gin.Default()

	// gin.H is a shortcut for map[string]interface{}
	r.GET("/someJSON", func(c *gin.Context) {
		c.JSON(http.StatusOK, gin.H{"message": "hey", "status": http.StatusOK})
	})

	r.GET("/moreJSON", func(c *gin.Context) {
		// You also can use a struct
		var msg struct {
			Name    string `json:"user"`
			Message string
			Number  int
		}
		msg.Name = "Lena"
		msg.Message = "hey"
		msg.Number = 123
		// Note that msg.Name becomes "user" in the JSON
		// Will output  :   {"user": "Lena", "Message": "hey", "Number": 123}
		c.JSON(http.StatusOK, msg)
	})

	r.GET("/someXML", func(c *gin.Context) {
		c.XML(http.StatusOK, gin.H{"message": "hey", "status": http.StatusOK})
	})

	r.GET("/someYAML", func(c *gin.Context) {
		c.YAML(http.StatusOK, gin.H{"message": "hey", "status": http.StatusOK})
	})

	r.GET("/someProtoBuf", func(c *gin.Context) {
		reps := []int64{int64(1), int64(2)}
		label := "test"
		// The specific definition of protobuf is written in the testdata/protoexample file.
		data := &protoexample.Test{
			Label: &label,
			Reps:  reps,
		}
		// Note that data becomes binary data in the response
		// Will output protoexample.Test protobuf serialized data
		c.ProtoBuf(http.StatusOK, data)
	})

	// Listen and serve on 0.0.0.0:8080
	r.Run(":8080")
}
```



### HTML模板渲染

- 使用 `LoadHTMLGlob()` 和 `LoadHTMLFiles()` ：

  ```go
  func main() {
  	router := gin.Default()
  	router.LoadHTMLGlob("templates/*")
  	//router.LoadHTMLFiles("templates/template1.html", "templates/template2.html")
  	router.GET("/index", func(c *gin.Context) {
  		c.HTML(http.StatusOK, "index.tmpl", gin.H{
  			"title": "Main website",
  		})
  	})
  	router.Run(":8080")
  }
  ```

- 使用自己的 `HTML` 渲染器，比如 `html/template` ：

  ```go
  import "html/template"
  
  func main() {
  	router := gin.Default()
  	html := template.Must(template.ParseFiles("file1", "file2"))
  	router.SetHTMLTemplate(html)
  	router.Run(":8080")
  }
  ```

- 如果转义符号发生了冲突，那么也可以使用自定义的转义字符：

  ```go
  r := gin.Default()
  r.Delims("{[{", "}]}")
  r.LoadHTMLGlob("/path/to/templates")
  ```



### HTTP服务器推送

`HTTP/2` 被设计用来解决 `HTTP/1.x` 的许多缺陷。现代网页使用许多资源：HTML、样式表、脚本、图像等等。在 `HTTP/1.x` ，这些资源都必须显式请求，这可能是一个缓慢的过程。浏览器首先获取HTML，然后在解析和计算页面时逐步请求更多的资源。由于服务器必须等待浏览器发出每个请求，网络通常是闲置的，没有得到充分利用。

为了改善延迟，`HTTP/2` 引入了服务器推送（ `server push` ），它允许服务器在显式请求资源之前将资源推送到浏览器。服务器通常知道一个页面需要多少额外的资源，并可以在响应初始请求时开始推送这些资源。这允许服务器充分利用空闲的网络，并提高页面加载时间。

![img](images/image-http-serverpush.svg)

在协议级别，`HTTP/2` 服务器推送是由 `PUSH_PROMISE` 帧驱动的。`PUSH_PROMISE` 描述服务器预测浏览器在不久的将来会发出的请求。只要浏览器接收到 `PUSH_PROMISE`，它就知道服务器将交付资源。如果浏览器后来发现它需要这个资源，它将等待推送完成，而不是发送一个新的请求，这减少了浏览器在网络上等待的时间。

举个例子，如果服务器知道 `app.js` 将会被浏览器去请求，那么处理器将会初始化一个 `push` ，如果 `http.Pusher` 是可用的：

```go
http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
    if pusher, ok := w.(http.Pusher); ok {
        // Push is supported.
        if err := pusher.Push("/app.js", nil); err != nil {
            log.Printf("Failed to push: %v", err)
        }
    }
    // ...
})
```

`Push` 调用为 `/app.js` 创建一个合成请求，将该请求合成到一个 `PUSH_PROMISE` 帧中，然后将合成请求转发给服务器的请求处理程序，后者将生成推送响应。`Push` 的第二个参数指定了要包含在 `PUSH_PROMISE` 中的其他标头。例如，如果对 `/app.js` 的响应在 `Accept-Encoding` 上发生变化，那么 `PUSH_PROMISE` 应该包含一个`Accept-Encoding` 值：

```go
http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
    if pusher, ok := w.(http.Pusher); ok {
        // Push is supported.
        options := &http.PushOptions{
            Header: http.Header{
                "Accept-Encoding": r.Header["Accept-Encoding"],
            },
        }
        if err := pusher.Push("/app.js", options); err != nil {
            log.Printf("Failed to push: %v", err)
        }
    }
    // ...
})
```

![img](images/image-http-networktimeline.png)

在发送响应的任何字节之前调用 `Push` 方法是一个好主意。否则，可能会意外地生成重复的响应。例如，假设你写了一个HTML响应的一部：

```html
<html>
<head>
    <link rel="stylesheet" href="a.css">...
```

然后调用 `Push("a.css", nil)` 。浏览器可能会在接收你的 `PUSH_PROMISE` 之前解析这段HTML，在这种情况下，除了接收你的 `PUSH_PROMISE` ，浏览器还会发送一个额外的请求 `a.css` 。现在，服务器将为 `a.css` 生成两个响应。在编写响应之前调用 `Push` 完全避免了这种可能性。

- 简单的 `server push` 代码：

  ```go
  package main
  
  import (
  	"html/template"
  	"log"
  
  	"github.com/gin-gonic/gin"
  )
  
  var html = template.Must(template.New("https").Parse(`
  <!DOCTYPE html>
  <html lang="en">
  <head>
      <meta charset="UTF-8">
      <title>Push</title>
      <script src="/assets/app.js"></script>
  </head>
  <body>
      <h1 style="color:darkseagreen;">Welcome, Ginner!</h1>
  </body>
  </html>
  `))
  
  func main() {
  	r := gin.Default()
  	r.Static("/assets", "./assets")
  	r.SetHTMLTemplate(html)
  
  	r.GET("/", func(c *gin.Context) {
  		if pusher := c.Writer.Pusher(); pusher != nil {
  			// use pusher.Push() to do server push
  			if err := pusher.Push("/assets/app.js", nil); err != nil {
  				log.Printf("Failed to push: %v", err)
  			}
  		}
  		c.HTML(200, "https", gin.H{
  			"status": "success",
  		})
  	})
  
  	// Run
  	r.Run(":8080")
  }
  ```

  ![image-20220916115602490](images/image-http-pushPage.png)



### 服务器多开

- 事实上， `go.Run()` 会阻塞之后的代码，经典 `ListenAndServe` （里有循环）；所以如果需要一次启动多个服务，我们需要多协程的帮助，这样，就能在多个端口监听了：

  ```go
  package main
  
  import (
  	"log"
  	"net/http"
  	"time"
  
  	"github.com/gin-gonic/gin"
  	"golang.org/x/sync/errgroup"
  )
  
  var (
  	g errgroup.Group
  )
  
  func router01() http.Handler {
  	e := gin.New()
  	e.Use(gin.Recovery())
  	e.GET("/", func(c *gin.Context) {
  		c.JSON(
  			http.StatusOK,
  			gin.H{
  				"code":  http.StatusOK,
  				"message": "Welcome server 01",
  			},
  		)
  	})
  
  	return e
  }
  
  func router02() http.Handler {
  	e := gin.New()
  	e.Use(gin.Recovery())
  	e.GET("/", func(c *gin.Context) {
  		c.JSON(
  			http.StatusOK,
  			gin.H{
  				"code":  http.StatusOK,
  				"message": "Welcome server 02",
  			},
  		)
  	})
  
  	return e
  }
  
  func main() {
  	server01 := &http.Server{
  		Addr:         ":8080",
  		Handler:      router01(),
  		ReadTimeout:  5 * time.Second,
  		WriteTimeout: 10 * time.Second,
  	}
  
  	server02 := &http.Server{
  		Addr:         ":8081",
  		Handler:      router02(),
  		ReadTimeout:  5 * time.Second,
  		WriteTimeout: 10 * time.Second,
  	}
  
  	g.Go(func() error {
  		return server01.ListenAndServe()
  	})
  
  	g.Go(func() error {
  		return server02.ListenAndServe()
  	})
  
  	if err := g.Wait(); err != nil {
  		log.Fatal(err)
  	}
  }
  ```



### 静态资源

- 三种方法加载静态资源，分别为使用 `http.FileServer` ， `http.FileSystem` ，以及 `http.FileSystem` 持有一个文件：

  ```go
  func main() {
  	router := gin.Default()
  	router.Static("/assets", "./assets")
  	router.StaticFS("/more_static", http.Dir("my_file_system"))
  	router.StaticFile("/favicon.ico", "./resources/favicon.ico")
  
  	// Listen and serve on 0.0.0.0:8080
  	router.Run(":8080")
  }
  ```



### 开箱即用的加密 `HTTPS`

- 一行就能用：

  ```go
  package main
  
  import (
  	"log"
  
  	"github.com/gin-gonic/autotls"
  	"github.com/gin-gonic/gin"
  )
  
  func main() {
  	r := gin.Default()
  
  	// Ping handler
  	r.GET("/ping", func(c *gin.Context) {
  		c.String(200, "pong")
  	})
  
  	log.Fatal(autotls.Run(r, "example1.com", "example2.com"))
  }
  ```

- 自定义的自动证书授权和管理：

  ```go
  package main
  
  import (
  	"log"
  
  	"github.com/gin-gonic/autotls"
  	"github.com/gin-gonic/gin"
  	"golang.org/x/crypto/acme/autocert"
  )
  
  func main() {
  	r := gin.Default()
  
  	// Ping handler
  	r.GET("/ping", func(c *gin.Context) {
  		c.String(200, "pong")
  	})
  
  	m := autocert.Manager{
  		Prompt:     autocert.AcceptTOS,
  		HostPolicy: autocert.HostWhitelist("example1.com", "example2.com"),
  		Cache:      autocert.DirCache("/var/www/.cache"),
  	}
  
  	log.Fatal(autotls.RunWithManager(r, &m))
  }
  ```

  

### 文件上传

- **单个文件**，文件名总是可选的，应用程序不能盲目地使用它，应该去掉路径信息，并将其转换为服务器文件系统规则，这意味着用户的文件名是不可信的：

  ```go
  func main() {
  	router := gin.Default()
  	// Set a lower memory limit for multipart forms (default is 32 MiB)
  	router.MaxMultipartMemory = 8 << 20  // 8 MiB
  	router.POST("/upload", func(c *gin.Context) {
  		// single file
  		file, _ := c.FormFile("file")
  		log.Println(file.Filename)
  
  		// Upload the file to specific dst.
  		c.SaveUploadedFile(file, dst)
  
  		c.String(http.StatusOK, fmt.Sprintf("'%s' uploaded!", file.Filename))
  	})
  	router.Run(":8080")
  }
  ```

  安全的做法：

  ```go
  file, err := c.FormFile("image")
  filename := filepath.Base(file.Filename)
  dst := filepath.Join("./test/images", filename)
  ```

- **多个文件**：

  ```go
  func main() {
  	router := gin.Default()
  	// Set a lower memory limit for multipart forms (default is 32 MiB)
  	router.MaxMultipartMemory = 8 << 20  // 8 MiB
  	router.POST("/upload", func(c *gin.Context) {
  		// Multipart form
  		form, _ := c.MultipartForm()
  		files := form.File["multi-image"]
  
  		for _, file := range files {
  			log.Println(file.Filename)
  
  			// Upload the file to specific dst.
  			c.SaveUploadedFile(file, dst)
  		}
  		c.String(http.StatusOK, fmt.Sprintf("%d files uploaded!", len(files)))
  	})
  	router.Run(":8080")
  }
  ```



### 基础身份验证

-  通过 `gin.BasicAuth` 中间件做简单的身份验证：

  ```go
  // simulate some private data
  var secrets = gin.H{
  	"foo":    gin.H{"email": "foo@bar.com", "phone": "123433"},
  	"austin": gin.H{"email": "austin@example.com", "phone": "666"},
  	"lena":   gin.H{"email": "lena@guapa.com", "phone": "523443"},
  }
  
  func main() {
  	r := gin.Default()
  
  	// Group using gin.BasicAuth() middleware
  	// gin.Accounts is a shortcut for map[string]string
  	authorized := r.Group("/admin", gin.BasicAuth(gin.Accounts{
  		"foo":    "bar",
  		"austin": "1234",
  		"lena":   "hello2",
  		"manu":   "4321",
  	}))
  
  	// /admin/secrets endpoint
  	// hit "localhost:8080/admin/secrets
  	authorized.GET("/secrets", func(c *gin.Context) {
  		// get user, it was set by the BasicAuth middleware
  		user := c.MustGet(gin.AuthUserKey).(string)
  		if secret, ok := secrets[user]; ok {
  			c.JSON(http.StatusOK, gin.H{"user": user, "secret": secret})
  		} else {
  			c.JSON(http.StatusOK, gin.H{"user": user, "secret": "NO SECRET :("})
  		}
  	})
  
  	// Listen and serve on 0.0.0.0:8080
  	r.Run(":8080")
  }
  ```

  访问一下 `/admin/secrets` ，有比较基础的弹窗：

  ![image-20220918215748383](C:/Users/10656/AppData/Roaming/Typora/typora-user-images/image-20220918215748383.png)





## `Gin` 源码



### 前置组件的源码

#### `net/http`

- `net/http` 是服务框架中最基础的一部分，其较为原始，也是最根本的使用方式如下：

```go
http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
    w.Write([]byte("Hello World"))
})

if err := http.ListenAndServe(":8000", nil); err != nil {
    fmt.Println("start http server fail:", err)
}
```

```go
HandleFunc(pattern string, handler func(ResponseWriter, *Request))
	👇
(mux *ServeMux) HandleFunc(pattern string, handler func(ResponseWriter, *Request))
	👇
func (mux *ServeMux) Handle(pattern string, handler Handler)
```

- 从 `http.HandleFunc` 一路导航下去，我们会发现使用了默认的 `ServeMux` ，最后调用了底层的注册操作，原理也很简单，就是一个 `map` 来缓存 `Uri` 和对应的路由操作：

```go
// Handle registers the handler for the given pattern.
// If a handler already exists for pattern, Handle panics.
func (mux *ServeMux) Handle(pattern string, handler Handler) {
	mux.mu.Lock()
	defer mux.mu.Unlock()

	if pattern == "" {
		panic("http: invalid pattern")
	}
	if handler == nil {
		panic("http: nil handler")
	}
	if _, exist := mux.m[pattern]; exist {
		panic("http: multiple registrations for " + pattern)
	}

	if mux.m == nil {
		mux.m = make(map[string]muxEntry)
	}
	e := muxEntry{h: handler, pattern: pattern}
	mux.m[pattern] = e
	if pattern[len(pattern)-1] == '/' {
		mux.es = appendSorted(mux.es, e)
	}

	if pattern[0] != '/' {
		mux.hosts = true
	}
}
```

- 而其中 `ServeMux` 长这样：

```go
// ServeMux also takes care of sanitizing the URL request path and the Host
// header, stripping the port number and redirecting any request containing . or
// .. elements or repeated slashes to an equivalent, cleaner URL.
type ServeMux struct {
	mu    sync.RWMutex
	m     map[string]muxEntry
	es    []muxEntry // slice of entries sorted from longest to shortest.
	hosts bool       // whether any patterns contain hostnames
}
```

- 当我们完成了路由配置后，我们需要启动我们的服务了。通过 `http.ListenAndServe` 来启动我们的服务端，其调用如下：

```go
// ListenAndServe listens on the TCP network address addr and then calls
// Serve with handler to handle requests on incoming connections.
// Accepted connections are configured to enable TCP keep-alives.
//
// The handler is typically nil, in which case the DefaultServeMux is used.
//
// ListenAndServe always returns a non-nil error.
func ListenAndServe(addr string, handler Handler) error {
   server := &Server{Addr: addr, Handler: handler}
   return server.ListenAndServe()
}
```

👇

```go
// ListenAndServe listens on the TCP network address srv.Addr and then
// calls Serve to handle requests on incoming connections.
// Accepted connections are configured to enable TCP keep-alives.
//
// If srv.Addr is blank, ":http" is used.
//
// ListenAndServe always returns a non-nil error. After Shutdown or Close,
// the returned error is ErrServerClosed.
func (srv *Server) ListenAndServe() error {
	if srv.shuttingDown() {
		return ErrServerClosed
	}
	addr := srv.Addr
	if addr == "" {
		addr = ":http"
	}
	ln, err := net.Listen("tcp", addr)
	if err != nil {
		return err
	}
	return srv.Serve(ln)
}
```

👇

```go
// Serve accepts incoming connections on the Listener l, creating a
// new service goroutine for each. The service goroutines read requests and
// then call srv.Handler to reply to them.
//
// HTTP/2 support is only enabled if the Listener returns *tls.Conn
// connections and they were configured with "h2" in the TLS
// Config.NextProtos.
//
// Serve always returns a non-nil error and closes l.
// After Shutdown or Close, the returned error is ErrServerClosed.
func (srv *Server) Serve(l net.Listener) error {
	...
	for {
		rw, err := l.Accept()
		if err != nil {
			select {
			case <-srv.getDoneChan():
				return ErrServerClosed
			default:
			}
			if ne, ok := err.(net.Error); ok && ne.Temporary() {
				if tempDelay == 0 {
					tempDelay = 5 * time.Millisecond
				} else {
					tempDelay *= 2
				}
				if max := 1 * time.Second; tempDelay > max {
					tempDelay = max
				}
				srv.logf("http: Accept error: %v; retrying in %v", err, tempDelay)
				time.Sleep(tempDelay)
				continue
			}
			return err
		}
		connCtx := ctx
		if cc := srv.ConnContext; cc != nil {
			connCtx = cc(connCtx, rw)
			if connCtx == nil {
				panic("ConnContext returned nil")
			}
		}
		tempDelay = 0
		c := srv.newConn(rw)
		c.setState(c.rwc, StateNew, runHooks) // before Serve can return
		go c.serve(connCtx)
	}
}
```

- 服务启动时会创建一个循环来实时获取所有的访问请求，当有新的 `HTTP` 请求到来时，也就是 `l.Accept()` ，当获得对应上下文后，单独开启一个线程处理这一类的访问请求：

```go
// Serve a new connection.
func (c *conn) serve(ctx context.Context) {
	...
	defer func() {
		if err := recover(); err != nil && err != ErrAbortHandler {
			...
		}
		...
	}()

	if tlsConn, ok := c.rwc.(*tls.Conn); ok {
		...
		if err := tlsConn.HandshakeContext(ctx); err != nil {
			...
		}
		// Restore Conn-level deadlines.
		...
	}

	// HTTP/1.x from here on.
	...
	for {
		w, err := c.readRequest(ctx)
		...
		// HTTP cannot have multiple simultaneous active requests.[*]
		// Until the server replies to this request, it can't read another,
		// so we might as well run the handler in this goroutine.
		// [*] Not strictly true: HTTP pipelining. We could let them all process
		// in parallel even if their responses need to be serialized.
		// But we're not going to implement HTTP pipelining because it
		// was never deployed in the wild and the answer is HTTP/2.
		inFlightResponse = w
		serverHandler{c.server}.ServeHTTP(w, w.req)
		inFlightResponse = nil
		w.cancelCtx()
		if c.hijacked() {
			return
		}
		w.finishRequest()
		...
	}
}
```

- 显然，我们会在 `serverHandler.ServeHTTP()` 中处理具体的路由：

```go
func (sh serverHandler) ServeHTTP(rw ResponseWriter, req *Request) {
	handler := sh.srv.Handler
	if handler == nil {
		handler = DefaultServeMux
	}
	if req.RequestURI == "*" && req.Method == "OPTIONS" {
		handler = globalOptionsHandler{}
	}

	if req.URL != nil && strings.Contains(req.URL.RawQuery, ";") {
		var allowQuerySemicolonsInUse int32
		req = req.WithContext(context.WithValue(req.Context(), silenceSemWarnContextKey, func() {
			atomic.StoreInt32(&allowQuerySemicolonsInUse, 1)
		}))
		defer func() {
			if atomic.LoadInt32(&allowQuerySemicolonsInUse) == 0 {
				sh.srv.logf("http: URL query contains semicolon, which is no longer a supported separator; parts of the query may be stripped when parsed; see golang.org/issue/25192")
			}
		}()
	}

	handler.ServeHTTP(rw, req)
}
```

👇

- `handler` 指代了一系列的接口，有原生的也有接下来要介绍的 `Gin` 的，目前我们着眼于最简单的 `ServeMux` 结构体的 `ServeHTTP` 实现：

```go
// ServeHTTP dispatches the request to the handler whose
// pattern most closely matches the request URL.
func (mux *ServeMux) ServeHTTP(w ResponseWriter, r *Request) {
	if r.RequestURI == "*" {
		if r.ProtoAtLeast(1, 1) {
			w.Header().Set("Connection", "close")
		}
		w.WriteHeader(StatusBadRequest)
		return
	}
	h, _ := mux.Handler(r) // 从map中找到已注册的方法
	h.ServeHTTP(w, r)
}
```

👇

```go
// ServeHTTP calls f(w, r).
func (f HandlerFunc) ServeHTTP(w ResponseWriter, r *Request) {
	f(w, r)
}
```

![图片](images/image-http-defaultServeMux)



---

### `Gin` 的源码

- 从 `Gin` 的 `hello world demo` 开始：

  ```go
  package main
  
  import "github.com/gin-gonic/gin"
  
  func main() {
      r := gin.Default()
      r.GET("/ping", func(c *gin.Context) {
          c.JSON(200, gin.H{
            "message": "pong",
          })
      })
      r.Run() // listen and serve on 0.0.0.0:8080
  }
  ```



#### `r.Run()`

```go
// Run attaches the router to a http.Server and starts listening and serving HTTP requests.
// It is a shortcut for http.ListenAndServe(addr, router)
// Note: this method will block the calling goroutine indefinitely unless an error happens.
func (engine *Engine) Run(addr ...string) (err error) {
	defer func() { debugPrintError(err) }()

	if engine.isUnsafeTrustedProxies() {
		debugPrint("[WARNING] You trusted all proxies, this is NOT safe. We recommend you to set a value.\n" +
			"Please check https://pkg.go.dev/github.com/gin-gonic/gin#readme-don-t-trust-all-proxies for details.")
	}

	address := resolveAddress(addr)
	debugPrint("Listening and serving HTTP on %s\n", address)
	err = http.ListenAndServe(address, engine.Handler())
	return
}
```

- `r.Run()` 用于开启 `Gin` 的监听，他的底层是标准库 `net` 和 `net/http` ，跟一般使用 `net/http` 来做服务监听是一致的。

- 比起之前的版本，`Gin` 框架中多出了一个代理检查的过程（`engine.isUnsafeTrustedProxies()`），远离并不难理解，`gin.Engine` 会持有一些受信任的 `CIDR` ，通过 `IP` 和 `IPMask` 来判断是否使用了全域（`0.0.0.0` 或者是 `: : :`）而不是指定的 `IP` ：

```go
type Engine struct {
	...
    trustedCIDRs     []*net.IPNet
	...
}
```

```go
// isUnsafeTrustedProxies checks if Engine.trustedCIDRs contains all IPs, it's not safe if it has (returns true)
func (engine *Engine) isUnsafeTrustedProxies() bool {
	return engine.isTrustedProxy(net.ParseIP("0.0.0.0")) || engine.isTrustedProxy(net.ParseIP("::"))
}
```

```go
// isTrustedProxy will check whether the IP address is included in the trusted list according to Engine.trustedCIDRs
func (engine *Engine) isTrustedProxy(ip net.IP) bool {
	if engine.trustedCIDRs == nil {
		return false
	}
	for _, cidr := range engine.trustedCIDRs {
		if cidr.Contains(ip) {
			return true
		}
	}
	return false
}
```

- 也可以通过 `(engine *Engine) SetTrustedProxies(trustedProxies []string) error` 来设置对应的代理检查规则。

```go
if err := router.SetTrustedProxies([]string{"127.0.0.1"}); err != nil {
	panic(err)
}
```

- 如果使用类似于 `CDN` 的服务，那么可以通过设置 `Engine.TrustedPlatform` 去跳过 `TrustedProxies` 检查。

```go
// Use predefined header gin.PlatformXXX
router.TrustedPlatform = gin.PlatformGoogleAppEngine
// Or set your own trusted request header for another trusted proxy service
router.TrustedPlatform = "X-CDN-IP"
```



#### `gin.ServeHTTP()`

```go
// ServeHTTP conforms to the http.Handler interface.
func (engine *Engine) ServeHTTP(w http.ResponseWriter, req *http.Request) {
	c := engine.pool.Get().(*Context)
	c.writermem.reset(w)
	c.Request = req
	c.reset()

	engine.handleHTTPRequest(c)

	engine.pool.Put(c)
}
```

- 经典的 `sync.Pool` 操作，为了能够复用一些已分配的内存，而不是将有限的资源浪费在分配和GC中，`Gin` 的做法是在 `ServeHTTP()` 中对上下文初始化，`gin.Context` 将成为新的信息载体，用 `Engine.handleHTTPRequest()` 来处理请求。



#### `Gin` 路由

- 为了弄清原理，我们得在这里按下暂停键，转而去看一下 `Gin` 另一个重要的部分，**路由**。上述代码只停留在了初始化 `gin.Context` ，为了能够处理到达的请求，首先需要的是去找到对应的处理函数。这是通用的 `RESTful` 规范，是 `Web` 开发中默认的规范了，不然显得不专业：

```http
RESTful:    DELETE  /user/hhf
No RESTful: GET     /deleteUser?name=hhf

GET    /user/{userID} HTTP/1.1
POST   /user/{userID} HTTP/1.1
PUT    /user/{userID} HTTP/1.1
DELETE /user/{userID} HTTP/1.1
```

- `net/http` 只做了一个很懒的哈希表来存储这些**请求—处理**关系，对于少量的请求这似乎没什么，但是实际使用中就不得不重写路由。，`Gin` 结合了 `julienschmidt/httprouter` 的代码，创造出了自己的树状路由。

```go
// GET is a shortcut for router.Handle("GET", path, handle).
func (group *RouterGroup) GET(relativePath string, handlers ...HandlerFunc) IRoutes {
	return group.handle(http.MethodGet, relativePath, handlers)
}
```

- 这儿用 `GET` 方法作为例子，显然其他 `RESTful` 方法是同理的。往下调用：👇

```go
func (group *RouterGroup) handle(httpMethod, relativePath string, handlers HandlersChain) IRoutes {
	absolutePath := group.calculateAbsolutePath(relativePath)
	handlers = group.combineHandlers(handlers)
	group.engine.addRoute(httpMethod, absolutePath, handlers)
	return group.returnObj()
}
```

- 过程很明确，先是通过 `group.calculateAbsolutePath()` 计算**绝对路径**，毕竟 `Gin` 引入了分组的概念，所以每一次注册路由传入的相当于**相对路径**，最后，当然是给 `Engine` 加上处理好的**请求—处理**了：

```go
type Engine struct {
    ...
    trees            methodTrees
	...
}

type methodTree struct {
	method string
	root   *node
}

type methodTrees []methodTree
```

```go
func (engine *Engine) addRoute(method, path string, handlers HandlersChain) {
	...
	root := engine.trees.get(method)
	if root == nil {
		root = new(node)
		root.fullPath = "/"
		engine.trees = append(engine.trees, methodTree{method: method, root: root})
	}
	root.addRoute(path, handlers)

	// Update maxParams
	...
}
```

- 事实上，并不是每一个节点 `node` 就是一个完整的路径字段，比如在 `/v1/ping` 和 `/v2/ping` 中，`node` 切割出来的子串是 `/` `v` 然后才是不同，更加类似于字典树，而不是机械地按照 `/` 来分割，以 `POST` 为例：

![image-gin-routerPOST](images/image-gin-routerPOST.svg)

- 建完树后长这样：

![image-gin-routerTreePOST](images/image-gin-routerTreePOST.svg)

- 所以， `Gin` 里的每个节点都设计成这样：

```go
type node struct {
	path      string
	indices   string
	wildChild bool
	nType     nodeType
	priority  uint32
	children  []*node // child nodes, at most 1 :param style node at the end of the array
	handlers  HandlersChain
	fullPath  string
}
```

| `node` 元素 |                 描述                 |
| :---------: | :----------------------------------: |
|   `path`    |           当前节点部分路径           |
| `fullpath`  | 当前节点完整长度，从 `root` 开始计算 |
|  `indices`  |   孩子节点中可能会出现的下一个字符   |
| `handlers`  |       处理函数链（有先后顺序）       |
|     ...     |                 ...                  |

#### 回到 `gin.ServeHTTP()`

- 让我们回到 `gin.ServeHTTP()` ，之前的过程可以去看 [`gin.ServeHTTP()`](#`gin.ServeHTTP()`)，接下来的部分从找到对应的路由开始：

```go
func (engine *Engine) handleHTTPRequest(c *Context) {
	...
	// Find root of the tree for the given HTTP method
	t := engine.trees
	for i, tl := 0, len(t); i < tl; i++ {
		if t[i].method != httpMethod {
			continue
		}
		root := t[i].root
		// Find route in tree
		value := root.getValue(rPath, c.params, c.skippedNodes, unescape)
		if value.params != nil {
			c.Params = *value.params
		}
		if value.handlers != nil {
			c.handlers = value.handlers
			c.fullPath = value.fullPath
			c.Next()
			c.writermem.WriteHeaderNow()
			return
		}
		if httpMethod != http.MethodConnect && rPath != "/" {
			if value.tsr && engine.RedirectTrailingSlash {
				redirectTrailingSlash(c)
				return
			}
			if engine.RedirectFixedPath && redirectFixedPath(c, root, engine.RedirectFixedPath) {
				return
			}
		}
		break
	}

	if engine.HandleMethodNotAllowed {
		for _, tree := range engine.trees {
			if tree.method == httpMethod {
				continue
			}
			if value := tree.root.getValue(rPath, nil, c.skippedNodes, unescape); value.handlers != nil {
				c.handlers = engine.allNoMethod
				serveError(c, http.StatusMethodNotAllowed, default405Body)
				return
			}
		}
	}
	c.handlers = engine.allNoRoute
	serveError(c, http.StatusNotFound, default404Body)
}
```



#### `Bind()`

- 之前的使用中就提及了 `ShouldBindWith()` 和 `MustBindWith()` 两种不同的绑定策略，前者返回错误信息供使用者自行处理，而后者则是直接写入 `HTTP 400` ，如果你试图重写状态码，会在日志文件里记录一个 `Warning` 。
- 事实上，源码里两者毫无区别， `MustBindWith()` 直接调用了 `ShouldBindWith()` ：

```go
// MustBindWith binds the passed struct pointer using the specified binding engine.
// It will abort the request with HTTP 400 if any error occurs.
// See the binding package.
func (c *Context) MustBindWith(obj any, b binding.Binding) error {
	if err := c.ShouldBindWith(obj, b); err != nil {
		c.AbortWithError(http.StatusBadRequest, err).SetType(ErrorTypeBind) // nolint: errcheck
		return err
	}
	return nil
}
```

```go
// ShouldBindWith binds the passed struct pointer using the specified binding engine.
// See the binding package.
func (c *Context) ShouldBindWith(obj any, b binding.Binding) error {
	return b.Bind(c.Request, obj)
}
```

- 这里是接口调用，看一个 `jsonBinding` 的实现：

```go
func (jsonBinding) Bind(req *http.Request, obj any) error {
	if req == nil || req.Body == nil {
		return errors.New("invalid request")
	}
	return decodeJSON(req.Body, obj)
}

func decodeJSON(r io.Reader, obj any) error {
	decoder := json.NewDecoder(r)
	if EnableDecoderUseNumber {
		decoder.UseNumber()
	}
	if EnableDecoderDisallowUnknownFields {
		decoder.DisallowUnknownFields()
	}
	if err := decoder.Decode(obj); err != nil {
		return err
	}
	return validate(obj)
}
```



#### 返回值处理

- `net/http` 提供了简单的返回值格式，即 `text/plain` ，同时，返回也只有简单的 `http.ResponseWriter.Write()` 函数支撑，并不支持结构体的直接写入：

```go
func main() {
    http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
        w.Write([]byte("Hello World"))
    })

    if err := http.ListenAndServe(":8000", nil); err != nil {
        fmt.Println("start http server fail:", err)
    }
}
```

- `Gin` 提供了多种样式的返回值，在此仅以 `JSON` 格式为例看看 `Gin` 在 `ResponseWriter` 基础上拓展了哪些：

```go
c.JSON(http.StatusOK, map[string]interface{}{
	"lang": "Go语言",
    "tag":  "test<br>Test",
})
```

👇

```go
// JSON serializes the given struct as JSON into the response body.
// It also sets the Content-Type as "application/json".
func (c *Context) JSON(code int, obj any) {
	c.Render(code, render.JSON{Data: obj})
}
```

👇

```go
func (c *Context) Render(code int, r render.Render) {
	c.Status(code)
	// write ContentType & Header
	if err := r.Render(c.Writer); err != nil {
		panic(err)
	}
}
```

👇

```go
// Render (JSON) writes data with custom ContentType.
func (r JSON) Render(w http.ResponseWriter) (err error) {
	if err = WriteJSON(w, r.Data); err != nil {
		panic(err)
	}
	return
}
```

👇

```go
// WriteJSON marshals the given interface object and writes it with custom ContentType.
func WriteJSON(w http.ResponseWriter, obj any) error {
	writeContentType(w, jsonContentType)
	jsonBytes, err := json.Marshal(obj)
	if err != nil {
		return err
	}
	_, err = w.Write(jsonBytes)
	return err
}
```

- 底层还是 `ResopnseWriter.Write()` 和 `json.Marshal()` 😀



### 后记 2022.09.20

不得不感叹 `Gin` 简洁明了，接下来就是 `net/http` 的源码了。😀