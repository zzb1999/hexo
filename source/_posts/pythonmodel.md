---
title: Python模块发布和安装
date: 2018-01-23 19:29:47
tags:
	- python
categories: Python
---
### 模块发布

1.mymodule目录结构体如下：
``` bash
.
├── setup.py
├── suba
│   ├── aa.py
│   ├── bb.py
│   └── __init__.py
└── subb
    ├── cc.py
    ├── dd.py
    └── __init__.py
```
2.编辑setup.py文件
py_modules需指明所需包含的py文件
``` bash
from distutils.core import setup

setup(name="ixysec", version="1.0", description="ixysec's module", author="ixysec", py_modules=['suba.aa', 'suba.bb', 'subb.cc', 'subb.dd'])
```
3.构建模块
python setup.py build
构建后目录结构
``` bash
.
├── build
│   └── lib.linux-i686-2.7
│       ├── suba
│       │   ├── aa.py
│       │   ├── bb.py
│       │   └── __init__.py
│       └── subb
│           ├── cc.py
│           ├── dd.py
│           └── __init__.py
├── setup.py
├── suba
│   ├── aa.py
│   ├── bb.py
│   └── __init__.py
└── subb
    ├── cc.py
    ├── dd.py
    └── __init__.py
```
4.生成发布压缩包
python setup.py sdist
打包后,生成最终发布压缩包ixysec-1.0.tar.gz , 目录结构
``` bash
.
├── build
│   └── lib.linux-i686-2.7
│       ├── suba
│       │   ├── aa.py
│       │   ├── bb.py
│       │   └── __init__.py
│       └── subb
│           ├── cc.py
│           ├── dd.py
│           └── __init__.py
├── dist
│   └── ixysec-1.0.tar.gz
├── MANIFEST
├── setup.py
├── suba
│   ├── aa.py
│   ├── bb.py
│   └── __init__.py
└── subb
    ├── cc.py
    ├── dd.py
    └── __init__.py
```
### 模块安装
1. 找到模块的压缩包
2. 解压
3. 进入文件夹
4. 执行命令python setup.py install