---
title: 【C 语言基础】函数
date: 2021-03-11 12:18:16
tags:
- C
cover: https://w.wallhaven.cc/full/e7/wallhaven-e7z65r.jpg
---

{ %  folding blue, 【基本知识】函数 % } 

1. 三要素：
    - `int is_prime(int x)`
      - int：返回值，没有返回值用 void
      - is_prime：函数名，贴合实际用处
      - (int x)：参数声明列表
2. 两大组成部分
    - 函数声明：
      - 向编译器声明是一个函数（编译器从主函数作为入口函数，按顺序进行编译），将函数定义放在调用之前，把函数声明当作定义放在前面
      - int is_prime(int);
      - 可以重复声明
    - 函数定义：函数的三要素 + 花括号内实现功能
      - 不能重复定义，唯一

3. 函数与数组的关系：映射
    - 函数是压缩的数组，数组是展开的函数：y = f(x)，arr[2] = 100

{ %  endfolding % } 

{ %  folding blue, 【基本知识】递归（套娃） % } 

# 概念

1. 递归：从程序调用自身的**编程技巧**，不是一种算法（递推是一种算法）

2. 递归程序的组成部分：
    - 边界条件处理：避免出现死循环
    - 针对于问题的**处理过程**和**递归过程**
    - 结果返回
      - return 关键字
      - 传出参数：当前参数接收的是地址 / 指针变量
    - 注意当前递归函数合理的**语义信息**的设定

3. 两大过程
   - 向下**递推**：函数调用
   - 向上**回归**：回溯

4. 验证：数学归纳法

5. 递归的背后：借助于系统栈（Linux 系统 8MB，Windows 系统 2MB）
   - 爆栈 / 栈溢出 / segment fault
   - 原因：① 临时数组开太大，内存上的栈区放不下；② 函数调用层数过多
   - 静态数组：通过类型直接定义的数组`int arr[10];`；当上述语句放在函数内部，放在栈区储存；放在函数外部，放在全局区存储
   - malloc / realloc / calloc 创造的数组存储在**堆区**

# 案例 → n 的阶乘

0! = 1：1! = 1，根据 1! = 1 * 0!，所以 0! = 1 而不是 0

```c
#include <stdio.h>

int fac(int n) {
    // 递归出口 / 边界条件
    if (n == 1) return 1;
    return n * fac(n - 1);
}   

int main() {
    int n;
    while (~scanf("%d", &n)) {
        printf("fac(%d) = %d\n", n, fac(n));
    }
    return 0;
}
```

# 头递归 & 尾递归


# 欧几里德算法 / 辗转相除



{ %  endfolding % } 

{ %  folding blue, 函数指针 % } 

`把一个函数当成参数传递给另一个函数使用`
- 指针是一个变量（变量的本质是存值，变量类型就是为了存储不同的值），指针变量存储的值是地址，函数指针变量存的是函数地址

```c
// int (*f1)(int)
// 意味着和上述相同，返回值为 int，参数列表为 int
// 分段函数
int g(int (*f1)(int), int (*f2)(int), int (*f3)(int), int x) {
    if (x < 0) {
        return f1(x);
    }
    if (x < 100) {
        return f2(x);
    }
    return f3(x);
}
```

PE-45（欧拉计划）函数指针的应用

满足三边形数、五边形数、六边形数

```c
#include <stdio.h>

typedef long long int1;
int1 Triangle(int1 n) {
    return n * (n + 1) >> 1;
}

int1 Pentagonal(int1 n) {
    return n * (3 * n - 1) >> 1;
}

int1 Hexagonal(int1 n) {
    return n * (2 * n - 1);
}

int1 binary_search(int1 (*arr)(int), int n, int1 x) {
    int head = 1, tail = n, mid;
    while (head <= tail) {
        mid = (head + tail) >> 1;
        if (arr(mid) == x) return 1;
        if (arr(mid) < x) head = mid + 1;
        else tail = mid - 1;
    }
    return 0;
}

int main() {
    int n = 143;
    while (1) {
        n++;
        int1 temp = Hexagonal(n);
        // 是六边形数一定是三角形数，所以下面语句可以注释
        if (!binary_search(Triangle, temp, temp)) continue;
        if (!binary_search(Pentagonal, temp, temp)) continue;
        printf("%lld\n", temp);
        break;
    }
}
```

{ %  endfolding % } 


{ %  folding blue, 变参函数 % } 

{ %  endfolding % } 

++i 比 i++ 速度快，但是可以忽略不计

- 数据表示：大整数