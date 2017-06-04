# C语言异常处理、try/catch简单实现


try/catch是一种错误处理机制，c++和object-c里面都有，c里面没有，这里简单在c里面实现一下：

要在c里面实现try/catch，先了解C标准库提供的两个特殊的函数：setjmp() 及 longjmp()，以及jmp_buf 异常结构

1.setjmp()和longjmp()
函数原型：

```
 int _Cdecl setjmp(jmp_buf jmpb);
 void _Cdecl longjmp(jmp_buf jmpb, int retval);
```
 
 其中_Cdecl是指调用约定，jum_buf是一个全局的异常结构类型，setjmp和longjmp是一对函数，setjmp函数的意义为设置跳转点，函数有这样一个特性：第一次调用返回0，用作初始化，第二次返回值是longjmp的入参retval；longjmp功能是跳转回来再执行一次setjmp，此时调用结构必不为0（貌似就算是0也会转换成1）
 
2.jmp_buf 异常结构

```
   typedef struct
   {
   unsigned j_sp;  // 堆栈指针寄存器
   unsigned j_ss;  // 堆栈段
   unsigned j_flag;  // 标志寄存器
   unsigned j_cs;  // 代码段
   unsigned j_ip;  // 指令指针寄存器
   unsigned j_bp; // 基址指针
   unsigned j_di;  // 目的指针
   unsigned j_es; // 附加段
   unsigned j_si;  // 源变址
   unsigned j_ds; // 数据段
   } jmp_buf;
```
jmp_buf 结构存放了程序当前寄存器的值，以确保使用 longjmp() 后可以跳回到该执行点上继续执行。

通过一段代码，可以理解2个函数的应用：

```
#include <stdio.h>
#include <stdlib.h>
#include <setjmp.h>
#include <string.h>
jmp_buf jmpbuffer;
 void f1(void)
{
    longjmp(jmpbuffer,11);
}

void f2(void)
{
    longjmp(jmpbuffer,22);
}
int main(void)
{
    int i = 0;
    i = setjmp(jmpbuffer);
    if(i==0)
    {
        printf("step:0\n");
        f1();
        printf("step:1\n");
        f2();
        printf("step:2\n");
    }
   else
   {
     switch(i)
     {
       
     case 11:
       	printf("In fun1\n");
     break;
  	 case 22:
    	 printf("In fun2\n");
     break;
  	 default:
     	printf("unkown error\n");
     break;
     }
	 exit(0);
   }
   return 1;
}
```

f5结果如下：
step:0
In fun1
请按任意键继续. . .
简单分析一下运行：
1，第一次运行到setjmp时，函数返回0，i=0，程序当前的环境保存给jmpbuffer
2，执行f1，调用longjmp，入参为11,此时跳转回setjmp
3，再次执行setjmp，函数返回11，i=11,跳到else里面，case 11，运行结束



接下来进入正题,c实现try/catch:

```
#include"stdio.h"  
#include"conio.h"  
#include"setjmp.h"  
jmp_buf Jump_Buffer;  
#define mTry if(!(err=setjmp(Jump_Buffer)))  
#define mCatch else  
#define mThrow(y) longjmp(Jump_Buffer,y);
int err=0;

double Test(double x,double y)  
{  
    if(y==0 || y>=10000)
	{
		mThrow(y);	
	} 
    else 
    	printf("result=%f\n",x/y);  
    return 0;  
}  
  
int main()  
{  
	double yy=0;
	printf("%d",(int)yy);
    double x,y;
	while (1){
    mTry{  
          puts("Input the 2 value:");  
          scanf("%lf%lf",&x,&y);  
          Test(x,y);  
      } 
	mCatch{
		printf("---------------------------input error,err = %d,y = %lf\n",err,y);  
      } 
	}
    return 0;
}
```


Test为要运行的函数，返回x/y，当分母=0或分母太大时抛出异常，err保存异常信息
f5结果如下：

```
0Input the 2 value:
1 2
result=0.500000
Input the 2 value:
2 0
---------------------------input error,err = 1,y = 0.000000
Input the 2 value:
1 444
result=0.002252
Input the 2 value:
1 1000001
---------------------------input error,err = 1000001,y = 1000001.000000
Input the 2 value:
2222222 1
result=2222222.000000
Input the 2 value:
```

这里可以看到一个细节：我们的err返回的是y的值，当y=0时，讲道理err应该=0，但是由于setjmp第二次运行时不能等于0，所以将值0转为1了