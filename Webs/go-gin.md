# Go-Gin 学习笔记

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

  