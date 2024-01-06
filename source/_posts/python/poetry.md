---
title: Poetry 简明笔记
date: 2023-01-06 23:08:29
tags: 
  - python
  - env
---


## 1 安装poetry：

官网好几种安装方式，此处使用pipx进行安装，pipx安装内容全部放到一个单独的虚拟环境中，poetry官网建议将poetry安装到虚拟环境中使用，使用pipx正合适并且也简单便捷。

```bash
pip install pipx
pipx install poetry

# 将pipx的管理的可执行文件目录追加环境变量中
echo 'export PATH=$PATH:$(pipx environment | grep PIPX_BIN_DIR=/ | awk -F= '\''{print $2}'\'')' >> ~/.zshrc
source ~/.zshrc

poetry -V
```

## 2 初始化项目

```bash
# 初始化当前文件夹，将现有文件夹变成一个poetry管理的项目
poetry init

# 切换到指定的python版本
poetry env use [python-executable-versionn]

# 进入虚拟环境
poetry shell

# 退出虚拟环境
exit
```

## 3 管理依赖

```bash
# 添加一个新的依赖到当前项目下，完成后会添加到pyproject.toml的依赖中
poetry add [package-name]

# 删除当前项目下的一个依赖，完成后会从pyproject.toml的依赖中移除
poetry remove [package-name]

# 已存在的poetry项目，安装pyproject.toml中的依赖
poetry install
```

## 4 使用国内源

一般使用`pip install requirements.txt`时，都通过`-i`指定国内仓库源，在`pyproject.toml`中追加以下内容后，无需在关心数据源问题。

```bash
[[tool.poetry.source]]
name = "tsinghua"
url = "https://pypi.tuna.tsinghua.edu.cn/simple/"
priority = "default"
```

## 5 其他

### 5.1 .venv文件位置

poetry是基于venv的增强，所以还是会创建一个venv文件夹用于存放虚拟环境的文件，默认在系统用户目录的缓存文件夹，修改一下配置后，venv文件夹默认会创建到项目的根文件夹内。

```bash
poetry config virtualenvs.in-project true
```

### 4.2 现有项目使用poetry管理依赖