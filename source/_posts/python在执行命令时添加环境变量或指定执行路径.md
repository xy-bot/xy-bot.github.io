---
title: python在执行命令时添加环境变量或指定执行路径
date: 2022-05-31 19:48:16
tags:
- python
categories:
- python常用函数
---
cwd: 命令的执行路径，相当于os.chdir('/home')提前切换到对应路径
env: 环境变量，某些执行路径需要添加必须的环境变量，例如fastboot依赖与adb路径下的环境变量
```
import subprocess, os

path = os.path.join(os.path.dirname(__file__), "test")
myenv = dict(os.environ)
myenv["PATH"] = myenv["PATH"] + ";" + path
print(myenv["PATH"])
command = "ls"
p = subprocess.Popen(command, shell=True, stdout=subprocess.PIPE,\
    bufsize=1, universal_newlines=True, cwd='/home', env=myenv)
print(p.stdout.readlines())
print(p.pid)

```