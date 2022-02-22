[TOC]

# 汇编语言(第4版)

## <font color='red'></font>



### 总线

+ 地址总线 	决定CPU的寻址能力
+ 控制总线     决定CPU对系统中其他器件的控制能力
+ 数据总线     决定CPU与其他器件进行数据传送时的一次数据传送量

`1Byte=8bit` 1字节等于8位 `1word=2Byte` 1字等于2个字节

==在进行数据传送或运算时,要注意指令的两个操作对象的位数应当是一致的==

<font color='red'>cpu执行汇编指令时认为ah和al是两个不相关的8位寄存器</font>

### 寄存器

+ 8086 CPU有14个寄存器且都是16位的

+ ```
  reg(寄存器):ax,bx,cx,dx,sp,bp,si,di,ip
  sreg(段寄存器):ds,ss,cs,es
  ```

1. ##### 通用寄存器

   + `AX,CX,DX`
   + 可分为两个8位寄存器 AH ,AL依此类推

2. ##### 段寄存器

   + `CS,DS,SS,ES` 

3. ##### 寻址寄存器

   + `BX,BP,SI,DI,SP` 

4. ##### 标志寄存器

   + `FLAG` 

     ```
     ZF 零标志位 结果为0,zf=1,如果结果不为0,那么zf=0
     PF 奇偶标志位 结果的所有bit位中1的个数为偶数,pf=1,如果为奇数,pf=0
     SF 符号标志位 结果为负,sf=1,如果非负,sf=0
     CF 进位标志位 在进行无符号数运算的时候,它记录了运算结果的最高有效位向更高位的进位值,或从更高位的借位值
     OF 溢出标志位 记录有符号数运算的结果是否发生溢出,如果发生溢出,OF=1,如果没有,OF=0
     DF 方向标志位 df=0,每次操作后si,di递增; df=1,每次操作后si,di递减
     ```

     

### Debug

+ -r  查看,改变CPU寄存器的内容
+ -d  查看内存中的内容
+ -e  改写内存中的内容
+ -u  将内存中的机器指令翻译成汇编指令
+ -t  执行一条机器指令
+ -a  以汇编指令的格式在内存中写入一条机器指令



### 地址加法器

+ 将两个16位地址合成为一个20位的物理地址

+ ==物理地址=段地址*16+偏移地址== 将16进制数据左移一位



### 内存中字的存储

+ 用16位寄存器来存储一个字
+ 内存单元是字节单元 则一个字要用两个地址连续的内存单元来存放
+ 如0,1两个内存单元用来存储一个字,这两个单元可以看作一个==起始地址为0的字单元==



### 段

+ 将若干地址连续的内存单元看作一个段
+ 一个段的起始地址一定是16的倍数
+ 偏移地址为16位 即寻址能力为`64KB` 所以一个段的长度最大为 `64KB`

1. 代码段 ==(code segment)==

   #### `CS和IP`

   + CS为代码段寄存器
   + `IP`为指令指针寄存器
   + `jmp 段地址:偏移地址` 指令的功能为,用指令中给出的段地址修改CS,偏移地址修改 `IP`
   + `jmp 某一合法寄存器`  指令的功能为:用寄存器中的值修改`IP`

2. 数据段 ==(data segment)==

   #### `DS和[address]`

   + 对于内存单元[...]来说 它的段地址默认放在`ds`中
   + 8086 CPU不支持将数据直接送入段寄存器的操作 要用一个寄存器来进行中转

3. 栈段 ==(stack segment)==

   #### `SS和SP`

   + ==以字为单位==
   + 任意时刻 `SS:SP`指向栈顶元素
   + `LIFO` last in first out (后进先出)
   + `pop` 出栈    1. 从`SS:SP`指向的字单元中读取数据   2. SP=SP+2
   + `push` 入栈 1. SP=SP-2  2.向`SS:SP`指向的字单元送入数据

4. 安全空间

   #### `0:200~0:2ff`



### 汇编程序执行过程

```
编程(edit)->x.asm
编译(masm)->x.obj
连接(link)->x.exe
加载(command)->内存中的程序
运行(cpu)
```



### 定位内存地址

```
[bx/si/di/bp+idata] 或 idata[bx/si/di/bp]
[bx+si+idata]和[bx+di+idata]
[bp+si+idata]和[bp+di+idata]
```



### 汇编指令

```
mov 标号,标号     //将数据送入寄存器
add 标号,标号     //将数据相加存入寄存器
adc 标号,标号     //带进位加法 标号=标号+标号+CF
sub 标号,标号     //将数据相减存入寄存器
sbb 标号,标号     //带借位减法 标号=标号-标号-CF
cmp 标号,标号     //比较指令 相当于减法指令,但不保存结果,可影响标志寄存器
and 标号,标号     //逻辑与 按位进行   and al,11011111b //将al变为大写字母
or  标号,标号     //逻辑或 按位进行   or  al,00100000b //将al变为小写字母
div 标号         //被除数为16(32)位时,默认在AX(DX和AX)中存放, 结果:高位存储余数,低位存储商
mul 标号         //一个8(16)位乘数默认放在AL(AX)中,结果默认放在AX(高位放在DX,低位放在AX)中
loop 标号	       //1.cx=cx-1 2.判断cx中的值 不为零则转至标号处执行程序,如果为零则向下执行 
push 标号        //入栈
pushf 
pop  标号        //出栈
popf
inc  标号        //标号+1
dec  标号        //标号-1
movsb           //串传送指令 将ds:si指向的内存单元中的字节送入es:di,然后根据df位的值,将si和di递增或递减
rep             //根据cx的值,重复执行后面的串传送指令
cld             //设置df=0,正向传送
sld             //设置df=1,逆向传送
```

### 伪指令

```
db           //定义字节(byte)型数据
dw           //定义字(word)形数据
dd           //定义双字(double word)型数据
db 3 dup (0) //相当于db 0,0,0
word ptr
byte ptr
start
end
ends
```

### 转移指令

1. #### 无条件转移指令

   ```
   操作符offset    ;取得标号的偏移地址
   段内短转移 jmp short 标号(转到标号处执行指令) ;对IP的修改范围为-128~127 即IP=IP+8位位移
   段内近转移 jmp near ptr 标号 ;IP=IP+16位位移
   段间转移   jmp far ptr 标号  ;用标号的段地址和偏移地址修改CS和IP
   jmp word ptr 内存单元地址 ;转移偏移地址
   jmp dword ptr 内存单元地址 ;高地址处的字是转移的目的段地址,低地址处是转移的目的偏移地址
   ```

   #### CALL和==RET==指令

   ```
   ret     ;相当于pop IP
   retf    ;相当于pop IP,pop CS
   iret    ;相当于pop IP,pop CS,popf
   call    ;将当前的IP或CS和IP压入栈中,转移
   call 标号 ;相当于push IP,jmp near ptr 标号
   call far ptr 标号 ;相当于push CS,push IP,jmp far ptr 标号
   ```

2. #### 条件转移指令

   ```
   jcxz 标号 ;如果(cx=0),转移到标号处执行
   je   标号 ;等于则转移   zf=1
   jne  标号 ;不等于则转移  zf=0
   jb   标号 ;低于则转移    cf=1
   jnb  标号 ;不低于则转移  cf=0
   ja   标号 ;高于则转移    cf=0且zf=0
   jna  标号 ;不高于则转移  cf=1或zf=1
   ```

3. #### 循环指令

   ```
   loop 标号 ;cx=cx-1 如果cx!=0,转移到标号处执行
   ```

4. #### 过程

5. #### 中断

   1. ###### 内中断

      + 中断类型码

        ```
        除法错误:0
        单步执行:1
        执行into指令:4
        执行int指令:该指令的格式位int n,指令中的n为字节型立即数,是提供给CPU的中断类型码
        ```

      + 中断向量表

        ```
        中断向量表在内存中保存,其中存放着256个中断源所对应的中断处理程序的入口
        CPU用中断类型码,通过查找中断向量表,就可以得到中断处理程序的入口地址。
        ```

      + 中断过程

        ```
        (从中断信息中)取得中断类型码
        标志寄存器的值入栈(因为在中断过程中要改变标志寄存器的值,所以先将其保存在栈中)
        设置标志寄存器的第8位TF和第九位IF的值为0
        CS的内容入栈
        IP的内容入栈
        从内存地址为中断类型码*4和中断类型码*4+2的两个字单元中读取中断处理程序入口地址设置IP和CS
        ```

        ```
        取得中断类型码N;
        pushf
        TF=0,IF=0
        push CS
        push IP
        (IP)=(N*4),(CS)=(N*4+2)
        ```

      + 0号中断

        ```
        assume cs:code
        
        code segment
        
        start:
        	mov ax,cs
        	mov ds,ax
        	mov si,offset do0	;设置ds:si指向源地址
        	mov ax,0
        	mov es,ax	
        	mov di,200h			;设置es:di指向目的地址
        	mov cx,offset do0end-offset do0		;设置cx为传输长度
        	cld					;设置传输方向为正
        	rep movsb			
        	
        	mov ax,0			
        	mov es,ax			;设置中断向量表
        	mov word ptr es:[0*4],200h	;0:0字单元存放偏移地址
        	mov word ptr es:[0*4+2],0	;0:2字单元存放段地址
        	
        	mov ax,1000h
        	mov bh,1
        	div   bh			;除法溢出进行0号中断
        
        	mov ax,4c00h
        	int 21h
        
        do0:
        	jmp short do0start
        	db "divide error!"	;将字符串存放在不会被覆盖的空间
        
        do0start:
        	mov ax,cs
        	mov ds,ax
        	mov si,202h			;设置ds:si指向字符串
        	
        	mov ax,0b800h
        	mov es,ax
        	mov di,12*160+36*2	;设置es:di指向显存空间的中间位置
        	mov cx,13           ;设置cx为字符串长度
        
        s:
        	mov al,[si]
        	mov es:[di],al
        	inc si
        	add di,2
        	loop s
        	
        	mov ax,4c00h
        	int 21h
        
        do0end:	nop
        
        code ends
        end start
        ```
        

      
      
      + int指令
      
        ```
        assume cs:code
        
        code segment
        	s1: db 'Good,better,best,','$'
        	s2: db 'Never let it rest,','$'
        	s3: db 'Till good is better,','$'
        	s4: db 'And better,best.','$'
        	s:  dw offset s1,offset s2,offset s3,offset s4
        	row:db 2,4,6,8
        	
        start:
        	mov ax,cs
        	mov ds,ax
        	mov bx,offset s
        	mov si,offset row
        	mov cx,4
        ok:
        	mov bh,0
        	mov dh,[si]
        	mov dl,0
        	mov ah,2
        	int 10h
        
        	mov dx,[bx]
        	mov ah,9
        	int 21h
        	inc si
        	add bx,2
        	loop ok
        	
        	mov ax,4c00h
        	int 21h
        code ends
        end start
        ```
      
      

### 端口

+ 端口的读写 ==(在in和out指令中,只能使用ax或al来存放数据)==

  ```
  in al,60h  ;从60h端口读入一个字节
  out 20h,al ;往20h端口写入一个字节
  ```

+ 逻辑移位指令

  ```
  shl 标号,标号 ;将一个寄存器或内存单元中的数据向左移位,最后移出的一位写入CF中,最低位用0补充
  shr 标号,标号 ;将一个寄存器或内存单元中的数据向右移位,最后移出的一位写入CF中,最高位用0补充
  如果移动位数大于1时,必须将移动位数放在cl中
  mov al,01010001b
  mov cl,3
  shl al,cl
  ```

+ CMOS RAM芯片

  + 端口地址为70h和71h
  + 70h为地址端口
  + 71h为数据端口

```
assume cs:code

data segment
	db '9,8,7,4,2,0'
data ends

stack segment
	db 8 dup (0)
stack ends

code segment

start:
	mov ax,stack
	mov ss,ax
	mov sp,0

	mov ax,data
	mov ds,ax
	mov si,0

	mov bx,0b800h
	mov es,bx
	mov di,2000

	mov cx,6
s:	
	push cx
	mov bl,[si]
	sub bl,30h
	mov byte ptr al,bl
	out 70h,al
	in al,71h
	
	mov ah,al	
	mov cl,4
	shr ah,cl
	and al,00001111b
	
	add ah,30h
	add al,30h
	
	mov byte ptr es:[di],ah
	add di,2
	mov byte ptr es:[di],al
	add di,2
	
	add si,2
	pop cx
	loop s	


	mov ax,4c00h
	int 21h
code ends
end start
```



### 外中断

1. #### 可屏蔽中断

   ```
   如果IF=1,当CPU检测到可屏蔽中断信息时，则CPU在执行完当前命令后响应中断.如果IF=0,则不响应可屏蔽中断
   sti  ;设置IF=1
   cli  ;设置IF=0
   ```

   

2. #### 不可屏蔽中断



### 直接定址表

```
assume cs:code


code segment
start:
	mov ax,cs
	mov ds,ax
	mov si,offset setscreen       ;ds:[si]指向源地址
	mov ax,0
	mov es,ax
	mov di,200h		;es:[di]指向目标地址
	mov cx,offset setscreenend-offset setscreen	;cx计算出地址长度
	cld			;方向为正
	rep movsb
	
	mov ax,0
	mov es,ax
	mov word ptr es:[7ch*4],200h
	mov word ptr es:[7ch*4+2],0		;设置中断向量
	

	mov ax,0002h
	int 7ch
	
	mov ax,4c00h
	int 21h

setscreen:
	jmp short set
	table dw sub1,sub2,sub3,sub4
set:
	push bx
	cmp ah,3
	ja sret
	mov bl,ah
	mov bh,0
	add bx,bx
	
	call word ptr table[bx]	
	
sret:
	pop bx
	iret




sub1:
	push bx
	push cx
	push es
	mov bx,0b800h	
	mov es,bx
	mov bx,0
	mov cx,2000
sub1s:
	mov byte ptr es:[bx],' '
	add bx,2
	loop sub1s
	pop es
	pop cx
	pop bx
	ret

sub2:
	push bx
	push cx
	push es
	mov bx,0b800h	
	mov es,bx
	mov bx,1
	mov cx,2000
sub2s:
	and byte ptr es:[bx],11110000b
	or es:[bx],al
	add bx,2
	loop sub2s

	pop es
	pop cx
	pop bx
	ret

sub3:
	push bx
	push cx
	push es
	mov cl,4
	shl al,cl
	mov bx,0b800h	
	mov es,bx
	mov bx,1
	mov cx,2000
sub3s:
	and byte ptr es:[bx],10001111b
	or es:[bx],al
	add bx,2
	loop sub3s

	pop es
	pop cx
	pop bx
	ret

sub4:
	push si
	push di
	push ds
	push cx
	push es
	
	mov si,0b800h
	mov es,si
	mov ds,si
	mov si,160
	mov di,0
	cld
	mov cx,24

sub4s:
	push cx
	mov cx,160
	rep movsb
	pop cx
	loop sub4s
		
	mov cx,80
	mov si,0
sub4s1:
	mov byte ptr [160*24+si],' '
	add si,2
	loop sub4s1

	pop es
	pop cx
	pop ds
	pop di
	pop si
	ret

setscreenend: nop

code ends
end start
```

### BIOS进行键盘输入和磁盘读写

##### 根据用户输入设置屏幕颜色

```
assume cs:code

code segment

start:
	mov ah,0
	int 16h

	mov ah,1
	cmp al,'r'
	je red
	cmp al,'g'
	je green
	cmp al,'b'
	je blue
	jmp short sret
red:
	shl ah,1
green:
	shl ah,1
blue:
	mov bx,0b800h
	mov es,bx
	mov bx,1
	mov cx,2000
s:
	and byte ptr es:[bx],11111000b
	or es:[bx],ah
	add bx,2
	loop s
sret:
	mov ax,4c00h
	int 21h
code ends
end start
```

##### 字符串输入

```
assume cs:code

code segment
start:
	
	call getstr		;输入字符串
	mov ax,4c00h	;程序结束 返回值0
	int 21h

getstr:
	push ax		
getstrs:
	mov ah,0		
	int 16h		;读取键盘缓冲区
	cmp al,20h	
	jb nochar		;ASCLL码小于20 不是字母
	mov ah,0
	call charstack	;字符入栈
	mov ah,2
	call charstack	;显示字符
	jmp getstrs	;获取下一个字符

nochar:
	cmp ah,0eh	
	je backspace	;退格键键码
	cmp ah,1ch
	je enter		;回车键键码
	jmp getstrs	;获取下一个字符

backspace:
	mov ah,1
	call charstack	;字符出栈
	mov ah,2
	call charstack	;重新显示字符
	jmp getstrs	;获取下一个字符

enter:
	mov al,0		;添加字符串结束标志0
	mov ah,0
	call charstack	;字符0入栈
	mov ah,2
	call charstack	;重新显示字符
	pop ax
	ret		;子程序返回

charstack:
	jmp short charstart		
table	dw charpush,charpop,charshow	;功能表
top	dw 0			;栈顶

charstart:
	push bx
	push dx
	push di
	push es
	
	cmp ah,2		
	ja sret		;无效输入退出
	mov bl,ah
	mov bh,0
	add bx,bx
	jmp word ptr table[bx] 	;根据ah值调用子程序

charpush:		;字符入栈
	mov bx,top
	mov [si][bx],al
	inc top
	jmp sret

charpop:		;字符出栈
	cmp top,0
	je sret		栈为空 无法出栈
	dec top
	mov bx,top
	mov al,[si][bx]
	jmp sret

charshow:
	mov bx,0b800h
	mov es,bx
	mov dx,1230h
	mov al,160
	mov ah,0
	mul dh
	mov di,ax		
	add dl,dl
	mov dh,0
	add di,dx		;es:[di]指向B800:160*12+30*2
	
	mov bx,0		;bx指向栈底

charshows:
	cmp bx,top
	jne noempty 	;将栈中字符显示在屏幕上
	mov byte ptr es:[di],' '
	jmp sret

noempty:
	mov al,[si][bx]
	mov es:[di],al
	mov byte ptr es:[di+2],' '
	inc bx
	add di,2
	jmp charshows

sret:
	pop es
	pop di
	pop dx
	pop bx
	ret

code ends
end start
```

