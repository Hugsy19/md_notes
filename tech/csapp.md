### 1. 计算机系统漫游

处理器的指令集架构和微体系架构：

- 指令集架构描述每条机器代码指令的效果
- 微体系架构描述处理器实际的实现方式

几类存储器：

- 主存是由一组动态随机存取存储器（DRAM）组成
- 高速缓存是用静态随机访问存储器（SRAM）的硬件技术实现

OS是应用程序与硬件之间插入的一层软件：

- 进程是OS对一个正在运行程序的抽象，是对处理器、主存、I/O设备的抽象
- 虚拟内存为进程提供虚拟地址空间，使每个进程看到的内存都一致，是对主存和磁盘I/O设备的抽象
- 文件就是字节序列，是对I/O设备的抽象

并发、并行和超线程：

- 并发（concurrency）指的是一个同时具有多个活动的系统
- 并行（parallelism）指的是用并发来使一个系统运行得更快
- 超线程又称同时多线程，是一项允许一个CPU执行多个控制流的技术

### 2. 信息的表示和处理

字节、字长、字节序：

- 大部分计算机都以8位（bits）的块作为最小的可寻址内存单位，称为一个字节（byte），在十六进制表示法下，一个字节的值域为`00-FF`

- 计算机的字长（word size）用来指明指针数据的的标称大小，字长的大小决定了虚拟空间地址的大小

- 很多对象都由多个字节组成，存储时要考虑它们的排列顺序，由此有了小端法（little endian）和大端法（big endian）两种规则：

  - 小端法：更低的有效位存放在更小的内存地址上
  - 大段法：更高的有效位存放在更高的内存地址上

- 当前大部分流行的OS都采用的小端模式，以下C代码可以打印各数据的字节表示：

  ```c
  #include <stdio.h>
  
  typedef unsigned char *byte_pointer;
  
  void show_bytes(byte_pointer start, int len) {
      size_t i;
      for (i = 0; i < len; ++i)
      	printf(" %.2x", start[i]);
      printf("\n");
  }
  
  int main() {
      int ival = 12345;
      float fval = (float) ival;
      show_bytes((byte_pointer) &ival, sizeof(ival));
      // Windows/Linux: 39 30 00 00 
      show_bytes((byte_pointer) &fval, sizeof(fval));
      // Windows/Linux: 00 e4 40 46
  }
  ```

- 文本数据比二进制数据更具有平台独立性，在使用ASCII码作为字符码的系统上都会有相同的结果
