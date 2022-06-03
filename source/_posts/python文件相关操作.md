---
title: python文件相关操作
date: 2022-06-03 11:38:54
tags:
- python
categories:
- python常用函数
---
## 文件复制
1.读取流方式复制
```
def copy_file(orign_file, new_file):
    """
    文件复制
    orign_file:旧文件
    new_file:新文件
    """
    try:
        if os.path.exists(new_file) == True:
            os.remove(new_file)
        with open(orign_file, encoding="utf-8", mode="r") as f:
            while True:
                # 一次性读取少数，防止因为大文件导致内存爆满
                content = f.read(2048)
                print("content:{}".format(content))
                if content == "":
                    break
                with open(new_file, encoding="utf-8", mode="a+") as f2:
                    f2.write(content)
    except Exception as e:
        print("copy_file fail:{}".format(e))
```
2.使用shutil的copy相关方法实现拷贝,注意拷贝的目标是文件夹，不是文件
```
# 复制文件
import shutil
src = "需要拷贝的文件"
des = "拷贝后文件夹"
if not os.path.exists(des):
    os.mkdir(des)
 # 非全拷贝，文件相关信息不会拷贝
shutil.copy(src,des)
# 全拷贝，文件相关信息会拷贝
shutil.copy2(src,des)
```
## 文件夹复制
1.使用逐个文件或文件夹复制的方式，此处使用到上方的copy_file文件流拷贝的方式
```
def copy_dir(src, des):
    """
    文件夹复制
    src:起始文件夹
    des:需要复制的目标文件夹，可以是多层不存在路径
    """
    if not os.path.exists(des):
        os.makedirs(des)
    for current_path, dir_list, file_list in os.walk(src):
        if len(file_list):
            for file in file_list:
                copy_file(file, os.path.join(des, file))
        if len(dir_list):
            for dir in dir_list:
                copy_dir(os.path.join(current_path, dir), os.path.join(des, dir))

copy_dir(src, des)
```
2.使用shutil的copytree方法复制文件夹，shutil会自动创建目标文件夹
```
# 复制文件夹
import shutil
src = "需要拷贝的文件夹"
des = "拷贝后文件夹"
shutil.copytree(src, des)
```
## 文件压缩
1.使用zipFile模块进行压缩，较为繁琐
```
def files_to_zip(dirpath, outFullName):
    """
    压缩指定文件夹
    :param dirpath: 目标文件夹路径
    :param outFullName: 压缩文件保存路径+xxxx.zip
    :return: 压缩文件的路径
    """
    try:
        out_path = os.path.dirname(outFullName)
        if not os.path.exists(out_path):
            os.makedirs(out_path)
        if not len(os.listdir(dirpath)):
            return
        zip = zipfile.ZipFile(outFullName,"w",zipfile.ZIP_DEFLATED)
        for current_path, _, file_list in os.walk(dirpath):
            if len(file_list):
                for filename in file_list:
			# 第一个参数:需要压缩的文件路径，第二个参数:被压缩后的压缩包中的路径
                    zip.write(os.path.join(current_path, filename), os.path.join(current_path.replace(dirpath, ''), filename))
        zip.close()
        return outFullName
    except Exception as e:
        print(f"zipDir error is {str(e)}")
        return ""
```
2.使用shutil中的make_archive进行压缩
base_name：压缩后的文件，不需要包括扩展名，需要是全路径，不能只是名称
format:压缩的格式： "zip", "tar", "gztar"
base_dir：开始压缩的文件夹路径，压缩文件里包含压缩文件的绝对路径
root_dir：同base_dir类似，区别在于压缩文件中不是绝对路径，是当前目录的路径
```
shutil.make_archive(base_name=None, format=None, base_dir=None,root_dir=None)
例子：
shutil.make_archive(base_name=new_zip_name,format='zip',root_dir=need_to_zip_path)
```
## 文件解压缩
1.使用zipFile解压文件，较为繁琐
```
def extract_files(zip_file_path, dest_path):
    """
    解压zip文件
    zip_file_path：需要解压的zip文件路径
    dest_path: 解压后的路径
    """
    try:
        zip = zipfile.ZipFile(zip_file_path, "r")
        zip.extractall(path=dest_path)
        # 解压后端所有文件路径（列表）
        file_names = zip.namelist()
        print("file_names:{}".format(file_names))
        zip.close()
    except Exception as e:
        print("extract_files fail:{}".format(e))
```
2.使用shuitl中的unpack_archive解压文件
filename：需要解压的压缩包全路径
extract_dir：解压后的文件夹：需是全路径
format：压缩的格式 "zip", "tar", "gztar"
```
shutil.unpack_archive(filename=None, extract_dir=None, format=None)
shutil.unpack_archive(filename=zip_path,extract_dir=extract_path, format='zip')
```
## 获取文件属性及列表
```
# 获取文件夹下的文件及文件夹列表
for item in os.listdir("文件夹"):
    print("item:{}".format(item))
# 获取文件和目录属性(如文件大小和修改日期、创建人)
for item in os.scandir("文件夹"):
    state = item.stat()
    print("name:",item.path)
    print("create_time:",state.st_ctime)
    print("modify_time:", state.st_mtime)
    print("size:",state.st_size)
```
## 文件夹创建
```
# 创建单个文件夹
os.mkdir("test")
# 创建多个递归文件夹，以’/‘作为分割
os.makedirs("2020/02/01")
```
## 文件名模式匹配
```
# 文件名模式匹配 startswith:以什么开头；endswith：以什么结尾
for item in os.listdir("文件夹"):
 if item.startswith("test"):
     print("test start is:",item)
 if item.endswith("file"):
     print("py end is:",item)
# 支持使用 * 和 ? 等通配符, 例如查找test*.text结尾的文件
import fnmatch
for item in os.listdir("文件夹"):
    if fnmatch.fnmatch(item, "test*.text"):
        print("item:",item)
# glob与fnmatch类似
import glob
# 查找当前目录以.py结尾的文件
print(glob.glob("*.py"))
# 支持 shell 样式的通配符来进行匹配 查找当前目录字符串中包含socket的文件夹及文件
print(glob.glob("*socket*"))
# 查找包含数字0-9的文件或文件夹
print(glob.glob("*[1-9]*"))
# 查找当前目录及子目录的以py结尾的文件琥珀文件夹， iglob:返回的是迭代器
for item in glob.iglob("**/*.py", recursive=True):
    print(item)
```
## 遍历目录和处理文件
```
# 遍历目录和处理文件 path:遍历到的路径； dirnames：路径下的文件夹（列表）； files：路径下的文件（列表）
for path, dirnames, files in os.walk("文件夹"):
    print("path:",path)
    print("dirnames:",dirnames)
    print("files:",files)
```
## 创建临时文件并写入内容
```
# 创建临时文件并写入内容
from tempfile import TemporaryFile
with TemporaryFile("w+t") as tf:
    tf.write("hello, world")
    # 回到文件开始读取文件， 不回到开始会读不到内容
    tf.seek(0)
    print(tf.read())
```
## 创建临时目录
```
# 创建临时目录
import tempfile
with tempfile.TemporaryDirectory() as tempdir:
    print("create TemporaryDirectory:", tempdir)
    print(os.path.exists(tempdir))
```
## 删除文件
```
# 删除文件
# 判断是否是文件
file = "文件"
if os.path.isfile(file):
    # remove与unlink类似
    os.remove(file) 
    # os.unlink(file)
else:
    print("not file")
```
## 删除目录
```
# 删除目录
# removedirs:必须要目录内为空才能删除，否则会抛出异常
os.removedirs("文件夹")
# 删除非空目录及完整目录树
import shutil
shutil.rmtree("文件夹")
```
## 移动文件
```
# 移动文件
import shutil
src = "需要移动的文件夹/文件"
des = "需要移动到的文件夹"
shutil.move(src, des)
```
## 重命名文件
```
os.rename("需要重命名的文件/文件夹", "重命名后的文件/文件夹")
```

