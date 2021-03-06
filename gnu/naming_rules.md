# 命令规则

为了方便测试管理，对测试过程中涉及的目录、文件名进行命令规则定义和约束。方便测试过程管理和测试结果来管理和后续测试结果分析时的文件检索。



## 分析

针对gnu项目，我们需要管理的信息有：

| 扩展指令集 | 目标操作系统体系架构 | 函数库 | abi二进制应用程序接口 |
| ---------- | -------------------- | ------ | --------------------- |
| B          | rv32                 | newlib |                       |
| K          | rv64                 | linux  |                       |
| P          |                      |        |                       |
| Zfinx      |                      |        |                       |
| Zce        |                      |        |                       |
| V          |                      |        |                       |



我们需要定义的目录或者文件名有：

1. 源码目录：
2. build目录：
3. prefix目录：
4. make日志：
5. gcc回归测试日志：
6. binutils回归测试日志：



## 1. 目录树结构参考

![](images/directory.png)



## 2. 目录命名规则

规则：扩展指令集-----目标操作系统架构-----函数库-----[-abi参数]

举例：b-----rv64-----newlib

|            | b-----rv64-----newlib          | b-----rv32-----linux          |
| ---------- | ------------------------------ | ----------------------------- |
| 源码目录   | $RISCV/b-ext                   | $RISCV/b-ext                  |
| build目录  | $RISCV/b-ext/build_rv64_newlib | $RISCV/b-ext/build_rv32_linux |
| prefix目录 | $RISCV/b_rv64_newlib           | $RISCV/b_rv32_linux           |



## 3. 日志文件命令规则

扩展指令集-----目标操作系统架构-----函数库-----文档类型-----[-abi参数]-----测试日期.log

|                      | b-----rv64-----newlib                           | b-----rv32-----linux                           |
| -------------------- | ----------------------------------------------- | ---------------------------------------------- |
| make日志             | b_rv64_newlib-build-20210701.log                | b_rv32_linux-build-20210701.log                |
| gcc回归测试日志      | b_rv64_newlib-reportgccnewlib-20210701.log      | b_rv32_linux-reportgccnewlib-20210701.log      |
| binutils回归测试日志 | b_rv64_newlib-reportbinutilsnewlib-20210701.log | b_rv32_linux-reportbinutilsnewlib-20210701.log |

linux函数库测试，回归测试命令是什么？



## 4. 测试文档命令规则

项目简称-----测试类型-----扩展指令集-----目标操作系统架构-----函数库

gnu-regression-b_rv64_newlib：gnu回归测试；测试内容b_rv64_newlib；