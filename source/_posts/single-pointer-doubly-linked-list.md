title: 使用单指针域实现双向链表
date: 2014-08-30 00:00
categories: Coding Things
tags:
- C-lang
---

## 数学基础

离散数学中的异或运算 `a⊕b`，具有以下性质：

- `a⊕b = b⊕a`
- `a⊕a = 0`
- `a⊕0 = a`
- `a⊕(a⊕b) = (a⊕a)⊕b = b`
- `(a⊕b)⊕b = a⊕(b⊕b) = a`

利用异或运算的这些性质，我们可以只用一个指针域，来实现一个双向链表。

## 单指针域双向链表的逻辑结构

下图是一个具有五个节点的双向链表的逻辑结构示意图，没有头结点。

![SignlePointerDoublyLinkedList](http://static-hiaero.b0.upaiyun.com/main/wp-content/uploads/2014/09/SinglePointerDoublyLinkedList.png)

其中每个节点的后半部分表示指针域，存储了它的前驱节点的指针与后继借点的指针的异或值。我们从最左边的节点开始遍历，就可以使用 `0 ^ (0^P1) = P1` 来得到下一个节点的指针（请注意，此处 `0^P1` 是一个整体，直接从节点的指针域中获得的）。继续往右走，又可以使用 `P0 ^ (P0^P3)` 来得到 `P3`，并以此类推。从右节点开始遍历也是同理（如：`(P4^0) ^ 0 = P4`）。

因此，按照上面的数据结构，就可以只使用一个指针域来实现双向链表了。

## 代码实现

```c
#include <stdio.h>
#include <stdlib.h>

#define OK 1
#define ERROR 0

typedef int Status;

typedef struct DLink
{
    int data;
    unsigned long link;
} DLink;

// Initialize a SPDLL
Status CreateList(DLink **L, DLink **R, int *Elements, int n);
// Traverse from Left to Right
Status DispL(DLink *L);
// Traverse from Right to Left
Status DispR(DLink *R);

int main()
{
    DLink *L, *R;
    int Elements[] = {1, 2, 3, 4, 5};
    CreateList(&L, &R, Elements, 5);
    printf("Traverse from Left to Right: ");
    DispL(L);
    printf("\n");
    printf("Traverse from Right to Left: ");
    DispR(R);
    printf("\n");
}

// Initialize a SPDLL
Status CreateList(DLink **L, DLink **R, int *Elements, int n)
{
    DLink *pre, *q;
    int i;
    unsigned long l = 0, r = 0;

    // L 是指向头结点的指针
    (*L) = (DLink *)malloc(sizeof(DLink));
    (*L)->data = Elements[0];

    pre = *L;
    for (i=1; i<n; i++)
    {
        // 这是生成的新节点
        q = (DLink *)malloc(sizeof(DLink));
        // 为新节点的数据域赋值
        q->data = *(Elements+i);
        /* l 为 pre 前驱节点的指针地址经强制转换的长整型值，
        q 为 pre 的后继节点的地址，
        因此将 l ^ (unsigned long)q 赋值给 pre 的指针域 */
        pre->link = l ^ (unsigned long)q;
        // 然后将分别将 l 与 pre 向前移动一个身位 :D
        l = (unsigned long)pre;
        pre = q;
    }
    // 最后，再将末尾节点的指针域赋值为次末尾节点与0的异或值即可
    pre->link = (unsigned long)l ^ r;
    /* R 是指向末尾节点的指针，如果要从右向左遍历这个链表的话，
    需要从尾指针开始，因此每个单指针双向非循环链表都需要一个头指针和尾指针 */
    (*R) = pre;
    return OK;
}

// Traverse from Left to Right
Status DispL(DLink *L)
{
    unsigned long l = 0, next;
    DLink *p = L;
    // 首先将首节点的值直接输出
    printf("%d ", p->data);
    while (1)
    {
        // 由当前节点 p 的前驱节点与 p 的指针域异或得到下一个节点的地址
        next = l ^ ((unsigned long)(p->link));
        l = (unsigned long)p;
        if (next == 0) break;
        p = (DLink *)next;
        printf("%d ", p->data);
    }
    return OK;
}

// Traverse from Right to Left
// 从右往左遍历，需要从尾指针开始，因此每个单指针双向非循环链表都需要一个头指针和尾指针
Status DispR(DLink *R)
{
    unsigned long r = 0, prior;
    DLink *p = R;
    printf("%d ", p->data);
    while (1)
    {
        prior = ((unsigned long)(p->link)) ^ r;
        r = (unsigned long)p;
        if (prior == 0) break;
        p = (DLink *)prior;
        printf("%d ", p->data);
    }
    return OK;
}
```

### 执行结果

```console
# gcc ./SinglePointDoubleLinkList.c -o ./SinglePointDoubleLinkList
# ./SinglePointDoubleLinkList
Traverse from Left to Right: 1 2 3 4 5
Traverse from Right to Left: 5 4 3 2 1
```
