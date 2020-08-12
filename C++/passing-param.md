---
layout: post
title: "passing param"
subtitle: 'c++参数传递'
author: "zhutao"
header-style: text
tags:
  - cpp
---

# 数组参数传递

### 普通传递

我们知道，普通函数参数传递的实质是将实参的值赋值给形参
例程序1.1

```cpp
//1.1
void printdz(int a)
{
  cout<<"函数里a的地址为"<<&a<<endl;
}

int main()
{
  int a;
  cout<<"主函数里a的地址为"<<&a<<endl;
  printdz(a);
  return 0;
}
```

>运行结果
>
>主函数里a的地址为0x61ff1c
>函数里a的地址为0x61ff00

### 一维数组传递

然而，由于在C和C++中数组不能直接复制，传递时只传递一个首地址，在函数中需要使用的时候再跟据首地址和下标去寻找对应的值。
至于为何数组不能直接复制，笔者在网上找到了两种解释，有所异同，读者可以自行鉴别。

- 1.是为了避免不必要的复制开销，因为数组的复制将导致连续的内存读与内存写，其时间开销取决于数组长度，有可能会变得非常大。
  为了避免复制数组的开销，才用指针代替数组。因此C语言使得当数组作为实参传递给函数的时候，将退化为同类型的指针，再传递指针的值。
  因此，在函数中修改数组值时，修改的是真的值。
- 2.当初开发C语言时为了可见性，dmr 在 C 语言里每个 operation都可以直接翻译为简单的汇编语句（指令），是 constant time。而数组赋值需要循环，一个 = 操作就不再是 constant time，这违背了初衷。
  这也是 C 语言没有乘方运算符的原因，因为没有对应的汇编指令，实现乘方需要用循环。
  后来 C 语言支持了 struct 的赋值，constant time 这一原则被打破了，这是后话。

下面，我们观察**数组作为函数参数**时的地址情况
例程序1.2

```cpp
//1.2
void printdz(int a[])
{
  cout<<"函数里a0的地址为"<<&a[0]<<endl;
  cout<<"函数里a的地址为"<<a<<endl; //a本身即为指针 无需取地址符
  cout<<"函数里a1的首地址为"<<&a[1]<<endl;
  a[1]=2;
}
int main()
{
  int a[10]={1};
  cout<<"主函数里a的首地址为"<<&a[0]<<endl;
  cout<<"开始的a1为 "<<a[1]<<endl;
  printdz(a);
  cout<<"函数里改过的a1 为"<<a[1]<<endl;
  return 0;
}
```

运行结果

>主函数里a[0]的首地址为0x61fee8 //void printdz(int a[10]) correct
>主函数里a的首地址为0x61fee8
>开始的a1为 0
>函数里a[0]的地址为0x61fee8
>函数里a的地址为0x61fee8
>函数里a1的首地址为0x61feec
>函数里改过的a1 为2

在这里我们可以看到函数里的首地址a[0]与主函数的首地址是相同的。且a已经是一个指针了。
由于是通过指针传递，因此无法得到数组的长度。
除了退化为指针传递，还可以直接通过指针传递和通过引用传递。

通过**指针传递**:
例程序1.3

```cpp
void printdzyy(int *aa) //传入指针 or void printdzyy(int aa[])
{
	cout<<"函数里参数的地址为"<<&aa[0]<<endl;
}
int main()
{
  int a[10]={1};
	int b[5]={1};
  cout<<"主函数里a[0]的首地址为"<<&a[0]<<endl;
	printdzyy(a);
  printdzyy(b);//传入指针时 编译器无法知道数组长度 因此可以随便传
	return 0;
}
```

通过**引用传递**:
例程序1.4

```cpp
//1.4
void printdzyy(int (&aa)[10]) //引用就可以传递数组长度 因此需要写出数组大小
{
	cout<<"函数里aa的地址为"<<aa<<endl;
}
int main()
{
  int a[10]={1};
	int b[5]={1};
  cout<<"主函数里a[0]的首地址为"<<&a[0]<<endl;
	printdzyy(a);
  //printdzyy(b);  incorrect  //如果传入b则编译不通过
	return 0;
}
```

如1.3 1.4程序可知，**传入指针时，编译器无法知道数组长度**，因此可以传递进去不同长度的数组,也可以不写出数组长度。但是传递引用时，同时将数组长度也传递进去了，所以传入时必须为相应长度的数组。并且，在**传递引用时，需要注意用（）将&aa括起来，否则编译器报错**。
对于引用，也叫“别名”，某种意义上可作为无地址的指针，`相比指针，引用不存在地址，必须被初始化，不存在NULL，不可改变对象，引用无法被多重引用。因此更加安全。`

### 二维数组传递

可以理解为数组的定义等同于指针的定义，即*a等同于a[],因此可以作如下变换

```cpp
//1.5
void func1(int iArray[][10])  //等同于 void func1(int (*iArray)[10])
{}  //等同于void func1(int iArray[10][10])
int main()  
{  
    int array[10][10];  
    func1(array);
}  
```

`注意*（iAarray）可通过，去掉括号会编译错误`

由于二维数组在栈内分配的内存是连续的，需要告诉编译器偏移的点，**所以第二维度不可省**，必须符合，而第一维度如一维数组一样随意。因此在上述代码1.5 中`void func1(int iArray[9][10])`不会报错，而`void func1(int iArray[10][9])`会报错。

同一维数组，这边推荐使用引用，使用引用时需要保证两个维度都要ok：

```cpp
//1.6
void func3(int (&pArray)[10][10])  
{  }  
int main()  
{  
    int array[10][10];  
    func3(array);  
}  
```

也可以指针的指针来表示二维数组，动态分配内存的形式，此时，严格来说，并不是二维数组。
此处代码源于 https://www.cnblogs.com/L-Lotus-F/p/4377998.html

```cpp
#include <iostream>
#include <stdio.h>
void out(double **a,int m, int n)
{
    int i, j;
    double b=0.0;
    for(i=0; i<m; i++)
    {for (j=0; j<n; j++)
        {
            a[i][j] = b;
            b += 1.2;
            printf("%5.1f",a[i][j]);
        }
        std::cout << std::endl; }   
}
int main(int argc, char * agrv)
{
    int i, j, m=2, n=3;
    double **a;
    a = new double*[m];
    for (i=0; i<m; i++)
        a[i] = new double[n];
    out(a,m,n);
    return 1;
}
```

### 总结

在传递数组参数时，首要推荐通过引用来传递参数，精准传入数组大小值，函数中参数定义为 `vtype （&name）[size][size]`,引用时传入名字即可。

在通过指针传递时，需要另外传入参数来传递数组的大小，\*的数量表示数组的维度。

