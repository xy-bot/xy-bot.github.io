---
title: python进程理解
date: 2022-06-03 11:42:06
tags:
- python
categories:
- python核心知识点
---
## 简介
线程理解中介绍过，再回顾一遍，一个应用程序由多个进程组成，一个进程由多个线程组成，由操作系统根据优先级、时间片来绝对线程的运行
## 进程
python的进程不同于线程，在主流的cpython解释器下，无论创建多少线程，都只会在一个cpu上运行，与java等语言有所区别，进程则与其他语言类似，会占用对应的cpu资源，因此进程相对于线程来说开销会大一点，进程适用于计算密集型程序
## 常见的进程创建方式
### 1.multiprocessing的Process创建进程
```
from multiprocessing import Process
import time

def run(name):
    time.sleep(10)
    print(f'process statrt:{name}')

p = Process(target=run, args=('test1',), name='test1')
p.start()
```
### 2.subprocess创建进程
subprocess创建进程一般用于命令的执行，详细信息可见subprocess文章
stderr=subprocess.STDOUT:代表将错误输出也输出到stdout
```
import subprocess

command = "adb devices"
p = subprocess.Popen(command, shell=True, stdout=subprocess.PIPE, stderr=subprocess.STDOUT)
print(p.stdout.readlines())
```
## 进程间通信
由于进程的内存空间是相对独立的，因此无法像线程使用全局变量的方式去进行进程间通信，需要借助一定的中间媒介，下面介绍几种进程间通信的方式。
### 1.Queue(队列)
此处不是queue中的队列，是mutiprocessing中的队列
```
from multiprocessing import Process, Queue

queue = Queue()
def put(msg):
    queue.put(msg)


def get():
    msg = queue.get()
    print(f"msg:{msg}")


p = Process(target=put, args=("message",))
p.start()
p2 = Process(target=get)
p2.start()
```

### 2.Pipe
适用于两个进程间通信，recv、send,  与socket.socketpair()类似
管道：适用于两个线程之间的数据交互，感觉类似于socket通信
```
from multiprocessing import Process, Pipe

send_p, recv_p = Pipe()
def send(msg):
    send_p.send(msg)

def recv():
    msg = recv_p.recv()
    print(f"msg:{msg}")

p = Process(target=send, args=('test',))
p.start()
p2 = Process(target=recv)
p2.start()
```
### 3.Manager
推荐使用这种方式，支持 list, dict, Namespace, Lock, RLock, Semaphore, BoundedSemaphore, Condition, Event, Barrier, Queue, Value and Array.
```
from multiprocessing import Process, Manager, Lock

manager = Manager()
# 字典类型
d = manager.dict({'count':0})
# 列表类型
l = manager.list([])
# 加锁，防止数据混乱
lock = Lock()
def start_process1(d, l:list):
    lock.acquire()
    d['count'] += 1
    l.append(d['count'])
    lock.release()
    print(d['count'])
    print(l)

p_list = []

for i in range(10):
    p = Process(target=start_process1, args=(d, l))
    p.start()
    p_list.append(p)

for p in p_list:
    p.join()
```
### 4.socket通信
支持跨进程、跨语言通信
例如：socket.socketpair()
```
from multiprocessing import Process
import socket

socket1, socket2 = socket.socketpair()
def send(msg):
    print(socket1)
    print(socket2)
    # 关闭不需要的socket
    socket2.close()
    socket1.send(msg.encode('utf8'))

def recv():
    print(socket1)
    socket1.close()
    print(socket2)
    print(socket2.recv(1024))

p = Process(target=send, args=('test',))
p.start()
p2 = Process(target=recv)
p2.start()
```
### 5.中间介质：文件、事件中心等
此处不再赘述
## 进程池
### 1.使用multiprocessing中的Pool模块
```
from multiprocessing import Pool
import time

def start_process(n):
    print(f'{n} process start')
    time.sleep(10)
    print(f'{n} process finish')
    return n

def callback(n):
    """
    回调参数为进程的return的返回值
    """
    print(f"callback:n:{n}")

def error_callback():
    print(f"error_callback")

pool = Pool(4)
for i in range(5):
    # 异步执行
    pool.apply_async(func=start_process, args=(str(i+1),),callback=callback)
pool.close()
pool.join()
for i in range(5):
    # 同步执行
    pool.apply(func=start_process, args=(str(i+1),))
pool.close()
pool.join()
# 批量对一个序列中的元素进行操作，同步
pool.map(func=start_process, iterable=[1,2,3,4])
pool.close()
pool.join()
# 批量对一个序列中的元素进行操作，异步
pool.map_async(func=start_process, iterable=[1,2,3,4], callback=callback)
pool.close()
pool.join()
print('process exec finish')
```
### 2.使用concurrent.futures下的ProcessPoolExecutor
默认如果不设置进程池的大小，则为cpu核心数或者1
ProcessPoolExecutor默认是异步的
#### 使用submit
```
from concurrent.futures import ProcessPoolExecutor
import time

def start_process(n, m):
    print(f'{n} process start')
    time.sleep(10)
    print(f'{n} process finish')
    return n, m

pool = ProcessPoolExecutor(4)
ret_list = []
for i in range(5):
    # 多个参数使用带参方式传入
    ret = pool.submit(fn=start_process, n=str(i+1), m=str(i))
    ret_list.append(ret)
for ret in ret_list:
    print(ret.result())
# 等待进程全部结束再继续运行，相当于join
pool.shutdown(wait=True)
```
### 使用map，需要注意的是多参数传入时，最好使用args传入
```
def start_process_muti(n, m):
    print(f'{n} process start')
    time.sleep(10)
    print(f'{n} process finish')
    return n, m

# 需要使用args接收多参数，防止出现缺少某一个位置参数的问题
def start_process_do(args):
    return start_process_muti(*args)

data = [(i,j) for i in range(5)  for j in range(5)]
print(data)
ret = pool.map(start_process_do, data)
print(list(ret))
```