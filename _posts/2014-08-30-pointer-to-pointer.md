---
layout: post
title: git里面一些不常用的功能
date:  2014-08-30 13:22:33   
category: C++
tags: Pointer
---

以下是经典程序（载自林锐的c\c++高质量编程）

~~~~~~~~~
void GetMemory(char *p,int num)
{
    p=(char*)malloc(sizeof(char)*num);       //p是形参指向的地址
}
void main()
{
    char *str=NULL;
    GetMemory(str,100);//str是实参指向的地址，不能通过调用函数来申请内存
    strcpy(str,"hello");
}
~~~~~~~~~

结果是编译能通过，却不能运行，为什么呢？
先说一下指针作为函数参数的意义：当将指针作为参数时，实参向形参传递的是地址，在函数执行过程中，既可以对该参数指针进行处理，也可以对该参数指针所指向的数据进行处理，（以上程序段来说就是可以对p或*p进行处理）。由于此时形参和实参都是指向同一个存储单元，因此当形参指针所指向的数据改变时，实参指针所指向的数据也作相应的改变，因此这时的形参可以作为输出参数使用。
按照上面的说法，这个程序应该没有问题的啊，实参str和形参p指向同一个存储单元，给形参分配的内存单元应该也给实参分配了才对啊，问题就是在这里
实参和形参是指向同一个地址，它们只是指向相同，但它们自身的地址不是同时申请的，就是说p在申请内存时，str并不可以通过调用Getmemory同时申请内存，所以尽管str调用了GetMemory，但它仍然是个空指针，所以进行strcpy是就不能运行。

要使程序可以运行，只要小小的改动就行了（用指向指针的指针）：

~~~~~~~~~
void GetMemory(char **p,int num)
{
    *p=(char*)malloc(sizeof(char)*num);       //此时*p就变成了是形参本身的地址
}
void main()
{
    char *str=NULL;
    GetMemory(&str,100);  //&str是实参的地址，所以实参和形参之间就可以直接调用
    strcpy(str,"hello");
    free(str);
}
~~~~~~~~~

程序就会如你所愿，输出hello了。

形式参数是实际参数的一个副本，因此当指针作为参数传递给函数的时候，函数只能对指针所指向的内存地址里面的内容进行操作而不能改变指针本身的值。因此如果在函数里面为指针申请空间后，形参的值改变而实参值并未改变。也就是说str的值仍然是NULL，此时如果把字符串拷贝到str指向的地址的话就出错了。

~~~~~~~~~~
#include <iostream>
using namespace std;

void swap1(int *p1, int *p2)
{
	int	temp = *p1;
	*p1 = *p2;
	*p2 = temp;
	
}

void swap2(int &p1, int &p2)
{
	int temp = p1;
	p1 = p2;
	p2 = temp;
}

void swap3(int *p1, int *p2)
{//很显然，这里只是改变了p1和p2的指向，但是并没有改变它们原来指向的内存单元的值。 
	int *temp = p1;//p1作为右值，使得temp指向p1所指向的单元。 
	p1 = p2;//p1指向p2指向的单元。
	/*这里不能写*p1 = p2，因为对于*p1是解引用，其在定义后，*p1就只能作为右值使用。 
	*/ 
	p2 = temp;
	//cout << " *temp = " << *temp << endl;
}

void swap4(int* &p1, int* &p2)
{
	int *temp = p1;
	p1 = p2;
	p2 = temp;
} 

void swap5(int **p1, int **p2)
{
	int *temp = *p1;
	*p1 = *p2;
	*p2 = temp;
}

void swap6(int **p1, int **p2)
{
	int ** temp = p1;
	p1 = p2;
	p2 = temp;
}

int main()
{
	int a = 3;
	int b = 4;
	
	/*int *ppp = NULL;
	*ppp = a;或者用ppp = &a;*/
	 
	int *p1 = &a;
	int *p2 = &b;
	
	int **q1 = &p1;
	int **q2 = &p2;
	
	
	//swap1(&a, &b);
	//swap2(a, b);
	//swap3(p1, p2);//结果：a = 3, b = 4
	//swap4(p1, p2);结果 a = 3, b = 4
	swap5(q1, q2);//改变了*p1 和 *p2所指向的值 
//	swap6(q1, q2);  没有改变*p1 和 *p2 所指向的值 
	cout << "a = " << a << ", " << "b = " << b << ", " << "*p1 = " << *p1 << ", " << "*p2 = " << *p2 << endl; 
	
		
	return 0;
}
~~~~~~~~~~

