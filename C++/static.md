---
layout: post
title: "static"
subtitle: 'static'
author: "zhutao"
header-style: text
tags:
  - cpp
---

类的静态成员有两种：**静态成员变量和静态成员函数**，语法是在普通成员变量和成员函数前加**static**关键字。

## 定义

```cpp
class CRect{
public:
	void show();//普通成员函数
	static void printTotal();//静态成员函数
private:
	int width, height;//普通成员变量
	static int totalNumber;//静态成员变量
	static int totalArea;//静态成员变量
};
```

## 存在原因

静态成员变量在本质上是**全局变量**。一个类，哪怕一个对象都不存在，其静态成员变量也是存在的。静态成员函数并不需要作用在某个具体的对象上，因此本质上是全局函数。设置静态成员的目的，是为了将某些和类紧密相关的全局变量和全局函数写到类里面，形式上成为一个整体，达到封装的效果，但其效果与定义全局变量/函数相同。

## 静态与静态成员变量/函数的区别

- 普通成员变量每个对象各自持有一份，而静态成员变量只有一份，被所有同类对象共享；
- 普通成员函数一定是作用在某个对象上的，而静态成员函数并不具体作用在某个对象上；
- 访问方式：访问普通成员时，要通过对象名.成员名的方式，指明要访问的成员变量是属于哪个对象的，或要调用的成员函数作用于哪个对象；**访问静态成员时，则可以通过类名::成员名的方式访问，不需要指明被访问的成员属于哪个对象或作用于哪个对象。**因此，甚至可以在还没有任何对象生成时就访问一个类的静态成员；
- 占用空间：当使用sizeof计算类所占用空间时**只计算非静态成员变量的值**，如sizeof(CRect)=8，两个int类型的width和height。
- 参数传递：普通成员函数在参数传递时编译器会隐藏地传递一个this指针.通过this指针来确定调用类产生的哪个对象;但是静态成员函数没有this指针,不知道应该访问哪个对象中的数据,所以在**程序中不可以用静态成员函数访问类中的普通变量**。

## 一些规则

- 对象与对象之间的成员变量是相互独立的。要想共用数据，则需要使用静态成员和静态方法。
- 只要在类中声明静态成员变量，即使不定义对象，也可以为静态成员变量分配空间，进而可以使用静态成员变量。（因为静态成员变量在对象创建之前就已经被分配了内存空间）
- 静态成员变量虽然在类中，但它并不是随对象的建立而分配空间的，也不是随对象的撤销而释放（一般的成员在对象建立时会分配空间，在对象撤销时会释放）。静态成员变量是在程序编译时分配空间，而在程序结束时释放空间。
  **静态成员的定义和声明要加个关键static。静态成员可以通过双冒号来使用，即<类名>::<静态成员名>**。
- 初始化静态成员变量要在类的外面进行。初始化的格式如下：**数据类型 类名::静态成员变量名 = 初值**；
- 不能用参数初始化表，对静态成员变量进行初始化。
- 既可以通过类名来对静态成员变量进行引用，也可以通过对象名来对静态成员变量进行引用。

## 案例一

通过类名调用类的普通成员函数与静态成员函数

```cpp
#include <iostream>
using namespace std;

class CRect{
public:
	void setParam(){}
	static void show(){}
};

int main()
{
	CRect::setParam();
	CRect::show();

	return 0;
}
/*
编译错误：error C2352: ‘CRect::setParam’ : illegal call of non-static member function
结论：不能通过类名来调用类的非静态成员函数
*/
```

## 案例二

通过类的对象调用类的静态成员函数与非静态成员函数

```cpp
#include <iostream>
using namespace std;

class CRect{
public:
	void setParam(){}
	static void show(){}
};

int main()
{
	CRect crt;
	crt.setParam();
	crt.show();

	return 0;
}
/*
编译通过
结论：可以通过类的对象来调用类的静态/非静态成员函数
*/
```

## 案例三

在类的静态成员函数中使用类的非静态成员变量

```cpp
#include <iostream>
using namespace std;

class CRect{
public:
	void setParam(){}
	static void show()
	{
		cout<<m_width<<endl;
	}
private:
	int m_width;
};

int main()
{
	CRect crt;
	crt.show();

	return 0;
}
/*
编译错误：error C2597: illegal reference to data member ‘CRect::m_width’ in a static member function
原因：静态成员函数属于整个类，在类实例化对象之前就已经分配空间了，而类的非静态成员必须在类实例化对象后才有内存空间，所以这个调用就会出错，就好比没有声明一个变量却提前使用它一样。
结论：类的静态成员函数中不能使用类的非静态成员
*/
```

## 案例四

类的非静态成员函数中使用静态成员变量

```cpp
#include <iostream>
using namespace std;

class CRect{
public:
	void setParam()//非静态成员函数中改变了静态成员变量的值
	{
		m_height = 10;
		cout<<m_height<<endl;
	}
	static void show()
	{
		
	}
private:
	int m_width;
	static int m_height;//非静态成员变量
};

int CRect::m_height = 0;	// ***********

int main()
{
	CRect crt;
	crt.setParam();

	return 0;
}
/*
编译通过
结论：类的非静态成员函数中可以使用静态成员变量（显而易见）
*/
```

## 案例五

在类的非静态成员函数中调用静态成员函数

```cpp
#include <iostream>
using namespace std;

class CRect{
public:
	void setParam()
	{
		show();
	}
	static void show()
	{
		cout<<"static member function"<<endl;
	}
private:
	int m_width;
	static int m_height;
};

int main()
{
	CRect crt;
	crt.setParam();

	return 0;
}
/*
编译通过
结论：综合例4、5，可以知道类的非静态成员函数中既可以使用静态成员变量，又可以调用静态成员函数，即类的非静态成员函数可以使用类的静态成员
*/
```

## 案例六

类的静态成员函数调用非静态成员函数

```cpp
#include <iostream>
using namespace std;

class CRect{
public:
	void setParam()
	{
		cout<<"non-static member function"<<endl;
	}
	static void show()
	{
		setParam();
	}
private:
	int m_width;
	static int m_height;
};

int main()
{
	CRect crt;
	crt.show();

	return 0;
}
/*
编译出错：error C2352: ‘CRect::setParam’ : illegal call of non-static member function
原因：没有声明一个函数却提前调用了它
结论：综合例3、6，可以得出类的静态成员函数中既不可以使用非静态成员变量，又不可以调用非静态成员函数，即类的静态成员函数不可以使用非静态成员。
*/
```

## 案例七

类的静态成员变量的使用

```cpp
#include <iostream>
using namespace std;

class CRect{
public:
	void setParam()
	{
		cout<<"non-static="<<m_height<<endl;
	}
	static void show()
	{
		cout<<"static="<<m_height<<endl;
	}
private:
	int m_width;
	static int m_height;
};

int main()
{
	CRect crt;
	crt.setParam();
	crt.show();

	return 0;
}
/*
编译错误：error LNK2001: unresolved external symbol “private: static int CRect::m_height” (?m_height@CRect@@0HA)
原因：类的静态成员变量在使用前没有进行初始化，在类外且在main函数外使用int CRect::m_height = 0;初始化静态成员变量即可
结论：类的静态成员变量必须先初始化再使用。
*/
```

## 案例八

类的静态成员变量的初始化位置

```cpp
class CRect{
public:
	
private:
	static m_height;
};

int CRect::m_height = 0;//A

int main()
{
	int CRect::m_height = 0;//B
	CRect crt;
	int CRect::m_height = 0;//C
	crt.setParam();
	crt.show();

	return 0;
}
/*
在A,B,C三处分别初始化类的静态成员变量时只有A出初始化正确，B和C处初始化编译出错：error C2655: ‘m_height’ : definition or redeclaration illegal in current scope
结论：类的静态成员变量初始化的位置为类外且在main函数前
*/
```

### 参考

[1]: https://www.cnblogs.com/codingmengmeng/p/5906282.html
[2]: http://c.biancheng.net/view/165.html