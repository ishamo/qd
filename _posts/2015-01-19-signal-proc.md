---
layout: post
title: 信号处理程序中的非局部转移的方法
date:  2015-01-19 12:09:51   
category: Unix
tags: signal
---

前面有篇日志提到用longjmp来进行非局部跳转，那是不是说它也可以用来进信号处理过程中的非局部转移呢？

当然可以，但是却不是一种稳妥的做法！

我们来看看《APUE》上面是怎么说的：
> 在freeBSD8.0和MACOS X 10.6.8中， setjmp和longjmp保存和恢复信号屏蔽字。但是，Linux3.2.0和Solaris10并不执行这种操作，虽然Linux支持提供BSD行为的选项。FreeBSD8.0和MAC OS X10.6.8提供函数_setjmp和_longjmp,它们也不保存和恢复信号屏蔽字。

简单的说，就是各种系统在进行信号处理跳转的时候，并没有规定此期间对再次到来的同样的信号应该执行什么样的行为，有些系统会维护一个阻塞的队列，这样就可以处理完发生的信号之后再接着处理阻塞队列中的信号，但是也有一些系统是直接不进行处理。所以为了兼顾这两种行为，我们的POSIX.1并没有指定setjmp和longjmp对信号屏蔽字的作用，而是定义了两个新函数sigsetjmp和siglongjmp，所以我们在信号处理程序中进行非局部跳转的时候就应当使用这两个函数！

~~~
#include <setjmp.h>
int sigsetjmp(sigjmp_buf env, int savemask);
							返回值：若直接调用，返回0；若从siglongjmp调用返回，则返回非0
void siglongjmp(sigjmp_buf env, int val);
~~~

给出一个示例：
~~~~
#include "apue.h"
#include <setjmp.h>
#include <time.h>

static void sig_usr1(int);
static void sig_alrm(int);
static sigjmp_buf jmpbuf;
static volatile sig_atomic_t canjump;

int main(void)
{
	if (signal (SIGUSR1, sig_usr1) == SIG_ERR)
		err_sys("signal(SIGUSR1) error");

	pr_mask("starting main: ");

	if (sigsetjmp(jmpbuf, 1)) {
		pr_mask("ending main: ");
		exit(0);
	}

	canjump = 1;
	for (; ; )
		pause();
}

static void sig_usr1(int signo)
{
	time_t starttime;

	if (canjump == 0)
		return;

	pr_mask("starting sig_usr1: ");
	alarm(3);
	starttime = time(NULL);
	for (; ; )
		if (time(NULL) > starttime + 5)
			break;

	pr_mask("finishing sig_usr1: ");
	canjump = 0;
	siglongjmp(jmpbuf, 1);
}

static void sig_alrm(int signo)
{
	pr_mask("in sig_alrm: ");
}
~~~~


~~~~~
#include "apue.h"
#include <errno.h>

void pr_mask(const char *str)
{
	sigset_t sigset;
	int errno_save;

	errno_save = errno;
	if (sigprocmask(0, NULL, &sigset) < 0){
		err_ret("sigprocmask error");
	}else {
		printf("%s", str);
		if (sigismember(&sigset, SIGINT))
			printf(" SIGINT");
		if (sigismember(&sigset, SIGQUIT))
			printf(" SIGQUIT");
		if (sigismember(&sigset, SIGUSR1))
			printf("SIGUSR1");
		if (sigismember(&sigset, SIGALRM))
			printf("SIGALRM");

		printf("\n");
	}

	errno = errno_save;
}
~~~~~
















