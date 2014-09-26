---
layout: post
title: "bash漏洞-环境变量"
date: 2014-09-26 13:37:04 +0800
comments: true
categories: 
  - Linux Admin
---

使用下列命令来检测。

```
env x='() { :;}; echo asdasd' bash -c "echo asd"
```

是否对Docker、LXC有影响？
