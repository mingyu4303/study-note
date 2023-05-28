# Gee框架

## 1. HTTP基础

http库最基础的两个函数： 

- HandleFunc: 注册路由的处理函数
- ListenAndServe: 设置服务器地址和服务处理实体，启动http服务

```go
func main() {
	http.HandleFunc("/", indexHandler)
	http.HandleFunc("/hello", helloHandler)
	log.Fatal(http.ListenAndServe(":9999", nil))
}

// handler echoes r.URL.Path
func indexHandler(w http.ResponseWriter, req *http.Request) {
	fmt.Fprintf(w, "URL.Path = %q\n", req.URL.Path)
}
```

**实现http.Handler接口**

```go
package http

type Handler interface {
    ServeHTTP(w ResponseWriter, r *Request)
}

func ListenAndServe(address string, h Handler) error
```

**重点：只要传入任何实现了 *ServerHTTP* 接口的实例，所有的HTTP请求，就都交给了该实例处理了。**

**Gee框架的雏形**：封装Handler实例（实现时叫做Engine），实现以下接口

```go
// HandlerFunc defines the request handler used by gee
type HandlerFunc func(http.ResponseWriter, *http.Request)

// Engine implement the interface of ServeHTTP
type Engine struct {
	router map[string]HandlerFunc
}

// New is the constructor of gee.Engine
func New() *Engine 

func (engine *Engine) addRoute(method string, pattern string, handler HandlerFunc) 

// GET defines the method to add GET request
func (engine *Engine) GET(pattern string, handler HandlerFunc)

// POST defines the method to add POST request
func (engine *Engine) POST(pattern string, handler HandlerFunc) 

// Run defines the method to start a http server
func (engine *Engine) Run(addr string) (err error) 

func (engine *Engine) ServeHTTP(w http.ResponseWriter, req *http.Request) 
```



