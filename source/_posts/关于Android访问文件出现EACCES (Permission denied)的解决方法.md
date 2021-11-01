---
title: 关于Android访问文件出现EACCES (Permission denied)的解决方法
date: 2021-11-01 10:50:41
tags:
- android
- 文件
- EACCES
---

# 前言
最近在编码，有个需求需要把文件写到getExternalStorageDirectory获取到的路径中，但是即便我在Manifest 里面申请了

    <uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE" />
    <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE"/>

并且，动态获取到了权限，打开时仍旧提示EACCES。

<!--more-->
# 不同机型的测试
在google pixel 2中测试，可以正常读写文件；
在鸿蒙系统2.0的机型中测试，会导致崩溃，查阅原因，即文件读写权限问题，提示EACCES。

# 原因分析
通过查阅网络上的资料，了解到可能导致的原因如下：

## 1. IO操作不建议放在UI线程中执行
IO操作，如文件读写，特别是大文件或者大量文件读写，网络接口等；如果操作放在UI线程中，则容易导致UI线程阻塞，严重时会影响app闪退或者报错。而在android 4.0之后，Ui线程就已经不允许联网进行IO操作了。因此，如果涉及到IO操作，可以**把IO操作放到其它线程中去，或者放到协程中进行操作**。

## 2. android 10之后的版本特殊情况，HuaWei Ex
这里从官方文档有查到的解释是：以 Android 10（API 级别 29）及更高版本为目标平台的应用在默认情况下被赋予了对外部存储设备的分区访问权限（即分区存储）。此类应用只能看到本应用专有的目录（通过 Context.getExternalFilesDir() 访问）以及特定类型的媒体。
这种分区存储限制了应用通过绝对路径去打开文件，不能通过File file = new File(filePath)去打开，这类路径不具有直接内核访问权限。要访问此类文件，应用必须使用 MediaStore，并调用 openFile() 等方法。

若一定要从绝对路径中打开文件，可以通过在 Manifest 的 application 标签里面加上

	android:requestLegacyExternalStorage="true"

这个配置的意思是使用旧版本的存储规则。

# 总结
综合测试下来，特别是HuaWei的机型，一般都会有这个问题。经过测试，增加了requestLegacyExternalStorage配置之后就可正常进行文件操作。

