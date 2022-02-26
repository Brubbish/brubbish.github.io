---
title: shell 学习
tag: 笔记
hidden: true
abbrlink: 46066
---

先按照菜鸟的章节进行划分toc，因为有一定linux命令基础所以可能会增删一些内容    

# Overall
- 运行方法：
```shell
sh ./xxx.sh #将文件名作为sh解释器的参数
# or
chmod +x ./xxx.sh   #作为可执行程序直接执行
./xxx.sh
```
- 用 **#!** 说明执行用的解释器，如
```shell
#!/bin/sh

#!/usr/bin/env sh #在系统环境中找sh

#!/usr/bin/env -S -P /usr/local/bin:/usr/bin:${Path} sh #在/usr/local/bin 和 /usr/bin 以及 PATH 定义的目录下找sh
```

# 变量

## 定义变量
```shell
a="aaaa"
b=1
for file in `ls ./Desktop`
for file in $(ls ./Desktop) #以上两句在每次循环中取一个Desktop下的文件名给file变量
```
> 命名只能用字母数字下划线，且不能以数字开头     
“=”两边不能有空格    
双引号和单引号等价

### 只读变量
在定义变量后使用 readonly *  ，如：    
```shell
a="Bruce"
readonly a
```
### 字符串
1. 单引号。      
单引号中的字符会原样输出，不能使用变量或转义字符     
不能出现单独一个单引号，但可以成对出现来对字符串进行拼接

2. 双引号。   
双引号中可以使用变量和转义字符

- 获取字符串长度：
```shell
echo ${#string}
```
- 提取字串
```shell
echo ${sting1:4}    #从左到右，开头下标为0
```
- 查找子串
```shell
string="runoob is a great site"
echo `expr index "$string" o`  # 输出 4
```

### 数组
> 1. array_name=(v0 v1 v2 ...) 
> 2. array_name=(   
    v0   
    v1   
    v2   
    ...   
)
> 3. array_name[0]=v0   
     array_name[1]=v1
     ...




## 使用变量
使用变量需要在变量名前添加dollar符：    
```shell
a_price=1
echo $a
echo ${a}
```
其中花括号可加可不加，但为了能让解释器正确识别变量，建议都加上花括号

## 删除变量
unset，如
```shell
a_price=1
unset a_price
b_price=${a_price}  #ERROR!
```

