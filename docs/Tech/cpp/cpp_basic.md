# C++语法基础

## C/C++字符串基础

!!! abstract "概述"
    本节涵盖C/C++字符串的基础知识，包括指针操作、格式化输出、字符串定义和常用操作函数。

### printf格式化输出

!!! info "printf格式说明符"
    printf函数支持多种格式说明符，用于输出不同类型的数据。

```c
#include <cstdio>

int main() {
    int num = 123;
    float f = 3.1415926;
    char ch = 'A';
    const char* str = "Hello, World!";

    printf("Number: %d\n", num);      // 整数
    printf("Float: %.2f\n", f);       // 浮点数（保留2位小数）
    printf("Hex: %x\n", num);         // 十六进制
    printf("Character: %c\n", ch);    // 字符
    printf("String: %s\n", str);      // 字符串
    printf("Pointer: %p\n", &num);    // 指针地址

    return 0;
}
```

### 转义字符总结

!!! tip "常用转义字符"
    转义字符用反斜杠(\)开头，表示特殊字符或控制字符。

| 转义字符 | 含义 |
|---------|------|
| `\n` | 换行符 |
| `\t` | 制表符 |
| `\\` | 反斜杠 |
| `\"` | 双引号 |
| `\0` | 字符串结束符 |
| `\r` | 回车符 |

### 字符串定义

!!! warning "字符串定义注意事项"
    不同的字符串定义方式有不同的内存分配和修改权限。

#### 基本定义方式

```c++
// 字符串字面量（只读）
const char* str0 = "Hello, World!";

// 可修改的字符串数组
char str2[] = "Hello, World!";

// 预分配空间
char str3[20];
str3[0] = 'H';
str3[1] = 'i';
str3[2] = '\0';  // 必须手动添加结束符
```

!!! danger "废弃用法"
    `char* str = "Hello, World!";` 在C++11中已被废弃，会产生编译警告。

#### 字符串输入方法

!!! info "多种输入方式"
    根据需要选择合适的字符串输入方法。

```c++
// 1. scanf - 读取单词（遇空格停止）
char str5[20];
scanf("%19s", str5);  // 限制长度防止溢出

// 2. fgets - 读取整行（C风格）
char str6[100];
while (fgets(str6, sizeof(str6), stdin)) {
    printf("%s", str6);
}

// 3. getline - 读取整行（C++风格）
#include <iostream>
#include <string>
string line;
while (getline(cin, line)) {
    cout << line << endl;
}
```

!!! tip "格式化字符集"
    scanf支持特殊格式化选项：
    
    - `%[a-z]`：读取小写字母
    - `%[^...]`：读取不包含特定字符的字符串
    - `%[...]`：读取包含特定字符的字符串

### 字符串操作函数

!!! note "头文件"
    使用这些函数需要包含 `#include <cstring>` 或 `#include <string.h>`

#### 基本操作

```c++
char* src = "Hello, World!";
char dest[50];

// 长度计算
int len = strlen(src);

// 字符串复制
strcpy(dest, src);          // 完整复制
strncpy(dest, src, 5);      // 复制前5个字符

// 字符串连接
strcat(dest, "!!!");        // 连接字符串
strncat(dest, "!!!", 3);    // 连接前3个字符

// 字符串比较
int cmp = strcmp(src, dest);     // 完整比较
int ncmp = strncmp(src, dest, 5); // 比较前5个字符
```

#### 高级操作

!!! example "字符集检查 - strspn"
    检查字符串前缀是否由指定字符集组成。

```c++
char str[] = "129th";
char accept[] = "1234567890";
int i = strspn(str, accept);
printf("str 前 %d 个字符都属于数字\n", i);  // 输出：3
```

!!! example "子字符串查找 - strstr"
    在字符串中查找子字符串的位置。

```c++
char haystack[] = "Hello, World!";
char needle[] = "World";
char* pos = strstr(haystack, needle);

if (pos != NULL) {
    printf("找到子字符串，位置: %ld\n", pos - haystack);
    printf("子字符串内容: %s\n", pos);
} else {
    printf("未找到子字符串\n");
}
```

!!! example "字符串分割 - strtok"
    按指定分隔符分割字符串。

```c++
char str[] = "Hello, World! This is C++.";
const char* delim = " ,.!";  // 多个分隔符

char* token = strtok(str, delim);
while (token != NULL) {
    printf("%s\n", token);
    token = strtok(NULL, delim);  // 后续调用传入NULL
}
```

!!! warning "strtok注意事项"
    strtok会修改原字符串，将分隔符替换为'\0'。如果需要保持原字符串不变，请先复制一份。

#### 类型转换

!!! example "字符串转数字 - atoi系列"
    将字符串转换为数值类型。

```c++
#include <cstdlib>

char str[100];
fgets(str, sizeof(str), stdin);

int num = atoi(str);           // 转换为整数
long lnum = atol(str);         // 转换为长整数
double dnum = atof(str);       // 转换为浮点数

printf("字符串 %s 转换为整数: %d\n", str, num);
```

!!! tip "更安全的转换"
    考虑使用 `strtol`、`strtod` 等函数，它们提供更好的错误检查机制。