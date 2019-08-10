---
title: go 错误总结
date: 2019-08-10 08:45:18
tags: Go
---


## 指针引用类型错误

```
cannot use user (type common.User) as type *common.User in argument to common.UserService.UpdateUser
```

解决办法
```
x := &common.User{}
```
基础没学好还是要多看书啊:(

[参考 stackoverflow](https://stackoverflow.com/questions/43900806/cannot-use-as-type-in-assignment-in-go)

---
## 项目中Makefile报错，看问题是制表符的问题，这个配置文件需要制表符而不是空格

```
Makefile:14: *** missing separator (did you mean TAB instead of 8 spaces?).  Stop.
```
解决办法

使用vi打开Makefile,把所有的空格替换成制表符 "\t"

```
:%s/^[ ]\+/\t/g
```
[参考 https://unix.stackexchange.com](https://unix.stackexchange.com/questions/125757/make-complains-missing-separator-did-you-mean-tab)

