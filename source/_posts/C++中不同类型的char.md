title: C++中不同类型的char
author: Jiahao Wu
tags:
  - C++
categories:
  - C++
date: 2021-03-01 13:03:00
---
C++中提供了四种不同的char类型，分别是char、wchar_t、char16_t、char32_t。我们主要来看下他们的不同。

# 占用空间不同

运行如下代码：
```C++
int main()
{
    cout << "size of char is " << sizeof(char) << endl;
    cout << "size of wchar_t is " << sizeof(wchar_t) << endl;
    cout << "size of char16_t is " << sizeof(char16_t) << endl;
    cout << "size of char32_t is " << sizeof(char32_t) << endl;
    return 0;
}
// 结果如下：
size of char is 1
size of wchar_t is 2
size of char16_t is 2
size of char32_t is 4
```
我们可以知道char、wchar_t、char16_t、char32_t分别是1、2、2、4个字节。  

# char和wchar_t

char是C和C++中的原始字符类型。  
wchar_t是定义和实现的宽字符类型。  
Char16_t和char32_t类型分别表示16位和32位宽字符；

char和wchar_t分别是ANSI和Unicode编码方式，即ANSI中的字符采用8bit，而UNICODE中的字符采用16bit。（对于字符来说ANSI以单字节存放英文字符，以双字节存放中文等字符,而Unicode下，英文和中文的字符都以双字节存放）Unicode码也是一种国际标准编码，采用二个字节编码，与ANSI码不兼容。目前，在网络、Windows系统和很多大型软件中得到应用。8bit的ANSI编码只能表示256种字符，表示26个英文字母是绰绰有余的，但是表示汉字，韩国语等有着成千上万个字符的非西方字符肯定就不够了，正是如此才引入了UNICODE标准。  

