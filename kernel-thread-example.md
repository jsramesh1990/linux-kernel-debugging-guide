
# Linux Kernel Thread Driver 

# 1. Introduction

A Kernel Thread is a thread that runs entirely inside:

```text
Kernel Space
````

Kernel threads are created and managed by the Linux kernel for performing:

* Background tasks
* Periodic operations
* Driver processing
* Monitoring
* Deferred work

Unlike user-space threads, kernel threads:

* Have no user memory
* Run with kernel privileges
* Access kernel APIs directly

---

# 2. What is a Kernel Thread?

A Kernel Thread is a schedulable kernel task created using:

```c
kthread_create()
```

or

```c
kthread_run()
```

It behaves similarly to a normal Linux process but executes only kernel code.

Kernel threads are represented internally by:

```c id="w5m8qx"
struct task_struct
```

---

# 3. Why Do We Use Kernel Threads?

Kernel threads are used when drivers need:

* Continuous background processing
* Polling hardware
* Periodic monitoring
* Asynchronous operations
* Deferred execution
* Long-running tasks

---

# 4. Real-Time Examples

| Driver/System   | Kernel Thread Usage    |
| --------------- | ---------------------- |
| Network Driver  | Packet handling        |
| Storage Driver  | Async I/O              |
| Sensor Driver   | Poll sensor data       |
| Watchdog Driver | Monitor hardware       |
| Thermal Driver  | Temperature monitoring |
| Audio Driver    | Stream processing      |

---

# 5. User Thread vs Kernel Thread

| Feature         | User Thread      | Kernel Thread      |
| --------------- | ---------------- | ------------------ |
| Runs In         | User space       | Kernel space       |
| Access Hardware | No               | Yes                |
| Uses libc       | Yes              | No                 |
| Scheduling      | Kernel scheduler | Kernel scheduler   |
| Privileges      | Restricted       | Full kernel access |

---

# 6. Kernel Thread Architecture

```text id="q3m9wy"
+-------------------------------+
| Driver Module                 |
+---------------+---------------+
                |
                v
+-------------------------------+
| kthread_run()                 |
+---------------+---------------+
                |
                v
+-------------------------------+
| Kernel Scheduler              |
+---------------+---------------+
                |
                v
+-------------------------------+
| Kernel Thread                 |
|-------------------------------|
| Infinite Loop                 |
| Background Processing         |
| Sleep/Delay                   |
+-------------------------------+
```

---

# 7. Kernel Thread Flow

```text id="r6m1vx"
Module Loaded
      ↓
Create Kernel Thread
      ↓
Thread Starts Running
      ↓
Background Task Executes
      ↓
Thread Stops During Module Exit
```

---

# 8. Important Kernel Thread APIs

| API                   | Purpose               |
| --------------------- | --------------------- |
| kthread_create()      | Create thread         |
| kthread_run()         | Create + start thread |
| wake_up_process()     | Wake thread           |
| kthread_stop()        | Stop thread           |
| kthread_should_stop() | Check stop request    |
| msleep()              | Sleep thread          |

---

# 9. Important Header Files

```c id="m4w7qy"
#include <linux/module.h>
#include <linux/kernel.h>
#include <linux/init.h>
#include <linux/kthread.h>
#include <linux/delay.h>
```

---

# 10. Basic Kernel Thread Example

## kernel_thread_driver.c

```c id="x8m2vy"
#include <linux/module.h>
#include <linux/kernel.h>
#include <linux/init.h>
#include <linux/kthread.h>
#include <linux/delay.h>

static struct task_struct *thread1;

static int thread_function(void *pv)
{
    while (!kthread_should_stop()) {

        printk(KERN_INFO "Kernel Thread Running\n");

        msleep(2000);
    }

    printk(KERN_INFO "Kernel Thread Stopped\n");

    return 0;
}

static int __init kernel_thread_init(void)
{
    printk(KERN_INFO "Kernel Thread Driver Loaded\n");

    thread1 = kthread_run(thread_function,
                          NULL,
                          "my_kernel_thread");

    if (IS_ERR(thread1)) {
        printk(KERN_ERR "Cannot create thread\n");
        return PTR_ERR(thread1);
    }

    return 0;
}

static void __exit kernel_thread_exit(void)
{
    kthread_stop(thread1);

    printk(KERN_INFO "Kernel Thread Driver Removed\n");
}

module_init(kernel_thread_init);
module_exit(kernel_thread_exit);

MODULE_LICENSE("GPL");
MODULE_AUTHOR("Your Name");
MODULE_DESCRIPTION("Simple Kernel Thread Example");
```

---

# 11. Makefile

```Makefile id="v1m5qx"
obj-m += kernel_thread_driver.o

KDIR := /lib/modules/$(shell uname -r)/build
PWD  := $(shell pwd)

all:
	make -C $(KDIR) M=$(PWD) modules

clean:
	make -C $(KDIR) M=$(PWD) clean
```

---

# 12. Compile Driver

```bash id="n9m4wx"
make
```

Output:

```bash id="p6m8qy"
kernel_thread_driver.ko
```

---

# 13. Insert Driver

```bash id="t2m7vx"
sudo insmod kernel_thread_driver.ko
```

---

# 14. Check Kernel Logs

```bash id="q5m1wy"
dmesg | tail
```

Expected:

```text id="y7m3qx"
Kernel Thread Driver Loaded
Kernel Thread Running
Kernel Thread Running
```

---

# 15. Remove Driver

```bash id="r4m8vx"
sudo rmmod kernel_thread_driver
```

Expected Logs:

```text id="u8m2wy"
Kernel Thread Stopped
Kernel Thread Driver Removed
```

---

# 16. Important APIs Explained

## kthread_run()

Creates and starts thread.

Example:

```c id="w1m9qx"
kthread_run(thread_function,
            data,
            "thread_name");
```

---

## kthread_should_stop()

Checks whether thread should terminate.

Example:

```c id="z5m4vy"
while (!kthread_should_stop())
```

---

## kthread_stop()

Stops kernel thread safely.

Example:

```c id="f7m2wx"
kthread_stop(thread1);
```

---

## msleep()

Suspends thread execution.

Example:

```c id="k3m8qy"
msleep(1000);
```

---

# 17. Thread Function Structure

Typical kernel thread:

```c id="g9m1vx"
static int thread_function(void *data)
{
    while (!kthread_should_stop()) {

        // Background work

        msleep(1000);
    }

    return 0;
}
```

---

# 18. Delayed Background Processing

Kernel threads are useful for:

* Polling hardware
* Sensor monitoring
* Logging
* Cleanup tasks
* Deferred driver operations

---

# 19. Kernel Thread vs Workqueue

| Feature          | Kernel Thread | Workqueue      |
| ---------------- | ------------- | -------------- |
| Dedicated Thread | Yes           | No             |
| Sleep Allowed    | Yes           | Yes            |
| Resource Usage   | Higher        | Lower          |
| Control          | Full control  | Kernel managed |

---

# 20. Kernel Thread vs Tasklet

| Feature          | Kernel Thread | Tasklet      |
| ---------------- | ------------- | ------------ |
| Context          | Process       | Atomic       |
| Sleep Allowed    | Yes           | No           |
| Heavy Processing | Suitable      | Not suitable |

---

# 21. Advantages of Kernel Threads

| Advantage             | Description             |
| --------------------- | ----------------------- |
| Full Control          | Dedicated execution     |
| Sleep Allowed         | Supports blocking APIs  |
| Background Processing | Long operations         |
| Continuous Execution  | Infinite loops possible |
| Flexible Scheduling   | Kernel-managed          |

---

# 22. Disadvantages of Kernel Threads

| Disadvantage           | Description            |
| ---------------------- | ---------------------- |
| More Memory Usage      | Dedicated thread stack |
| Synchronization Needed | Shared resources       |
| Scheduling Overhead    | Context switching      |
| Complex Management     | Manual stop/start      |

---

# 23. Common Interview Questions

## Q1. What is a Kernel Thread?

A thread running completely in kernel space.

---

## Q2. Why Use Kernel Threads?

For long-running background operations inside drivers.

---

## Q3. Difference Between Workqueue and Kernel Thread?

Workqueue uses shared worker threads.

Kernel thread uses dedicated thread.

---

## Q4. Can Kernel Threads Sleep?

Yes.

Kernel threads run in process context.

---

## Q5. Why Use kthread_should_stop()?

To terminate thread safely.

---

# 24. Common Errors

## Error: Thread Never Stops

Cause:

* Missing:

```c id="b2m7qy"
kthread_should_stop()
```

Fix:

* Add stop condition

---

## Error: Kernel Crash During Exit

Cause:

* Thread still running

Fix:

```c id="h6m1vx"
kthread_stop()
```

---

## Error: CPU Usage 100%

Cause:

* Infinite loop without sleep

Fix:

```c id="m4w8qy"
msleep()
```

---

# 25. Kernel Thread Debugging

## Check Kernel Logs

```bash id="v7m3wx"
dmesg | tail
```

---

## View Kernel Threads

```bash id="q9m5vx"
ps -ef | grep kthread
```

---

## Monitor Scheduling

```bash id="t1m8qy"
top
```

---

# 26. Advanced Kernel Thread Topics

After learning basic kernel threads, move to:

* Thread synchronization
* Real-time scheduling
* Multi-threaded drivers
* CPU affinity
* Freezable threads
* RT Linux

---

# 27. Thread Synchronization

Kernel threads often require:

| Mechanism        | Purpose           |
| ---------------- | ----------------- |
| Mutex            | Sleepable locking |
| Spinlock         | Fast locking      |
| Semaphore        | Resource sharing  |
| Atomic variables | Counters          |

---

# 28. Freezable Kernel Threads

Used during:

* Suspend
* Hibernate
* Power management

APIs:

```c id="x5m2wy"
set_freezable()
try_to_freeze()
```

---

# 29. Real-Time Scheduling

Kernel threads can use:

```c id="z8m4qx"
sched_setscheduler()
```

for:

* FIFO scheduling
* Real-time priorities

---

# 30. Best Practices

## Always Stop Threads

Before unloading module:

```c id="g3m7vx"
kthread_stop()
```

---

## Avoid Busy Waiting

Bad:

```c id="k6m1wy"
while(1)
```

Good:

```c id="n9m5qx"
msleep()
```

---

## Protect Shared Resources

Use mutexes/spinlocks.

---

## Keep Thread Logic Simple

Avoid unnecessary complexity.

---

# 31. Real Hardware Platforms

Kernel threads are widely used on:

* Raspberry Pi 5
* BeagleBone Black
* NVIDIA Jetson Nano
* STM32MP157

---

# 32. Real Linux Kernel Thread Examples

| Driver/System | Usage                   |
| ------------- | ----------------------- |
| kswapd        | Memory reclaim          |
| kworker       | Background kernel tasks |
| watchdog      | Hardware monitoring     |
| migration     | CPU balancing           |
| network stack | Packet handling         |

---
