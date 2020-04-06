# plt和got总结

### 函数调用：
例如printf函数存在于动态链接库glibc中，所以程序编译链接的时候其实并不知道printf
函数的地址，除此之外还有一个问题就是代码段中的数据是无法修改的，所以需要利用数据段中的一小段代码计算偏移量
### 延迟重定位：
```asm
void system@plt(){
    address:        jmp system@got;链接的时候，链接器先将这个地方的地址填写为lookup_system的地址
                    ret
    lookup_system:  调用重定位函数查找system的地址，并且写到system@got中
                    goto address
}
```
当程序`第一次`调用system函数的时候，由于真实的system地址并没有被写入system@got中，所以会先执行跳转到lookup_system的代码，并且调用了查找函数将system@got中的地址`改为真实的system地址`，随后执行了goto跳转到了address的地方，此时system@got已经被修改为真实的system地址，所以程序进入system中执行。之后的程序如果`再次调用`system函数，那么会直接进入真实的system函数地址，因为system@got的位置已经被正确的修正了，而lookup_system的代码将被`废弃`。