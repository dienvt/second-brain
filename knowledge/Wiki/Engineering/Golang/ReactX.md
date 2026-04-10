---
title: "ReactX"
date: 2026-01-14
tags:
  - engineering
  - golang
---

```
go get -u github.com/reactivex/rxgo/v2
```

Describe observable (manager)
```go
rxgo.Just(1, 2, 3)()
```

Describe subscribe (consumer)
```Go
ch := observable.Observe()
for item := range ch{
	if item.Error() {
	    return item.E
	}
	fmt.Println(item.V)
}

```

How to stop consumer