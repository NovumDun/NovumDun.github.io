# 方案概述

本方案提供了一种嵌入式C/C++程序调试信息压缩方法。将调试信息进行编码并生成编码记录，将编码存储在生成的代码中，运行时输出调试编码及调试参数到PC上位机，上位机根据编码记录找到格式化字符串，并根据格式化字符串格式化调试参数，生成调试信息并输出。

# 现有技术的技术方案

在嵌入式领域中，C/C++代码中常常需要加入调试信息以便定位错误及原因或者获取相关信息。目前常用的方法是通过printf或类似函数将格式字符串格式化，生成调试信息，再通过串口或者其他外设进行输出。

# 现有技术的缺点

1. 必须保存较长的格式字符串在程序中，对资源有限的单片机来说会消耗一定的内存空间，例如简单的printf(“Errno %d\n”, 3)仅格式字符串就占用10个字节。
2. 生成的调试信息较长，以之前的printf(“Errno %d\n”, 3)为例，生成的调试信息“Errno 3\n”至少占用9个字节，如果调试信息需要保存在设备内部，也需要占用较多资源。
3. 目前常用串口作为调试信息输出外设，速率较低，较长的调试信息会花费较多的时间用于传输，当调试打印被频繁调用时容易产生阻塞的情况。

# 本方案要解决的技术问题

本方案要解决的技术问题是因调试信息较长而占用的内存较多，以及产生的调试信息的较长和调试信息传输的时间较长的问题。

# 本方案的详细阐述

## 实现STR_BUF_WRITE_STRU宏

```c
#include "macro_paras_opt.h"
#include "macro_struct.h"
#include "macro_type.h"

#define SET_COUNTER(cnt) \      // 
    FILE_ADDR + cnt;

#ifdef __PYTHON_SCOPE_PRE
#define PYTHON_SCOPE_PRE 
#endif

#define GEN_NAME(name) MACRO_CAT(name, __LINE__)

#define _STATIC_PARAS(num, para)    PYTHON_SCOPE_PRE para;
#define STATIC_PARAS(num, para)     MACRO_PARAS_NOTDEAL_0PARAS(_STATIC_PARAS, para)     // 
#define STR_PARAS(num, para)     MACRO_STR_NUM(para),   // 

#define STR_BUF_WRITE_STRU(str, ...)                                                                     \
    do                                                                                                   \
    {                                                                                                    \
        MACRO_PARAS_ENUM_OPT(STATIC_PARAS, GEN_TEMP_VARS(__VA_ARGS__));                                  \
        GEN_STRUCT_AUTO(GET_TEMP_VARS(__VA_ARGS__));                                                     \
        SET_TEMP_VARS(__VA_ARGS__);                                                                      \
        SET_STRUCT_AUTO(GET_TEMP_VARS(__VA_ARGS__));                                                     \
        GEN_STRUCT(frame, unsigned char preamble; unsigned char len; unsigned short global_cnt;          \
                   GET_STRUCT_AUTO_TYPE() GET_STRUCT_AUTO_VAR();                                         \
                   unsigned char tail;);                                                                 \
        GET_STRUCT_VAR_MEMBER(frame, preamble) = (0xAA & 0xFC) | (0x00);                                 \
        GET_STRUCT_VAR_MEMBER(frame, len) = sizeof(GET_STRUCT_TYPE(frame)) - 2;                          \
        GET_STRUCT_VAR_MEMBER(frame, tail) = 0x55;                                                       \
        PYTHON_SCOPE_PRE char *GEN_NAME(fast_print_str) = str;                                           \
        PYTHON_SCOPE_PRE int GEN_NAME(sizeof_stru_member)[] = {GET_MACRO_PARAS_SIZE(__VA_ARGS__)};       \
        PYTHON_SCOPE_PRE int GEN_NAME(typeof_stru_member)[] = {GET_MACRO_PARAS_TYPE_FIX(__VA_ARGS__)};   \
        PYTHON_SCOPE_PRE char *GEN_NAME(fast_print_paras_str)[] = {                                      \
            MACRO_PARAS_ENUM_OPT(STR_PARAS, GET_TEMP_VARS(__VA_ARGS__))};                                \
        GET_STRUCT_VAR_MEMBER(frame, global_cnt) = SET_COUNTER(__COUNTER__);                             \
        GET_STRUCT_VAR_MEMBER(frame, GET_STRUCT_AUTO_VAR()) = GET_STRUCT_AUTO_VAR();                     \
        console_output_bin((char *)&GET_STRUCT_VAR(frame), sizeof(GET_STRUCT_TYPE(frame)));              \
    } while (0)

STR_BUF_WRITE_STRU("hello %d %d %d %d %d", 0, 1, 2, 3, 4);  // __LINE__ = 37
```

macro analysis.

```c
MACRO_PARAS_ENUM_OPT(STATIC_PARAS, GEN_TEMP_VARS(__VA_ARGS__));                                 // 生成临时变量
                                                                                                // static typeof(0) temp_37_5;
                                                                                                // static typeof(1) temp_37_4;
                                                                                                // static typeof(2) temp_37_3;
                                                                                                // static typeof(3) temp_37_2;
                                                                                                // static typeof(4) temp_37_1;
                                                                                                // ;
GEN_STRUCT_AUTO(GET_TEMP_VARS(__VA_ARGS__));                                                    // 生成临时结构体及变量，用于存放临时变量
                                                                                                // struct gen_auto37_stru
                                                                                                // {
                                                                                                //     typeof(temp_37_5) temp_37_5;
                                                                                                //     typeof(temp_37_4) temp_37_4;
                                                                                                //     typeof(temp_37_3) temp_37_3;
                                                                                                //     typeof(temp_37_2) temp_37_2;
                                                                                                //     typeof(temp_37_1) temp_37_1;
                                                                                                // } __attribute__((__packed__)) gen_auto37;
SET_TEMP_VARS(__VA_ARGS__);                                                                     // 初始化临时变量
                                                                                                // temp_37_5 = 0;
                                                                                                // temp_37_4 = 1;
                                                                                                // temp_37_3 = 2;
                                                                                                // temp_37_2 = 3;
                                                                                                // temp_37_1 = 4;
SET_STRUCT_AUTO(GET_TEMP_VARS(__VA_ARGS__));                                                    // 初始化结构体变量
                                                                                                // gen_auto37.temp_37_5 = temp_37_5;
                                                                                                // gen_auto37.temp_37_4 = temp_37_4;
                                                                                                // gen_auto37.temp_37_3 = temp_37_3;
                                                                                                // gen_auto37.temp_37_2 = temp_37_2;
                                                                                                // gen_auto37.temp_37_1 = temp_37_1;
GEN_STRUCT(frame, unsigned char preamble; unsigned char len; unsigned short global_cnt;         // 
            GET_STRUCT_AUTO_TYPE() GET_STRUCT_AUTO_VAR();                                       // 
            unsigned char tail;);                                                               // 生成临时结构体及变量，用于发送调试信息帧
                                                                                                // struct gen_frame37_stru
                                                                                                // {
                                                                                                //     unsigned char preamble;
                                                                                                //     unsigned char len;
                                                                                                //     unsigned short global_cnt;
                                                                                                //     struct gen_auto37_stru gen_auto37;
                                                                                                //     unsigned char tail;
                                                                                                // } __attribute__((__packed__)) gen_frame37;
GET_STRUCT_VAR_MEMBER(frame, preamble) = (0xAA & 0xFC) | (0x00);                                // gen_frame37.preamble = (0xAA & 0xFC) | (0x00);
GET_STRUCT_VAR_MEMBER(frame, len) = sizeof(GET_STRUCT_TYPE(frame)) - 2;                         // gen_frame37.len = sizeof(struct gen_frame37_stru) - 2;
GET_STRUCT_VAR_MEMBER(frame, tail) = 0x55;                                                      // gen_frame37.tail = 0x55;
PYTHON_SCOPE_PRE char *GEN_NAME(fast_print_str) = str;                                          // 获取格式字符串
                                                                                                // PYTHON_SCOPE_PRE char *fast_print_str37 = "hello %d %d %d %d %d";
PYTHON_SCOPE_PRE int GEN_NAME(sizeof_stru_member)[] = {GET_MACRO_PARAS_SIZE(__VA_ARGS__)};      // 获取调试参数位宽
                                                                                                // PYTHON_SCOPE_PRE int sizeof_stru_member37[] = {(sizeof(0)), (sizeof(1)), (sizeof(2)), (sizeof(3)), (sizeof(4)), (0), (0), (0), (0), (0)};
PYTHON_SCOPE_PRE int GEN_NAME(typeof_stru_member)[] = {GET_MACRO_PARAS_TYPE_FIX(__VA_ARGS__)};  // 获取调试参数类型
                                                                                                // PYTHON_SCOPE_PRE int typeof_stru_member37[] = {(__builtin_types_compatible_p(typeof(0), char) ? 1 : 
                                                                                                // (__builtin_types_compatible_p(typeof(0), unsigned char) ? 2 : 
                                                                                                // (__builtin_types_compatible_p(typeof(0), short) ? 3 : 
                                                                                                // (__builtin_types_compatible_p(typeof(0), unsigned short) ? 4 : 
                                                                                                // (__builtin_types_compatible_p(typeof(0), int) ? 5 : 
                                                                                                // (__builtin_types_compatible_p(typeof(0), unsigned int) ? 6 : 
                                                                                                // (__builtin_types_compatible_p(typeof(0), float) ? 7 : 
                                                                                                // (__builtin_types_compatible_p(typeof(0), double) ? 8 : 
                                                                                                // (__builtin_types_compatible_p(typeof(0), typeof(const char *)) ? 9 : 0))))))))), 
                                                                                                // (__builtin_types_compatible_p(typeof(1), char) ? 1 : 
                                                                                                // (__builtin_types_compatible_p(typeof(1), unsigned char) ? 2 : 
                                                                                                // (__builtin_types_compatible_p(typeof(1), short) ? 3 : 
                                                                                                // (__builtin_types_compatible_p(typeof(1), unsigned short) ? 4 : 
                                                                                                // (__builtin_types_compatible_p(typeof(1), int) ? 5 : 
                                                                                                // (__builtin_types_compatible_p(typeof(1), unsigned int) ? 6 : 
                                                                                                // (__builtin_types_compatible_p(typeof(1), float) ? 7 : 
                                                                                                // (__builtin_types_compatible_p(typeof(1), double) ? 8 : 
                                                                                                // (__builtin_types_compatible_p(typeof(1), typeof(const char *)) ? 9 : 0))))))))), 
                                                                                                // (__builtin_types_compatible_p(typeof(2), char) ? 1 : 
                                                                                                // (__builtin_types_compatible_p(typeof(2), unsigned char) ? 2 : 
                                                                                                // (__builtin_types_compatible_p(typeof(2), short) ? 3 : 
                                                                                                // (__builtin_types_compatible_p(typeof(2), unsigned short) ? 4 : 
                                                                                                // (__builtin_types_compatible_p(typeof(2), int) ? 5 : 
                                                                                                // (__builtin_types_compatible_p(typeof(2), unsigned int) ? 6 : 
                                                                                                // (__builtin_types_compatible_p(typeof(2), float) ? 7 : 
                                                                                                // (__builtin_types_compatible_p(typeof(2), double) ? 8 : 
                                                                                                // (__builtin_types_compatible_p(typeof(2), typeof(const char *)) ? 9 : 0))))))))), 
                                                                                                // (__builtin_types_compatible_p(typeof(3), char) ? 1 : 
                                                                                                // (__builtin_types_compatible_p(typeof(3), unsigned char) ? 2 : 
                                                                                                // (__builtin_types_compatible_p(typeof(3), short) ? 3 : 
                                                                                                // (__builtin_types_compatible_p(typeof(3), unsigned short) ? 4 : 
                                                                                                // (__builtin_types_compatible_p(typeof(3), int) ? 5 : 
                                                                                                // (__builtin_types_compatible_p(typeof(3), unsigned int) ? 6 : 
                                                                                                // (__builtin_types_compatible_p(typeof(3), float) ? 7 : 
                                                                                                // (__builtin_types_compatible_p(typeof(3), double) ? 8 : 
                                                                                                // (__builtin_types_compatible_p(typeof(3), typeof(const char *)) ? 9 : 0))))))))), 
                                                                                                // (__builtin_types_compatible_p(typeof(4), char) ? 1 : 
                                                                                                // (__builtin_types_compatible_p(typeof(4), unsigned char) ? 2 : 
                                                                                                // (__builtin_types_compatible_p(typeof(4), short) ? 3 : 
                                                                                                // (__builtin_types_compatible_p(typeof(4), unsigned short) ? 4 : 
                                                                                                // (__builtin_types_compatible_p(typeof(4), int) ? 5 : 
                                                                                                // (__builtin_types_compatible_p(typeof(4), unsigned int) ? 6 : 
                                                                                                // (__builtin_types_compatible_p(typeof(4), float) ? 7 : 
                                                                                                // (__builtin_types_compatible_p(typeof(4), double) ? 8 : 
                                                                                                // (__builtin_types_compatible_p(typeof(4), typeof(const char *)) ? 9 : 0))))))))), 0, 0, 0, 0, 0};
PYTHON_SCOPE_PRE char *GEN_NAME(fast_print_paras_str)[] = {                                     // 
    MACRO_PARAS_ENUM_OPT(STR_PARAS, GET_TEMP_VARS(__VA_ARGS__))};                               // PYTHON_SCOPE_PRE char *fast_print_paras_str37[] = {
                                                                                                //     "temp_37_5",
                                                                                                //     "temp_37_4",
                                                                                                //     "temp_37_3",
                                                                                                //     "temp_37_2",
                                                                                                //     "temp_37_1",
                                                                                                // };
GET_STRUCT_VAR_MEMBER(frame, global_cnt) = SET_COUNTER(__COUNTER__);                            // gen_frame37.global_cnt = 0 + 1; ;
GET_STRUCT_VAR_MEMBER(frame, GET_STRUCT_AUTO_VAR()) = GET_STRUCT_AUTO_VAR();                    // gen_frame37.gen_auto37 = gen_auto37;
console_output_bin((char *)&GET_STRUCT_VAR(frame), sizeof(GET_STRUCT_TYPE(frame)));             // console_output_bin((char *)&gen_frame37, sizeof(struct gen_frame37_stru));
```

result.

```c
do
{
    static typeof(0) temp_37_5;
    static typeof(1) temp_37_4;
    static typeof(2) temp_37_3;
    static typeof(3) temp_37_2;
    static typeof(4) temp_37_1;
    ;
    struct gen_auto37_stru
    {
        typeof(temp_37_5) temp_37_5;
        typeof(temp_37_4) temp_37_4;
        typeof(temp_37_3) temp_37_3;
        typeof(temp_37_2) temp_37_2;
        typeof(temp_37_1) temp_37_1;
    } __attribute__((__packed__)) gen_auto37;
    temp_37_5 = 0;
    temp_37_4 = 1;
    temp_37_3 = 2;
    temp_37_2 = 3;
    temp_37_1 = 4;
    gen_auto37.temp_37_5 = temp_37_5;
    gen_auto37.temp_37_4 = temp_37_4;
    gen_auto37.temp_37_3 = temp_37_3;
    gen_auto37.temp_37_2 = temp_37_2;
    gen_auto37.temp_37_1 = temp_37_1;
    struct gen_frame37_stru
    {
        unsigned char preamble;
        unsigned char len;
        unsigned short global_cnt;
        struct gen_auto37_stru gen_auto37;
        unsigned char tail;
    } __attribute__((__packed__)) gen_frame37;
    gen_frame37.preamble = (0xAA & 0xFC) | (0x00);
    gen_frame37.len = sizeof(struct gen_frame37_stru) - 2;
    gen_frame37.tail = 0x55;
    PYTHON_SCOPE_PRE char *fast_print_str37 = "hello %d %d %d %d %d";
    PYTHON_SCOPE_PRE int sizeof_stru_member37[] = {(sizeof(0)), (sizeof(1)), (sizeof(2)), (sizeof(3)), (sizeof(4)), (0), (0), (0), (0), (0)};
    PYTHON_SCOPE_PRE int typeof_stru_member37[] = {(__builtin_types_compatible_p(typeof(0), char) ? 1 : 
                                        (__builtin_types_compatible_p(typeof(0), unsigned char) ? 2 : 
                                        (__builtin_types_compatible_p(typeof(0), short) ? 3 : 
                                        (__builtin_types_compatible_p(typeof(0), unsigned short) ? 4 : 
                                        (__builtin_types_compatible_p(typeof(0), int) ? 5 : 
                                        (__builtin_types_compatible_p(typeof(0), unsigned int) ? 6 : 
                                        (__builtin_types_compatible_p(typeof(0), float) ? 7 : 
                                        (__builtin_types_compatible_p(typeof(0), double) ? 8 : 
                                        (__builtin_types_compatible_p(typeof(0), typeof(const char *)) ? 9 : 0))))))))), 
                                        (__builtin_types_compatible_p(typeof(1), char) ? 1 : 
                                        (__builtin_types_compatible_p(typeof(1), unsigned char) ? 2 : 
                                        (__builtin_types_compatible_p(typeof(1), short) ? 3 : 
                                        (__builtin_types_compatible_p(typeof(1), unsigned short) ? 4 : 
                                        (__builtin_types_compatible_p(typeof(1), int) ? 5 : 
                                        (__builtin_types_compatible_p(typeof(1), unsigned int) ? 6 : 
                                        (__builtin_types_compatible_p(typeof(1), float) ? 7 : 
                                        (__builtin_types_compatible_p(typeof(1), double) ? 8 : 
                                        (__builtin_types_compatible_p(typeof(1), typeof(const char *)) ? 9 : 0))))))))), 
                                        (__builtin_types_compatible_p(typeof(2), char) ? 1 : 
                                        (__builtin_types_compatible_p(typeof(2), unsigned char) ? 2 : 
                                        (__builtin_types_compatible_p(typeof(2), short) ? 3 : 
                                        (__builtin_types_compatible_p(typeof(2), unsigned short) ? 4 : 
                                        (__builtin_types_compatible_p(typeof(2), int) ? 5 : 
                                        (__builtin_types_compatible_p(typeof(2), unsigned int) ? 6 : 
                                        (__builtin_types_compatible_p(typeof(2), float) ? 7 : 
                                        (__builtin_types_compatible_p(typeof(2), double) ? 8 : 
                                        (__builtin_types_compatible_p(typeof(2), typeof(const char *)) ? 9 : 0))))))))), 
                                        (__builtin_types_compatible_p(typeof(3), char) ? 1 : 
                                        (__builtin_types_compatible_p(typeof(3), unsigned char) ? 2 : 
                                        (__builtin_types_compatible_p(typeof(3), short) ? 3 : 
                                        (__builtin_types_compatible_p(typeof(3), unsigned short) ? 4 : 
                                        (__builtin_types_compatible_p(typeof(3), int) ? 5 : 
                                        (__builtin_types_compatible_p(typeof(3), unsigned int) ? 6 : 
                                        (__builtin_types_compatible_p(typeof(3), float) ? 7 : 
                                        (__builtin_types_compatible_p(typeof(3), double) ? 8 : 
                                        (__builtin_types_compatible_p(typeof(3), typeof(const char *)) ? 9 : 0))))))))), 
                                        (__builtin_types_compatible_p(typeof(4), char) ? 1 : 
                                        (__builtin_types_compatible_p(typeof(4), unsigned char) ? 2 : 
                                        (__builtin_types_compatible_p(typeof(4), short) ? 3 : 
                                        (__builtin_types_compatible_p(typeof(4), unsigned short) ? 4 : 
                                        (__builtin_types_compatible_p(typeof(4), int) ? 5 : 
                                        (__builtin_types_compatible_p(typeof(4), unsigned int) ? 6 : 
                                        (__builtin_types_compatible_p(typeof(4), float) ? 7 : 
                                        (__builtin_types_compatible_p(typeof(4), double) ? 8 : 
                                        (__builtin_types_compatible_p(typeof(4), typeof(const char *)) ? 9 : 0))))))))), 0, 0, 0, 0, 0};
    PYTHON_SCOPE_PRE char *fast_print_paras_str37[] = {
        "temp_37_5",
        "temp_37_4",
        "temp_37_3",
        "temp_37_2",
        "temp_37_1",
    };
    gen_frame37.global_cnt = 0 + 1;
    ;
    gen_frame37.gen_auto37 = gen_auto37;
    console_output_bin((char *)&gen_frame37, sizeof(struct gen_frame37_stru));
} while (0);
```

* 使用宏STR_BUF_WRITE_STRU来代替printf以实现调试信息生成功能。在STR_BUF_WRITE_STRU(“Errno %d\n”, 3)中，称“Errno %d\n”为格式字符串，称“3”为调试参数。
* 使用编译器预定义宏__COUNTER__表示当前C文件中该STR_BUF_WRITE_STRU调用处前文的STR_BUF_WRITE_STRU调用数量，STR_BUF_WRITE_STRU通过__COUNTER__以获取该STR_BUF_WRITE_STRU在当前C文件中的有效序号。
* STR_BUF_WRITE_STRU根据STR_BUF_WRITE_STRU地址空间起始地址和有效序号生成整个工程唯一的STR_BUF_WRITE_STRU地址。STR_BUF_WRITE_STRU地址('SET_COUNTER')是一个整数，实现对调试信息的编码，位长可根据工程的需求适应，2字节长的STR_BUF_WRITE_STRU地址空间即包含表示65536个STR_BUF_WRITE_STRU地址。STR_BUF_WRITE_STRU地址空间在下文定义。
* STR_BUF_WRITE_STRU根据调试参数生成结构体类型及变量，结构体成员类型与调试参数类型一一对应，结构体成员按照单字节对齐，并且将结构体变量成员的值一一对应的初始化为调试参数的值。
* 通过STR_BUF_WRITE_STRU获取格式字符串、调试参数类型、调试参数位宽、STR_BUF_WRITE_STRU调用行号、调试参数结构体类大小（以byte为单位）。
* 通过STR_BUF_WRITE_STRU调用调试信息输出驱动，传递STR_BUF_WRITE_STRU地址、调试参数结构体变量起始地址、调试参数结构体类大小三个参数来输出STR_BUF_WRITE_STRU地址值、调试参数值。
* 使用预定义的宏控制 STR_BUF_WRITE_STRU 在工程预处理和工程编译过程中预处理阶段展开的结果，下文使用PYTHON_SCOPE_PRE表示该宏。

## 工程预处理

* 使用脚本遍历整个工程，记录全部直接调用STR_BUF_WRITE_STRU的C文件，及每个文件中STR_BUF_WRITE_STRU出现的次数。
* 根据记录的文件及文件中STR_BUF_WRITE_STRU出现的次数，为每个文件分配一段STR_BUF_WRITE_STRU地址空间，文件的地址空间长度应不小于文件中STR_BUF_WRITE_STRU出现的次数，且任意文件地址空间之间没有覆盖。STR_BUF_WRITE_STRU地址空间起始地址在编译时以预定义宏的方式传入文件中，下文中以FILE_ADDR表示。
* 使用预处理器对记录的C文件进行预处理，生成预处理文件。对每个C文件进行预处理时，将相应的FILE_ADDR和'PYTHON_SCOPE_PRE=static'作为预定义宏传入文件。对应每个STR_BUF_WRITE_STRU调用处，都生成全局的调试参数结构体类及变量，及全局的包含格式字符串、调试参数类型、调试参数位宽信息的变量，还有全局的包含调用行号、STR_BUF_WRITE_STRU地址信息的变量。使用脚本对预处理生成的文件进行分析，获取STR_BUF_WRITE_STRU调用行号、STR_BUF_WRITE_STRU地址、格式字符串信息。
* 使用clang对预处理文件编译生成LLVM IR文件。使用软件工具对LLVM IR文件进行分析，获取STR_BUF_WRITE_STRU格式字符串、调试参数类型、调试参数位宽信息。
* 根据获取STR_BUF_WRITE_STRU的调用行、STR_BUF_WRITE_STRU地址、格式字符串、调试参数类型、调试参数位宽信息，生成STR_BUF_WRITE_STRU调用记录文件。文件以STR_BUF_WRITE_STRU地址为索引包含STR_BUF_WRITE_STRU的调用行号、格式字符串、调试参数类型、调试参数位宽及所在文件路径。

## 工程编译

对记录的包含STR_BUF_WRITE_STRU的C文件，编译的时候将FILE_ADDR和'__PYTHON_SCOPE_PRE='作为预定义宏传入每个文件。对应每个STR_BUF_WRITE_STRU调用处，生成局部的包含调试参数结构体类及变量、调试参数结构体类大小、STR_BUF_WRITE_STRU地址的变量以及调试信息输出驱动调用代码。

## 程序运行

程序运行前，在PC端运行服务程序及终端显示程序。服务程序用于打开PC端连接的调试接口，并通过TCP端口连接终端显示程序。终端显示程序可用Putty。服务程序读取STR_BUF_WRITE_STRU调用记录文件中的信息，等待调试接口输入。
程序运行到STR_BUF_WRITE_STRU调用处时，STR_BUF_WRITE_STRU地址值和调试参数值被输出接口以字节流的形式输出。
服务程序收到字节流后对字节流进行分析，先获取STR_BUF_WRITE_STRU地址，检索STR_BUF_WRITE_STRU调用记录信息，查找出相应记录，然后按照记录中的格式字符串、调试参数类型、调试参数位宽信息将调试参数数据转化为调试信息，并输出到终端显示程序中。

