# tarsier-test-job



## 新成员入门

如果你是刚刚加入的新人，那么我们计划通过让所有的新人都跑一遍构建和回归测试来快速入门。

### 1. PLCT内部项目简介

| 项目名              | 简介                                 | 说明                                                         |
| ------------------- | ------------------------------------ | ------------------------------------------------------------ |
| riscv-gnu-toolchain | C/C++/Fortran/Rust等语言的编译工具链 | PLCT目前维护的是gcc和binutils两个子模块；按照子指令集扩展需求，又分为zfinx、 B、 K、 P 、zce  5个扩展分支在github开发维护 |
| llvm                | C/C++/Fortran/Rust等语言的编译工具链 |                                                              |
| openJDK             | Java编译工具                         |                                                              |
| V8                  | Javascript                           |                                                              |
| ……                  |                                      |                                                              |



### 2. 软件的构建和测试

#### （1）riscv-gnu-toolchain（下简称gnu）

   1. 搭建测试环境

      - 环境要求：

        ```
        硬件环境：x86-64电脑
        
        软件环境：ubuntu20.04-64（因为目前我们常用的是ubuntu20.04-64和ubuntu18.04-64，建议新入门同学如果是重新搭建环境，建议运行环境保持一致，便于沟通.但是实际上，Fedora、Debian、Gentoo、ArchLinux等其它linux操作系统都是可以的。后续未特别说明，默认都是64位系统）
        
        网络环境：能够非常稳定流畅的访问github
        
        存储空间：大于100G（gnu源码包8G，单次构建超14G；llvm更大源码80G左右？，总之存储空间大点好）
        ```

      

      - 环境搭建方式：

        - （1）在自己的电脑设备上搭建运行环境（测试环境）——》刚入职实习生（前4周）

        - （2）在PLCT服务器上找@wuwei申请服务器——》正式员工/实习生

          

      - 环境搭建主要内容

        1. ubuntu20.04的搭建

        2. 窗口会话分离的tmux会话

        3. 新建一个普通用户并授权sudo

        4. 最后切换到普通用户下执行本文档的测试操作

           

      - 常见的问题和解决办法

        - 国内网络从github下载源码困难

          解决思路：

          ```
          1. 不用git clone的方法下载第一份源码；而是从镜像网址下载历史版本的tar压缩包;镜像地址：https://mirror.iscas.ac.cn/plct/
          2. 将压缩包解压的gnu的源设置为riscv-gnu-toolchain
          3. 从riscv-gnu-toolchain源上获取最新的更新
          ```

          命令：

          ```
          如果从mirror上下载riscv-gnu-toolchain，需要fetch和merge：
          ​```shell
           $wget https://mirror.iscas.ac.cn/plct/riscv-gnu-toolchain.20210207.tbz
           $tar xjvf riscv-gnu-toolchain.20210207.tbz
           $cd riscv-gnu-toolchain
           $git fetch origin master
           $git merge origin/master
           $git submodule update --init --recursive
          ​```
          ```

          

      - 环境搭建参考文档

        - 本地环境搭建
        - [服务器docker上环境搭建](https://github.com/xijing21/tarsier-test-job/blob/main/gnu/test_environment-docker.md)

      

   2. 按照测试操作单完成gnu的构建和测试用例的运行

      - 测试步骤

        1. 准备测试环境

        2. 按照测试操作单执行测试操作，并记录测试过程日志

        3. 将运行测试用例的结果填写并反馈

           

      - 测试操作单

           1. B 分支：[gnu-b-rv64测试操作单](https://github.com/xijing21/tarsier-test-job/blob/main/gnu/gnu-regression-b_rv64_newlib.md)

           2.  K分支：

           3.  P 分支：

           4. zfinx 分支:

           5. zce  分支：开发中，暂未达到可测阶段

              

        **不同分支，除了代码源获取地址、configure参数不一样，环境要求、操作步骤一样，参考b分支代码执行。不同分支的源码地址、configure参数，为了更加简洁的展示，请在gnu测试用例文档[gnu-testcases](https://github.com/xijing21/tarsier-test-job/blob/main/gnu/gnu-testcases.xlsx)文档中查看并执行。**

        

      - 测试报告

        - [gnu回归测试测试报告](https://github.com/xijing21/tarsier-test-job/blob/main/gnu/gnu-regression-b_rv64_newlib-report.docx) ：可以参考[例子](https://github.com/xijing21/tarsier-test-job/blob/main/gnu/gnu-regression-b_rv64_newlib-report-demo.docx)

        

#### （2）LLVM

​	新手入门级测试用例文档待完成。

​	当前可参考xiaoou的系列文档：https://github.com/mollybuild/RISCV-Measurement

```
1. 构建RISCV LLVM并运行回归测试

https://github.com/mollybuild/RISCV-Measurement/blob/master/Build-RISCV-LLVM-and-run-testsuite.md
```



#### （3）openJDK

​	新手入门级测试用例文档待完成。

​	当前可参考xiaoou的系列文档：https://github.com/mollybuild/RISCV-Measurement

```
【Java on RISC-V】交叉编译OpenJDK11 for RV32G（ZERO VM）

https://zhuanlan.zhihu.com/p/344502147
```



### 3. 建议的学习方法

1. 第一阶段：新人加入后，可以先按照上述的构建和测试文档，依次将gnu、llvm、openjdk构建、测试运行一遍；有个整体的了解和印象。执行过程中，请保持良好的记录习惯，建议以文档、博客、专栏等任意公开文档方式记录执行经过和遇到的问题。

2. 第二阶段：逐个项目自行去寻找[第一阶段]中自己遇到的问题，弄明白每一步操作在做什么。

3. 第三阶段：根据安排，或者自己的兴趣，在某个项目（gnu、llvm、openjdk、v8等）、某个方向（研发、测试、文档、翻译、宣传）等各方面找到自己的目标和定位。

   > 目前各项目都在研发中，很多项目处于初级阶段，后续甚至会有新的项目启动。我们需要很多人才一起来进行贡献。我们会结合个人的发展规划、兴趣及能力综合考虑，给予展示各自才华和能力的机会。



## 测试人员等级简介

关于测试，我们有不同等级的测试人员：

- 高级：

  - 能够根据需求设计和编写测试用例来验证功能是否达到质量目标。

    > 编译工具不像上层软件，这里的测试用例需要编程能力+编译理论基础，目前是由开发人员完成的。当你做到这一步，那么以为这能力匹配开发甚至超越一般开发人员。

  - 自动化测试、DevOps、性能测试等能力的测试人员。

    > 能够写自动化测试脚本、能够构建或者完善自动化测试环境、能够进行性能测试和评测的人员。
    >
    > 能力更多的是脚本编译能力、测试工具、测试方法使用能力等；

    

- 中级

  - 能够在新的硬件设备上（没有文档的）很好的执行环境搭建、安装、测试等。并输出文档。

  - 能够修改、维护高级测试编写的脚本等；能够写一些测试用例。

  

- 初级

  按照测试用例，测试文档，测试流程执行测试、管理bug。



当你具备测试中级水平，就可以根据自己的发展规划和兴趣去沟通和协商后续工作重心了。

欢迎大家推荐自己的同学和朋友加入PLCT。加入第一步：准备好简历发送到wuwei2016@iscas.ac.cn 或 xijing@nj.iscas.ac.cn 。

