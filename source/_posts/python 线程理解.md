---
title: python线程理解
date: 2022-06-03 11:41:18
tags:
- python
categories:
- python核心知识点
---
## 简介
一个应用程序由多个进程组成，一个进程有多个线程，一个线程则是操作系统调度的最小单位，当应用程序运行时，操作系统根据优先级和时间片调度线程（决定此时此刻执行哪个线程）。
## python的线程
python存在多个解释器，cpython、jpython等，但目前主流常用的则是cpython，其存在GIL锁，从而导致无论你启多少个线程，你有多少个cpu, Python在执行的时候会淡定的在同一时刻只允许一个线程运行，因此适用于io密集型程序，由于读写io需要时间且python线程只能在一个cpu运行，可以在io的时间中来回切换线程，进一步提高效率
## python为什么要加锁
进行定期自动内存回收
	加入GIL主要的原因是为了降低程序的开发的复杂度，比如现在的你写python不需要关心内存回收的问题，因为Python解释器帮你自动定期进行内存回收，你可以理解为python解释器里有一个独立的线程，每过一段时间它起wake up做一次全局轮询看看哪些内存数据是可以被清空的，此时你自己的程序 里的线程和 py解释器自己的线程是并发运行的，假设你的线程删除了一个变量，py解释器的垃圾回收线程在清空这个变量的过程中的clearing时刻，可能一个其它线程正好又重新给这个还没来及得清空的内存空间赋值了，结果就有可能新赋值的数据被删除了，为了解决类似的问题，python解释器简单粗暴的加了锁，即当一个线程运行时，其它人都不能动，这样就解决了上述的问题，  这可以说是Python早期版本的遗留问题。
## python线程的常用创建方式
### 1.类方式创建，需要继承threading.Thread类，必须要在构造函数中初始化thread的init方法，线程方法需要重写run方法
```
import threading, time, os

class MyThread(threading.Thread):

    def __init__(self, name):
        threading.Thread.__init__(self, name=name)
        self.name = name

    def run(self):
        time.sleep(10)
        print(f"MyThread run:{self.name}")

print(f'pid:{os.getpid()}')

t = MyThread(name='test')
t2 = MyThread(name='test2')
t.run()
t2.run()
```
### 2.函数式创建线程，需要注意的是在函数传参时如果只有一个值，需要在后面添加逗号
```
import threading, time

def run(name):
    time.sleep(10)
    print(f"Thread run:{name}")

t1 = threading.Thread(target=run, args=('test1',))
t2 = threading.Thread(target=run, args=('test2',))
t1.start()
t2.start()
```
## python 线程间通信
### 1.全局变量，一个线程写入，一个线程读取
```
from typing import List

MSG_LIST:List = []

import threading

def send(msg):
    global MSG_LIST
    MSG_LIST.append(msg)

def get():
    global MSG_LIST
    for msg in MSG_LIST:
        print(f'recvmsg:{msg}')
        MSG_LIST.remove(msg)

t_list = []
for i in range(5):
    t = threading.Thread(target=send, args=('msg'+str(i),))
    t2 = threading.Thread(target=get)
    t.start()
    t2.start()
    t.join()
    t2.join()
```
![image](https://img2022.cnblogs.com/blog/2850640/202205/2850640-20220526140934113-715783870.png)
### 2.队列
与全局变量类似，不过方便了编写
```
import threading, queue

q = queue.Queue()

def send(msg):
    global q
    q.put(msg)

def get():
    global q
    print(f'msg:{q.get_nowait()}')

t_list = []
for i in range(5):
    t = threading.Thread(target=send, args=('msg'+str(i),))
    t2 = threading.Thread(target=get)
    t.start()
    t2.start()
    t.join()
    t2.join()
```
![image](https://img2022.cnblogs.com/blog/2850640/202205/2850640-20220526141248719-1576161512.png)
## python线程数据不一致问题
在多线程操作数据时，会导致脏数据、数据不一致的情况，如下所示：
```
count = 0

import threading, time

def run(name):
    global count
    time.sleep(5)
    count += 1
    print(f'name:{name}, count:{count}')

t_list = []
for i in range(5):
    t:threading.Thread = threading.Thread(target=run, args=('thread' + str(i+1), ))
    t_list.append(t)
    t.start()

for t in t_list:
    t.join()
```
![image](https://img2022.cnblogs.com/blog/2850640/202205/2850640-20220526142957080-213390805.png)
为了解决上述问题，需要采取加锁的方式，方式如下：
### 1.互斥锁(Lock)
使用threading中的Lock模块，只需要将函数或方法中的操作逻辑加上对应的锁即可，此方法可以解决一般的数据不一致问题，但是如果锁发生嵌套，则可能会出现死锁问题
```
count = 0

import threading, time

lock = threading.Lock()
def run(name):
    global count
    lock.acquire()
    time.sleep(5)
    count += 1
    print(f'name:{name}, count:{count}')
    lock.release()
    
t_list = []
for i in range(5):
    t:threading.Thread = threading.Thread(target=run, args=('thread' + str(i+1), ))
    t_list.append(t)
    t.start()

for t in t_list:
    t.join()
```
![image](https://img2022.cnblogs.com/blog/2850640/202205/2850640-20220526143334979-1007813961.png)
### 2.如上Lock所写，Lock嵌套可能会出现死锁的问题，如下由于Lock使用了嵌套，则会导致线程发生阻塞，导致死锁问题
```
count = 0

import threading, time

lock = threading.Lock()
def run(name):
    global count
    lock.acquire()
    time.sleep(5)
    count += 1
    lock.acquire()
    count -= 1
    lock.release()
    print(f'name:{name}, count:{count}')
    lock.release()
    
t_list = []
for i in range(5):
    t:threading.Thread = threading.Thread(target=run, args=('thread' + str(i+1), ))
    t_list.append(t)
    t.start()

for t in t_list:
    t.join()
```
因此需要使用嵌套锁RLock
```
count = 0

import threading, time

lock = threading.RLock()
def run(name):
    global count
    lock.acquire()
    time.sleep(5)
    count += 1
    lock.acquire()
    count += 1
    lock.release()
    print(f'name:{name}, count:{count}')
    lock.release()
    
t_list = []
for i in range(5):
    t:threading.Thread = threading.Thread(target=run, args=('thread' + str(i+1), ))
    t_list.append(t)
    t.start()

for t in t_list:
    t.join()
```
![image](https://img2022.cnblogs.com/blog/2850640/202205/2850640-20220526144603174-1498665663.png)
### 3.信号量(Semaphore)
互斥锁 同时只允许一个线程更改数据，而Semaphore是同时允许一定数量的线程更改数据
```
count = 0

import threading, time

lock = threading.Semaphore(4)
def run(name):
    global count
    lock.acquire()
    time.sleep(5)
    count += 1
    print(f'name:{name}, count:{count}')
    lock.release()
    
t_list = []
for i in range(5):
    t:threading.Thread = threading.Thread(target=run, args=('thread' + str(i+1), ))
    t_list.append(t)
    t.start()

for t in t_list:
    t.join()
```
结果虽然都是一样的，但是在实际运行过程中，则会4个线程‘同时’运行，此处同时起始只是人眼可见的，实际上还是依次运行的，不过运行较快
![image](https://img2022.cnblogs.com/blog/2850640/202205/2850640-20220526145003936-1343146497.png)
Semaphore原理：相当于早晚班倒
	内部存在一个计数器，acquire时-1 release时+1， 当计数器为0时则阻塞， 当有线程释放时则会继续执行相关信息，不需要全部释放后才可以继续执行
	计数器的数量取决于设置的n的数量， 默认是1
### 4.条件(Condition)
```
import threading, time

count = 0
con = threading.Condition()
def producer():
    global count
    con.acquire()
    time.sleep(5)
    count += 1
    con.notify()
    con.release()

def customer():
    global count
    con.acquire()
    con.wait(timeout=10)
    print(f'count:{count}')
    con.release()

c = threading.Thread(target=customer)
c.start()
p = threading.Thread(target=producer)
p.start()

```
Condition原理: 类似与生产消费者模式
    python 条件变量Condition也需要关联互斥锁，同时Condition自身提供了wait/notify/notifyAll方法，用于阻塞/通知其他并行线程，可以访问共享资源了。
    Condition提供了一种多线程通信机制，假如线程1需要数据，那么线程1就阻塞等待，这时线程2就去制造数据，线程2制造好数据后，通知线程1可以去取数据了，然后线程1去获取数据。
    内部使用互斥锁，双层锁结构
### 5.event(事件):适用于两个线程间通信
```
import threading, time

count = 0

event = threading.Event()

def run1():
    global count
    count += 1
    print(f'event:flag:before:{event.isSet()}')
    time.sleep(10)
    event.set()
    print(f'event:flag:after:{event.isSet()}')

def run2():
    global count
    event.wait()
    print(f'count:{count}')

t1 = threading.Thread(target=run2)
t2 = threading.Thread(target=run1)
t1.start()
t2.start()
```
event原理:
    python线程的事件用于主线程控制其他线程的执行，事件主要提供了三个方法 set、wait、clear。
    默认 event flag为false
    事件处理的机制：全局定义了一个“Flag”，如果“Flag”值为 False，那么当程序执行 event.wait 方法时就会阻塞，如果“Flag”值为True，那么event.wait 方法时便不再阻塞。
        clear：将“Flag”设置为False
        set：将“Flag”设置为True
## 线程池
### 使用submit创建单个线程
```
from concurrent.futures import ThreadPoolExecutor
import time

def run(n, m):
    print(f'{n} thread is starting')
    time.sleep(3)
    print(f'{n} thread is finish')
    return n, m

t = ThreadPoolExecutor(4)
ret_list = []
for i in range(5):
    # 创建单个线程
    ret = t.submit(run, n=i, m=i+1)
    ret_list.append(ret)

for ret in ret_list:
    print(ret.result())
```
### 使用map批量创建线程
需要使用args中间传递值，结果会在最后返回
```
from concurrent.futures import ThreadPoolExecutor
import time

def run(n, m):
    print(f'{n} thread is starting')
    time.sleep(3)
    print(f'{n} thread is finish')
    return n, m

def run_map(args):
    print(args)
    print(*args)
    return run(*args)

t = ThreadPoolExecutor(4)
ret = t.map(run_map, [(1,2)])
print(list(ret))
```
## 死锁问题
1.什么是死锁？
死锁是由于两个或以上的线程互相持有对方需要的资源，且都不释放占有的资源，导致这些线程处于等待状态，程序无法执行。
2.产生死锁的四个必要条件
1.互斥性：线程对资源的占有是排他性的，一个资源只能被一个线程占有，直到释放。
2.请求和保持条件：一个线程对请求被占有资源发生阻塞时，对已经获得的资源不释放。
3.不剥夺：一个线程在释放资源之前，其他的线程无法剥夺占用。
4.循环等待：发生死锁时，线程进入死循环，永久阻塞。
3.产生死锁的原因
在多线程的场景，比如线程A持有独占锁资源a，并尝试去获取独占锁资源b同时，线程B持有独占锁资源b，并尝试去获取独占锁资源a。
这样线程A和线程B相互持有对方需要的锁，从而发生阻塞，最终变为死锁。
造成死锁的原因可以概括成三句话：
1.不同线程同时占用自己锁资源
2.这些线程都需要对方的锁资源
3.这些线程都不放弃自己拥有的资源
## 注意事项
1.建议创建线程时对每一个线程起一个名称，方便后期出现问题后定位问题