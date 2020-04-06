### 1.strcspn
```C
size_t strcspn(const char *str1, const char *str2)
```
```C
#include <stdio.h>
#include <string.h>


int main ()
{
   const char str1[] = "ABCDEF4960910";
   const char str2[] = "013";
   int len = strcspn(str1, str2);
   printf("第一个匹配的字符是在 %d\n", len + 1);
   return(0);
}
```
结果为10，匹配`第二个字符串`在`第一个字符串``第一次`出现的`位置`
### 2.strdup
strdup函数是C语言中常用的一种字符串拷贝库函数，一般和`free`函数成对出现。
`strdup在内部调用了malloc为变量分配内存`，不需要使用返回的字符串时，需要用free释放相应的内存，否则会造成`内存泄漏`。
返回一个指针，指向复制字符串分配的空间，如果分配失败则返回NULL。
### 3.strcpy
C 库函数 char *strcpy(char *dest, const char *src) 把 src 所指向的字符串复制到 dest
### 4.strspn
C 库函数 size_t strspn(const char *str1, const char *str2) 检索字符串 str1 中第一个不在字符串 str2 中出现的字符下标。
通俗的讲，就是返回两个字符串从哪里开始不同的地方
如:`"abc","a"`从b开始不同，返回1
`"abcd", "abc"`从d开始不同，返回3