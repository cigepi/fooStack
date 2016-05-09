title: Getopt 使用教程并与 Getopts 比较
date: 2014-08-15 12:45
categories: Tech Logs
tags:
- Bash
- getopt
- getopts
- Shell
---

This articale has a [English Verison]({filename}getopt-versus-getopts-en.md).

## 概述

`getopt` 与 `getopts` 都是 Bash 中用来获取与分析命令行参数的工具，常用在 Shell 脚本中被用来分析脚本参数。


## 比较


1.	getopts 是 Shell 内建命令，getopt 是一个独立外部工具
2.	getopts 使用语法简单，getopt 使用语法较复杂
3.	getopts 不支持长参数（如：`--option` ），getopt 支持
4.	getopts 不会重排所有参数的顺序，getopt 会重排参数顺序（这里的区别下面会说明）
5.	getopts 出现的目的是为了代替 getopt 较快捷的执行参数分析工作


## 逐步了解 getopt

一个简单的 `getopt` 示例

```bash
#!/bin/bash

ARGS=`getopt -o "ao:" -l "arg,option:" -n "getopt.sh" -- "$@"`

eval set -- "${ARGS}"

while true; do
    case "${1}" in
        -a|--arg)
        shift;
        echo -e "arg: specified"
        ;;
        -o|--option)
        shift;
        if [[ -n "${1}" ]]; then
            echo -e "option: specified, value is ${1}"
            shift;
        fi
        ;;
        --)
        shift;
        break;
        ;;
    esac
done
```

执行示例

```console
# ./getopt.sh -a
arg: specified
# ./getopt.sh -a -o Apple
arg: specified
option: specified, value is Apple
# ./getopt.sh --arg --option Apple
arg: specified
option: specified, value is Apple
```

同样功能的 `getopts` 代码

```bash
#!/bin/bash

while getopts "ao:" OPT; do
    case ${OPT} in
        "a")
        echo -e "a: specified, short for arg"
        ;;
        "o")
        echo -e "o: specified, short for option, value is ${OPTARG}"
        ;;
    esac
done
```

执行示例

```console
# ./getopts.sh -a -o Apple
a: specified, short for arg
o: specified, short for option, value is Apple
```


`getopt` 的参数重排功能


```bash
#!/bin/bash

echo "${@}"
ARGS=`getopt -o "ao:" -l "arg,option:" -n "getopt_2.sh" -- "$@"`
eval set -- "${ARGS}"
echo "${@}"

while true; do
    case "${1}" in
        -a|--arg)
        shift;
        echo -e "arg: specified"
        ;;
        -o|--option)
        shift;
        if [[ -n "${1}" ]]; then
            echo -e "option: specified, value is ${1}"
            shift;
        fi
        ;;
        --)
        shift;
        break;
        ;;
    esac
done

echo "${@}"
```

执行示例

```console
# ./getopt_2.sh -o Apple Orange Banana
-o Apple Orange Banana
-o Apple -- Orange Banana
option: specified, value is Apple
Orange Banana
# ./getopt_2.sh Orange Banana -o Apple
Orange Banana -o Apple
-o Apple -- Orange Banana
option: specified, value is Apple
Orange Banana
```


##	进一步了解 `getopt`

如上文中的最后一个示例，getopt 对参数顺序进行了重排，这样可以将带 `-` 或 `--` 的参数写在其他参数的前面，也可以写在后面，而 getopts 是没有这样的能力的，具体没有的原因就是因为 getopts 直接进入了 `while` 循环处理参数，而 getopt 有一个 `set -- ${ARGS}` 的过程。另外还要注意到的是，在使用 getopt 处理完参数之后，`"${@}"` 变量“被清洗干净了” ，里面包含了所有不带 `-` 或 `--` 的参数，所以你可以继续使用 `${1}`，`${2}` 等来调用他们。

综上，getopts 语法简单又快速，getopt 虽然稍复杂，但是是功能上选。
3Rd
