---
title: Blog 第一帖 - 字符坑 BOM
description: UFT-8编码文件下存在的BOM问题
date: 2016-05-18T00:36:00+0800
lastmod: '2021-07-15T16:36:00+0800'
image: bom.jpg
slug: bom
categories:
    - others
tags:
    - Encoding
    - BOM
    - UTF-8
---

## 字符编码

字集码是把字符集中的字符编码为指定集合中某一对象（例如：比特模式、自然数序列、8位组或者电脉冲），以便文本在计算机中存储和通过通信网络的传递。

常见的例子包括将拉丁字母表编码成摩斯电码和ASCII。其中，ASCII将字母、数字和其它符号编号，并用7比特的二进制来表示这个整数。通常会额外使用一个扩充的比特，以便于以1个字节的方式存储。
常见的字符编码有： ASCII、UTF-8、Unicode、GBK等

详见[WikiPedia](https://zh.wikipedia.org/wiki/%E5%AD%97%E7%AC%A6%E7%BC%96%E7%A0%81)

---

## ASCII

**ASCII**  ( **A** merican  **S** tandard **C** ode for **I** nformation **I** nterchange)       即美国信息交换标准代码。

ASCII第一次以规范标准的型态发表是在1967年，最后一次更新则是在1986年，至今为止共定义了128个字符；其中33个字符无法显示（一些终端提供了扩展，使得这些字符可显示为诸如笑脸、扑克牌花式等8-bit符号），且这33个字符多数都已是陈废的控制字符。控制字符的用途主要是用来操控已经处理过的文字。在33个字符之外的是95个可显示的字符，包含用键盘敲下空白键所产生的空白字符也算1个可显示字符（显示为空白）。

ASCII的局限在于只能显示26个基本拉丁字母、阿拉伯数目字和英式标点符号，因此只能用于显示现代美国英语（而且在处理英语当中的外来词如naïve、café、élite等等时，所有重音符号都不得不去掉，即使这样做会违反拼写规则）。而EASCII虽然解决了部分西欧语言的显示问题，但对更多其他语言依然无能为力。因此现在的软件系统大多采用Unicode。

详见[WikiPedia](https://zh.wikipedia.org/wiki/ASCII)

---

## Unicode
--------
![Unicode](https://upload.wikimedia.org/wikipedia/commons/thumb/a/ab/Unicode_logo.svg/108px-Unicode_logo.svg.png)

**Unicode** 是为了解决传统的字符编码方案的局限而产生的，例如ISO 8859-1所定义的字符虽然在不同的国家中广泛地使用，可是在不同国家间却经常出现不兼容的情况。

很多传统的编码方式都有一个共同的问题，即容许电脑处理双语环境（通常使用拉丁字母以及其本地语言），但却无法同时支持多语言环境（指可同时处理多种语言混合的情况）。

目前，几乎所有电脑系统都支持基本拉丁字母，并各自支持不同的其他编码方式。Unicode为了和它们相互兼容，其首256字符保留给ISO 8859-1所定义的字符，使既有的西欧语系文字的转换不需特别考量；并且把大量相同的字符重复编到不同的字符码中去，使得旧有纷杂的编码方式得以和Unicode编码间互相直接转换，而不会丢失任何信息。举例来说，全角格式区块包含了主要的拉丁字母的全角格式，在中文、日文、以及韩文字形当中，这些字符以全角的方式来呈现，而不以常见的半角形式显示，这对竖排文字和等宽排列文字有重要作用。

详见[WikiPedia](https://zh.wikipedia.org/wiki/Unicode)

---

## UTF-8
-------

**UTF-8** （8-bit Unicode Transformation Format）是一种针对Unicode的可变长度字符编码，也是一种前缀码。

它可以用来表示Unicode标准中的任何字符，且其编码中的第一个字节仍与ASCII兼容，这使得原来处理ASCII字符的软件无须或只须做少部分修改，即可继续使用。因此，它逐渐成为电子邮件、网页及其他存储或发送文字的应用中，优先采用的编码。

**UTF-8** 现已经作为通用的字符编码，应用于各中网页编码，数据编码，数据库字符编码等。编码的统一能够写出的程序或网页在中文环境下大大减少乱码的出现。dddddddddddddddddd

详见[WikiPedia](https://zh.wikipedia.org/wiki/UTF-8)

---

## GBK
-------
**汉字内码扩展规范** ，称GBK，全名为《汉字内码扩展规范(GBK)》1.0版，由中华人民共和国全国信息技术标准化技术委员会1995年12月1日制订，国家技术监督局标准化司和电子工业部科技与质量监督司1995年12月15日联合以《技术标函[1995]229号》文件的形式公布。

GBK的K为汉语拼音Kuo Zhan（扩展）中“扩”字的声母。英文全称Chinese Internal Code Extension Specification。

GBK 只为“技术规范指导性文件”，不属于国家标准。国家质量技术监督局于2000年3月17日推出了GB 18030-2000标准，以取代GBK。

---

## BOM ( Byte Order Mark  ) 
-------
**这个才是重点，BOM头。**

在UTF-8编码文件中BOM在文件头部，占用三个字节，用来标示该文件属于UTF-8编码，现在已经有很多软件识别BOM头，但是还有些不能识别BOM头，比如Windows自带的记事本软件，这也是用记事本编辑UTF-8编码后执行就会出错的原因了。

Windows下

- 支持BOM的编辑器
Notepad++

- 不支持BOM的编辑器
Windows自带的记事本 （**坑**）

正因有BOM头的存在，使我在本地Jekyll的调试环境中，页面不能正常显示。

**请使用Windows的童鞋一定注意该问题**

目前已经切换为 [Hugo](https://github.com/gohugoio/hugo) + [GitHub Action](https://github.com/features/actions) 的方式来完成静态网站的生成
