  

sample

```Plain
// ChangePassword godoc
// @Summary Change user's password
// @Introduction Change user's password
// @Tags users
// @Accept json
// @Produce json
// @Param request body dto.ChangePasswordRequest true "request"
// @Param request body dto.ChangePasswordRequest true "request"
// @Success 204 {object} object ""
// @Failure 400 {object} dto.HTTPError
// @Failure 500 {object} dto.HTTPError
// @Security ApiKeyAuth
// @Router /users/password [PUT]
```

  

# Attribute

```Bash
// @Param id path string false "path params"
// @Param enumstring query string false "string enums" Enums(A, B, C)
// @Param enumint query int false "int enums" Enums(1, 2, 3)
// @Param enumnumber query number false "int enums" Enums(1.1, 1.2, 1.3)
// @Param string query string false "string valid" minlength(5) maxlength(10)
// @Param int query int false "int valid" minimum(1) maximum(10)
// @Param default query string false "string default" default(A)
// @Param collection query []string false "string collection" collectionFormat(multi)
```

```Go
type Foo struct {
    Bar string `minLength:"4" maxLength:"16"`
    Baz int `minimum:"10" maximum:"20" default:"15"`
    Qux []string `enums:"foo,bar,baz"`
}
```

```Go
type JSONResult struct {
		// event/{id}/...
		ID `param:"id"`

		// Code demonstrate process result
    Code    int          `json:"code"  example:"1"`
    Message string       `json:"message" example:"example message"`
    Data    interface{}  `json:"data"`
		Ignored int     `swaggerignore:"true"`

		// Sort order:
		// * asc - Ascending, from A to Z.
		// * desc - Descending, from Z to A.
		Order string `enums:"asc,desc"`
}
```