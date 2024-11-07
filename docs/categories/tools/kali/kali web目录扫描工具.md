---
title: kali linux web目录扫描工具汇总
author: Seso
date: 2024/05/6 11:05
categories:
 - kali
 - 扫描工具
tags:
 - 扫描工具
 - kali
---

# kali linux web目录扫描工具汇总



> 在渗透中，我们需要得到网站web服务器的路劲。如`管理员后台`,站点的敏感文件如（站点备份、数据库备份）等等。在kali中有很多这样的优秀工具，本文将为你一一介绍。

# 01 Gobuster

`Gobuster` 是一个开源工具，主要用于网站目录扫描和子域名收集。安装也很简单，只需执行下面命令即可！

```bash
apt-get install gobuster
```

![img](https://blog-1304715799.cos.ap-nanjing.myqcloud.com/imgs/d9f6ccd382b5ddc0c4f37e905fc232e27690e65a.png%401256w_528h_!web-article-pic.avif?imageSlim)

**使用参数：**

- `-u` : –url 网站的域名或者IP
- `-w` : 用户自定义字典
- `-x` : 自定义文件类型如`php` `jsp`

**示例**

```bash
gobuster dir -u http://192.168.0.106/bolt/ -w /usr/share/wordlists/dirb/big.txt -x php
```

![img](https://blog-1304715799.cos.ap-nanjing.myqcloud.com/imgs/783b9e42aa9abbbb565e16df79c7a1f04bf14c88.png%401256w_638h_!web-article-pic.avif?imageSlim)

# 02 DIRB

`DIRB` 是一个 Web 内容扫描器。它是 kali linux 内置的工具，通过对 Web 服务器发起基于字典的攻击并分析响应来工作，但请记住它是内容扫描器而不是漏洞扫描器。 使用也很简单，在DIRB后面直接加目标域名即可。

```bash
dirb https://bbskali.cn
```

![img](https://blog-1304715799.cos.ap-nanjing.myqcloud.com/imgs/a03e2dec9e18f45efe4129f4e30435869d6a1ea9.png%401256w_860h_!web-article-pic.avif?imageSlim)

# 03 dirsearch

`Dirsearch` 是一个用 Python 编写的暴力扫描工具，用于查找隐藏的 Web 目录和文件。它可以在 `Windows`、`Linux`和 `macOS`上运行，而且终端界面配色也比较漂亮。**安装**`dirsearch`在kali中默认没有安装，我们只需执行下面命令进行安装。

```bash
apt-get install dirsearch
```

**使用**

```bash
dirsearch -u https://bbskali.cn
```



![img](https://blog-1304715799.cos.ap-nanjing.myqcloud.com/imgs/6cfc86efd6b42ebcac8d56d01fdb0938290979d5.png%401256w_1002h_!web-article-pic.avif?imageSlim)

# 04 Wfuzz

`Wfuzz` 在 Kali Linux内置的，因此我们可以通过在终端上键入`wfuzz`来启动它。**参数**`-u` : 目标网址`-w` : 单词表

```javascript
wfuzz -u https://bbskali.cn -w /usr/share/dirb/wordlists/common.txt --hc 400,404,403
```



# 05 Metasploit

利用`metasploit` 框架,我们也可以对网站目录进行扫描。

```javascript
use auxiliary/scanner/http/dir_scanner
set rhosts bbskali.cn
set path secnhack/
set dictionary /usr/share/dirb/wordlists/common.txt
run
```

![img](https://blog-1304715799.cos.ap-nanjing.myqcloud.com/imgs/17ebac2982ffd3bbd88b42dc47e7bce2059388b6.png%401256w_658h_!web-article-pic.avif?imageSlim)

# 06 DirBuster

`DirBuster`是kali自带的一款图形化工具。终端执行命令在其中提交目标详细信息，如下所示。

![img](https://blog-1304715799.cos.ap-nanjing.myqcloud.com/imgs/841796ddb00ef5e3a3e01026c59b27e0019cc186.png%401256w_838h_!web-article-pic.avif?imageSlim)

效果如下：

![img](https://blog-1304715799.cos.ap-nanjing.myqcloud.com/imgs/dab57e398bcc42e09c09d52ffbfebb277d707896.png%401256w_858h_!web-article-pic.avif?imageSlim)