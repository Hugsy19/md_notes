### vim



### gdb

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

### cmake

#### find_pacakge

- 查找包（第三方库）并返回其相关细节
- 有两种搜索模式：
  - Module Mode：未指定搜索模式下优先使用的模式，搜索名为`Find<PackageName>.cmake`的文件，搜索路径依次为：
    - 变量`CMAKE_MODULE_PATH`指定的路径
    - Cmake的安装路径`/path/to/cmake/Modules/`
  - Config Mode：搜索名为`<lovercasePackageName>-config.cmake`或`<PackageName>Config.cmake`文件，指定了具体版本时则找对应版本号的文件