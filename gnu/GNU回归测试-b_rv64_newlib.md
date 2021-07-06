# GNU回归测试-b_rv64_newlib

## 1. 测试说明

### 测试目的

通过前后两次(代码提交前和代码提交后)回归测试结果的比对：

1. 验证之前版本产生的所有缺陷已全部被修复；

2. 确认修复这些缺陷没有引发新的缺陷。

   

## 测试对象

github上，riscv-gnu-toolchain项目 B扩展对应的代码的回归测试。

代码源：

riscv-gnu-toolchain ：https://github.com/riscv/riscv-gnu-toolchain

riscv-gcc：https://github.com/pz9115/riscv-gcc/tree/riscv-gcc-10.2.0-rvb

riscv-binutils-gdb：https://github.com/pz9115/riscv-binutils-gdb/tree/riscv-binutils-experiment

> 说明：riscv-gcc和riscv-binutils-gdb的git地址可能会变更或过时。在测试前需要跟测试主管确认并更换为测试任务单中的测试地址。



## 测试环境要求

### 环境要求

硬件环境：x86-64电脑

软件环境：ubuntu20.04-64

网络环境：能够非常稳定流畅的访问github

存储空间：大于50G（源码包： 8G  单次构建：14G）



### 测试准备

#### 1、docker+ubuntu20.04准备

参考文档《搭建docker测试环境》，包含：

1. ubuntu20.04的搭建
2. 窗口会话分离的tmux会话
3. 新建一个普通用户并授权sudo
4. 最后切换到普通用户下执行本文档的测试操作



## 预期结果

本次回归测试的测试结果与上一次回归测试对比，没有产生新的缺陷。



## 测试步骤

### 准备工作

#### 0、请按照【测试准备】步骤做好环境准备工作

#### 1、连接docker服务器

1. 远程连接服务器

   ```
   C:\Users\cz>ssh p9
   Welcome to Ubuntu 18.04.5 LTS (GNU/Linux 4.15.0-144-generic x86_64)
   
    * Documentation:  https://help.ubuntu.com
    * Management:     https://landscape.canonical.com
    * Support:        https://ubuntu.com/advantage
   New release '20.04.2 LTS' available.
   Run 'do-release-upgrade' to upgrade to it.
   
   
   Welcome to Alibaba Cloud Elastic Compute Service !
   
   Last login: Tue Jul  6 10:38:25 2021 from 111.196.246.16
   xijing@p9-plct:~$
   ```

   请注意$符号前面的变化：用户名@机器人名

   

3. 查看所有的tmux会话

   ```
   # 显示所有会话
   xijing@p9-plct:~$ tmux ls
   xj: 1 windows (created Mon Jun  7 10:06:04 2021) [157x85]
   ```

   

4.  连接之前建立的会话

   ```
   xijing@p9-plct:~$ xijing@p9-plct:~$ tmux a -t xj
   #成功执行后，会进入到会话窗口（会话窗口最下面左下角显示[xj] 0:docker*   如下图绿色部分所示）
   ```

   

4. 检查当前用户是否为自定义用户

   检查命令行$符号前的用户和主机名信息，是否是自定义的用户名（**切记所有测试操作不要在root用户下执行**）。

   如：xj@6d592fc325bf:/$

   从规则上看，就能看出目前是在docker容器ubuntu系统下的xj用户下。

   

#### 2、预装软件包

1. 普通用户下，sudo安装gnu测试工作所依赖的软件包。

   ```
   xj@6d592fc325bf:/$ sudo apt update
   
   xj@6d592fc325bf:/$ sudo apt upgrade
   
   xj@6d592fc325bf:/$ sudo apt install autoconf automake autotools-dev curl python3 libmpc-dev libmpfr-dev libgmp-dev gawk build-essential bison flex texinfo gperf libtool patchutils bc zlib1g-dev libexpat-dev cmake ninja-build pkg-config libglib2.0-dev libpixman-1-dev python git libfdt-dev libncurses5-dev libncursesw5-dev device-tree-compiler
   ```



2. 创建测试工作目录

   ```
   # 切换到home目录下
   xj@6d592fc325bf:/$ cd

   # 在home目录下创建新目录（如果是在/目录下创建新目录，会有权限问题）
   xj@e2ba8bd04169:~$ mkdir RISCV
   xj@e2ba8bd04169:~$  cd RISCV
   xj@e2ba8bd04169:~/RISCV$ 
   ```



### 配置环境变量

根据规划的目录，配置环境变量：

   ```
# 配置环境变量
xj@e2ba8bd04169:~$ vim ~/.bashrc
----在文件开头加入以下两行-------------
export RISCV=~/RISCV
export PATH=$RISCV/b_rv64_newlib/bin/:$PATH


# 使得环境变量生效
xj@e2ba8bd04169:~$ source ~/.bashrc

# 检查是否生效
xj@e2ba8bd04169:~$ echo $PATH
/home/xijing/RISCV/b_rv64_newlib/bin/:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin
xj@e2ba8bd04169:~$ echo $RISCV
/home/xijing/RISCV
   ```



### 下载源码并构建

#### 3、下载riscv-gnu-toolchain代码，更新gcc、binutils-gdb子模块的B扩展代码（待测目标代码），并记录版本号

1. 下载riscv-gnu-toolchain总代码包，并查询记录其版本信息

```
   # git clone --recursive 用于循环克隆git子项目 
   xj@e2ba8bd04169:~/RISCV$  git clone --recursive https://github.com/riscv/riscv-gnu-toolchain.git

   # 查看并输出commmitid版本信息
   xj@e2ba8bd04169:~/RISCV$ cd riscv-gnu-toolchain
   xj@e2ba8bd04169:~/RISCV/riscv-gnu-toolchain$ git rev-parse HEAD 

```

   

2. 将代码拷贝一份重命名为b-ext（可选操作，当计划在同一台设备上测试多个分支时避免混淆）

   ```
   xj@e2ba8bd04169:~/RISCV/riscv-gnu-toolchain$ cd ..
   xj@e2ba8bd04169:~/RISCV$ cp riscv-gnu-toolchain b-ext
   xj@e2ba8bd04169:~/RISCV$ cd b-ext
   xj@e2ba8bd04169:~/RISCV/b-ext$ 
   ```

   

3. 更新gcc子模块的代码为待测试代码

   ```
   xj@e2ba8bd04169:~/RISCV/b-ext$ cd riscv-gcc

   # 添加一个远程仓库pz9115
   xj@e2ba8bd04169:~/RISCV/b-ext/riscv-gcc$ git remote add pz9115 https://github.com/pz9115/riscv-gcc.git

   # 将远程仓库pz9115的最新内容拉到本地
   xj@e2ba8bd04169:~/RISCV/b-ext/riscv-gcc$ git fetch pz9115
   remote: Enumerating objects: 1011, done.
   remote: Counting objects: 100% (999/999), done.
   remote: Compressing objects: 100% (303/303), done.
   remote: Total 1011 (delta 859), reused 778 (delta 680), pack-reused 12
   Receiving objects: 100% (1011/1011), 410.93 KiB | 5.96 MiB/s, done.
   Resolving deltas: 100% (859/859), completed with 39 local objects.
   From https://github.com/pz9115/riscv-gcc
    * [new branch]              BK                         -> pz9115/BK
    * [new branch]              k-dev                      -> pz9115/k-dev
    * [new branch]              p-ext-andes                -> pz9115/p-ext-andes
    * [new branch]              q-ext                      -> pz9115/q-ext
    * [new branch]              riscv-gcc-10.1.0-rvv       -> pz9115/riscv-gcc-10.1.0-rvv
    * [new branch]              riscv-gcc-10.1.0-rvv-zfh   -> pz9115/riscv-gcc-10.1.0-rvv-zfh
    * [new branch]              riscv-gcc-10.2.0-rvb       -> pz9115/riscv-gcc-10.2.0-rvb
    * [new branch]              riscv-gcc-10.2.0-zfinx     -> pz9115/riscv-gcc-10.2.0-zfinx
    * [new branch]              riscv-gcc-experiment-p-ext -> pz9115/riscv-gcc-experiment-p-ext
    
   #  checkout代码
   xj@e2ba8bd04169:~/RISCV/b-ext/riscv-gcc$ git checkout pz9115/riscv-gcc-10.2.0-rvb
   Previous HEAD position was 03cb20e5433 Update 2 C++ coroutine testcases from upstream.
   HEAD is now at fcd1b5d046c Add testcases with for zbb zbe zbp

   # 查询riscv-gcc代码版本
   xj@e2ba8bd04169:~/RISCV/b-ext/riscv-gcc$ git rev-parse HEAD                                                          
   fcd1b5d046c6fb087a2dff83ca4cda3d28f10ed1    
   ```

   

4. 更新riscv-binutils代码包：

   ```
   xj@e2ba8bd04169:~/RISCV/b-ext/riscv-gcc$ cd ../riscv-binutils/

   # 添加一个远程仓库pz9115
   xj@e2ba8bd04169:~/RISCV/b-ext/riscv-binutils$ git remote add pz9115 https://github.com/pz9115/riscv-binutils-gdb.git

   # 将远程仓库pz9115的最新内容拉到本地
   xj@e2ba8bd04169:~/RISCV/b-ext/riscv-binutils$  git fetch pz9115
   remote: Enumerating objects: 408, done.
   remote: Counting objects: 100% (393/393), done.
   remote: Compressing objects: 100% (129/129), done.
   remote: Total 408 (delta 307), reused 323 (delta 264), pack-reused 15
   Receiving objects: 100% (408/408), 481.07 KiB | 6.17 MiB/s, done.
   Resolving deltas: 100% (307/307), completed with 24 local objects.
   From https://github.com/pz9115/riscv-binutils-gdb
    * [new branch]            BK                              -> pz9115/BK
    * [new branch]            b-dev                           -> pz9115/b-dev
    * [new branch]            riscv-binutils-2.35-zfinx       -> pz9115/riscv-binutils-2.35-zfinx
    * [new branch]            riscv-binutils-2.36-k-ext       -> pz9115/riscv-binutils-2.36-k-ext
    * [new branch]            riscv-binutils-experiment       -> pz9115/riscv-binutils-experiment
    * [new branch]            riscv-binutils-experiment-p-ext -> pz9115/riscv-binutils-experiment-p-ext
    
   #  checkout代码
   xj@e2ba8bd04169:~/RISCV/b-ext/riscv-binutils$ git checkout pz9115/riscv-binutils-experiment
   Previous HEAD position was f35674005e This is 2.36.1 release
   HEAD is now at d52e3ccf96 Fix the indents problems and update INSN_ALIAS FLAG

   # 查询并获取riscv-binutils版本信息
   xj@e2ba8bd04169:~/RISCV/b-ext/riscv-binutils$ git rev-parse HEAD                                                     
   d52e3ccf969016bd9db01a7e58e7902456d2c9e5      
   ```

   

#### 4、构建riscv-gnu-toolchain：newlib+rv64

1. 创建构建文件目录

   ```
   xj@e2ba8bd04169:~/RISCV/b-ext/riscv-binutils$ cd ..
   xj@e2ba8bd04169:~/RISCV/b-ext$ mkdir build_rv64_newlib
   xj@e2ba8bd04169:~/RISCV/b-ext$ cd build_rv64_newlib
   ```

   

2. 用newlib库构建riscv64的目标文件

   ```
   # configure
   xj@e2ba8bd04169:~/RISCV/b-ext/build_rv64_newlib$ ../configure --prefix=$RISCV/b_rv64_newlib/ --with-arch=rv64gc_zba_zbb_zbc_zbe_zbf_zbm_zbp_zbr_zbs_zbt --with-abi=lp64 --with-multilib-generator="rv64gc_zba_zbb_zbc_zbe_zbf_zbm_zbp_zbr_zbs_zbt-lp64--"
   checking for gcc... gcc
   checking whether the C compiler works... yes
   checking for C compiler default output file name... a.out
   checking for suffix of executables...
   checking whether we are cross compiling... no
   checking for suffix of object files... o
   checking whether we are using the GNU C compiler... yes
   checking whether gcc accepts -g... yes
   checking for gcc option to accept ISO C89... none needed
   checking for grep that handles long lines and -e... /usr/bin/grep
   checking for fgrep... /usr/bin/grep -F
   checking for grep that handles long lines and -e... (cached) /usr/bin/grep
   checking for bash... /bin/bash
   checking for __gmpz_init in -lgmp... yes
   checking for mpfr_init in -lmpfr... yes
   checking for mpc_init2 in -lmpc... yes
   checking for curl... /usr/bin/curl
   checking for wget... /usr/bin/wget
   checking for ftp... /usr/bin/ftp
   configure: creating ./config.status
   config.status: creating Makefile
   config.status: creating scripts/wrapper/awk/awk
   config.status: creating scripts/wrapper/sed/sed

   # 构建并将构建过程记录到log文档中
   xj@e2ba8bd04169:~/RISCV/b-ext/build_rv64_newlib$ make 2>&1|tee b_rv64_newlib-build-20210701.log
   ```

   

### 运行测试用例测试

#### 5、运行回归测试用例进行测试并输出测试结果到日志

3. 运行gcc回归测试用例进行测试

   ```
   xj@e2ba8bd04169:~/RISCV/b-ext/build_rv64_newlib$ make report-gcc-newlib 2>&1|tee b_rv64_newlib-reportgccnewlib-20210701.log

   ```

   

4. 查看gcc回归测试结果

   当测试出现类似的结果显示时，回归测试操作本身就完成了：

   ```
                  ========= Summary of gcc testsuite =========
                               | # of unexpected case / # of unique unexpected case
                               |          gcc |          g++ |     gfortran |
    rv64gc_zba_zbb_zbc_zbe_zbf_zbm_zbp_zbr_zbs_zbt/   lp64/ medlow |18241 /  3421 | 9167 /  2263 |      - |
   make: *** [Makefile:913: report-gcc-newlib] Error 1
   ```

   

5. 运行binutils回归测试用例进行测试

   ```
   xj@e2ba8bd04169:~/RISCV/b-ext/build_rv64_newlib$ make report-binutils-newlib 2>&1|tee b_rv64_newlib-reportbinutilsnewlib-20210701.log
   ```

6. 查看binutils回归测试结果

   当测试出现类似的结果显示时，回归测试操作本身就完成了：

   ```
   		=== ld Summary ===

   # of expected passes		539
   # of unexpected failures	14
   # of expected failures		11
   # of unsupported tests		203

   		=== ld: Unexpected fails for rv64gc_zba_zbb_zbc_zbe_zbf_zbm_zbp_zbr_zbs_zbt lp64 medlow ===
   FAIL: Run pr26391-5
   FAIL: Run pr26391-6

                  ========= Summary of binutils testsuite =========
                               | # of unexpected case
                               |     binutils |           ld |          gas |
    rv64gc_zba_zbb_zbc_zbe_zbf_zbm_zbp_zbr_zbs_zbt/   lp64/ medlow |            0 |            2 |            0 |
   make: *** [Makefile:934: report-binutils-newlib] Error 1
   ```

   

#### 6、查看测试结果

​	如上。看到如上的测试结果输出，则表示回归测试完成。



### 测试结果管理

#### 7、上传测试结果日志

将上述生成的log文件进行上传

暂定义测试结果保存到 https://github.com/xijing21/tarsier-testresult-gnu.git 

   ```
xj@e2ba8bd04169:~/RISCV$ git clone https://github.com/xijing21/tarsier-testresult-gnu.git
xj@e2ba8bd04169:~/RISCV$ cd tarsier-testresult-gnu
xj@e2ba8bd04169:~/RISCV/tarsier-testresult-gnu$ cp ~/RISCV/b-ext/build_rv64_newlib/*-20210701.log  .
xj@e2ba8bd04169:~/RISCV/tarsier-testresult-gnu$ git add .
xj@e2ba8bd04169:~/RISCV/tarsier-testresult-gnu$ git commit -m "b_rv64_newlib-20210701"
xj@e2ba8bd04169:~/RISCV/tarsier-testresult-gnu$ git push
   ```



#### 8、在测试任务单中记录测试结果

在测试任务单中记录测试结果：

测试结果上传的路径：

构建日志：https://github.com/xijing21/tarsier-testresult-gnu/blob/main/b_rv64_newlib-build-20210701.log

gcc回归测试结果：https://github.com/xijing21/tarsier-testresult-gnu/blob/main/b_rv64_newlib-reportgccnewlib-20210701.log

binutils回归测试结果：https://github.com/xijing21/tarsier-testresult-gnu/blob/main/b_rv64_newlib-reportbinutilsnewlib-20210701.log



todo：计划将任务单以y站issue方式进行管理，因此测试的结果就贴到测试任务的issue中去。

#### 9、测试结果分析

在https://github.com/xijing21/tarsier-testresult-gnu/ 中，找到上一次的【b_rv64_newlib-】测试结果，将相邻两次的测试结果贴出，并进行对比分析。

怎么对比和分析：todo



#### 10、bug管理

当对比分析发现了新的bug时，将bug提交到gnu的bug管理系统中去。

怎么判断bug？todo

