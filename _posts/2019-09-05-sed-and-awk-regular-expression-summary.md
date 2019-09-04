---
layout: post
title:  "sed和awk工具，以及正则表达式总结"
date:   2019-09-05 01:53:49
categories: 操作系统
tags: Linux
---
sed和awk命令正则表达式的使用总结

正则表达式
-------------------

以下符号可以在grep、egrep、sed、awk中使用:

| 符号    | 描述                 |
| ------- | -------------------- |
| `.`     | 任意1个字符          |
| `.*`    | 任意字符             |
| `[a-z]` | 任意一个小写字符     |
| `^`     | 以什么字符开始       |
| `$`     | 以此匹配的字符结尾   |
| `*`     | `*`前面的字符重复N次 |


以下表达式符号为可用列表，不一定所有的都可以使用:

| 符号      | 描述                                 |
| --------- | ------------------------------------ |
| `(A|B)`   | A 或者是 B                           |
| `\<`      | 单词锚点开始                         |
| `\>`      | 单词锚点结束                         |
| `\( \)`   | 一组字符、可能是\1,\2等              |
| `?`       | 断言，匹配前面的子表达式零次或一次。 |
| `\{n\}`   | 前面的子表达式重复n次                |
| `\{n,m\}` | 前面的子表达式重复n至m次             |
| `\{n,\}`  | 前面的子表达式至少重复n次            |



sed：流编辑
-------------
`sed`一般用在文本内容的查找和替换，譬如：

```bash
cmd | sed 's/old/new/g' #在cmd的输出结果中，把old字符全部用new替换
sed 's/old/new/g' infile > outfile # 把infile文件中的old字符全部替换为new
```

`sed` 功能包括:

| 功能              | 描述                                        |
| ----------------- | ------------------------------------------- |
| `s/old/new/`      | 替换old为new                                |
| `s/old/new/g`     | 替换所有的old为new                          |
| `/RE/s/old/new/g` | 匹配包含RE行的内容把old替换为new            |
| `3s/old/new/g`    | 替换第3行中所有的old为new                   |
| `3,6s/old/new/g`  | 替换第3行到6行中所有的old为new              |
| `/RE/d`           | 删除包含RE的行                              |
| `1d`              | 删除第1行                                   |
| `10,25d`          | 删除第10到15行                              |
| `/A/,/B/d`        | 删除以A开头或B结尾的行                      |
| `s/RE/-&-/g`      | 查找RE字符，在RE前后加入`-`.&代表匹配的字符 |
| `/RE/p`           | 输出包含RE字符的行号，需要使用`sed -n`      |
| `1,10p`           | 打印1-10行,需要使用`sed -n                  |


例子:

  ```bash
 cat /etc/passwd | sed 's/:.*//'            # 列出用户名
   cat /var/adm/messages | sed 's/.*fail.*/!!!&!!!/'    # 打印出经过
   ifconfig hme0 | sed '/inet/!d;/inet/s/.*inet \(.*\) netm.*/\1/;'
    # 删除没有以 "inet"开头的行, 并且答应IP.
  ```




AWK 编程语言
------------------------

Awk是一个扩展文本处理语言，它一般用在打印字段或者一列文本。譬如：


```bash
cmd | awk '{ print $2 }' #输出cmd命令结果的第二列
awk '{ print $2 }' infile #打印文件第二列内容
```

print -输出声明的变量:

```bash
{ print $2 }#打印第二列2
{ print $2,$4 }         #打印第2和4列
{ print $2 "\t" $4 }         #打印第2和4列，并且在中间加入tab空行
{ print "user:",$1,"uid",$3 }    #按格式化文本的方式输出内容
{ printf "%10s %-5s\n",$1,$2 }  #格式化列，并在输出内容$1,$2，右边加10个空白字符，左边加5个空白字符
```

查找--代码在` { print ... }`之间:

```bash
/RE/ #包含RE字符的内容
BEGIN            #任何在BEGIN之后列出的操作（在{}内）将在Unix awk开始扫描输入之前执行
END          #而END之后列出的操作将在扫描完全部的输入之后执行。
NR == 4 #行号为4
NR > 2 #行号大于3
NR > 2 && NR < 11   #行号在3和10之间
$3 ~ /RE/       if #第三列包含 RE
$9 !~ /RE/      if #第9列不包含RE
```

内置变量 - awk中的变量,同样在` { ... }`语句中:

```bash
NR          #行号
FS = ":"         #设置输出格式以:间隔
OFS = "\t"      set #输出内容以tab间隔
name = $1        #变量name的值为$1
sum = sum + $5 #变量sum为sum+$5列
sum += $5        #和上面一样
x = 5            #变量 x = 5
print x          #输出变量 x
print "name:" #输出的内容前加上name:
```

例子:

 ```bash
ls -l | awk '{ print "file",$9,"has size",$5,"bytes" }   # 格式化ls命令结果
   cat /etc/passwd | awk 'BEGIN { FS=":" } { print $1 }'    # 列出用户名
   IP=`ifconfig hme0 | awk '/inet/ { print $2 }'`       # 查询IP时，并赋值给变量IP
   cat logfile.txt | awk '                  # report
    BEGIN  { print "My report on blah\n" 
            printf "%10s %-20s | %s\n","hostname","time","fault" }
    /fail/ { print "AHOY! The following line is a failure!" }
          { sum = sum + $5; printf "%10s %-20s | %s\n",$2,$3,$1 }
    END    { print "total faults:",sum }'
 ```



**参考阅读**

- [regexp, sed & awk](http://www.brendangregg.com/Unix/re-sed-awk.txt)



