@[toc]
## 简介

使用内核线程需要包含头文件`#include <linux/kthread.h>`，下面整理了一下常用的api接口，如下表格所示；

| 函数                                                         | 功能           |
| ------------------------------------------------------------ | -------------- |
| `struct task_struct * kthread_create(threadfn, data, namefmt, arg...) ` | 创建一个线程   |
| `struct task_struct * kthread_run(threadfn, data, namefmt, ...)` | 创建线程并运行 |
| `int kthread_stop(struct task_struct *k)`                    | 停止线程       |



## 例程

下面的代码简单实现了创建一个线程，循环60秒，每秒打印`count`的数值和传入线程执行函数的参数`our_thread`，所以预期结果是该模块会打印`1 thread_func:thread1`字符串的数据。

```c
#include <linux/module.h>
#include <linux/init.h>
#include <linux/module.h>
#include <linux/time.h>

#include <linux/sched.h>
#include <linux/kthread.h>
#include <linux/delay.h>

static struct task_struct *thread_body;
static char  our_thread[8]="thread1";

static int thread_func(void *buff) {

	static int count = 0;
	char *data = (char*)buff;
	unsigned long j0, j1;
	int delay = 60*HZ;

	printk(KERN_INFO "In thread 1");
	
	j0 = jiffies;
	j1 = j0 + delay;
	
	while (time_before(jiffies, j1)){
		schedule();
		msleep(1000);
		printk("%d thread_func:%s \n",++count,data);
	}

	return 0;
}

static int __init demo_thread_init(void){
    
    printk(KERN_INFO "in demo_thread_init\n");
	
    //这里可以也使用 kthread_run ，kthread_run中已经包含了wake_up_process操作
    thread_body = kthread_create(thread_func, (char*)our_thread, "thread1");
	
    if((thread_body))
    {
        wake_up_process(thread_body);
    }

	return 0;
}

module_init(demo_thread_init);

static void __exit demo_thread_exit(void){
	int ret;
	ret = kthread_stop(thread_body);
	if(!ret){
		printk(KERN_INFO "Thread stopped\n"); 
	}
}
module_exit(demo_thread_exit);

MODULE_LICENSE("GPL");	
```



## 运行结果

```c
[    4.496344] 1 thread_func:thread1
[    5.499766] 2 thread_func:thread1
...
[   57.673065] 54 thread_func:thread1
[   58.676418] 55 thread_func:thread1
[   59.679734] 56 thread_func:thread1
[   60.683070] 57 thread_func:thread1
...
```



## 参考

https://www.programering.com/a/MDN4IjMwATk.html

http://tuxthink.blogspot.com/2011/02/kernel-thread-creation-1.html
