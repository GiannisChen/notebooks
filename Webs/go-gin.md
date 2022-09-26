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



### `sqlx`

- `sqlx` 是在Go标准库 `database/sql` 基础上的拓展，他提供了基于 `database/sql` 的一定封装，同样，也没对操作过程过度封装。
- `sqlx` 将使用标准的 `error` 变量来指示正在返回的错误。



#### `sqlx` 安装 

- 我们将使用 `sqlx` 和 `clickhouse-go` 的组合来完成数据库的基础操作，包括增删改查（`CRUD`）。

  ```shell
  $ go get github.com/jmoiron/sqlx
  $ go get github.com/ClickHouse/clickhouse-go/v2
  ```

- 该连接数据库了：

  ```go
  import (
  	_ "github.com/ClickHouse/clickhouse-go/v2"
  	"github.com/jmoiron/sqlx"
  	"sync"
  )
  
  var Conn *sqlx.DB
  var once sync.Once
  
  func MustInitDB() {
  	once.Do(func() {
  		dsn := "clickhouse://default1:12345678@192.168.47.128:9000/gin"
  		Conn = sqlx.MustConnect("clickhouse", dsn)
  		Conn.SetMaxOpenConns(20)
  		Conn.SetMaxIdleConns(10)
  	})
  }
  ```

  由于我们相当于采用了第三方包 `clickhouse-go` ，因此需要多做一个引用来覆盖原有的配置，`sqlx.DB` 自身维持了一个连接池，所以我们大胆地使用全局变量来初始化。



#### `CRUD`

- 首先我们需要定义一下 `format` 格式：

  ```go
  type User struct {
  	FirstName string `form:"firstName" db:"firstName"`
  	LastName  string `form:"lastName" db:"lastName"`
  	Email     string `form:"email" db:"email" binding:"required,email"`
  }
  ```

  其中 `db:"lastName"` 就是做自动填充或者自动绑定时需要的标签名；

- 接下来把 `CRUD` 端上来罢~

  - **`GetAllUsers`**：

    ```go
    func GetAllUsers(users *[]User) error {
    	if err := Conn.Select(users, "SELECT * FROM gin.users"); err != nil {
    		return err
    	}
    	return nil
    }
    ```

  - **`GetUser`**：

    ```go
    func GetUser(user *User, email string) error {
    	if err := Conn.Get(user, "SELECT * FROM gin.users WHERE email=? LIMIT 1", email); err != nil {
    		return err
    	}
    	return nil
    }
    ```

  - **`InsertUsers`**：

    ```go
    func InsertUsers(users ...User) error {
    	if len(users) == 0 {
    		return fmt.Errorf("insert 0 user(s)")
    	}
    
    	for _, user := range users {
    		if _, err := Conn.NamedExec("INSERT INTO gin.users (email,firstName,lastName) VALUES (:email,:firstName,:lastName)", user); err != nil {
    			return err
    		}
    	}
    	return nil
    }
    ```

  - **`UpdateUsers`**：

    ```go
    func UpdateUsers(users ...User) error {
    	for _, user := range users {
    		if _, err := Conn.NamedExec("ALTER TABLE gin.users UPDATE firstName=:firstName,lastName=:lastName WHERE email=:email", user); err != nil {
    			return err
    		}
    	}
    	return nil
    }
    ```

  - **`DeleteUser`**：

    ```go
    func DeleteUser(email string) error {
    	if _, err := Conn.Exec("ALTER TABLE gin.users DELETE WHERE email=?", email); err != nil {
    		return err
    	}
    
    	return nil
    }
    ```

#### 细节

- 具体用法参考：https://jmoiron.github.io/sqlx/

#### 特殊的 `sqlx.In`

- **插入**

  - 前提是需要结构体实现 `driver.Valuer` 接口：

    ```go
    func (u User) Value() (driver.Value, error) {
    	return []interface{}{u.Email, u.FirstName, u.LastName}, nil
    }
    ```

  - 使用`sqlx.In`实现批量插入代码如下：

    ```go
    // BatchInsertUsers2 使用sqlx.In帮我们拼接语句和参数, 注意传入的参数是[]interface{}
    func BatchInsertUsers2(users []interface{}) error {
    	query, args, _ := sqlx.In(
    		"INSERT INTO users (email, firstName, lastName) VALUES (?), (?), (?)",users...) 	
        // 如果arg实现了 driver.Valuer, sqlx.In 会通过调用 Value()来展开它
    	fmt.Println(query) // 查看生成的querystring
    	fmt.Println(args)  // 查看生成的args
    	_, err := DB.Exec(query, args...)
    	return err
    }
    ```

- **查询**

  - 在`sqlx`查询语句中实现In查询和FIND_IN_SET函数。即实现`SELECT * FROM user WHERE id IN (3, 2, 1);`和`SELECT * FROM user WHERE id IN (3, 2, 1) ORDER BY FIND_IN_SET(id, '3,2,1');`：

    ```go
    // QueryByIDs 根据给定emails查询
    func QueryByEmails(emails []string)(users []User, err error){
    	// 动态填充id
    	query, args, err := sqlx.In("SELECT firstName, lastName FROM users WHERE email IN (?)", emails)
    	if err != nil {
    		return
    	}
    	// sqlx.In 返回带 `?` bindvar的查询语句, 我们使用Rebind()重新绑定它
    	query = DB.Rebind(query)
    
    	err = DB.Select(&users, query, args...)
    	return
    }
    ```



#### 参考

- https://www.liwenzhou.com/posts/Go/sqlx/
- https://www.liwenzhou.com/posts/Go/sqlx_bulk_insert/
- https://jmoiron.github.io/sqlx/
- https://github.com/ClickHouse/clickhouse-go



### `JWT`

#### 介绍

- `JSON Web Token` 是一个开放标准 ( [RFC 7519](https://tools.ietf.org/html/rfc7519) )，它定义了一种紧凑且自包含的方式，用于在各方之间以 `JSON` 对象的形式安全传输信息。此信息可以验证和信任，因为它是数字签名的。`JWT` 可以使用密钥（使用`HMAC` 算法）或使用 `RSA` 或 `ECDSA` 的公钥/私钥对进行签名。

- 虽然 `JWT` 可以加密以在各方之间提供保密性，但我们将专注于*签名*令牌。签名的令牌可以验证其中包含的声明的*完整性*，而加密的令牌会向第三方*隐藏*这些声明。当使用公钥/私钥对对令牌进行签名时，签名还能证明只有持有私钥的一方才是签署它的一方。

- 以下是 `JWT` 的**使用场景**：

  - **授权**：这是使用 `JWT` 最常见的场景。用户登录后，每个后续请求都将包含 `JWT`，从而允许用户访问该令牌允许的路由、服务和资源。单点登录是当今广泛使用 `JWT` 的一项功能，因为它的开销很小并且能够在不同的域中轻松使用。
  - **信息交换**：`JWT` 是在各方之间安全传输信息的好方法。因为可以对 `JWT` 进行签名（例如，使用公钥/私钥对），所以您可以确定发件人就是他们所说的那个人。此外，由于使用标头和有效负载计算签名，您还可以验证内容没有被篡改。

- `JWT` **结构**由以点 ( `.`) 分隔的三部分组成，它们是：（`xxxxx.yyyyy.zzzzz`）

  - **标头**（`Header`）

    标头*通常*由两部分组成：令牌的类型，即 `JWT`，以及正在使用的签名算法，例如 `HMAC` `SHA256` 或 `RSA` ，比如下面的 `JSON` 将被 `Base64Url` 方法加密成为 `JWT` 的第一部分。

    ```json
    {
      "alg": "HS256",
      "typ": "JWT"
    }
    ```

  - **有效载荷**（`Payload`）

    令牌的第二部分是有效负载，其中包含声明（`claims`）。声明是关于实体（通常是用户）和附加数据的声明。声明分为三种类型：**注册声明**、**公开声明**和**私人声明**：

    - [**注册声明**](https://tools.ietf.org/html/rfc7519#section-4.1)：这些是一组预定义的声明，它们不是强制性的，但建议使用，以提供一组有用的、可互操作的声明。其中一些是： **iss**（`issuser`）、 **exp**（`expiration time`）、 **sub**（`subject`）、 **aud**（`audience`）[等](https://tools.ietf.org/html/rfc7519#section-4.1)。

      > 请注意，声明名称只有三个字符，只要 JWT 是紧凑的。

    - [**公共声明**](https://tools.ietf.org/html/rfc7519#section-4.2)：这些可以由使用 `JWT` 的人随意定义。但是为了避免冲突，它们应该在[IANA JSON Web Token Registry](https://www.iana.org/assignments/jwt/jwt.xhtml)中定义，或者定义为包含抗冲突命名空间的 `URI`。

    - [**私人声明**](https://tools.ietf.org/html/rfc7519#section-4.3)：这些是为在同意使用它们的各方之间共享信息而创建的自定义声明，既不是**注册声明**也不是**公共声明**。

    ```json
    {
      "sub": "1234567890",
      "name": "John Doe",
      "admin": true
    }
    ```

    > 请注意，对于已签名的令牌，此信息虽然受到保护以防篡改，但任何人都可以读取。除非已加密，否则**请勿**将机密信息放入 `JWT` 的有效负载或标头元素中。

  - **签名**（`Signature`）

    要创建签名部分，您必须获取编码的标头、编码的有效负载、秘密、标头中指定的算法，并对其进行签名。例如，如果您想使用 `HMAC SHA256` 算法，签名将通过以下方式创建：

    ```c
    HMACSHA256(base64UrlEncode(header) + "." + base64UrlEncode(payload), secret)
    ```

    签名用于验证消息在此过程中没有被更改，并且在使用私钥签名的令牌的情况下，它还可以验证 `JWT` 的发送者就是它所说的那个人。

  - **总结**

    输出是三个用点分隔的 `Base64-URL` 字符串，可以在 `HTML` 和 `HTTP` 环境中轻松传递，同时与基于 `XML` 的标准（如 `SAML`）相比更紧凑。

    ![JWT.io 调试器](images/legacy-app-auth.png)

#### `JWT` 使用流程

- 在身份验证中，当用户使用其凭据成功登录时，将返回一个 `JSON Web Token`。由于令牌是凭据，因此必须非常小心以防止出现安全问题，通常，您不应将令牌保留超过所需的时间。由于缺乏安全性，也**不应该**在浏览器存储中存储敏感的会话数据。

- 每当用户想要访问受保护的路由或资源时，用户代理应该发送 `JWT`，通常在**Authorization**标头中使用**Bearer**模式。标头的内容应如下所示：

  ```json
  Authorization: Bearer <token>
  ```

![JSON Web 令牌如何工作](images/client-credentials-grant.png)

1. 应用程序或客户端向授权服务器请求授权。这是通过不同的授权流程之一执行的。例如，一个典型的符合[OpenID Connect](http://openid.net/connect/)的Web应用程序将使用[授权代码流](http://openid.net/specs/openid-connect-core-1_0.html#CodeFlowAuth)`/oauth/authorize`通过端点。
2. 当授权被授予时，授权服务器向应用程序返回一个访问令牌。
3. 应用程序使用访问令牌访问受保护的资源（如API）。

#### 代码

犯懒了，不想写了，看这吧：https://www.liwenzhou.com/posts/Go/json-web-token



### `swagger`  接口文档

#### 介绍

- `swagger` 本质上是一种用于描述使用 `JSON` 表示的 `RESTful` API的接口描述语言。`swagger` 与一组开源软件工具一起使用，以设计、构建、记录和使用 `RESTful` Web服务。`swagger` 包括自动文档，代码生成和测试用例生成。

- 在前后端分离的项目开发过程中，如果后端同学能够提供一份清晰明了的接口文档，那么就能极大地提高大家的沟通效率和开发效率。可是编写接口文档历来都是令人头痛的，而且后续接口文档的维护也十分耗费精力。

- 最好是有一种方案能够既满足我们输出文档的需要又能随代码的变更自动更新，而 `swagger` 正是那种能帮我们解决接口文档问题的工具。

- 使用 `gin-swagger` 库以使用 `swagger 2.0` 自动生成 `RESTful` API文档。
- **参考**：
  - **注释规范**：https://swaggo.github.io/swaggo.io/declarative_comments_format/general_api_info.html
  - **项目地址**：https://github.com/swaggo/gin-swagger

#### 代码

1. 在 `main.go` 中配置（通过注释，这点就没有Java好了）：

   ```go
   // @title gin-test
   // @version 1.0
   // @description a test for gin
   // @termsOfService http://swagger.io/terms/
   
   // @contact.name GiannisChen
   // @contact.url http://www.swagger.io/support
   // @contact.email support@swagger.io
   
   // @license.name Apache 2.0
   // @license.url http://www.apache.org/licenses/LICENSE-2.0.html
   
   // @host localhost:8080
   // @BasePath /
   
   // @securityDefinitions.apikey  ApiKeyAuth
   // @in                          header
   // @name                        Authorization
   // @description					Description for what is this security definition being used
   func main() {
       ...
   }
   ```

   - `host` 得写完整；
   - `BasePath` 会作为公共前缀拼接上去；
   - 开了 `JWT` 的认证，这和Java是一样的，配合上一章节使用；

2. 在 `controller` 层注册具体路由：

   ```go
   // TestSwagger swagger test
   // @Summary
   // @Description for test
   // @Tags swagger
   // @Accept application/json
   // @Produce application/json
   // @Param object query models.User false "query param"
   // @Security ApiKeyAuth
   // @Success 200 {object} _Response
   // @Router /admin/swagger [post]
   func TestSwagger(c *gin.Context) {
   	var user models.User
   	if err := c.ShouldBind(&user); err != nil {
   		c.JSON(400, _Response{Code: 400, Message: err.Error()})
   		return
   	}
   	if marshal, err := json.Marshal(user); err != nil {
   		c.JSON(400, _Response{Code: 400, Message: err.Error()})
   	} else {
   		c.JSON(200, _Response{Code: 200, Message: string(marshal)})
   	}
   }
   
   // AuthHandler generate auth
   // @Summary
   // @Description for test
   // @Tags auth
   // @Accept application/json
   // @Produce application/json
   // @Security ApiKeyAuth
   // @Success 200 {object} _Response
   // @Router /auth [get]
   func AuthHandler(c *gin.Context) {
   	str, _ := auth.GenRegisteredClaims()
   	c.JSON(http.StatusOK, _Response{Code: 200, Message: "Bearer:" + str})
   }
   
   // _Response 帖子列表接口响应数据
   type _Response struct {
   	Code    int    `json:"code"`    // 业务响应状态码
   	Message string `json:"message"` // 提示信息
   }
   ```

3. 准备好了注释，需要自动生成环节了：

   安装 `swag` 工具：

   ```shell
   $ go get -u github.com/swaggo/swag/cmd/swag
   ```

   在**项目根目录**下生成：

   ```shell
   $ swag init
   ```

   执行完后会有日志和新的文件夹：

   > ```
   > 2022/09/23 22:03:39 Generate swagger docs....
   > 2022/09/23 22:03:39 Generate general API Info, search dir:./
   > 2022/09/23 22:03:39 Generating models.User
   > 2022/09/23 22:03:39 Generating controller._Response
   > 2022/09/23 22:03:39 create docs.go at  docs/docs.go
   > 2022/09/23 22:03:39 create swagger.json at  docs/swagger.json
   > 2022/09/23 22:03:39 create swagger.yaml at  docs/swagger.yaml
   > ```

   > ```bash
   > ./docs
   > ├── docs.go
   > ├── swagger.json
   > └── swagger.yaml
   > ```

4. 导入相关包：

   ```shell
   $ go get github.com/swaggo/files
   $ go get github.com/swaggo/gin-swagger
   ```

5. 然后需要注册 `swagger` 的访问路由：

   ```go
   import swaggerFiles "github.com/swaggo/files"
   import gs "github.com/swaggo/gin-swagger"
   
   r.GET("/swagger/*any", gs.WrapHandler(swaggerFiles.Handler))
   ```

6. 最后在 `main.go` 中引入我们自己的 `docs` ：

   ```go
   import 	_ "Gin-Test/test/docs"
   ```

7. 大功告成！！！

   ![image-20220923224346873](images/gintest-web-swagger.png)



### 限流策略（`rate limit`）

- 限流又称为流量控制（流控），通常是指限制到达系统的并发请求数。限流虽然会影响部分用户的使用体验，但是却能在一定程度上报障系统的稳定性，不至于崩溃（大家都没了用户体验）。

  而互联网上类似需要限流的业务场景也有很多：

  - 电商系统的秒杀、微博上突发热点新闻、双十一购物节、12306抢票等等，这些场景下的用户请求量通常会激增，远远超过平时正常的请求量，此时如果不加任何限制很容易就会将后端服务打垮，影响服务的稳定性。
  - 一些厂商公开的API服务通常也会限制用户的请求次数，比如百度地图开放平台等会根据用户的付费情况来限制用户的请求数等。

- 常用的限流策略有两种：

  - [**漏桶**](#漏桶)（`leaky-bucket algorithm`）
  - [**令牌桶**](#令牌桶)（`token bucket algorithm`）

#### 漏桶

- 漏桶法的关键点在于漏桶始终按照固定的速率运行，但是它并不能很好的处理有大量突发请求的场景，毕竟在某些场景下我们可能需要提高系统的处理效率，而不是一味的按照固定速率处理请求。

- 参考：https://github.com/uber-go/ratelimit

  ```go
  package main
  
  import (
  	"fmt"
  	"go.uber.org/ratelimit"
  	"time"
  )
  
  func main() {
  	limiter := ratelimit.New(10)
  	prev := time.Now()
  	for i := 0; i < 10; i++ {
  		now := limiter.Take()
  		fmt.Println(i, now.Sub(prev))
  		prev = now
  	}
  }
  
  // 0 0s
  // 1 100ms
  // 2 100ms
  // 3 100ms
  // 4 100ms
  // 5 100ms
  // 6 100ms
  // 7 100ms
  // 8 100ms
  // 9 100ms
  ```

- 限制器是一个接口类型，有三种不同的实现方式（`atomic limiter`，`mutex limiter`，`unlimited`），不过官方只使用了其中一种（`atomic limiter`）：

  ```go
  type state struct {
  	last     time.Time			// 上一次的时刻
  	sleepFor time.Duration		// 需要等待的时间
  }
  
  type atomicLimiter struct {
  	state unsafe.Pointer
  	//lint:ignore U1000 Padding is unused but it is crucial to maintain performance
  	// of this rate limiter in case of collocation with other frequently accessed memory.
  	padding [56]byte // cache line size - state pointer size = 64 - 8; created to avoid false sharing.
  
  	perRequest time.Duration	// 每次的时间间隔
  	maxSlack   time.Duration	// 最大的富余量
  	clock      Clock			// 时钟
  }
  ```

- `newAtomicBased`初始化一个限流器，用 `atomic.StorePointer()` 来确保同步运行和并发：

  ```go
  func newAtomicBased(rate int, opts ...Option) *atomicLimiter {
  	// TODO consider moving config building to the implementation
  	// independent code.
  	config := buildConfig(opts)
  	perRequest := config.per / time.Duration(rate)
  	l := &atomicLimiter{
  		perRequest: perRequest,
  		maxSlack:   -1 * time.Duration(config.slack) * perRequest,
  		clock:      config.clock,
  	}
  
  	initialState := state{
  		last:     time.Time{},
  		sleepFor: 0,
  	}
  	atomic.StorePointer(&l.state, unsafe.Pointer(&initialState))
  	return l
  }
  ```

- `Take()` 阻塞一定时间并返回：

  ```go
  // Take blocks to ensure that the time spent between multiple
  // Take calls is on average time.Second/rate.
  func (t *atomicLimiter) Take() time.Time {
  	var (
  		newState state
  		taken    bool
  		interval time.Duration
  	)
  	for !taken {
  		now := t.clock.Now()
  
  		previousStatePointer := atomic.LoadPointer(&t.state)
  		oldState := (*state)(previousStatePointer)
  
  		newState = state{
  			last:     now,
  			sleepFor: oldState.sleepFor,
  		}
  
  		// If this is our first request, then we allow it.
  		if oldState.last.IsZero() {
  			taken = atomic.CompareAndSwapPointer(&t.state, previousStatePointer, unsafe.Pointer(&newState))
  			continue
  		}
  
  		// sleepFor calculates how much time we should sleep based on
  		// the perRequest budget and how long the last request took.
  		// Since the request may take longer than the budget, this number
  		// can get negative, and is summed across requests.
  		newState.sleepFor += t.perRequest - now.Sub(oldState.last)
  		// We shouldn't allow sleepFor to get too negative, since it would mean that
  		// a service that slowed down a lot for a short period of time would get
  		// a much higher RPS following that.
  		if newState.sleepFor < t.maxSlack {
  			newState.sleepFor = t.maxSlack
  		}
  		if newState.sleepFor > 0 {
  			newState.last = newState.last.Add(newState.sleepFor)
  			interval, newState.sleepFor = newState.sleepFor, 0
  		}
  		taken = atomic.CompareAndSwapPointer(&t.state, previousStatePointer, unsafe.Pointer(&newState))
  	}
  	t.clock.Sleep(interval)
  	return newState.last
  }
  ```

- 利用 `atomic.CompareAndSwapPointer()` 来达到并行访问的目的，复用了 `atomicLimiter` 的结构但是每次更新 `state` 这个结构体，最后通过 `Clock.Sleep` 来限流。

- `unlimited` 干脆啥都没做，而 `mutexLimiter` 详见：https://www.liwenzhou.com/posts/Go/ratelimit/

#### 令牌桶

<img src="images/lingpaitong.jpg" alt="令牌桶原理" style="zoom: 33%;" />

- 令牌桶其实和漏桶的原理类似，令牌桶按固定的速率往桶里放入令牌，并且只要能从桶里取出令牌就能通过，令牌桶支持突发流量的快速处理。对于从桶里取不到令牌的场景，我们可以选择等待也可以直接拒绝并返回。

- 参考：https://github.com/juju/ratelimit

- 创建令牌桶：

  ```go
  // 创建指定填充速率和容量大小的令牌桶
  func NewBucket(fillInterval time.Duration, capacity int64) *Bucket
  // 创建指定填充速率、容量大小和每次填充的令牌数的令牌桶
  func NewBucketWithQuantum(fillInterval time.Duration, capacity, quantum int64) *Bucket
  // 创建填充速度为指定速率和容量大小的令牌桶
  // NewBucketWithRate(0.1, 200) 表示每秒填充20个令牌
  func NewBucketWithRate(rate float64, capacity int64) *Bucket
  ```

- 取出令牌的方法：

  ```go
  // 取token（非阻塞）
  func (tb *Bucket) Take(count int64) time.Duration
  func (tb *Bucket) TakeAvailable(count int64) int64
  
  // 最多等maxWait时间取token
  func (tb *Bucket) TakeMaxDuration(count int64, maxWait time.Duration) (time.Duration, bool)
  
  // 取token（阻塞）
  func (tb *Bucket) Wait(count int64)
  func (tb *Bucket) WaitMaxDuration(count int64, maxWait time.Duration) bool
  ```

- 虽说是令牌桶，但是我们没有必要真的去生成令牌放到桶里，我们只需要每次来取令牌的时候计算一下，当前是否有足够的令牌就可以了，具体的计算方式可以总结为下面的公式：

  > ```bash
  > 当前令牌数 = 上一次剩余的令牌数 + (本次取令牌的时刻-上一次取令牌的时刻)/放置令牌的时间间隔 * 每次放置的令牌数
  > ```

  ```go
  // currentTick returns the current time tick, measured
  // from tb.startTime.
  func (tb *Bucket) currentTick(now time.Time) int64 {
  	return int64(now.Sub(tb.startTime) / tb.fillInterval)
  }
  ```

  ```go
  // adjustavailableTokens adjusts the current number of tokens
  // available in the bucket at the given time, which must
  // be in the future (positive) with respect to tb.latestTick.
  func (tb *Bucket) adjustavailableTokens(tick int64) {
  	lastTick := tb.latestTick
  	tb.latestTick = tick
  	if tb.availableTokens >= tb.capacity {
  		return
  	}
  	tb.availableTokens += (tick - lastTick) * tb.quantum
  	if tb.availableTokens > tb.capacity {
  		tb.availableTokens = tb.capacity
  	}
  	return
  }
  ```

- 获得令牌：

  ```go
  // takeAvailable is the internal version of TakeAvailable - it takes the
  // current time as an argument to enable easy testing.
  func (tb *Bucket) takeAvailable(now time.Time, count int64) int64 {
  	if count <= 0 {
  		return 0
  	}
  	tb.adjustavailableTokens(tb.currentTick(now))
  	if tb.availableTokens <= 0 {
  		return 0
  	}
  	if count > tb.availableTokens {
  		count = tb.availableTokens
  	}
  	tb.availableTokens -= count
  	return count
  }
  ```

- 在 `Gin` 使用限流中间件：

  ```go
  func RateLimitMiddleware(fillInterval time.Duration, cap int64) func(c *gin.Context) {
  	bucket := ratelimit.NewBucket(fillInterval, cap)
  	return func(c *gin.Context) {
  		// 如果取不到令牌就中断本次请求返回 rate limit...
  		if bucket.TakeAvailable(1) < 1 {
  			c.String(http.StatusOK, "rate limit...")
  			c.Abort()
  			return
  		}
  		c.Next()
  	}
  }
  ```



### `Viper`

- 参考并翻译自：https://github.com/spf13/viper

- 是适用于Go应用程序的完整**配置**解决方案。它被设计用于在应用程序中工作，并且可以处理**所有类型的配置需求和格式**。

#### 安装

```go
$ go get github.com/spf13/viper
```

#### 特性

- 设置默认值
- 从 `JSON` 、 `TOML` 、 `YAML` 、 `HCL` 、 `envfile` 和 `Java properties` 格式的配置文件读取配置信息
- 实时监控和重新读取配置文件（可选）
- 从环境变量中读取
- 从远程配置系统（**etcd**或**Consul**）读取并监控配置变化
- 从命令行参数读取配置
- 从buffer读取配置
- 显式配置值

#### 功能

1. 查找、加载和反序列化 `JSON` 、 `TOML` 、 `YAML` 、 `HCL` 、 `INI` 、 `envfile` 和 `Java properties` 格式的配置文件。
2. 提供一种机制为你的不同配置选项设置默认值。
3. 提供一种机制来通过命令行参数覆盖指定选项的值。
4. 提供别名系统，以便在不破坏现有代码的情况下轻松重命名参数。
5. 当用户提供了与默认值相同的命令行或配置文件时，可以很容易地分辨出它们之间的区别。

#### 优先级

![gin-viper-priority](images/gin-viper-priority.svg)



#### 存

##### 设置默认值

- 一个好的配置系统应该支持**默认值**。`Key` 不需要默认值，但如果没有通过配置文件、环境变量、远程配置或命令行标志（`flag`）设置 `Key` 对应的 `Value`，则默认值非常有用。

  ```go
  viper.SetDefault("ContentDir", "content")
  viper.SetDefault("LayoutDir", "layouts")
  viper.SetDefault("Taxonomies", map[string]string{"tag": "tags", "category": "categories"})
  ```

##### 读取配置文件

- `Viper` 需要最少知道在哪里查找配置文件的配置。`Viper` 支持`JSON`、`TOML`、`YAML`、`HCL`、`envfile`和`Java properties`格式的配置文件。`Viper` 可以**搜索多个路径**，但目前单个 `Viper` 实例**只支持单个配置文件**。`Viper` 不默认任何配置搜索路径，将默认决策留给应用程序。下面是一个如何使用 `Viper` 搜索和读取配置文件的示例。不需要任何特定的路径，但是至少应该提供一个配置文件预期出现的路径：

  ```go
  viper.SetConfigFile("./config.yaml") // 指定配置文件路径
  viper.SetConfigName("config") // 配置文件名称(无扩展名)
  viper.SetConfigType("yaml") // 如果配置文件的名称中没有扩展名，则需要配置此项
  viper.AddConfigPath("/etc/appname/")   // 查找配置文件所在的路径
  viper.AddConfigPath("$HOME/.appname")  // 多次调用以添加多个搜索路径
  viper.AddConfigPath(".")               // 还可以在工作目录中查找配置
  err := viper.ReadInConfig() // 查找并读取配置文件
  if err != nil { // 处理读取配置文件的错误
  	panic(fmt.Errorf("Fatal error config file: %s \n", err))
  }
  ```

  在加载配置文件出错时，你可以像下面这样处理**找不到配置文件**的特定情况：

  ```go
  if err := viper.ReadInConfig(); err != nil {
      if _, ok := err.(viper.ConfigFileNotFoundError); ok {
          // 配置文件未找到错误；如果需要可以忽略
      } else {
          // 配置文件被找到，但产生了另外的错误
      }
  }
  
  // 配置文件找到并成功解析
  ```

  > 你也可以有不带扩展名的文件，并以编程方式指定其格式。对于位于用户 `$HOME` 目录中的配置文件没有任何扩展名，如 `.bashrc` 。

- 那么，如果我**不指定后缀名**呢？当你使用如下方式读取配置时，viper会从 `./conf` 目录下查找任何以 `config` 为文件名的配置文件，如果同时存在 `./conf/config.json` 和 `./conf/config.yaml` 两个配置文件的话， `viper` 会读取 `config.json` ，而不是 `config.yaml` ：

  ```go
  viper.SetConfigName("config")
  viper.AddConfigPath("./conf")
  ```

- 原理很直白，有一个**全局变量**，按顺序调用就行了：

  ```go
  var SupportedExts = []string{"json", "toml", "yaml", "yml", "properties", "props", "prop", "hcl", "tfvars", "dotenv", "env", "ini"}
  ```

  ```go
  viper.SetConfigType("yaml") 
  ```

  ```go
  func SetConfigType(in string) { v.SetConfigType(in) }
  ```

  ```go
  func (v *Viper) SetConfigType(in string) {
  	if in != "" {
  		v.configType = in
  	}
  }
  ```

- **傻逼**的地方来了，顺着源码一路往下找：

  ```go
  // ReadInConfig will discover and load the configuration file from disk
  // and key/value stores, searching in one of the defined paths.
  func ReadInConfig() error { return v.ReadInConfig() }
  ```

  ```go
  func (v *Viper) ReadInConfig() error {
  	...
  	filename, err := v.getConfigFile()
  	...
  }
  ```

  ```go
  func (v *Viper) searchInPath(in string) (filename string) {
  	...
  	for _, ext := range SupportedExts {
  		...
  		if b, _ := exists(v.fs, filepath.Join(in, v.configName+"."+ext)); b {
  			...
  			return filepath.Join(in, v.configName+"."+ext)
  		}
  	}
  
  	if v.configType != "" {
  		if b, _ := exists(v.fs, filepath.Join(in, v.configName)); b {
  			return filepath.Join(in, v.configName)
  		}
  	}
  
  	return ""
  }
  ```

- 也就是说，如果我也是个**傻逼**，把读取的代码写成这样👇，然后把文件配置成这样：

  ```go
  viper.SetConfigName("config")        // 配置文件名称(无扩展名)
  viper.SetConfigType("yaml")          // 如果配置文件的名称中没有扩展名，则需要配置此项
  viper.AddConfigPath("./test/config") // 查找配置文件所在的路径
  ```

  <img src="images/gin-viper-ops.png" alt="image-20220926154856107" style="zoom: 67%;" />

- 我们读取的**结果是**：😓

  ```bash
  content_json layouts_json map[category:categories_json tag:tags_json]
  ```

- 预期结果呢：

  ```bash
  content_yaml layouts_yaml map[category:categories_yaml tag:tags_yaml]
  ```

  ![一想到我是个傻逼,许多问题就迎刃而解了](images/viper-fuck-me.jpg)

> 让我们收拾一下心情，继续吧🙂



##### 写入配置文件

- 从配置文件中读取配置文件是有用的，但是有时你想要**存储**在**运行时**所做的所有修改。为此，可以使用下面一组命令，每个命令都有自己的用途：

  > **程序**影响**配置文件**

  - `WriteConfig()` - 将当前的 `viper` 配置写入预定义的路径并**覆盖**（如果存在的话）。如果没有预定义的路径，则报错。
  - `SafeWriteConfig()` - 将当前的 `viper` 配置写入预定义的路径。如果没有预定义的路径，则报错。如果存在，将**不会覆盖**当前的配置文件。
  - `WriteConfigAs()` - 将当前的 `viper` 配置写入给定的文件路径。将**覆盖**给定的文件（如果它存在的话）。
  - `SafeWriteConfigAs()` - 将当前的 `viper` 配置写入给定的文件路径。**不会覆盖**给定的文件（如果它存在的话）。

  ```go
  viper.WriteConfig() // 将当前配置写入“viper.AddConfigPath()”和“viper.SetConfigName”设置的预定义路径
  viper.SafeWriteConfig()
  viper.WriteConfigAs("/path/to/my/.config")
  viper.SafeWriteConfigAs("/path/to/my/.config") // 因为该配置文件写入过，所以会报错
  viper.SafeWriteConfigAs("/path/to/my/.other_config")
  ```

##### 监控并重新读取配置文件

- `viper` 支持在**运行时实时**读取配置文件的功能，需要重新启动服务器以使配置生效的日子已经一去不复返了， `viper` 驱动的应用程序可以在运行时读取配置文件的更新，而不会错过任何消息。只需告诉 `viper` 实例 `watchConfig()` 。可选地，你可以为 `viper` 提供一个回调函数，以便在每次发生更改时运行。

  > **配置文件**影响**程序**

  ```go
  viper.WatchConfig()
  viper.OnConfigChange(func(e fsnotify.Event) {
    // 配置文件发生变更之后会调用的回调函数
  	fmt.Println("Config file changed:", e.Name)
  })
  ```

- 底层原理是经典的 `fsnotify` : https://github.com/fsnotify/fsnotify

##### 从io.Reader读取配置

- `viper` 预先定义了许多配置源，如文件、环境变量、标志和远程K/V存储，但你不受其约束。你还可以实现自己所需的配置源并将其提供给 `viper`：

  ```go
  viper.SetConfigType("yaml") // 或者 viper.SetConfigType("YAML")
  
  // 任何需要将此配置添加到程序中的方法。
  var yamlExample = []byte(`
  Hacker: true
  name: steve
  hobbies:
  - skateboarding
  - snowboarding
  - go
  clothing:
    jacket: leather
    trousers: denim
  age: 35
  eyes : brown
  beard: true
  `)
  
  viper.ReadConfig(bytes.NewBuffer(yamlExample))
  
  viper.Get("name") // 这里会得到 "steve"
  ```

##### 覆盖设置

- 这些可能来自命令行标志，也可能来自你自己的应用程序逻辑：

  ```go
  viper.Set("Verbose", true)
  viper.Set("LogFile", LogFile)
  ```

##### 注册和使用别名

- 别名允许多个键引用单个值：

  ```go
  viper.RegisterAlias("loud", "Verbose")  // 注册别名（此处loud和Verbose建立了别名）
  
  viper.Set("verbose", true) // 结果与下一行相同
  viper.Set("loud", true)   // 结果与前一行相同
  
  viper.GetBool("loud") // true
  viper.GetBool("verbose") // true
  ```

##### 使用环境变量

- `viper` 完全支持环境变量。这使 `Twelve-Factor App` 开箱即用。有五种方法可以帮助与ENV协作:

  - `AutomaticEnv()`

  - `BindEnv(string...) : error`

  - `SetEnvPrefix(string)`

  - `SetEnvKeyReplacer(string...) *strings.Replacer`

  - `AllowEmptyEnv(bool)`

> *使用 `ENV` 变量时，务必要意识到 `viper` 将 `ENV` 变量视为区分大小写。*

- `viper` 提供了一种机制来确保 `ENV` 变量是惟一的。通过使用 `SetEnvPrefix` ，你可以告诉 `viper` 在读取环境变量时使用前缀。`BindEnv `和 `AutomaticEnv `都将使用这个前缀。

  `BindEnv` 使用一个或两个参数。第一个参数是键名称，第二个是环境变量的名称。环境变量的名称区分大小写。如果没有提供 `ENV` 变量名，那么 `viper` 将自动假设 `ENV` 变量与以下格式匹配：前缀 `+ “_” +` 键名全部大写。当你显式提供 `ENV` 变量名（第二个参数）时，它 **不会** 自动添加前缀。例如，如果第二个参数是 `id` ，`viper` 将查找环境变量 `ID` 。

  在使用 `ENV` 变量时，需要注意的一件重要事情是，每次访问该值时都将读取它。`viper` 在调用 `BindEnv` 时不固定该值。

  `AutomaticEnv` 是一个强大的助手，尤其是与 `SetEnvPrefix` 结合使用时。调用时，`viper` 会在发出 `viper.Get` 请求时随时检查环境变量。它将应用以下规则。它将检查环境变量的名称是否与键匹配（如果设置了 `EnvPrefix` ）。

  `SetEnvKeyReplacer` 允许你使用 `strings.Replacer` 对象在一定程度上重写 `Env` 键。如果你希望在 `Get()` 调用中使用 `-` 或者其他什么符号，但是环境变量里使用 `_` 分隔符，那么这个功能是非常有用的。可以在 `viper_test.go` 中找到它的使用示例。

  或者，你可以使用带有 `NewWithOptions` 工厂函数的 `EnvKeyReplacer` 。与 `SetEnvKeyReplacer` 不同，它接受 `StringReplacer` 接口，允许你编写自定义字符串替换逻辑。

  默认情况下，空环境变量被认为是未设置的，并将返回到下一个配置源。若要将空环境变量视为已设置，请使用 `AllowEmptyEnv` 方法。

- `ENV` 示例：

  ```go
  SetEnvPrefix("spf") // 将自动转为大写
  BindEnv("id")
  
  os.Setenv("SPF_ID", "13") // 通常是在应用程序之外完成的
  
  id := Get("id") // 13
  ```

##### 使用Flags

- `viper` 具有绑定到标志的能力。具体来说，`viper` 支持[Cobra](https://github.com/spf13/cobra)库中使用的`Pflag`。与`BindEnv`类似，该值不是在调用绑定方法时设置的，而是在访问该方法时设置的。这意味着你可以根据需要尽早进行绑定，即使在`init()`函数中也是如此。

- 对于单个标志，`BindPFlag()`方法提供此功能：

  ```go
  serverCmd.Flags().Int("port", 1138, "Port to run Application server on")
  viper.BindPFlag("port", serverCmd.Flags().Lookup("port"))
  ```

- 你还可以绑定一组现有的 `pflags` （`pflag.FlagSet`），举个例子：

  ```go
  pflag.Int("flagname", 1234, "help message for flagname")
  
  pflag.Parse()
  viper.BindPFlags(pflag.CommandLine)
  
  i := viper.GetInt("flagname") // 从viper而不是从pflag检索值
  ```

- 在 `viper` 中使用 `pflag` 并不阻碍其他包中使用标准库中的 `flag` 包。`pflag` 包可以通过导入这些 `flags` 来处理 `flag` 包定义的 `flags` 。这是通过调用 `pflag` 包提供的便利函数 `AddGoFlagSet()` 来实现的：

  ```go
  package main
  
  import (
  	"flag"
  	"github.com/spf13/pflag"
  )
  
  func main() {
  
  	// 使用标准库 "flag" 包
  	flag.Int("flagname", 1234, "help message for flagname")
  
  	pflag.CommandLine.AddGoFlagSet(flag.CommandLine)
  	pflag.Parse()
  	viper.BindPFlags(pflag.CommandLine)
  
  	i := viper.GetInt("flagname") // 从 viper 检索值
  
  	...
  }
  ```

- 如果你不使用`Pflag`，`viper` 提供了两个Go接口来绑定其他 `flag` 系统。`FlagValue`表示单个 `flag` 。这是一个关于如何实现这个接口的非常简单的例子，一旦你的 `flag` 实现了这个接口，你可以很方便地告诉 `viper` 绑定它：

  ```go
  type myFlag struct {}
  func (f myFlag) HasChanged() bool { return false }
  func (f myFlag) Name() string { return "my-flag-name" }
  func (f myFlag) ValueString() string { return "my-flag-value" }
  func (f myFlag) ValueType() string { return "string" }
  
  viper.BindFlagValue("my-flag-name", myFlag{})
  ```

- `FlagValueSet`代表一组 `flags` ，这是一个关于如何实现这个接口的非常简单的例子，一旦你的 `flag set` 实现了这个接口，你就可以很方便地告诉 `viper` 绑定它：

  ```go
  fSet := myFlagSet{
  	flags: []myFlag{myFlag{}, myFlag{}},
  }
  viper.BindFlagValues("my-flags", fSet)
  ```

##### 远程Key/Value存储支持

- 在 `viper` 中启用远程支持，需要在代码中匿名导入 `viper/remote` 这个包。

  ```GO
  import _ "github.com/spf13/viper/remote"
  ```

- `viper` 将读取从 `Key/Value` 存储（例如 `etcd` 或 `Consul` ）中的路径检索到的配置字符串（如 `JSON`、`TOML`、`YAML`、`HCL`、`envfile `和 `Java properties` 格式）。这些值的优先级高于默认值，但是会被从磁盘、`flag` 或环境变量检索到的配置值覆盖。

  > 也就是说Viper加载配置值的优先级为：磁盘上的配置文件 > 命令行标志位 > 环境变量 > 远程 `Key/Value` 存储 > 默认值。

- `Viper` 使用[crypt](https://github.com/bketelsen/crypt)从K/V存储中检索配置，这意味着如果你有正确的 `gpg` 密匙，你可以将配置值加密存储并自动解密。加密是可选的。

- 你可以将远程配置与本地配置结合使用，也可以独立使用。

- `crypt`有一个命令行助手，你可以使用它将配置放入K/V存储中。`crypt`默认使用在[http://127.0.0.1:4001](http://127.0.0.1:4001/)的 `etcd` ：

  ```SHELL
  $ go get github.com/bketelsen/crypt/bin/crypt
  $ crypt set -plaintext /config/hugo.json /Users/hugo/settings/config.json
  ```

  确认值已经设置：

  ```shell
  $ crypt get -plaintext /config/hugo.json
  ```

  > 有关如何设置加密值或如何使用 `Consul` 的示例，请参见`crypt`文档。

##### 远程Key/Value存储示例-未加密

- `etcd`

  ```go
  viper.AddRemoteProvider("etcd", "http://127.0.0.1:4001","/config/hugo.json")
  viper.SetConfigType("json") // 因为在字节流中没有文件扩展名，所以这里需要设置下类型。支持的扩展名有 "json", "toml", "yaml", "yml", "properties", "props", "prop", "env", "dotenv"
  err := viper.ReadRemoteConfig()
  ```

- `Consul` —— 你需要 `Consul` `Key/Value` 存储中设置一个 `Key` 保存包含所需配置的 `JSON` 值。例如，创建一个 `Key` `MY_CONSUL_KEY`将下面的值存入`Consul` `key/value` 存储：

  ```json
  { "port": 8080, "hostname": "liwenzhou.com" }
  ```

  ```go
  viper.AddRemoteProvider("consul", "localhost:8500", "MY_CONSUL_KEY")
  viper.SetConfigType("json") // 需要显示设置成json
  err := viper.ReadRemoteConfig()
  
  fmt.Println(viper.Get("port")) // 8080
  fmt.Println(viper.Get("hostname")) // liwenzhou.com
  ```

- `Filestore`

  ```go
  viper.AddRemoteProvider("firestore", "google-cloud-project-id", "collection/document")
  viper.SetConfigType("json") // 配置的格式: "json", "toml", "yaml", "yml"
  err := viper.ReadRemoteConfig()
  ```

  > 当然，你也可以使用`SecureRemoteProvider`。

##### 远程Key/Value存储示例-加密

```go
viper.AddSecureRemoteProvider("etcd","http://127.0.0.1:4001","/config/hugo.json","/etc/secrets/mykeyring.gpg")
viper.SetConfigType("json") // 因为在字节流中没有文件扩展名，所以这里需要设置下类型。支持的扩展名有 "json", "toml", "yaml", "yml", "properties", "props", "prop", "env", "dotenv"
err := viper.ReadRemoteConfig()
```

##### 监控etcd中的更改-未加密

```go
// 或者你可以创建一个新的viper实例
var runtime_viper = viper.New()

runtime_viper.AddRemoteProvider("etcd", "http://127.0.0.1:4001", "/config/hugo.yml")
runtime_viper.SetConfigType("yaml") // 因为在字节流中没有文件扩展名，所以这里需要设置下类型。支持的扩展名有 "json", "toml", "yaml", "yml", "properties", "props", "prop", "env", "dotenv"

// 第一次从远程读取配置
err := runtime_viper.ReadRemoteConfig()

// 反序列化
runtime_viper.Unmarshal(&runtime_conf)

// 开启一个单独的goroutine一直监控远端的变更
go func(){
	for {
	    time.Sleep(time.Second * 5) // 每次请求后延迟一下

	    // 目前只测试了etcd支持
	    err := runtime_viper.WatchRemoteConfig()
	    if err != nil {
	        log.Errorf("unable to read remote config: %v", err)
	        continue
	    }

	    // 将新配置反序列化到我们运行时的配置结构体中。你还可以借助channel实现一个通知系统更改的信号
	    runtime_viper.Unmarshal(&runtime_conf)
	}
}()
```



#### 取

- 在 `viper` 中，有几种方法可以根据值的类型获取值。存在以下功能和方法:

  - `Get(key string) : interface{}`

  - `GetBool(key string) : bool`

  - `GetFloat64(key string) : float64`

  - `GetInt(key string) : int`

  - `GetIntSlice(key string) : []int`

  - `GetString(key string) : string`

  - `GetStringMap(key string) : map[string]interface{}`

  - `GetStringMapString(key string) : map[string]string`

  - `GetStringSlice(key string) : []string`

  - `GetTime(key string) : time.Time`

  - `GetDuration(key string) : time.Duration`

  - `IsSet(key string) : bool`

  - `AllSettings() : map[string]interface{}`

- 需要认识到的一件重要事情是，每一个 `Get` 方法在找不到值的时候都会返回零值。为了检查给定的键是否存在，提供了 `IsSet()` 方法：

  ```go
  viper.GetString("logfile") // 不区分大小写的设置和获取
  if viper.GetBool("verbose") {
      fmt.Println("verbose enabled")
  }
  ```

##### 访问嵌套的键

- 访问器方法也接受深度嵌套键的格式化路径。例如，如果加载下面的 `JSON` 文件：

  ```json
  {
      "host": {
          "address": "localhost",
          "port": 5799
      },
      "datastore": {
          "metric": {
              "host": "127.0.0.1",
              "port": 3099
          },
          "warehouse": {
              "host": "198.0.0.1",
              "port": 2112
          }
      }
  }
  ```

- `viper` 可以通过传入`.`分隔的路径来访问嵌套字段：

  ```go
  GetString("datastore.metric.host") // (返回 "127.0.0.1")
  ```

- 这遵守上面建立的优先规则；搜索路径将遍历其余配置注册表，直到找到为止。例如，在给定此配置文件的情况下， `datastore.metric.host` 和 `datastore.metric.port` 均已定义（并且可以被覆盖）。如果另外在默认值中定义了 `datastore.metric.protocol` ，`viper` 也会找到它。

- 然而，如果 `datastore.metric` 被直接赋值覆盖（被 `flag`，环境变量，`set()`方法等等…），那么 `datastore.metric `的所有子键都将变为未定义状态，它们被高优先级配置级别**“遮蔽”（shadowed）**了。

- 最后，如果存在与分隔的键路径匹配的键，则返回其值。例如：

  ```go
  {
      "datastore.metric.host": "0.0.0.0",
      "host": {
          "address": "localhost",
          "port": 5799
      },
      "datastore": {
          "metric": {
              "host": "127.0.0.1",
              "port": 3099
          },
          "warehouse": {
              "host": "198.0.0.1",
              "port": 2112
          }
      }
  }
  
  GetString("datastore.metric.host") // 返回 "0.0.0.0"
  ```

##### 提取子树

- 从 `viper` 中提取子树。例如，`viper` 实例现在代表了以下配置：

  ```yaml
  app:
    cache1:
      max-items: 100
      item-size: 64
    cache2:
      max-items: 200
      item-size: 80
  ```

- 执行后：

  ```go
  subv := viper.Sub("app.cache1")
  ```

- `subv` 现在就代表：

  ```yaml
  max-items: 100
  item-size: 64
  ```

- 假设我们现在有这么一个函数：

  ```go
  func NewCache(cfg *Viper) *Cache {...}
  ```

- 它基于 `subv` 格式的配置信息创建缓存。现在，可以轻松地分别创建这两个缓存，如下所示：

  ```go
  cfg1 := viper.Sub("app.cache1")
  cache1 := NewCache(cfg1)
  
  cfg2 := viper.Sub("app.cache2")
  cache2 := NewCache(cfg2)
  ```

##### 反序列化

- 你还可以选择将所有或特定的值解析到结构体、map等。有两种方法可以做到这一点：

  - `Unmarshal(rawVal interface{}) : error`
  - `UnmarshalKey(key string, rawVal interface{}) : error`

  ```go
  type config struct {
  	Port int
  	Name string
  	PathMap string `mapstructure:"path_map"`
  }
  
  var C config
  
  err := viper.Unmarshal(&C)
  if err != nil {
  	t.Fatalf("unable to decode into struct, %v", err)
  }
  ```

- 如果你想要解析那些键本身就包含 `.` (默认的键分隔符）的配置，你需要修改分隔符：

  ```go
  v := viper.NewWithOptions(viper.KeyDelimiter("::"))
  
  v.SetDefault("chart::values", map[string]interface{}{
      "ingress": map[string]interface{}{
          "annotations": map[string]interface{}{
              "traefik.frontend.rule.type":                 "PathPrefix",
              "traefik.ingress.kubernetes.io/ssl-redirect": "true",
          },
      },
  })
  
  type config struct {
  	Chart struct{
          Values map[string]interface{}
      }
  }
  
  var C config
  
  v.Unmarshal(&C)
  ```

- `viper` 还支持解析到嵌入的结构体：

  ```go
  /*
  Example config:
  
  module:
      enabled: true
      token: 89h3f98hbwf987h3f98wenf89ehf
  */
  type config struct {
  	Module struct {
  		Enabled bool
  
  		moduleConfig `mapstructure:",squash"`
  	}
  }
  
  // moduleConfig could be in a module specific package
  type moduleConfig struct {
  	Token string
  }
  
  var C config
  
  err := viper.Unmarshal(&C)
  if err != nil {
  	t.Fatalf("unable to decode into struct, %v", err)
  }
  ```

  > `viper` 在后台使用[github.com/mitchellh/mapstructure](https://github.com/mitchellh/mapstructure)来解析值，其默认情况下使用`mapstructure`标签。

##### 序列化成字符串

- 你可能需要将viper中保存的所有设置序列化到一个字符串中，而不是将它们写入到一个文件中。你可以将自己喜欢的格式的序列化器与 `AllSettings()` 返回的配置一起使用：

  ```go
  import (
      yaml "gopkg.in/yaml.v2"
      // ...
  )
  
  func yamlStringSettings() string {
      c := viper.AllSettings()
      bs, err := yaml.Marshal(c)
      if err != nil {
          log.Fatalf("unable to marshal config to YAML: %v", err)
      }
      return string(bs)
  }
  ```

#### 单个 `viper` 还是多个？

- `viper` 是开箱即用的。你不需要配置或初始化即可开始使用 `viper` 。由于大多数应用程序都希望使用单个中央存储库管理它们的配置信息，所以 `viper` 包提供了这个功能。它**类似**于**单例模式**。在上面的所有示例中，它们都以其单例风格的方法演示了如何使用 `viper` 。

- 使用多个 `viper` 是被允许的，每个都有自己独特的一组配置和值。每个人都可以从不同的配置文件，KV存储区等读取数据。每个都可以从不同的配置文件、键值存储等中读取。 `viper` 包支持的所有功能都被镜像为 `viper` 实例的方法：

  ```go
  x := viper.New()
  y := viper.New()
  
  x.SetDefault("ContentDir", "content")
  y.SetDefault("ContentDir", "foobar")
  
  //...
  ```

#### 案例

- https://www.liwenzhou.com/posts/Go/viper_tutorial/#autoid-1-6-1

---



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



#### 中间件

- 中间件的注册和调用其实没有多少技术含量，其实是一个调用方程数组（其实官方叫法应该叫切片来着~）。一般调用 `Engine.Use()` 来注册中间件：

```go
// Use attaches a global middleware to the router. i.e. the middleware attached through Use() will be
// included in the handlers chain for every single request. Even 404, 405, static files...
// For example, this is the right place for a logger or error management middleware.
func (engine *Engine) Use(middleware ...HandlerFunc) IRoutes {
	engine.RouterGroup.Use(middleware...)
	engine.rebuild404Handlers()
	engine.rebuild405Handlers()
	return engine
}
```

👇

```go
// Use adds middleware to the group, see example code in GitHub.
func (group *RouterGroup) Use(middleware ...HandlerFunc) IRoutes {
	group.Handlers = append(group.Handlers, middleware...)
	return group.returnObj()
}
```

- 很简单的 `append()` 方程，只要加到 `group.Handlers` 里就是了，包括在注册路由时候添加到 `node` 节点里：

```go
// HandlersChain defines a HandlerFunc slice.
type HandlersChain []HandlerFunc
```

```go
func (group *RouterGroup) handle(httpMethod, relativePath string, handlers HandlersChain) IRoutes {
	absolutePath := group.calculateAbsolutePath(relativePath)
	handlers = group.combineHandlers(handlers)
	group.engine.addRoute(httpMethod, absolutePath, handlers)
	return group.returnObj()
}
```

```go
func (group *RouterGroup) combineHandlers(handlers HandlersChain) HandlersChain {
	finalSize := len(group.Handlers) + len(handlers)
	assert1(finalSize < int(abortIndex), "too many handlers")
	mergedHandlers := make(HandlersChain, finalSize)
	copy(mergedHandlers, group.Handlers)
	copy(mergedHandlers[len(group.Handlers):], handlers)
	return mergedHandlers
}
```

- 接下来考虑中间件的执行，不难发现，执行过程就是方程切片从头遍历到尾罢了，这里有几个比较典型的迭代方程，`gin.Context.Next()` ，`gin.Context.Set()` ，`gin.Context.Get()` 和 `gin.Context.Abort()` ：

```go
// Next should be used only inside middleware.
// It executes the pending handlers in the chain inside the calling handler.
// See example in GitHub.
func (c *Context) Next() {
	c.index++
	for c.index < int8(len(c.handlers)) {
		c.handlers[c.index](c)
		c.index++
	}
}
```

```go
// Abort prevents pending handlers from being called. Note that this will not stop the current handler.
// Let's say you have an authorization middleware that validates that the current request is authorized.
// If the authorization fails (ex: the password does not match), call Abort to ensure the remaining handlers
// for this request are not called.
func (c *Context) Abort() {
	c.index = abortIndex
}
```

![image-gin-nextRun](images/image-gin-nextRun.svg)

- 在 `handersChain` 中传播时，可以通过中间件设置默写参数以供后面的处理函数使用：

```go
// Set is used to store a new key/value pair exclusively for this context.
// It also lazy initializes  c.Keys if it was not used previously.
func (c *Context) Set(key string, value any) {
	c.mu.Lock()
	if c.Keys == nil {
		c.Keys = make(map[string]any)
	}

	c.Keys[key] = value
	c.mu.Unlock()
}

// Get returns the value for the given key, ie: (value, true).
// If the value does not exist it returns (nil, false)
func (c *Context) Get(key string) (value any, exists bool) {
	c.mu.RLock()
	value, exists = c.Keys[key]
	c.mu.RUnlock()
	return
}
```

![image-gin-handlerChanSetGet](images/image-gin-handlerChanSetGet.svg)



### 后记 2022.09.20

不得不感叹 `Gin` 简洁明了，接下来就是 `net/http` 的源码了。😀