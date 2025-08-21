[https://github.com/avelino/awesome-go#contents](https://github.com/avelino/awesome-go#contents)

- Web framework
    
    https://github.com/gin-gonic/gin
    
    https://github.com/labstack/echo
    
- Watch and build
    
    https://github.com/cosmtrek/air
    
    https://github.com/githubnemo/CompileDaemon.git
    

  

# Gin bind

```Go
type Person struct {
	A string `uri:"name" binding:"required"`
	Name    string `form:"name"`
	Address string `form:"address"`

	data any `json:"any"`
}
```