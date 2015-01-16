---
layout: post
title: 使用longjmp来避免慢速系统调用被中断
date:  2015-01-16 20:51:42   
category: Unix
tags: longjmp
---

《APUE》告诉我们在处理信号的时候一定要非常小心，特别是要考虑会否存在竞争条件。如果要对I/O操作设置时间限制，那么可以使用longjmp来让系统如预期的那样工作，不管系统是否重新启用被中断的系统调用。当然还有一种选择是使用select或poll函数。

	#include "apue.h"
	#include <setjmp.h>

	static void sig_alrm(int);
	static jmp_buf env_alrm;

	int main(void)
	{
		int n;
		char line[MAXLINE];

		if (signal(SIGALRM, sig_alrm) == SIG_ERR)
			err_sys("signal(SIGALRM) error");
		if (setjmp(env_alrm) != 0)
			err_quit("read timeout");

		alarm(10);
		if ((n = read(STDIN_FILENO, line, MAXLINE)) < 0)
			err_sys("read error");
		alarm(0);

		write(STDOUT_FILENO, line, n);
		exit(0);
	}

	static void sig_alrm(int signo)
	{
		longjmp(env_alrm, 1);
	}
