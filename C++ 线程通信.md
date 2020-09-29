# C++ 线程通信

## 一、基本概念

线程之间通信的两个基本问题是互斥和同步，

+ 线程同步是指线程之间所具有的一种制约关系，一个线程的执行依赖另一个线程的消息，当它没有得到另一个线程的消息时应等待，直到消息到达时才被唤醒。

+ 线程互斥是指对于共享的操作系统资源（指的是广义的”资源”，而不是Windows的.res文件，譬如全局变量就是一种共享资源），在各线程访问时的排它性。当有若干个线程都要使用某一共享资源时，任何时刻最多只允许一个线程去使用，其它要使用该资源的线程必须等待，直到占用资源者释放该资源。

线程的同步可分用户模式的线程同步和内核对象的线程同步两大类，

+ 用户模式中线程的同步方法主要有原子访问和临界区等方法。其特点是同步速度特别快，适合于对线程运行速度有严格要求的场合。
+ 内核对象的线程同步则主要由事件、等待定时器、信号量以及信号灯等内核对象构成。由于这种同步机制使用了内核对象，使用时必须将线程从用户模式切换到内核模式，而这种转换一般要耗费近千个CPU周期，因此同步速度较慢，但在适用性上却要远优于用户模式的线程同步方式。

在WIN32中（区别于Linux，其实也差不多），通信机制主要有以下几种：
 （1）事件(Event);
 （2）信号量(semaphore);
 （3）互斥量(mutex);
 （4）临界区(Critical section)。

## 二、Win32中的四种通信方式

### 2.1 临界区

临界区（Critical  Section）是一段独占对某些共享资源访问的代码，在任意时刻只允许一个线程对共享资源进行访问。如果有多个线程试图同时访问临界区，那么在有一个线程进入后其他所有试图访问此临界区的线程将被挂起，并一直持续到进入临界区的线程离开。临界区在被释放后，其他线程可以继续抢占，并以此达到用原子方式操作共享资源的目的。

临界区在使用时以`CRITICAL_SECTION`结构对象保护共享资源，并分别用`EnterCriticalSection()`和`LeaveCriticalSection()`函数去标识和释放一个临界区。所用到的`CRITICAL_SECTION`结构对象必须经过`InitializeCriticalSection()`的初始化后才能使用，而且必须确保所有线程中的任何试图访问此共享资源的代码都处在此临界区的保护之下。否则临界区将不会起到应有的作用，共享资源依然有被破坏的可能。

```c++
#include "stdafx.h"
#include<windows.h>
#include<iostream>
using namespace std;

int number = 1;	//定义全局变量
CRITICAL_SECTION Critical;		//定义临界区句柄

unsigned long __stdcall ThreadProc1(void* lp) {
	while (number < 100) {
		EnterCriticalSection(&Critical);
		cout << "thread 1 :"<<number << endl;
		++number;
		_sleep(100);
		LeaveCriticalSection(&Critical);
	}

	return 0;
}

unsigned long __stdcall ThreadProc2(void* lp) {
	while (number < 100) {
		EnterCriticalSection(&Critical);
		cout << "thread 2 :"<<number << endl;
		++number;
		_sleep(100);
		LeaveCriticalSection(&Critical);
	}

	return 0;
}

int main() {
	InitializeCriticalSection(&Critical);	//初始化临界区对象
	CreateThread(NULL, 0, ThreadProc1, NULL, 0, NULL);
	CreateThread(NULL, 0, ThreadProc2, NULL, 0, NULL);

	Sleep(10*1000);
	system("pause");
    return 0;
}
```

### 2.2 事件

### 2.3 信号量

信号量是维护0到指定最大值之间的同步对象。信号量状态在其计数大于0时是有信号的，而其计数是0时是无信号的。信号量对象在控制上可以支持有限数量共享资源的访问。

信号量的特点和用途可用下列几句话定义：
 （1）如果当前资源的数量大于0，则信号量有效；
 （2）如果当前资源数量是0，则信号量无效；
 （3）系统决不允许当前资源的数量为负值；
 （4）当前资源数量决不能大于最大资源数量。

**创建信号量**

函数原型为：

```c++
 HANDLE CreateSemaphore (
  　PSECURITY_ATTRIBUTE psa, //信号量的安全属性
　  LONG lInitialCount, //开始时可供使用的资源数
　  LONG lMaximumCount, //最大资源数
   PCTSTR pszName);     //信号量的名称
```

**释放信号量**

通过调用`ReleaseSemaphore`函数，线程就能够对信标的当前资源数量进行递增，该函数原型为：

```c++
BOOL WINAPI ReleaseSemaphore(
  　HANDLE hSemaphore,   //要增加的信号量句柄
  　LONG lReleaseCount, //信号量的当前资源数增加lReleaseCount
  　LPLONG lpPreviousCount  //增加前的数值返回
   );
```

**打开信号量**

和其他核心对象一样，信号量也可以通过名字跨进程访问，打开信号量的API为：

```c++
 HANDLE OpenSemaphore (
  　DWORD fdwAccess,      //access
  　BOOL bInherithandle,  //如果允许子进程继承句柄，则设为TRUE
  　PCTSTR pszName  //指定要打开的对象的名字
  );
```

代码示例：

```c++
#include "stdafx.h"
#include<windows.h>
#include<iostream>
using namespace std;

int number = 1;	//定义全局变量
HANDLE hSemaphore;	//定义信号量句柄

unsigned long __stdcall ThreadProc1(void* lp) {
	long count;
	while (number < 100) {
		WaitForSingleObject(hSemaphore, INFINITE);	//等待信号量为有信号状态
		cout << "thread 1 :"<<number << endl;
		++number;
		_sleep(100);
		ReleaseSemaphore(hSemaphore, 1, &count);
	}

	return 0;
}

unsigned long __stdcall ThreadProc2(void* lp) {
	long count;
	while (number < 100) {
		WaitForSingleObject(hSemaphore, INFINITE);	//等待信号量为有信号状态
		cout << "thread 2 :"<<number << endl;
		++number;
		_sleep(100);
		ReleaseSemaphore(hSemaphore, 1, &count);
	}

	return 0;
}

int main() {
	hSemaphore = CreateSemaphore(NULL, 1, 100, "sema");
	CreateThread(NULL, 0, ThreadProc1, NULL, 0, NULL);
	CreateThread(NULL, 0, ThreadProc2, NULL, 0, NULL);

	Sleep(10*1000);
	system("pause");
    return 0;
}
```

### 2.4 互斥量

采用互斥对象机制。 只有拥有互斥对象的线程才有访问公共资源的权限，因为互斥对象只有一个，所以能保证公共资源不会同时被多个线程访问。互斥不仅能实现同一应用程序的公共资源安全共享，还能实现不同应用程序的公共资源安全共享。

示例代码：

```c++
#include "stdafx.h"
#include<windows.h>
#include<iostream>
using namespace std;

int number = 1;	//定义全局变量
HANDLE hMutex;	//定义互斥对象句柄

unsigned long __stdcall ThreadProc1(void* lp) {
	while (number < 100) {
		WaitForSingleObject(hMutex, INFINITE);
		cout << "thread 1 :"<<number << endl;
		++number;
		_sleep(100);
		ReleaseMutex(hMutex);
	}

	return 0;
}

unsigned long __stdcall ThreadProc2(void* lp) {
	while (number < 100) {
		WaitForSingleObject(hMutex, INFINITE);
		cout << "thread 2 :"<<number << endl;
		++number;
		_sleep(100);
		ReleaseMutex(hMutex);
	}

	return 0;
}

int main() {
	hMutex = CreateMutex(NULL, false, "mutex");		//创建互斥对象
	CreateThread(NULL, 0, ThreadProc1, NULL, 0, NULL);
	CreateThread(NULL, 0, ThreadProc2, NULL, 0, NULL);

	Sleep(10*1000);
	system("pause");
    return 0;
}
```





条件变量，借由它，一个线程可以唤醒一个或多个其他等待中的线程。

- 你必须同时包含< mutex >和< condition_variable >，并声明一个mutex和一个condition_variable变量；
- 那个通知“条件已满足”的线程（或多个线程之一）必须调用notify_one()或notify_all()，以便条件满足时唤醒处于等待中的一个条件变量；
- 那个等待"条件被满足"的线程必须调用wait()，可以让线程在条件未被满足时陷入休眠状态，当接收到通知时被唤醒去处理相应的任务；

一个疑问：若发出“条件已满足”的通知时，等待线程尚未被调用，会怎样？？？？？