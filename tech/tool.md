## gdb

* break/b：设置断点
* run/r：运行
* next/n：下一行
* step/s：单步执行
* print/p：打印
* backtrace/bt：查看调用栈
* watch：设置观察点
* quit/q：退出
* layout：分割窗口，显示源码
* refresh：刷新窗口

## cmake

### 变量

- CMAKE_BINARY_DIR：构建目录的根路径

- CMAKE_SOURCE_DIR：cmake命令行中给出的顶层源目录
- CMAKE_CURRENT_BINARY_DIR：当前CmakeLists.txt所在的构建目录
- CMAKE_CURRENT_SOURCE_DIR：当前CmakeLists.txt所在的源码目录

### 指令

#### find_pacakge(PackageName ...)

- 查找包（第三方库）并返回其相关细节
- 有两种搜索模式：
  - Module Mode：默认的模式，搜索名为`Find<PackageName>.cmake`的文件，搜索路径依次为：
    - 变量`CMAKE_MODULE_PATH`指定的路径
    - Cmake的安装路径`/path/to/cmake/Modules/`
  - Config Mode：备选查找模式，可设置更多搜索路径，搜索名为`<lower-case-package-name>-config.cmake`或`<PackageName>Config.cmake`文件，指定了具体版本时则找对应版本号的文件

#### configure_file(input output ...)

- 解析input中的变量值并复制为output