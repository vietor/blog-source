---
title: "用打印程序崩溃时“调用堆栈”的GCC代码"
date: 2016-04-08 13:16:28 +0800
categories: [program]
tags: [gcc]
---

## 主要代码

``` c
#include <stdio.h>
#include <time.h>
#include <string.h>
#include <signal.h>
#include <unistd.h>
#include <execinfo.h>

static void signal_dump(int sig)
{
  FILE* fp;
	time_t curTime;
	struct tm* pTm, xTm;
	void* array[20];
	char **strings, filename[1024];
	size_t i, size;
	struct sigaction act, oact;

	time(&curTime);
	pTm = localtime_r(&curTime, &xTm);
	sprintf(filename, "%04d-%02d-%02d-stackdump.log", pTm->tm_year + 1900, pTm->tm_mon + 1, pTm->tm_mday);
	fp = fopen(filename, "a");
	if(fp) {
		fprintf(fp, "%02d:%02d:%02d PID[%d] Exception: %s\nStack trace:\n", pTm->tm_hour, pTm->tm_min, pTm->tm_sec, (int)getpid(), strsignal(sig));
		size = backtrace(array, sizeof(array) / sizeof(array[0]));
		strings = backtrace_symbols(array, size);
		for(i = 0; i < size; i++) {
			fprintf(fp, "%s\n", strings[i]);
		}
		fclose(fp);
	}

	memset(&act, 0, sizeof(act));
	act.sa_handler = SIG_DFL;
	act.sa_flags = 0;
	sigaction(sig, &act, &oact);
	raise(sig);
}

void register_signal_dump()
{
	size_t i;
	struct sigaction act, oact;
	static int signals[] = {SIGSEGV, SIGABRT, SIGBUS, SIGFPE, SIGSTKFLT};
	static int ignore_signals[] = {SIGPIPE};

	for(i = 0; i < sizeof(signals) / sizeof(signals[0]); i++) {
		memset(&act, 0, sizeof(act));
		act.sa_handler = signal_dump;
		act.sa_flags = 0;
		sigaction(signals[i], &act, &oact);
	}
	for(i = 0; i < sizeof(ignore_signals) / sizeof(ignore_signals[0]); i++) {
		memset(&act, 0, sizeof(act));
		act.sa_handler = SIG_IGN;
		act.sa_flags = 0;
		sigaction(ignore_signals[i], &act, &oact);
	}
}
```

## 使用说明
1）一般程序崩溃时会收到SIGSEGV等信号；开始时通过调用**register_signal_dump**完成信号处理注册。  
2）通过addr2line命令可以进一步获得更多详细信息。

``` bash
cat 2013-09-12-stackdump.log| grep "\[0x"| awk -F[ '{print $2}'| awk -F] '{print $1}' | xargs addr2line $1 -e <程序文件>
```
