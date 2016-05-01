---
layout: post
title:  "Golang:储存单位转换判断"
date:   2016-05-01 23:32:49
categories: Golang
tags: Golang
---

这段代码是把存储单位转换通过移位运算得出需要的结果.源码来自于Google的开源项目`zoekt`.[api.go][2]

```
func (s Stats) HumanBytesLoaded() string {
	suffix := ""
	b := s.BytesLoaded
	if s.BytesLoaded > (1 << 30) {
		suffix = "G"
		b = s.BytesLoaded / (1 << 30)
	} else if s.BytesLoaded > (1 << 20) {
		suffix = "M"
		b = s.BytesLoaded / (1 << 20)
	} else if s.BytesLoaded > (1 << 10) {
		suffix = "K"
		b = s.BytesLoaded / (1 << 10)
	}

	return fmt.Sprintf("%d%s", b, suffix)
}
```

`b := s.BytesLoaded`的值是byte的单位:

- 1GB=1024*1024*1024=1<<30=1073741824
- 1MB=1024*1024=1<<20=1048576
- 1KB=1<<10=1024

```
//测试代码
package main

import (
	"fmt"
	"os"
	"strconv"
)

func main() {
	var com string
	value,_:=strconv.ParseInt(os.Args[1],10,64)
	com = load(value)
	fmt.Printf("%s", com)
}
func  load(s int64) string {
	suffix := ""
	b := s
	if s > (1 << 30) {
		suffix = "G"
		b = s / (1 << 30)
	} else if s > (1 << 20) {
		suffix = "M"
		b = s / (1 << 20)
	} else if s > (1 << 10) {
		suffix = "K"
		b = s / (1 << 10)
	}
	return fmt.Sprintf("%d%s\n", b, suffix)
}
```

```
//运行测试
➜ /home/sec/go/src/main >go run byte.go 10000000
9M
```


参考资料:

[Google zoekt source][1]

[1]: https://github.com/google/zoekt/blob/master/api.go
[2]: https://github.com/google/zoekt/blob/master/api.go#L69