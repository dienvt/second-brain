---
title: "Context Value Comparison"
date: 2026-01-14
tags:
  - engineering
  - golang
---


package main

import (
	"context"
	"fmt"
)

type sessionKey struct{}

func cmp(a, b any) bool {
	return a == b
}

func main() {
	ctx := context.WithValue(context.Background(), &sessionKey{}, "foo")
	fmt.Println(ctx.Value(&sessionKey{}))
	foo := &sessionKey{}
	bar := &sessionKey{}
	if foo == bar {
		fmt.Println("true")
	}
	if cmp(foo, bar) {
		fmt.Println("false")
	}
	
	// console
	// foo
	// false
}

tai.pham+mob_11
