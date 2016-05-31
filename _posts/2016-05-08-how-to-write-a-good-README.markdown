---
layout: post
title:  "如何写一份好的README"
date:   2016-05-08 23:32:49
categories: Thinking
tags: openSource
---



![项目Logo](https://raw.githubusercontent.com/jehna/readme-best-practices/master/sample-logo.png)

# 项目名称
> 关于项目的简单描述

简短的介绍你的项目，项目如何使用，并且这个项目主要解决什么问题，如何使用

## 安装/入门

使用简洁的语言快速的介绍如何使用，就像编程语言一样，如何写一个简单的Hello world或者是运行。

```shell
packagemanager install awesome-project
awesome-project start
awesome-project "Do something!"  # prints "Nah."
```
当执行某个命令时，应该附上这个命令执行后的结果

## 开发测试

Here's a brief intro about what a developer must do in order to start developing
the project further:

```shell
git clone https://github.com/your/awesome-project.git
cd awesome-project/
packagemanager install
```

每一步的结果介绍

### 构建

如果你的项目在构建时还需要其它步骤,例如编译安装。需要在这里也要写出来:

```shell
./configure
make
make install
```

在这里我们应该注明编译安装之后成功的结果,譬如:成功界面截图

### 部署和发布

如果你项目部署到生产服务器上有一些特别的说明。则需要在这里介绍,告诉使用者应该如何去部署并发布他们的应用。

```shell
packagemanager deploy awesome-project -s server.com -u username -p password
```

在这里应该解释一下上面的代码是什么意思。

## 特点

这个项目可以用来做什么,有那些特点?

* 主要功能
* 可以用来做什么
* 如果可能,实际上可以怎么样做

## 使用配置

如果你的项目在使用过程中还需要一些配置信息,一定不要忘记在这里注明,格式如下:

#### Argument 1
Type: `String`  
Default: `'default value'`

State what an argument does and how you can use it. If needed, you can provide
an example below.

Example:
```bash
awesome-project "Some other value"  # Prints "You're nailing this readme!"
```

#### Argument 2
Type: `Number|Boolean`  
Default: 100

Copy-paste as many of these as you need.

## 如何贡献代码

告诉开发者可以给项目贡献代码,但是得遵循一定的骨折

When you publish something open source, one of the greatest motivations is that
anyone can just jump in and start contributing to your project.

These paragraphs are meant to welcome those kind souls to feel that they are
needed. You should state something like:

"If you'd like to contribute, please fork the repository and use a feature
branch. Pull requests are warmly welcome."

If there's anything else the developer needs to know (e.g. the code style
guide), you should link it here. If there's a lot of things to take into
consideration, it is common to separate this section to its own file called
`CONTRIBUTING.md` (or similar). If so, you should say that it exists here.

## 版权(授权)

最重要的一个部分：告诉使用者，你的开源项目究竟承认何种版权，并且附上版权细则。就像MIT license下面这样：

“
Copyright (C) 2016

Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the "Software"), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.”


