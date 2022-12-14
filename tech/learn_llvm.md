### 构建

* `cmake -G Ninja/Unix Makefiles -D DLLVM_ENABLE_PROJECTS=clang -DCMAKE_BUILD_TYPE=RELEASE -DLLVM_TARGETS_TO_BUILD=X86 -DCMAKE_INSTALL_PREFIX=path/to/install path/to/llvm`

* CMake变量：
  
  * CMAKE_C/CXX_COMPILER：默认值为环境变量CC/CXX
  
  * CMAKE_C_FLAGS/CMAKE_FLAGS：默认值为环境变量CFLAGS/CXXFLAGS
  
  * CMAKE_INSTALL_PREFIX：安装路径，unix上默认为/usr/local
  
  * CMAKE_BUILD_TYPE：构建类型
    
    * DEBUG：含调试符号
    
    * RELEASE：发布版本
    
    * RELWITHDEBINFO：含调试符号的发布版本
    
    * MINSIZEREL：优化文件大小的发布版本

* LLVM变量：
  
  * LLVM_TARGETS_TO_BUILD：构建目标，区分大小写，多个要以”；“分隔
  
  * LLVM_ENABLE_PROJECTS：项目列表，不区分大小写，项目源码必须与llvm目录在同一级
  
  * LLVM_ENABLE_ASSERTTIONS：开启断言检查
  
  * LLVM_APPEND_VC_REV：工具中加入版本信息

### 简介

* 编译器和工具
  
  * Clang编译器：源码位于clang目录，提供了一组可对C/C++源码进行词法分析、解析、语义分析并生成LLVM IR的库，clang只是基于这些库的编译器驱动程序
  
  * lld链接器：源码位于lld目录，支持多种格式
  
  * lldb调试器：源码位于lldb目录，类似gdb
  
  * C/C++其他工具：源码位于clang-tools-extra目录，如代码静态检查工具clang-tidy

* LLVM核心库：源码位于llvm目录，为主流CPU提供了一组带优化器、代码生成器的库以及相关工具：
  
  * llc：以IR文件作为输入，输出对应的汇编或二进制
  
  * bugpoint：查找导致编译器崩溃的最小测试用例

* 运行时库：源码都位于同名目录
  
  * compiler-rt：为硬件不支持的低级功能提供特定于目标的支持，如为i386机器提供64位除法、各sanitizer、模糊及分析库
  
  * libunwind：提供基于DWARF标准的堆栈展开帮助函数，这些函数通常用于C++等语言的异常处理，由C编写
  
  * libcxxabi：在libunwind上实现C++的异常处理，并为其提供标准C++函数
  
  * libcxx：C++标准库的实现，另外pstl提供了并行版本的STL算法
  
  * libclc：OpenCL的运行时库，OpenCL是异构并行计算的标准，以助于将计算任务转移到GPU上
  
  * libc：提供完整的C库
  
  * openmp：提供OpemMP API的支持，OpenMP可助于多线程编程

* 项目结构：
  
  * 各项目都以CMakeLists.txt来描述项目的构建，额外的cmake模块或支持文件放到cmake子目录，现成的模块则放到cmake/modules
  
  * 库的头文件放在include，源文件放在lib，lib目录下又分为各种库的目录
  
  * 应用程序的源码放在tools和utils，前者包含最终面向用户的程序，后者则一般是编译或测试期间使用的内部应用程序
  
  * unittest目录是以Google Test作为框架的单元测试套，主要用于测试单个函数及一些无法通过额外方式测试的独立功能
  
  * test目录是以LIT作为框架的回归测试套，主要用于测试特定特性或触发特定bug 的小段代码，用例所使用的语言与所测试的llvm部分有关，但都由llvm-lit来执行

* 新建项目
  
  - 作为附加项目与LLVM一起构建：`-DLLV M_EXTERNAL_PROJECTS=project_name -DLLVM_EXTERNAL_TINYLANG_SOURCE_DIR=path/to/project`
  
  - 作为独立项目构建：`-DLLVM_DIR=path/to/llvm/lib/cmake/llvm`
  
  - 项目不使用CMake进行组织，查找LLVM相关组件的库名：`llvm-config --libs modules_name`
- 交叉编译
  
  - 为不同的CPU体系结构生成代码、创建对应的可执行文件的过程
  
  - 目标系统的名字一般由“三元组表达式”来表示：CPU架构-供应商-OS，如`x86_64_pc_win32`、`aarch64-unknown-linux-gnu`，后者因没有获取到linux版本的发行商而显示unkonwn
  
  - 交叉编译需要借助`llvm-tblgen`：`-DCMAKE_CROSSCOMPILING=True \
    -DLLVM_TABLEGEN=path/to/llvm-tblgen \
    -DLLVM_DEFAULT_TARGET_TRIPLE=aarch64-linux-gnu \
    -DLLVM_TARGET_ARCH=AArch64 \
    -DLLVM_TARGETS_TO_BUILD=AArch64 \
    -DCMAKE_C_COMPILER=aarch64-linux-gnu-gcc -DCMAKE_CXX_COMPILER=aarch64-linux-gnu-g++ \`

### 编译器结构

- 
