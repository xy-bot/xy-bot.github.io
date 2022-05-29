---
title: python将命令输出写入文件或临时缓存
date: 2022-05-29 16:45:27
tags:
---
## python将命令输出写入文件
将文件写入到对应文件，方便后期处理或保存
```
def write_file(file_path):
    with open(file=file_path, mode="w", encoding="utf-8") as out_file:
        command = "ifconfig"
        p = subprocess.Popen(command, shell=True, stdout=out_file, \
            stdin=subprocess.PIPE, bufsize=1, universal_newlines=True)
        print(p.pid)

file_path = os.path.join(os.path.dirname(__file__), "test.text")
write_file(file_path)
```
## python将命令输出写入临时文件或者缓存
```
# 临时文件,一般用于保存临时信息
import tempfile
def write_temp():
    temp = tempfile.SpooledTemporaryFile(max_size=1024 * 10)
    out_temp = temp.fileno()
    command = "ifconfig"
    p = subprocess.Popen(command, shell=True, stdout=out_temp, \
        bufsize=1, universal_newlines=True)
    print(p.pid)
    print(out_temp.conjugate())

```
## tempfile不同临时文件的区别
```
tempfile.TemporaryFile([mode=’w+b'[, bufsize=-1[, suffix=”[, prefix=’tmp'[, dir=None]]]]])
```
该函数返回一个 类文件 对象(file-like)用于临时数据保存（实际上对应磁盘上的一个临时文件）。当文件对象被close或者被del的时候，临时文件将从磁盘上删除。mode、bufsize参数的单方与open()函数一样；suffix和prefix指定了临时文件名的后缀和前缀；dir用于设置临时文件默认的保存路径。返回的类文件对象有一个file属性，它指向真正操作的底层的file对象。
```
tempfile.NamedTemporaryFile([mode=’w+b'[, bufsize=-1[, suffix=”[, prefix=’tmp'[, dir=None[, delete=True]]]]]])
```
tempfile.NamedTemporaryFile函数的行为与tempfile.TemporaryFile类似，只不过它多了一个delete参数，用于指定类文件对象close或者被del之后，是否也一同删除磁盘上的临时文件（当delete = True的时候，行为与TemporaryFile一样）。
```
tempfile.SpooledTemporaryFile([max_size=0[, mode=’w+b'[, bufsize=-1[, suffix=”[, prefix=’tmp'[, dir=None]]]]]])
```
tempfile.SpooledTemporaryFile函数的行为与tempfile.TemporaryFile类似。不同的是向类文件对象写数据的时候，数据长度只有到达参数max_size指定大小时，或者调用类文件对象的fileno()方法，数据才会真正写入到磁盘的临时文件中。
