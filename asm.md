#1. 汇编

### 字节大小
db 一个字节 = 8bit
dw 两个字节 = 16bit
dd 两个两个字节 也就是4个字节 = 32bit
qw 四倍字节 = 64bit //一般用于64位程序
### lea指令
作用为取偏移地址
如:mov ax, [1000h]   则取1000h处的`数据`放ax中
  lea ax, [1000h]   则取1000h放在ax中
### cmp比较指令
进行比较两个操作数的大小，相当于sub指令，只不过不保存值，只影响标志寄存器，`sub会影响值和标志寄存器`
例如:cmp 操作对象1, 操作对象2
将操作对象1-操作对象2,`值不保存，仅影响标志寄存器`
### test指令
和and指令一样，`不保存结果，只影响标志位寄存器`
### ret指令
#####retf指令为远返回，段间返回
>如:
>push 4000   段
>push 5000   ip寄存器

将得到4000:5000
#####retn为段内返回
`只压ip寄存器`
ret默认retn
ret near 为retn
ret far 为retf
### __fast_call
fast call时传参数默认优先选择ecx和edx
### __read
`__readfsbyte(unsiged long offset)` 从fs段offset处读取byte类型
`__readfsword(unsiged long offset)`从fs段offset处读取word类型
`__readsdword(unsiged long offset)`从fs段offset处读取dword类型
`__readfsqword(unsiged long offset)`从fs段offset处读取qword类型

