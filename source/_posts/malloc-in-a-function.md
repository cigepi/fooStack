title: 在函数内部使用 malloc 分配内存
date: 2014-08-29 10:38
categories: Coding Things
tags:
- C-lang
---

## 错误的例子

下面这段程序中，`main` 函数中的 `printf` 语句会正确的打印 5 吗？

**不会**

```c
#include <stdio.h>
#include <stdlib.h>

void ChangeN(int *p);

int main(void)
{
    // 定义了一个 int 类型的指针
    int *i;
    // 该函数试图给指针 i，分配一段内存并赋一个初值
    ChangeN(i);
    // 打印 i 指向的地址空间的值
    printf("%d\n", *i);
}

void ChangeN(int *p)
{
    p = malloc(sizeof(int));
    *p = 5;
}
```

#### 执行结果

```cosole
# gcc ./int.c -o ./int
# ./int
Segmentation fault
```

#### 拙计

1. 在 `main` 函数中，指针变量 `i` 被声明之后，还只是一个空指针，并没有被分配空间
2. 指针 `p` 作为形参，代码执行进入函数内后，`p` 依旧是一个空指针，而且与 `i` 不是同一个
3. 函数 `ChangeN` 在其内部新申请了一块内存空间，并将这块内存空间的地址赋予了指针变量 `p` 中，但函数外的指针 `i` 并没有获得该地址。因此在 `ChangeN` 函数执行结束后，指针 `p` 被销毁了，他指向的地址空间也无从找寻了，`i` 还依旧是那个空指针，肯定就已经无法访问到 `p` 原本指向的空间了。**对上面的程序稍加修改，就能很好的说明这个问题**

### 例子二

```c
#include <stdio.h>
#include <stdlib.h>

void ChangeN(int *p);

int main(void)
{
    int *i;
    ChangeN(i);
    // 修改1, 改为打印 i 指向的地址
    printf("i is located at %p\n", i);
}

void ChangeN(int *p)
{
    p = malloc(sizeof(int));
    *p = 5;
    printf("***** Start ChangeN *****\n");
    // 修改2，打印函数内 p 分配空间后，指向的空间地址
    printf("p is located at %p\n", p);
    printf("*p = %d\n", *p);
    printf("***** End ChangeN *****\n");
}
```

#### 例子二执行结果

```cosole
# gcc ./int_1.c -o int_1
# ./int_1
***** Start ChangeN *****
p is located at 0x1110010
*p = 5
***** End ChangeN *****
i is located at (nil)
```

## 正确的例子

正确的在函数内为外部的变量分配内存空间的方法为：

1. 在函数外部申请指针变量 `i`
2. 将函数的形参设为指向指针的指针 `p`
3. 将指针变量 `i` 的地址作为实参传递给函数
4. 在函数内申请内存空间后，将地址赋给 `*p`

```c
#include <stdio.h>
#include <stdlib.h>

// 将函数的形参声明为指向指针的指针
void ChangeN(int **p);

int main(void)
{
    int *i;
    // 将指针变量的变量地址传给函数
    ChangeN(&i);
    printf("%d\n", *i);
}

void ChangeN(int **p)
{
    // 为指针 p 指向的指针分配内存空间
    *p = malloc(sizeof(int));
    // 为指针 p 指向的指针赋值
    **p = 5;
}
```

#### 执行结果

```console
# gcc ./int_2.c -o ./int_2
# ./int_2
5
```
#
