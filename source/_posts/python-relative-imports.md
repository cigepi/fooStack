title: Python 包的相对导入讲解
date: 2014-01-31 08:19
categories: Coding Things
tags:
- Python
---

## 两个常见错误

### *ValueError: Attempted relative import in non-package*

#### 错误重现

目录树

```shell-session
case1/
├── cat
│   ├── __init__.py
│   └── cat.py
├── dog
│   ├── __init__.py
│   └── dog.py
└── main.py
```

代码


```console
# cat case1/cat/cat.py
from .. import dog
```

执行

```cosole
# python case1/cat/cat.py
```

#### 错误原因

python 使用一个模块的属性 `__name__`来决定他在包结构中的位置，所以当直接执行 `cat.py` 时，`__name__ = '__main__'`，`cat.py` 被当作顶层模块来处理了，自然不可能导入上层的任何对象了。

### *ValueError: Attempted relative import beyond toplevel package*

#### 错误重现

目录树

```shell-session
case2/
├── cat
│   ├── __init__.py
│   ├── cat.py
│   └── cat.pyc
├── dog
│   ├── __init__.py
│   └── dog.py
├── __init__.py
└── main.py
```

代码

```shell-session
# cat case2/cat/cat.py
from .. import dog

# cat case2/main.py
import cat.cat
```

执行

```cosole
# python case2/main.py
```

#### 错误原因

这里的 `case2` 是一个包，但当你直接执行 `main.py` 的时候，就没有把完整的 `case2` 当作一个包来处理了，可想而知，下层的 `cat.py` 自然找不到上层包了。**即想要相对导入成功，必须让下层的被导入对象是上层包或上层包内的对象。**

## 正确执行的例子

目录树

```shell-sesion
case3/
├── alpaca.py
├── main.py
└── pets
    ├── __init__.py
    ├── cat
    │   ├── __init__.py
    │   └── cat.py
    └── dog
        ├── __init__.py
        └── dog.py
```

代码

```shell-session
# cat case3/pets/cat/cat.py
from ..dog import dog
from .. import dog

# cat case3/main.py
from pets.cat import cat
```

执行

```console
# python case3/main.py
```

请注意，这里的 `cat.py` 中是不能导入 `alpaca` 的，因为 `pets` 已经是这个包树的顶层了。

[示例代码下载](http://hiaero.net/wp-content/uploads/2014/01/python-relative-imports.zip)
