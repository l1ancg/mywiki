---
title: Linux三大利器之awk
date: 2021-12-05 18:24:33
tags:
  - shell
  - linux
---

## 命令

### 命令分类
- 内建命令：属于shell的组成部分，不需要子线程执行
- 外部文件命令：通常位于/bin、/sbin、/usr/bin、/usr/sbin等目录中，父进程fork子进程，子线程执行外部命令

### 常用命令
#### 关键和目录
- cd：进入指定路径
- pwd：打印当前路径
- ls：查看当前文件夹的文件列表
- touch：创建空文件、
- cp、mv、rm：复制、移动、删除一个文件或目录
- mkdir：创建、删除一个目录
- file：文件类型
- cat：文件文本内容
- more：打开一个文件以交互式的阅读，只允许向下滚动
- less：类似more，可以上下滚动，不会加载全部文件
- tail、head：查看一个文件的末尾、开头

#### 系统管理：
- ps：显示线程运行信息
- top：动态的现实线程运行信息
- kill、killall：向线程发送终止信号

#### 磁盘空间
- mount：磁盘挂在操作、查看磁盘挂载信息
- umount:取消磁盘挂载
- df：查看当前目录占用磁盘空间大小
- du：查看指定文件占用磁盘空间大小

#### 数据文件
- sort：文本文件排序
- grep：查找文件中包好的字符串，可使用正则表达式
- gzip：zip格式文件压缩、解压缩
- tar：文件的归档、压缩、解压缩

查看命令手册：man [command]

## 脚本

### 指定执行脚本的工具：
#!/bin/bash

### 执行状态吗：
- $?：查看上一个命令的执行状态码，0：命令执行成功
- exit 1：指定执行状态码

### 示例
脚本内容
```shell
#!/bin/bash

echo "current date is ${date}"
echo "current UID is ${UID}"
echo "current user HOME is ${HOME}"
exit 1
```
执行脚本
```shell
> chmod +x script.sh
> ./script.sh
```
查看执行状态码
```shell
> echo $?
> 1
```