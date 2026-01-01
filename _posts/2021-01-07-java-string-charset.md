---
title: JAVA String Charset
date: 2021-01-07 16:25:05
tags: [java, string]
categories: [Programming, JAVA]
---

# JAVA String Charset

JAVA的String使用上一般不会考虑到编码字符集的问题，即encode charset。但如果出现数据交互，就需要考虑了，因为不同环境就可能存在差别。如果编码对不上，可能会出现乱码。JAVA中我经常见到的是`?`乱码（多个问号），不排除其他可能。

# An Example

host A用utf-8编码`String`，以`byte[]`形式发送到host B，如果host B只简单的用
```
new String(receivedBytes)
```
来转成String使用，这里使用的charset就是default charset。
> The Java platform depends heavily on a property called the default charset. The Java Virtual Machine (JVM) determines the default charset during start-up.

可以用下面的方法测试出当前平台的default charset。
```
Charset.defaultCharset().displayName();
```
我个人的jvm default charset是`US-ASCII`，所以不设置String的charset就出现乱码。

## 解决方法
1. `new String`时指定charset
1. 修改JVM的default charset，方法自行搜索

# Further

即使String的编码正确了，也不代表console output或output file就不会乱码。涉及了多个模块，可能需要更多的配置，请随机应变。