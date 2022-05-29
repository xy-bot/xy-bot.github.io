---
title: python在管道中执行命令
date: 2022-05-29 16:44:24
tags:
---
## python在管道中执行命令
在实际开发中，可能在执行命令过程中，需要在命令的管道中输入相应命令后继续执行，因此需要在执行命令后在命令的管道中输入相应指令
## 方法一
直接使用communicate向管道传入所需指令,注意如果是多个命令，需要在command中间添加\n，例如：ls\nifconfig
```
def write_pipe1():
    command = "adb shell"
    p = subprocess.Popen(command, shell=True, stdout=subprocess.PIPE, \
        stdin=subprocess.PIPE, bufsize=1, universal_newlines=True)
    inner_command = "ifconfig"
    output, error = p.communicate(inner_command)
    print(f"output:\n{output}\n error:{error}")

write_pipe1()


```
## 方法二
向管道中写入数据，使用communicate获取管道中的输出， 多个命令时与方法一类似
```
def write_pipe2():
    command = "adb shell"
    p = subprocess.Popen(command, shell=True, stdin=subprocess.PIPE, stdout=subprocess.PIPE, \
        stderr=subprocess.PIPE, bufsize=1, universal_newlines=True)
    inner_command = "ifconfig"
    p.stdin.write(inner_command)
    # 刷新缓冲区
    p.stdin.flush()
    output, error = p.communicate()
    print(f"output:\n{output}\n error:{error}")

write_pipe2()

```
