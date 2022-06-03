---
title: python zip、*、**理解
date: 2022-06-03 11:49:10
tags:
- python
categories:
- python核心知识点
---
## zip函数
zip()一般传入可迭代对象（不止一个），将对象中对应的元素打包成一个个元组，然后返回由这些元组组成的zip对象（python2返回元组），需要手动使用tuple、list等序列转换成可使用的对象。
### 打包
```
# 传入一个可迭代对象
data = [i for i in range(3)]
print(list(zip(data)))
# 传入多个可迭代对象，元素个数相对应
data = [i for i in range(3)]
data2 = [j for j in range(3)]
print(list(zip(data, data2)))
# 传入多个可迭代对象，元素个数不对应,会选取元素最少的可迭代对象作为对应
data = [i for i in range(3)]
data2 = [j for j in range(4)]
print(list(zip(data, data2)))
```
![image](https://img2022.cnblogs.com/blog/2850640/202205/2850640-20220527152150166-1044581752.png)
以最后一个为例，流程为
```
[0, 1, 2]
[0, 1, 2]
一一对应，转变为
[(0, 0), (1, 1), (2, 2)]
```
### 解包
只需要在打包的对象前面添加*即可
```
data = [i for i in range(3)]
data2 = [j for j in range(4)]
ret = list(zip(data, data2))
print(ret)
print(*ret)
# 恢复成原来的两个可迭代对象，不过是以列表格式
print(list(zip(*ret)))
```
![image](https://img2022.cnblogs.com/blog/2850640/202205/2850640-20220527154643552-1035923288.png)
### *在不同场景下的区别
1.在非函数或方法中的参数中，类似于在zip中的，就是解包操作，将元组解开为多个元组，也可以再进行zip还原成原来的可迭代对象
```
print(	list(zip(*ret)))
```
2.在函数或方法的参数中的*
作为可变参数，传入时以元组形式呈现
```
def test(m, *args):
    print(m, args)

test('sa', 'asd', 'qwe')
```
![image](https://img2022.cnblogs.com/blog/2850640/202205/2850640-20220527160007224-1938654954.png)
需要注意的是，目前发现当必选参数添加形参名称时，后面的可变参数无法正常设置，如下所示
```
test(m='sa', 'asd', 'qwe')
```
## ** 的使用场景
一般出现在函数或方法的可变关键字参数，以字典形式呈现
```
def test(m, **kwargs):
    print(m, kwargs)

test(1, a=1,b=2)
```
![image](https://img2022.cnblogs.com/blog/2850640/202205/2850640-20220527160916126-1323116094.png)
