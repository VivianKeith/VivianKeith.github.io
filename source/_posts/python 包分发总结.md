---
title: python 包分发总结
copyright: true
top: 0
date: 2021-12-30 19:12:15
tags: python
categories: 技术
---

# 前言

平常我们习惯了使用 pip 来安装一些第三方模块，这个安装过程之所以简单，是因为模块开发者为我们默默地为我们做了所有繁杂的工作，而这个过程就是**==打包==**。

打包，就是将你的源代码进一步封装，并且将所有的项目部署工作都事先安排好，这样使用者拿到后即装即用，不用再操心如何部署的问题（如果你不想对照着一堆部署文档手工操作的话）。

不管你是在工作中，还是业余准备自己写一个可以上传到 PyPI 的项目，你都要学会如何打包你的项目。

# 1. 分发包类型
## 1.1 二进制包
二进制包提前根据平台信息构建包，安装时省去了编译的过程，直接进行解压安装，所以安装速度较源码包来说更快。
由于不同平台的编译出来的包无法通用，所以在发布时，需事先编译好多个平台的包。
二进制包的常见格式有：
- dumb
包括：gztar（Unix默认），bztar，xztar，ztar，tar，zip（windows默认）
- rpm
- pkgtool
- sdux
- wininst
- msi

### wheel
wheel是python新的发行标准，旨在替代传统的egg，pip >=1.4的版本均支持wheel， 使用wheel作为你python库的发行文件，有如下好处：

1.  纯Python和本机C扩展软件包的安装速度更快
2.  避免执行任意代码进行安装。
3.  C扩展的安装不需要在Linux，Windows或macOS上进行编译
4.  允许更好地缓存以进行测试和持续集成
5.  在安装过程中创建.pyc文件，以确保它们与使用的Python解释器匹配
6.  跨平台和机器的安装更加一致

本质上，wheel是一个zip压缩文件，将.whl扩展名替换为.zip，你就可以使用zip应用程序打开它，可以想象pip在安装wheel文件时，其过程也正是对它进行解压，然后复制到site-packges目录下，当然，实际的过程要比我刚才描述的要复杂一些，毕竟有很多事情要做，比如安装依赖。下面的代码向你展示如何解压一个wheel安装文件：

```python
from zipfile import ZipFile

with ZipFile("notebook-7.0.0-py3-none-any.whl", allowZip64=True) as z:
    z.extractall("./notebook")
```

解压后，可以在notebook目录下找到两个文件夹，分别是notebook-7.0.0.dist-info 和 notebook， notebook是源文件，notebook-7.0.0.dist-info是一些关键的安装信息， entry_points.txt中记录的是notebook的命令入口信息，METADATA记录了notebook的元信息，比如安装依赖，适用的平台，支持的python版本等等，pip就是根据这里的信息对库进行安装的。

### egg
Egg 格式是由 setuptools 在 2004 年引入，而 Wheel 格式是由 PEP427 在 2012 年定义。Wheel 的出现是为了替代 Egg，它的本质是一个zip包，其现在被认为是 Python 的二进制包的标准格式。
以下是 Wheel 和 Egg 的主要区别：
-   Wheel 有一个官方的 PEP427 来定义，而 Egg 没有 PEP 定义
-   Wheel 是一种分发格式，即打包格式。而 Egg 既是一种分发格式，也是一种运行时安装的格式，并且是可以被直接 import
-   Wheel 文件不会包含 .pyc 文件
-   Wheel 使用和 PEP376 兼容的 .dist-info 目录，而 Egg 使用 .egg-info 目录
-   Wheel 有着更丰富的命名规则。
-   Wheel 是有版本的。每个 Wheel 文件都包含 wheel 规范的版本和打包的实现
-   Wheel 在内部被 sysconfig path type 管理，因此转向其他格式也更容易

## 1.2 源码包
源码包安装的过程，是先解压，再编译，最后才安装，所以它是跨平台的，由于每次安装都要进行编译，相对二进包安装方式来说安装速度较慢。

源码包的本质是一个压缩包，其常见的格式有：

![](https://pic1.zhimg.com/v2-dffd77a245997a841eb426d9c0ce5184_b.jpg)

# 2. 打包工具
Python 发展了这么些年了，项目打包工具也已经很成熟了。他们都有哪些呢？

你可能听过 disutils、 distutils 、distutils2、setuptools等等，好像很熟悉，却又很陌生，他们都是什么关系呢？

## tldr:
>setuptools是增强版的distutils，基本满足大型项目的安装和发布，掌握setuptools就可以了。

## 包分发始祖：distutils
`distutils` 是 Python 的一个标准库，从命名上很容易看出它是一个分发（distribute）工具（utlis），它是 Python 官方开发的一个分发打包工具，所有后续的打包工具，全部都是基于它进行开发的。
`distutils` 的精髓在于编写 setup.py，它是模块分发与安装的指导文件。
那么如何编写 setup.py 呢？这里面的内容非常多，我会在后面进行详细的解析，请你耐心往下看。
你有可能没写过 setup.py ，但你绝对使用过 setup.py 来做一些事情，比如下面这条命令，我们经常用它来进行模块的安装。
```bash
$ python setup.py install
```

这样的安装方法是通过源码安装，与之对应的是通过二进制软件包的安装，同样我也会在后面进行介绍。

## 分发工具升级：setuptools
`setuptools` 是 distutils 增强版，不包括在标准库中。其扩展了很多功能，能够帮助开发者更好的创建和分发 Python 包。大部分 Python 用户都会使用更先进的 setuptools 模块。
**distribute**，或许你在其他地方也见过它，这里也提一下。
distribute 是 setuptools 有一个分支版本，分支的原因可能是有一部分开发者认为 setuptools 开发太慢了。但现在，distribute 又合并回了 setuptools 中。因此，我们可以认为它们是同一个东西。
还有一个大包分发工具是distutils2，其试图尝试充分利用distutils，detuptools 和 distribute 并成为 Python 标准库中的标准工具。但该计划并没有达到预期的目的，且已经是一个废弃的项目。
因此，setuptools 是一个优秀的，可靠的 Python 包安装与分发工具。

那么如何在一个干净的环境中安装 setuptools 呢？
主要以下几种方法：

1. 源码安装：在 [https://pypi.org/project/setuptools/#files](https://link.zhihu.com/?target=https%3A//pypi.org/project/setuptools/%23files) 中下载 zip 包 解压执行 `python setup.py install` 安装
2. 通过引导程序安装：下载引导程序，它可以用来下载或者更新最新版本的 setuptools
```bash
$ wget http://peak.telecommunity.com/dist/ez_setup.py

# 安装
$ python ez_setup.py

# 更新，以下两种任选
$ python ez_setup.py –U setuptools
$ pip install -U setuptools
```

3. 包管理工具直接安装：`sudo apt-get install python-setuptools`
4. pip 安装：`pip install setuptools`

# 3. 使用 setuptools 构建分发包
## step1：编写`setup.py`
比如一个`Hibiscus`项目，需要在项目根目录下编写一个`setup.py`文件，项目结构如下:

```text
├── Hibiscus
│   ├── __init__.py
│   ├── decorator
│   │   ├── __init__.py
│   │   ├── __pycache__
│   │   └── func_decorator.py
│   └── func_utils
│       ├── __init__.py
│       └── utils.py
└── setup.py
```

`setup.py`中需要引入`setuptools`，通过配置`setup`函数的参数完成对分发包的控制。一个简单的`setup.py`内容示例如下：
```python
from setuptools import setup, find_packages

setup(
    name='Hibiscus',
    version='0.0.1',
    author='酷python',
    author_email='pythonlinks@163.com',
    description='打包示例',
    url='http://www.coolpython.net',
    packages=find_packages(),
    classifiers=[
        "Programming Language :: Python :: 3",
        "License :: OSI Approved :: MIT License",
        "Operating System :: OS Independent",
    ],
    python_requires='>=3.6',
)
```

`setup`函数的参数具体含义可以看[这里](http://www.coolpython.net/python_senior/project/op_py_setup_install.html)和[这里](https://zhuanlan.zhihu.com/p/276461821)。


## step2：打包
有了上面的 setup.py 文件，我们就可以打出各种安装包，主要分为两类：sdist 和 bdist。 
sdist指的是source distribution，源码分发包。
bdist指的是built distribution, 二进制分发包。

通过执行`python setup.py sdist/bdist`来进行构建对应的源码包或二进制包。

下面分别进行说明：

### 构建源码包
构建源码包的命令为：
```bash
$ python setup.py sdist
```

使用 sdist 将根据当前平台创建默认格式的存档。在类 Unix 平台上，将创建后缀后为 `.tar.gz` 的 gzip 压缩的tar文件分发包，而在Windows上为`.zip`文件。
当然，你也可以通过指定你要的发布包格式来打破这个默认行为：

```bash
$ python setup.py sdist --formats=gztar,zip
```

执行后，项目目录下便会多出 dist 和 .egg-info 目录，dist 内保存了我们打好的包，上面命令使用 --formats 指定了打出 .tar.gz 和 .zip 包，如果不指定则如上表根据具体平台默认格式打包。 包的名称为 setup.py 中定义的 name, version以及指定的包格式，格式如：firstApp01-0.0.1.tar.gz。
前文说过，源码包实质上是一个压缩包，我们可以指定的源码包格式有哪些呢？
![](https://pic2.zhimg.com/v2-7416d2c5ffd4fde928d7bd214ea59d39_b.jpg)

对以上的格式，有几点需要注意一下：
-   在版本3.5中才添加了对 `xztar` 格式的支持
-   zip 格式需要你事先已安装相应的模块：zip程序或zipfile模块（已成为Python的标准库）
-   ztar 格式正在弃用，请尽量不要使用

另外，如果您希望归档文件的所有文件归root拥有，可以这样指定

```bash
$ python setup.py sdist --owner=root --group=root
```


### 构建二进制包
构建源码包的命令为：
```bash
$ python setup.py bdist
```

二进制包相比于源码包，由于预先构建好，所以安装更快。使用上，和 sdist 一样，可以使用 --formats 指定包格式。如：
```bash
$ python setup.py bdist --formats=rpm
```
可选的格式有：`tar.gz`,`rpm`，`srpm`，`wininst`，`msi`等。
同时为了简化操作，setuptools 提供了如下命令：
1. `python setup.py bdist_wininst/bdist_msi`
**note**: 在windows中我们习惯了双击 exe/msi 进行软件的安装，Python 模块的安装也同样支持 打包成 exe/msi 这样的二进制软件包。
2. `python setup.py bdist_rpm`
**note**:在 Linux 中，大家也习惯了使用 rpm 来安装包，对此你可以使用这条命令实现 rpm 包的构建
3. `python setup.py bdist_egg`
**note**:若你喜欢使用 easy_install 或者 pip 来安装离线包。你可以将其打包成 egg 包。==注意==：egg 包是过时的，下文所述的 wheel 包才是推荐的新标准。

如果我们的项目，需要安装到多个平台下，既有 Windows 也有 Linux，按照上面的方法，多种格式我们要执行多次命令，为了方便，我们可以一步到位，执行如下命令，即可生成多个格式的进制：
```bash
$ python setup.py bdist
```

**构建 wheel 包**
Wheel Wheel 也是一种 built 包，而且是官方推荐的打包方式。也许你曾经遇见或使用过 egg 包，但现在 [wheel](https://wheel.readthedocs.io/en/stable/) 是官方推荐的打包方式。
使用 wheel 打包，首先要安装 wheel：
```bash
$ pip install wheel
```

然后使用 `bdist_wheel` 打包：

```bash
$ python setup.py bdist_wheel
```

执行成功后，目录下除了 dist 和 .egg-info 目录外，还有一个 build 目录用于存储打包中间数据。 wheel 包的名称如 firstApp01-0.0.1-py3-none-any.whl，其中 py3 指明只支持 Python3。可以使用参数 --universal，包名如 mfirstApp-0.0.1-py2.py3-none-any.whl，表明 wheel 包同时支持 Python2 和 Python3使用 universal 也成为通用 wheel 包，反之称为纯 wheel 包。

也可以使用 wheel 直接打包：
```bash
$ pip wheel --wheel-dir=./wheel/
```
--wheel-dir 指定生成.whl文件的存储位置，上面的命令，是进入到setup.py文件所在目录执行的，因此使用的./ 表示当前目录，你可以在任意位置执行上面的命令，但是最后一部分必须是setup.py所在的目录。在setup.py所在的目录里，请将库的安装依赖写入到requirements.txt文件中，在制作.whl 安装包时会将requirements.txt里的安装依赖写入到dist-info 里的METADATA文件中。


# 分发包怎么安装？
## 从项目安装
正常情况下，我们都是通过以上构建的源码包或者二进制包进行模块的安装。
但在编写 `setup.py`的过程中，可能不能一步到位，需要多次调试，这时候如何测试自己写的 setup.py 文件是可用的呢？
这时候你可以使用这条命令，它会将你的模块安装至系统全局环境中：
```bash
$ python setup.py install
```
或者：
```bash
$ pip install .
```
应用开发过程中会频繁变更，每次安装都需要先卸载旧版本很麻烦。使用 develop 开发模式安装的话，实际代码不会拷贝到 site-packages 下，而是除一个指向当前应用的链接（.egg-link）。这样当前位置的源码改动就会马上反映到 site-packages，在修改包之后不用再安装就能生效，便于调试。使用如下：
```bash
$ python setup.py develop
```
或者：
```bash
$ pip install -e .
```

## 从分发包安装
构建好分发包后，可以使用pip安装：
- 安装二进制包（wheel）：
```bash
$ pip install dist/Hibiscus-0.0.1-py3-none-any.whl

```
- 安装源码包（tar.gz）：
```bash
$ pip install dist/Hibiscus-0.0.1.tar.gz
```
这种方法先编译出wheel文件，然后再进行安装，如下是其安装过程

```bash
Processing ./Hibiscus-0.0.1.tar.gz
Building wheels for collected packages: Hibiscus
  Building wheel for Hibiscus (setup.py) ... done
  Created wheel for Hibiscus: filename=Hibiscus-0.0.1-py3-none-any.whl size=2382 sha256=e4f46223ac138f0b30c3c7fdad3f3284f964144d35e31188fb3d58761aced350
  Stored in directory: /Users/kwsy/Library/Caches/pip/wheels/6f/9f/44/951064df5b5bba5bc41bf713804006c271b2a6946e139dd79c
Successfully built Hibiscus
Installing collected packages: Hibiscus
Successfully installed Hibiscus-0.0.1
```

- 解压tar.gz文件然后使用setup.py安装

对tar.gz解压，解压后的文件目录如下

```text
├── Hibiscus.egg-info
│   ├── PKG-INFO
│   ├── SOURCES.txt
│   ├── dependency_links.txt
│   └── top_level.txt
├── PKG-INFO
├── build
│   ├── bdist.macosx-10.9-x86_64
│   └── lib
│       └── hibiscus
│           ├── __init__.py
│           ├── decorator
│           │   ├── __init__.py
│           │   └── func_decorator.py
│           └── func_utils
│               ├── __init__.py
│               └── utils.py
├── dist
│   └── Hibiscus-0.0.1-py3.6.egg
├── hibiscus
│   ├── __init__.py
│   ├── decorator
│   │   ├── __init__.py
│   │   └── func_decorator.py
│   └── func_utils
│       ├── __init__.py
│       └── utils.py
├── setup.cfg
└── setup.py
```

执行命令：
```shell
$ python3 setup.py install
```

注意看dist目录里的Hibiscus-0.0.1-py3.6.egg，就是这个文件被安装到了site-packages目录下。

==使用pip安装和python命令安装的区别==：

使用python命令从项目源码直接进行安装：

```bash
$ python3 setup.py install
```

经过这种方式安装，会将Hibiscus-0.0.1-py3.6.egg文件安装到site-packages目录下，而如果使用pip命令安装，则会在site-packages目录下创建一个名为hibiscus的目录, 在这个目录下有程序的源码。

==简单总结==：pip安装后可以在site-packages目录下找到源码，而使用python命令直接安装，则只能找到一个.egg文件。

##  卸载
无论用那种方式安装，都可以用`pip uninstall`进行卸载。

# 4. 将分发包上传到 PyPI
我么已经知道了如何打包自己的项目，若你觉得自己开发的模块非常不错，想要 share 给其他人使用，你可以将其上传到 [PyPi ](https://pypi.org/)（Python Package Index）上，它是 Python 官方维护的第三方包仓库，用于统一存储和管理开发者发布的 Python 包。

## 直接使用 setuptools 上传
如果要发布自己的包，需要先到 pypi 上注册账号。然后创建 `~/.pypirc` 文件，此文件中配置 PyPI 访问地址和账号。如的.pypirc文件内容请根据自己的账号来修改。
典型的 .pypirc 文件
```text
[distutils]
index-servers = pypi

[pypi]
username:xxx
password:xxx
```

然后使用这条命令进行信息注册，完成后，你可以在 PyPi 上看到项目信息。
```bash
$ python setup.py register
```

注册完了后，你还要上传源码包，别人才使用下载安装
```bash
$ python setup.py upload
```

或者也可以使用 `twine` 工具注册上传，它是一个专门用于与 pypi 进行交互的工具，详情可以参考官网：

## 使用 twine 上传
虽然 setuptools 支持使用 setup.py upload 上传包文件到 PyPI，但只支持 HTTP 而被新的 twine 取代，它是一个专门用于与 pypi 进行交互的工具同样的，需要先安装 [twine](https://www.ctolib.com/twine.html)：
```bash
$ pip install twine
```

使用 twine 上传 使用 upload：
```bash
$ twine upload dist/*
```

输入 username 和 password 即上传至 PyPI。如果不想每次输入账号密码，同样可以在家目录下创建 `~/.pypirc` 文件，内容如下：
```
[distutils]
index-servers =
    pypi
    pypitest

[pypi]
username: 
password: 

[pypitest]
repository: https://test.pypi.org/legacy/
username: 
password:
复制代码
```

填上自己的账号密码即可，这里配置了官方的 pypi 和 pypitest，若要配置其他仓库，按格式添加。回到 PyPI 主页即可看到上传的分发包。

## PyPI 上传推荐配置

官网例子参考：

```python
from setuptools import setup, find_packages
setup(
    name="HelloWorld",
    version="0.1",
    packages=find_packages(),
    scripts=['say_hello.py'],

    # Project uses reStructuredText, so ensure that the docutils get
    # installed or upgraded on the target machine
    install_requires=['docutils>=0.3'],

    package_data={
        # If any package contains *.txt or *.rst files, include them:
        '': ['*.txt', '*.rst'],
        # And include any *.msg files found in the 'hello' package, too:
        'hello': ['*.msg'],
    },

    # metadata to display on PyPI
    author="Me",
    author_email="me@example.com",
    description="This is an Example Package",
    keywords="hello world example examples",
    url="http://example.com/HelloWorld/",   # project home page, if any
    project_urls={
        "Bug Tracker": "https://bugs.example.com/HelloWorld/",
        "Documentation": "https://docs.example.com/HelloWorld/",
        "Source Code": "https://code.example.com/HelloWorld/",
    },
    classifiers=[
        'License :: OSI Approved :: Python Software Foundation License'
    ]

    # could also include long_description, download_url, etc.
)

```

参考：https://setuptools.pypa.io/en/latest/setuptools.html#


# 参考
https://cloud.tencent.com/developer/article/1683436
https://packaging.python.org/en/latest/tutorials/packaging-projects/
https://zhuanlan.zhihu.com/p/276461821
https://zhuanlan.zhihu.com/p/354110980
https://juejin.cn/post/6844903906158313485
https://www.jianshu.com/p/ea9973091fdf
http://www.coolpython.net/python_senior/project/op_py_setup_install.html
https://blog.konghy.cn/2018/04/29/setup-dot-py/
