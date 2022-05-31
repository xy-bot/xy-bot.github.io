---
title: python不同平台进程的启动与终止
date: 2022-05-31 19:17:48
tags:
- python
categories:
- python常用函数
---
## Liunx进程的启动与终止
在使用subprocess创建进程时需要将所有进程设置为一个进程组
preexec_fn：只在 Unix 平台下有效，用于指定一个可执行对象（callable object），它将在子进程运行之前被调用
close_fds：在执行子进程之前,将关闭除0、1和2以外的所有文件描述符；表示子进程将不会继承父进程的输入、输出、错误管道
```
import subprocess, os, signal, time

process_pid = None

# linux
def start_process():
    command = "adb shell"
    p = subprocess.Popen(command, shell=True, stdout=subprocess.PIPE, \
        preexec_fn=os.setpgrp, close_fds=True)
    print("adb shell")
    global process_pid
    process_pid = p.pid
    print(process_pid)

start_process()
time.sleep(5)
os.killpg(process_pid, signal.SIGKILL)
```
## Windows进程的启动与终止
由于preexec_fn不适用于Windows系统，因此采取另外方式
### 方式一：
使用psutils遍历进程ID的子进程依次杀，此方法也适用于Linux系统
```
# 进程开启
import subprocess, psutil, time
command = "adb shell"
process = subprocess.Popen(command, shell=True, \
    stdout=subprocess.PIPE, stderr=subprocess.PIPE)
process_id = process.pid
print(process_id)
print("adb shell")

# 进程终止
def kill(proc_pid):
    parent_proc = psutil.Process(proc_pid)
    # recursive=True：获取所有的父子进程
    for child_proc in parent_proc.children(recursive=True):
        child_proc.kill()
    parent_proc.kill()
time.sleep(5)

kill(process_id)

```
### 方式二：
直接使用Windows的taskkill -t:进程树，-f:强制杀死
```
taskkill /pid process_id -t -f
```
