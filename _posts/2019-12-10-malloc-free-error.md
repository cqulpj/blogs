---
layout: post
title:  "C语言malloc_free踩坑记之free报错"
author: lpj
categories: [ C, 技术 ]
image: assets/images/5.jpg
tags: featured
---

在写一个字符串解析的小程序时，因为传进来的有可能时const char*，所以在程序里首先malloc了一块内存区域将原始字符串进行了复制以进行后续处理，处理完了释放掉，测试的时候发现有个别测试用例会运行时异常，经过log调试把问题缩小到free处，即释放时出错。问题抽象如下：

## 一、问题

有一个const char *变量cs存储有字符串，用malloc申请跟cs长度相同的内存区域赋给指针ps，然后用strcpy将cs的内容复制到ps，最后释放ps，程序概率性报错，代码如下：
```c
int main()
{
    char *cs="12345678901212345678901", *ps=NULL;

    ps = (char *)malloc(strlen(cs));
    strcpy(ps, cs);
    free(ps);

    return 0;
}
```

经进一步测试，当cs字符串的长度为24时会报错，24以下的其他值不报错

## 二、分析

1. 先不管概率性报错这个现象，只从代码上分析，有一个比较严重的问题：strcpy复制时，其实复制了strlen(cs)+1个字节的内容，因为它要复制字符串结束符'\0'，具体说比如cs内容是"hello", 按照程序执行逻辑，ps分配了5个字节的内存，然后strcpy语句执行完，从ps地址开始的6个字节内容都被改变了，第六个字节是'\0'，所以malloc那句代码分配长度应该为strlen(cs)+1，这样改以后程序就不会有报错的情况出现

2. 然后再说概率报错的现象，如果将cs内容改为"hello"，上边的程序是能正常运行的，也就是说free语句不会报错，经过在strcpy前、strcpy后、free后对ps所指向内存区域的数据进行对比发现，在ps有效内存区域（分配的长度）后有几个字节在free后会发生改变，结合网上查到的内容，malloc分配内存后，会在这段内存区域后面保存该次malloc的信息，free的时候也会根据这个记录信息进行内存操作，因此可以得出结论：报错是因为strcpy复制时多出来那个字节修改了malloc的记录信息，那为什么有时候不报错呢，这个没找到答案，猜测可能是字节对齐引起的，比如malloc分配的区域对N字节取整补齐后才是记录信息

## 三、结论

理解strcpy复制的操作内容，如果要动态分配区域作为strcpy的目标地址，那malloc长度至少要是strlen(cs)+1，如果长度刚好是strlen(cs)，那strcpy操作会越界一个字节，可能造成异常结果

