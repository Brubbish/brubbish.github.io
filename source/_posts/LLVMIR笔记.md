---
abbrlink: 0
hidden: true
---
# 标识符
- 全局标识符（函数、全局变量）以'@'开头
- 本地标识符（寄存器名、类型）以'%'开头

## 三种格式
命名值：格式为带前缀的字符串，如%foo、@abcd，匹配方式：[%@][-a-zA-Z$._][-a-zA-Z$._0-9]*    
未命名值：格式为带前缀的数值，如@1、%12，一般为临时变量，在整个llvmir中按序号命名    
数值：


# 链接类型
暂无涉及

# 结构体类型

```
%mytype = type { %mytype*, i32 }
```

# 全局变量

```
@<GlobalVarName> = [Linkage] [PreemptionSpecifier] [Visibility]
                   [DLLStorageClass] [ThreadLocal]
                   [(unnamed_addr|local_unnamed_addr)] [AddrSpace]
                   [ExternallyInitialized]
                   <global | constant> <Type> [<InitializerConstant>]
                   [, section "name"] [, comdat [($name)]]
                   [, align <Alignment>] (, !name !N)*
```

# 函数

# 别名

# 元数据
形如：
```
!0 = !{i8 0, i8 9}
!1 = !{i8 0, i8 4}
```

# LLVM IR 属性组


```
declare i8 @llvm.ctpop.i8(i8) #0
attributes #0 = { nofree nosync nounwind readnone speculatable willreturn }
```


# 数据布局
指定数据如何布局在内存中
```
target datalayout = "e-m:e-p:64:64-i64:64-f80:128-n8:16:32:64-S128"
```


# 实验化简


## 删除
### 注释
以';'分隔到行尾    

## 统一
### 寄存器
以@开头，

arm64：
x64：rax、rbx、rcx、rdx、rbp、rsp、rsi、rdi、r8、r9、r10、r11、r12、r13、r14、r15... 从前5-231行读取

![1](https://pic1.zhimg.com/80/v2-383b34d6c1c89be379e03570d4af8ffc_720w.jpg)


### 临时变量
opt自动排序过


## 函数体（外）
label：粗粒度出现
对齐

函数声明

https://llvm.liuxfe.com/docs/langref/identifiers
