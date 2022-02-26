---
title: VEX IR学习
tags:
  - 课题
abbrlink: 52443
date: 2021-07-10
---

搞这个的原因是课题那边，把基本块转成VEX IR后需要进行一些modify。老师：你这周就先把VEX IR （和LLVM IR ）看懂

所以这篇的要求也就是能看懂和在变量名之类的做一点优化（统一）就行了

<!--more-->
## 代码1

从程序里转了一段中间代码：

```
IRSB {
   t0:Ity_I32 t1:Ity_I32 t2:Ity_I32 t3:Ity_I32 t4:Ity_I32 t5:Ity_I32 t6:Ity_I32 t7:Ity_I32 t8:Ity_I32 t9:Ity_I32 t10:Ity_I32 t11:Ity_I32 t12:Ity_I32 t13:Ity_I32 t14:Ity_I32 t15:Ity_I32 t16:Ity_I32 t17:Ity_I32 t18:Ity_I32 t19:Ity_I32 t20:Ity_I32 t21:Ity_I32 t22:Ity_I32 t23:Ity_I32 t24:Ity_I32 t25:Ity_I32 t26:Ity_I32 t27:Ity_I32

   00 | ------ IMark(0x80491b0, 1, 0) ------
   01 | t0 = GET:I32(ebp)
   02 | t13 = GET:I32(esp)
   03 | t12 = Sub32(t13,0x00000004)
   04 | PUT(esp) = t12
   05 | STle(t12) = t0
   06 | ------ IMark(0x80491b1, 2, 0) ------
   07 | PUT(ebp) = t12
   08 | ------ IMark(0x80491b3, 3, 0) ------
   09 | t2 = Sub32(t12,0x00000018)
   10 | PUT(cc_op) = 0x00000006
   11 | PUT(cc_dep1) = t12
   12 | PUT(cc_dep2) = 0x00000018
   13 | PUT(cc_ndep) = 0x00000000
   14 | PUT(esp) = t2
   15 | PUT(eip) = 0x080491b6
   16 | ------ IMark(0x80491b6, 7, 0) ------
   17 | t15 = Add32(t12,0xfffffffc)
   18 | STle(t15) = 0x00000000
   19 | PUT(eip) = 0x080491bd
   20 | ------ IMark(0x80491bd, 7, 0) ------
   21 | t17 = Add32(t12,0xfffffff8)
   22 | STle(t17) = 0x00000000
   23 | ------ IMark(0x80491c4, 6, 0) ------
   24 | PUT(eip) = 0x080491ca
   25 | ------ IMark(0x80491ca, 3, 0) ------
   26 | STle(t2) = 0x0804a021
   27 | ------ IMark(0x80491cd, 3, 0) ------
   28 | t20 = Add32(t12,0xfffffff8)
   29 | PUT(eax) = t20
   30 | PUT(eip) = 0x080491d0
   31 | ------ IMark(0x80491d0, 4, 0) ------
   32 | t22 = Add32(t2,0x00000004)
   33 | STle(t22) = t20
   34 | PUT(eip) = 0x080491d4
   35 | ------ IMark(0x80491d4, 5, 0) ------
   36 | t25 = Sub32(t2,0x00000004)
   37 | PUT(esp) = t25
   38 | STle(t25) = 0x080491d9
   NEXT: PUT(eip) = 0x08049060; Ijk_Call
}
```
angr输出的反编译代码：
```assembly
>>> irsb.pp()
0x80491b0:	push	ebp
0x80491b1:	mov	ebp, esp
0x80491b3:	sub	esp, 0x18
0x80491b6:	mov	dword ptr [ebp - 4], 0
0x80491bd:	mov	dword ptr [ebp - 8], 0
0x80491c4:	lea	eax, [0x804a021]
0x80491ca:	mov	dword ptr [esp], eax
0x80491cd:	lea	eax, [ebp - 8]
0x80491d0:	mov	dword ptr [esp + 4], eax
0x80491d4:	call	0x8049060
```
对应ida中选中的部分：   
![20210428124520](https://raw.githubusercontent.com/Brubbish/pic_bed/master/blog/20210428124520.png)




### 分析1
首先第一行表示函数的开始（IRSB在此处无意义）    
IRSB:

后续的一堆tx:Ity blabla 代表临时变量，同时指明了变量类型    
```
quote:Temporary variables. VEX uses temporary variables as internal registers.
These temporaries are strongly typed (i.e., "64-bit integer" or "32-bit float").
```

IMark没有实际意义，仅作为一个statement，括号中的内容分别代表了在程序中的起始地址、指令长度和（不知道，不过只见过值=0）     

对寄存器操作前需要读取该寄存器[GET]   
操作结束后将值写回[PUT]



t0 = GET:I32(ebp)：t0 gets ebp, which is a 32bit integer    

t12 = Sub32(t13,0x00000004): t12 = t13 - 0x4      

PUT(esp) = t12: esp = t12      
&nbsp;&nbsp; [Update a register with the value of the given IR Expression.]     

STle(t12) = t0: [t12] = t0     
&nbsp;&nbsp;Update a location in memory, "le" in STle (and LDle) stands for "little-endian"     

```
0~7行大概干了个
push ebp
mov ebp,esp
```
09行的命令很直白，但10~15指的是啥呢...似乎跟10~13出现的"cc_*"有挺大关系，但“cc_”只出现在arm架构的文档中：    
```
    # NOTE 2: Something is goofy w/r/t archinfo and VEX; cc_op3 is used in ccalls, but there's
    # no cc_op3 in archinfo, angr itself uses cc_depn instead.  We do the same.
```


17~22对应了：
```
mov     [ebp-4], 0
mov     [ebp-4], 0
```
t15=t12+0xfffffffc，此时t12=ebp。看起来很奇怪，但其实把t15指向了ebp-4的位置，（不过为啥不用减法了？...）   

24~26行实现了两条语句：
```
0x80491c4:	lea	eax, [0x804a021]
0x80491ca:	mov	dword ptr [esp], eax
```
24行：没有实际意义    
26行：    
此时t2=esp，把esp指向的值改为0x0804a021d    
相当于mov [esp],[0x804a021]    
所以其实是再进行了优化的（可能这就是RISC吧...)    

28~34:    
```
0x80491cd:	lea	eax, [ebp - 8]
0x80491d0:	mov	dword ptr [esp + 4], eax
```
但并没有像上面一样绕过eax，直接对内存的进行修改，应该是因为针对一个基本块分析的时候要保存寄存器最后的值，不能再优化了    

最后的跳转：   
放一个官方对于跳转命令的整理
```
typedef
   enum {
      Ijk_INVALID=0x1A00, 
      Ijk_Boring,         /* not interesting; just goto next */
      Ijk_Call,           /* guest is doing a call */
      Ijk_Ret,            /* guest is doing a return */
      Ijk_ClientReq,      /* do guest client req before continuing */
      Ijk_Yield,          /* client is yielding to thread scheduler */
      Ijk_EmWarn,         /* report emulation warning before continuing */
      Ijk_EmFail,         /* emulation critical (FATAL) error; give up */
      Ijk_NoDecode,       /* current instruction cannot be decoded */
      Ijk_MapFail,        /* Vex-provided address translation failed */
      Ijk_InvalICache,    /* Inval icache for range [CMSTART, +CMLEN) */
      Ijk_FlushDCache,    /* Flush dcache for range [CMSTART, +CMLEN) */
      Ijk_NoRedir,        /* Jump to un-redirected guest addr */
      Ijk_SigILL,         /* current instruction synths SIGILL */
      Ijk_SigTRAP,        /* current instruction synths SIGTRAP */
      Ijk_SigSEGV,        /* current instruction synths SIGSEGV */
      Ijk_SigBUS,         /* current instruction synths SIGBUS */
      Ijk_SigFPE,         /* current instruction synths generic SIGFPE */
      Ijk_SigFPE_IntDiv,  /* current instruction synths SIGFPE - IntDiv */
      Ijk_SigFPE_IntOvf,  /* current instruction synths SIGFPE - IntOvf */
      Ijk_Privileged,     /* current instruction should fail depending on privilege level */
      /* Unfortunately, various guest-dependent syscall kinds.  They
	 all mean: do a syscall before continuing. */
      Ijk_Sys_syscall,    /* amd64/x86 'syscall', ppc 'sc', arm 'svc #0' */
      Ijk_Sys_int32,      /* amd64/x86 'int $0x20' */
      Ijk_Sys_int128,     /* amd64/x86 'int $0x80' */
      Ijk_Sys_int129,     /* amd64/x86 'int $0x81' */
      Ijk_Sys_int130,     /* amd64/x86 'int $0x82' */
      Ijk_Sys_int145,     /* amd64/x86 'int $0x91' */
      Ijk_Sys_int210,     /* amd64/x86 'int $0xD2' */
      Ijk_Sys_sysenter    /* x86 'sysenter'.  guest_EIP becomes 
                             invalid at the point this happens. */
   }
   IRJumpKind;
```
## 代码2
```
IRSB {
   t0:Ity_I32 t1:Ity_I32 t2:Ity_I32 t3:Ity_I32 t4:Ity_I32 t5:Ity_I32 t6:Ity_I1 t7:Ity_I32 t8:Ity_I32 t9:Ity_I32 t10:Ity_I32 t11:Ity_I32 t12:Ity_I32 t13:Ity_I32 t14:Ity_I1 t15:Ity_I1

   00 | ------ IMark(0x80491fd, 4, 0) ------
   01 | t5 = GET:I32(ebp)
   02 | t4 = Add32(t5,0xfffffff8)
   03 | t2 = LDle:I32(t4)
   04 | PUT(cc_op) = 0x00000006
   05 | PUT(cc_dep1) = t2
   06 | PUT(cc_dep2) = 0x0000000a
   07 | PUT(cc_ndep) = 0x00000000
   08 | PUT(eip) = 0x08049201
   09 | ------ IMark(0x8049201, 6, 0) ------
   10 | t14 = CmpEQ32(t2,0x0000000a)
   11 | t13 = 1Uto32(t14)
   12 | t11 = t13
   13 | t15 = 32to1(t11)
   14 | t6 = t15
   15 | if (t6) { PUT(eip) = 0x8049207; Ijk_Boring }
   NEXT: PUT(eip) = 0x08049221; Ijk_Boring
}
```
对应汇编代码
```
0x80491fd:	cmp	dword ptr [ebp - 8], 0xa
0x8049201:	jne	0x8049221
```
suprise...汇编好简洁    

03行：t2=[t4]
```    
LDle：The value stored at a memory address, with the address specified by another IR Expression.
```

11行：
```
 Iop_1Uto32, /* :: Ity_Bit -> Ity_I32, unsigned widen */
```
看一下寄存器的定义，t14的大小也是一个bit
所以这个命令的作用就是把它扩展成32位的无符号数，放到t13里    

13行：    
```
 Iop_32to1,  /* :: Ity_I32 -> Ity_Bit, just select bit[0] */
```
再取最低位放到t15里   

讲真没看懂这波操作为啥要这么多次转换



## 分析3_来波大的

```
>>> irsb.vex.pp()
IRSB {
   t0:Ity_I64 t1:Ity_I32 t2:Ity_I32 t3:Ity_I32 t4:Ity_I64 t5:Ity_I64 t6:Ity_I32 t7:Ity_I32 t8:Ity_I32 t9:Ity_I64 t10:Ity_I64 t11:Ity_I64 t12:Ity_I64 t13:Ity_I64 t14:Ity_I64 t15:Ity_I64 t16:Ity_I64 t17:Ity_I64 t18:Ity_I128 t19:Ity_I128 t20:Ity_I64 t21:Ity_I64 t22:Ity_I32 t23:Ity_I32 t24:Ity_I32 t25:Ity_I64 t26:Ity_I64 t27:Ity_I64 t28:Ity_I64 t29:Ity_I32 t30:Ity_I32 t31:Ity_I64 t32:Ity_I64 t33:Ity_I64 t34:Ity_I64 t35:Ity_I64 t36:Ity_I32 t37:Ity_I64 t38:Ity_I64 t39:Ity_I64 t40:Ity_I64 t41:Ity_I64 t42:Ity_I64 t43:Ity_I64 t44:Ity_I32 t45:Ity_I64 t46:Ity_I64 t47:Ity_I64 t48:Ity_I32 t49:Ity_I64 t50:Ity_I64 t51:Ity_I64 t52:Ity_I64 t53:Ity_I64 t54:Ity_I32 t55:Ity_I64 t56:Ity_I64 t57:Ity_I64 t58:Ity_I64 t59:Ity_I64 t60:Ity_I64 t61:Ity_I64 t62:Ity_I64 t63:Ity_I64 t64:Ity_I32 t65:Ity_I64 t66:Ity_I64 t67:Ity_I64 t68:Ity_I64 t69:Ity_I64 t70:Ity_I64 t71:Ity_I64 t72:Ity_I32 t73:Ity_I64 t74:Ity_I64 t75:Ity_I64 t76:Ity_I64 t77:Ity_I64 t78:Ity_I64 t79:Ity_I32 t80:Ity_I64 t81:Ity_I64 t82:Ity_I64 t83:Ity_I64 t84:Ity_I128 t85:Ity_I64 t86:Ity_I64 t87:Ity_I64 t88:Ity_I64 t89:Ity_I64 t90:Ity_I64 t91:Ity_I64 t92:Ity_I64 t93:Ity_I64 t94:Ity_I64 t95:Ity_I32 t96:Ity_I32 t97:Ity_I64 t98:Ity_I64 t99:Ity_I64 t100:Ity_I64 t101:Ity_I64 t102:Ity_I64 t103:Ity_I32 t104:Ity_I64 t105:Ity_I64 t106:Ity_I1

   00 | ------ IMark(0x401163, 3, 0) ------
   01 | t27 = GET:I64(rbp)
   02 | t26 = Add64(t27,0xffffffffffffffd4)
   03 | t29 = LDle:I32(t26)
   04 | t28 = 32Uto64(t29)
   05 | ------ IMark(0x401166, 3, 0) ------
   06 | t30 = 64to32(t28)
   07 | t1 = Add32(t30,0x00000001)
   08 | t34 = 32Uto64(t1)
   09 | ------ IMark(0x401169, 3, 0) ------
   10 | t36 = 64to32(t34)
   11 | t35 = 32Sto64(t36)
   12 | PUT(rip) = 0x000000000040116c
   13 | ------ IMark(0x40116c, 4, 0) ------
   14 | t40 = Shl64(t35,0x02)
   15 | t39 = Add64(t27,t40)
   16 | t38 = Add64(t39,0xffffffffffffffe0)
   17 | t44 = LDle:I32(t38)
   18 | t43 = 32Uto64(t44)
   19 | PUT(rip) = 0x0000000000401170
   20 | ------ IMark(0x401170, 4, 0) ------
   21 | t45 = Add64(t27,0xffffffffffffffd4)
   22 | t48 = LDle:I32(t45)
   23 | t47 = 32Sto64(t48)
   24 | PUT(rip) = 0x0000000000401174
   25 | ------ IMark(0x401174, 4, 0) ------
   26 | t51 = Shl64(t47,0x02)
   27 | t50 = Add64(t27,t51)
   28 | t49 = Add64(t50,0xffffffffffffffe0)
   29 | t54 = 64to32(t43)
   30 | t7 = LDle:I32(t49)
   31 | t6 = Add32(t54,t7)
   32 | t58 = 32Uto64(t6)
   33 | PUT(rip) = 0x0000000000401178
   34 | ------ IMark(0x401178, 4, 0) ------
   35 | t61 = Shl64(t47,0x02)
   36 | t60 = Add64(t27,t61)
   37 | t59 = Add64(t60,0xffffffffffffffe0)
   38 | t64 = 64to32(t58)
   39 | STle(t59) = t64
   40 | PUT(rip) = 0x000000000040117c
   41 | ------ IMark(0x40117c, 4, 0) ------
   42 | t66 = Add64(t27,0xffffffffffffffd8)
   43 | t68 = LDle:I64(t66)
   44 | PUT(rip) = 0x0000000000401180
   45 | ------ IMark(0x401180, 4, 0) ------
   46 | t69 = Add64(t27,0xffffffffffffffd4)
   47 | t72 = LDle:I32(t69)
   48 | t71 = 32Sto64(t72)
   49 | PUT(rip) = 0x0000000000401184
   50 | ------ IMark(0x401184, 5, 0) ------
   51 | t75 = Shl64(t71,0x02)
   52 | t74 = Add64(t27,t75)
   53 | t73 = Add64(t74,0xffffffffffffffe0)
   54 | t79 = LDle:I32(t73)
   55 | t78 = 32Sto64(t79)
   56 | ------ IMark(0x401189, 4, 0) ------
   57 | t16 = Mul64(t78,t68)
   58 | ------ IMark(0x40118d, 3, 0) ------
   59 | ------ IMark(0x401190, 2, 0) ------
   60 | t81 = Sar64(t16,0x3f)
   61 | ------ IMark(0x401192, 5, 0) ------
   62 | PUT(rcx) = 0x0000000000000002
   63 | ------ IMark(0x401197, 3, 0) ------
   64 | t84 = 64HLto128(t81,t16)
   65 | t106 = CmpEQ64(0x0000000000000002,0x0000000000000000)
   66 | if (t106) { PUT(rip) = 0x401197; Ijk_SigFPE_IntDiv }
   67 | t19 = DivModS128to64(t84,0x0000000000000002)
   68 | t88 = 128HIto64(t19)
   69 | PUT(rdx) = t88
   70 | PUT(rip) = 0x000000000040119a
   71 | ------ IMark(0x40119a, 4, 0) ------
   72 | t89 = Add64(t27,0xffffffffffffffd8)
   73 | STle(t89) = t88
   74 | PUT(rip) = 0x000000000040119e
   75 | ------ IMark(0x40119e, 3, 0) ------
   76 | t92 = Add64(t27,0xffffffffffffffd4)
   77 | t95 = LDle:I32(t92)
   78 | t94 = 32Uto64(t95)
   79 | ------ IMark(0x4011a1, 3, 0) ------
   80 | t96 = 64to32(t94)
   81 | t22 = Add32(t96,0x00000001)
   82 | PUT(cc_op) = 0x0000000000000003
   83 | t98 = 32Uto64(t96)
   84 | PUT(cc_dep1) = t98
   85 | PUT(cc_dep2) = 0x0000000000000001
   86 | t100 = 32Uto64(t22)
   87 | PUT(rax) = t100
   88 | PUT(rip) = 0x00000000004011a4
   89 | ------ IMark(0x4011a4, 3, 0) ------
   90 | t101 = Add64(t27,0xffffffffffffffd4)
   91 | t103 = 64to32(t100)
   92 | STle(t101) = t103
   93 | ------ IMark(0x4011a7, 5, 0) ------
   NEXT: PUT(rip) = 0x0000000000401159; Ijk_Boring
}
```

angr的汇编代码

```
>>> irsb.pp()
0x401163:	mov	eax, dword ptr [rbp - 0x2c]
0x401166:	add	eax, 1
0x401169:	movsxd	rcx, eax
0x40116c:	mov	eax, dword ptr [rbp + rcx*4 - 0x20]
0x401170:	movsxd	rcx, dword ptr [rbp - 0x2c]
0x401174:	add	eax, dword ptr [rbp + rcx*4 - 0x20]
0x401178:	mov	dword ptr [rbp + rcx*4 - 0x20], eax
0x40117c:	mov	rcx, qword ptr [rbp - 0x28]
0x401180:	movsxd	rdx, dword ptr [rbp - 0x2c]
0x401184:	movsxd	rdx, dword ptr [rbp + rdx*4 - 0x20]
0x401189:	imul	rcx, rdx
0x40118d:	mov	rax, rcx
0x401190:	cqo	
0x401192:	mov	ecx, 2
0x401197:	idiv	rcx
0x40119a:	mov	qword ptr [rbp - 0x28], rdx
0x40119e:	mov	eax, dword ptr [rbp - 0x2c]
0x4011a1:	add	eax, 1
0x4011a4:	mov	dword ptr [rbp - 0x2c], eax
0x4011a7:	jmp	0x401159
```

这段代码是
```c
	int a[] = {1,2,3,4,5};
	long long j=1;
	for(int i=0;i<3;i++){
		a[i]+=a[i+1];
		j=j*a[i]%2;
	}
```
的循环部分


01~04：执行后    
```
t26=rbp-0x2c
t29=dowrd ptr [rbp-0x2c]=eax
t28=eax(64bits)
```
结合下一步的操作，不是很懂意义何在...


11行：出现了vex ir 指令，Sto，S指的是singed widen；类似的Uto中，U指的是unsigned widen    
对于这行代码的理解可以参考汇编中的movsxd：   
```
 MOVSXD r64, r/m32       Move doubleword to quadword with sign-extension.
这是64位代码的指令，该指令将32位寄存器或地址转换为32位值，并将其符号扩展为64位寄存器。符号扩展是获取源的最高位（符号位）的值，并使用它来填充目标的所有高位。
```

到57行前都是些重复的语句   

57行：   
mul64：官方文档中写道，mul代表的是signless的，相对的有以下乘法：   
```
      /* -- Ordering not important after here. -- */
      ...
      /* Widening multiplies */
      Iop_MullS8, Iop_MullS16, Iop_MullS32, Iop_MullS64,
      Iop_MullU8, Iop_MullU16, Iop_MullU32, Iop_MullU64,

(看上去和arm的精简指令集挺像的)
```
看到这行产生了个疑惑：imul是带符号的，但mul64是个无符号的乘法      
再往下看看，发现，   
```
t16=Mul64(t78,t68)
t81=Sar64(t16,0x3f)
```
完成了以下三条汇编指令：
```
0x401189:	imul	rcx, rdx
0x40118d:	mov	rax, rcx
0x401190:	cqo	
```
sar64：官方文档中没有描述作用，猜测和x64汇编相似，即算术右移（用符号位补）     
cqo：将rax中的符号复制到rdx的每个位上    


64~70行：  
简单的一条    
```
idiv rcx
```
被转换成了七行ir
```
   64 | t84 = 64HLto128(t81,t16)
   65 | t106 = CmpEQ64(0x0000000000000002,0x0000000000000000)
   66 | if (t106) { PUT(rip) = 0x401197; Ijk_SigFPE_IntDiv }
   67 | t19 = DivModS128to64(t84,0x0000000000000002)
   68 | t88 = 128HIto64(t19)
   69 | PUT(rdx) = t88
   70 | PUT(rip) = 0x000000000040119a
```
涉及到了几个没有见过的VEX IR 指令：    
64HLto128：两个64位的寄存器里的值组合扩展到128位的寄存器中  
DivModS128to64：:: V128,I64 -> V128，of which lo half is div and hi half is mod     
128HIto64：128位数据中的高64位复制到64位寄存器中

82~85行，出现了“cc_*”的“寄存器”，但似乎对理解这个基本块的VEX IR完全没有影响...（不会是因为代码片段还不够长吧orz）    




### 第二段

```
>>> irsb.vex.pp()
IRSB {
   t0:Ity_I32 t1:Ity_I32 t2:Ity_I32 t3:Ity_I64 t4:Ity_I64 t5:Ity_I64 t6:Ity_I64 t7:Ity_I64 t8:Ity_I1 t9:Ity_I64 t10:Ity_I64 t11:Ity_I64 t12:Ity_I64 t13:Ity_I64 t14:Ity_I64 t15:Ity_I64 t16:Ity_I64 t17:Ity_I1 t18:Ity_I32 t19:Ity_I32 t20:Ity_I1

   00 | ------ IMark(0x401159, 4, 0) ------
   01 | t5 = GET:I64(rbp)
   02 | t4 = Add64(t5,0xffffffffffffffd4)
   03 | t2 = LDle:I32(t4)
   04 | PUT(cc_op) = 0x0000000000000007
   05 | t15 = 32Uto64(t2)
   06 | t6 = t15
   07 | PUT(cc_dep1) = t6
   08 | PUT(cc_dep2) = 0x0000000000000003
   09 | PUT(rip) = 0x000000000040115d
   10 | ------ IMark(0x40115d, 6, 0) ------
   11 | t18 = 64to32(0x0000000000000003)
   12 | t19 = 64to32(t6)
   13 | t17 = CmpLT32S(t19,t18)
   14 | t16 = 1Uto64(t17)
   15 | t13 = t16
   16 | t20 = 64to1(t13)
   17 | t8 = t20
   18 | if (t8) { PUT(rip) = 0x401163; Ijk_Boring }
   NEXT: PUT(rip) = 0x00000000004011ac; Ijk_Boring
}


反汇编结果：
```asm
0x401159:	cmp	dword ptr [rbp - 0x2c], 3
0x40115d:	jge	0x4011ac
```

第一部分13行：cmpLT32S   
官方文档中的解释很短：
```
      /* Standard integer comparisons */
      Iop_CmpLT32S, Iop_CmpLT64S,
      Iop_CmpLE32S, Iop_CmpLE64S,
      Iop_CmpLT32U, Iop_CmpLT64U,
      Iop_CmpLE32U, Iop_CmpLE64U,
```

该结果为在x86汇编中为指令jge服务（大于等于则跳转）   
所以小于3时t8=t20=t16=1     
因此cmpLT应该是“前小于后返回1”     
所以"CmpL*"是只能比较小于或等于？   

*注：试了一下还真是这样的，形成的中间代码会把其他判断方式转换成"<"/"="*    
*注2：在微软的文档搜到了SSE2指令集，里边的比较指令和 vex ir 长得还蛮像的↓*    
![20210516225639](https://raw.githubusercontent.com/Brubbish/pic_bed/master/blog/20210516225639.png)

## 来个aarch64的
用aarch64-linux-gnu-gcc编译了相同的代码：

```c
#include<stdio.h>
#include<stdlib.h>

struct a{
	int a;
	int* b;
}aa;
int main(){
	int a=1;
	double b=1;
	scanf("%d",&a);
	switch(a){
		case 1:
			printf("a=2");
			break;
		case 2:
			printf("a=1");
			break;
		default:
			printf("a==null");
			break;
	}
	scanf("%d",&aa.a);
	aa.b = (int*)malloc(10);
	if (aa.a ==a)
		aa.b[1] = aa.a;
}
```
用angr加载的时候会产生警告，但（对于这个样例）没有影响      
![20210510184101](https://raw.githubusercontent.com/Brubbish/pic_bed/master/blog/20210510184101.png)     
*注：上一个应该是因为找不到动态链接库，编译选项设置静态编译后可以正常加载*     

发现vex ir用到的寄存器是根据程序cpu架构改变的。如i386、amd64下用到PUT(rip)、PUT(eip)，在aarch64下则是PUT(pc)    

选取下面这句代码的中间代码：
```c
    *(_DWORD *)(qword_4999C8 + 4) = aa;
```

### aarch64

![20210510213959](https://raw.githubusercontent.com/Brubbish/pic_bed/master/blog/20210510213959.png)

```
   00 | ------ IMark(0x400778, 4, 0) ------
   01 | ------ IMark(0x40077c, 4, 0) ------
   02 | PUT(pc) = 0x0000000000400780
   03 | ------ IMark(0x400780, 4, 0) ------
   04 | t4 = LDle:I64(0x00000000004999c8)
   05 | ------ IMark(0x400784, 4, 0) ------
   06 | t7 = Add64(t4,0x0000000000000004)
   07 | ------ IMark(0x400788, 4, 0) ------
   08 | ------ IMark(0x40078c, 4, 0) ------
   09 | PUT(pc) = 0x0000000000400790
   10 | ------ IMark(0x400790, 4, 0) ------
   11 | t34 = LDle:I32(0x00000000004999c0)
   12 | t57 = 32Uto64(t34)
   13 | t33 = t57
   14 | PUT(pc) = 0x0000000000400794
   15 | ------ IMark(0x400794, 4, 0) ------
   16 | t58 = 64to32(t33)
   17 | t37 = t58
   18 | STle(t7) = t37
   19 | ------ IMark(0x400798, 4, 0) ------
   20 | ------ IMark(0x40079c, 4, 0) ------
   21 | PUT(x1) = 0x0000000000000000
   22 | ------ IMark(0x4007a0, 4, 0) ------
   23 | PUT(pc) = 0x00000000004007a4
   24 | ------ IMark(0x4007a4, 4, 0) ------
   25 | t19 = LDle:I64(0x0000000000497f50)
   26 | PUT(x0) = t19
   27 | PUT(pc) = 0x00000000004007a8
   28 | ------ IMark(0x4007a8, 4, 0) ------
   29 | t46 = GET:I64(xsp)
   30 | t45 = Add64(t46,0x0000000000000028)
   31 | t21 = LDle:I64(t45)
   32 | PUT(pc) = 0x00000000004007ac
   33 | ------ IMark(0x4007ac, 4, 0) ------
   34 | t23 = LDle:I64(t19)
   35 | ------ IMark(0x4007b0, 4, 0) ------
   36 | t27 = Sub64(t21,t23)
   37 | PUT(x2) = t27
   38 | PUT(cc_op) = 0x0000000000000004
   39 | PUT(cc_dep1) = t21
   40 | PUT(cc_dep2) = t23
   41 | PUT(cc_ndep) = 0x0000000000000000
   42 | ------ IMark(0x4007b4, 4, 0) ------
   43 | PUT(x3) = 0x0000000000000000
   44 | PUT(pc) = 0x00000000004007b8
   45 | ------ IMark(0x4007b8, 4, 0) ------
   46 | t60 = CmpEQ64(t21,t23)
   47 | t59 = 1Uto64(t60)
   48 | t55 = t59
   49 | t61 = 64to1(t55)
   50 | t49 = t61
   51 | if (t49) { PUT(pc) = 0x4007c0; Ijk_Boring }
   NEXT: PUT(pc) = 0x00000000004007bc; Ijk_Boring
}
```



### amd64
![20210510213751](https://raw.githubusercontent.com/Brubbish/pic_bed/master/blog/20210510213751.png)
```
   00 | ------ IMark(0x40126d, 7, 0) ------
   01 | t9 = LDle:I64(0x0000000000404028)
   02 | ------ IMark(0x401274, 4, 0) ------
   03 | t10 = Add64(t9,0x0000000000000004)
   04 | PUT(rdx) = t10
   05 | PUT(rip) = 0x0000000000401278
   06 | ------ IMark(0x401278, 6, 0) ------
   07 | t13 = LDle:I32(0x0000000000404020)
   08 | t29 = 32Uto64(t13)
   09 | t12 = t29
   10 | PUT(rip) = 0x000000000040127e
   11 | ------ IMark(0x40127e, 2, 0) ------
   12 | t30 = 64to32(t12)
   13 | t14 = t30
   14 | STle(t10) = t14
   15 | ------ IMark(0x401280, 5, 0) ------
   16 | PUT(rax) = 0x0000000000000000
   17 | PUT(rip) = 0x0000000000401285
   18 | ------ IMark(0x401285, 4, 0) ------
   19 | t18 = GET:I64(rbp)
   20 | t17 = Add64(t18,0xfffffffffffffff8)
   21 | t19 = LDle:I64(t17)
   22 | PUT(rip) = 0x0000000000401289
   23 | ------ IMark(0x401289, 9, 0) ------
   24 | t21 = GET:I64(fs)
   25 | t20 = Add64(0x0000000000000028,t21)
   26 | t6 = LDle:I64(t20)
   27 | t5 = Xor64(t19,t6)
   28 | PUT(cc_op) = 0x0000000000000014
   29 | PUT(cc_dep1) = t5
   30 | PUT(cc_dep2) = 0x0000000000000000
   31 | PUT(rcx) = t5
   32 | PUT(rip) = 0x0000000000401292
   33 | ------ IMark(0x401292, 2, 0) ------
   34 | t32 = CmpEQ64(t5,0x0000000000000000)
   35 | t31 = 1Uto64(t32)
   36 | t27 = t31
   37 | t33 = 64to1(t27)
   38 | t22 = t33
   39 | if (t22) { PUT(rip) = 0x401299; Ijk_Boring }
   NEXT: PUT(rip) = 0x0000000000401294; Ijk_Boring
}
```