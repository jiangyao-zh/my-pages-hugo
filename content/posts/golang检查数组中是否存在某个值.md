---
title: 'GoLang检查数组中是否存在某个值'
date: 2022-06-10 10:12:52
tags: [Go]
categories: ["开发"]
draft: false
---

## golang检查数组中是否存在某个值

* 类似PHP用的in_array功能实现
    >in_array(mixed $needle, array $haystack, bool $strict = false): bool

在Go 1.18 版本支持泛型以后，可以在**pkg.go.dev**中的slices包下查找[Contains](https://pkg.go.dev/golang.org/x/exp@v0.0.0-20220609121020-a51bd0440498/slices#Contains)方法用作实现,例如：

```go
package main

import (
 "fmt"

 "golang.org/x/exp/slices"
)

func main() {
    var date string = "createdAt"
    dateType := []string{"createdAt", "updatedAt"}
    if slices.Contains[string](dateType, date) {
        fmt.Println("yes")
    } else {
        fmt.Println("no")
    }
}
```
