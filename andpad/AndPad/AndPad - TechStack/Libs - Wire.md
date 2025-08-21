https://github.com/google/wire

DI: Dependency Injection in Golang
Core concepts: **Providers** and **Injectors**

## Makefile
```
gen:
	go get github.com/google/wire/cmd/wire  
	cd internal/registry; go run github.com/google/wire/cmd/wire
```

### Provider
meaning like java Consumer func. Call to produce a value.


Many Provider functions can group in to a Type name SuperSet

```go
package foobarbaz

import (
    // ...
    "github.com/google/wire"
)

// ...

func ProviderFoo() Foo {
// ....
}

func ProviderBar(int bar) Bar {
// ... do sth
}
func ProviderBar(int bar, string foo) (Bar,err){
// ... sth
}

var SuperSet = wire.NewSet(ProvideFoo, ProvideBar, ProvideBaz)
```

## Injectors
Developer define how providers will be inject in `wire.go` then wire will generate the `wire_gen.go` which is impl just like developers do.
wire.go should include
`// +build wireinject`

S1: define wire.go

