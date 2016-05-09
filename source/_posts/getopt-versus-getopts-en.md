lang: en
title: Getopt versus Getopts
date: 2014-08-15 12:45
categories: Tech Logs
tags:
- Bash
- getopt
- getopts
- Shell
---

这篇文章有一个[中文版]({filename}getopt-versus-getopts.md)。

I wrote a [Chinese version]({filename}getopt-versus-getopts.md) of this post months ago to interpret some basic differce between bash command `getopt` and `getopts`. Now I am writing the English version to try to help me get a Bash Scripting related job on freelancer. :D

## Overview

Both `getopt` and `getopts` are used to get and parse the options and arguments of your shell script. In other word, if you write a shell script, now you want to add some user-defined options(eg. '--help') to you new software, use this two commands.

## What is the difference

1. `getopts` is a bash builtin command, `getopt` is an external software
2. `getopts` has a simple syntax, `getopt` is more complex but powerfull
3. `getopts` does not support long parameters like `--help`, `getopt` does support
4. `getopts` dose not reorder your parameters but `getopt` dose (will explain detailed below)
5. `getopts` is be constructed to do `getopt`'s job simpler and quicker

## Get closer to `getopt`

a simple `getopt` example

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

execution results

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

the same functions implemented by `getopts`

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

execution results

```console
# ./getopts.sh -a -o Apple
a: specified, short for arg
o: specified, short for option, value is Apple
```

the parameters reorder ability of `getopt`

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

execution results

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

## How dose `getopt` reorder the parameters but `getopts` not

Just like the last example above, `getopt` rearranged your parameters' sequence. The meaning is that you can write parameters with `-` or `--` behind or ahead other plain parameters(eg. `Orange`) when use `getopt`, but you cannot do this with `getopts`. The underlying reason is `getopts` put your parameters into the `while` loop directly, but `getopts` deal your parameters with the command `set - ${ARGS}` before put them into the `while` loop.

Another import thing your should notice is, the variable `${@}` is 'cleaned up' after processed by `set - ${ARGS}`. You can still use `${1}`, `${2}` to get all parameters which are not led by `-` or `--`.
