## GCC总体结构

### 调试选项

选项的形式主要有三类：

- `-ftree-dump-[switch]/-[options]`：输出编译过程中与AST、GIMPLE等树节点相关的调试信息
- `-ftree-ipa-[switch]`：输出IPA相关的调试信息
- `-ftree-rtl-[pass]`：输出RTL中间表示相关调试信息

输出的调试文件名称格式为：`filename.c.[num][r/t].[name]`

- num为编号
- t表示是基于tree的GIMPLE处理过程，r表示是基于RTL的处理过程
- name为调试文件中的信息名，如cfg即表示控制流图（Control Flow Graph）

### 目录结构

![gcc目录结构](https://raw.githubusercontent.com/Hugsy19/Picbed/master/tech/gcc_struct.jpg)

- config*：gcc编译配置相关
- lib*/：通用库、语言库目录，如libgcc是与cpp语言相关的库
- gcc/：gcc核心代码，语言机器都无关的核心处理代码
  - gcc/{lang}：各种编程语言相关的前端分析程序
  - gcc/config/{target}：目标机器相关的机器描述文件
  - gcc/gen*.[ch]：根据机器描述文件生成与目标机器相关代码

### 源码编译

`./configure`配置：检查gcc源码的编译、安装环境，生成makefile

- `--prefix=`：安装目录
- `--program-prefix/suffix=`：gcc程序的前/后缀名
- `--build=`：生成编译器程序的机器平台信息
- `--host=`：生成的编译器程序所要运行的机器平台信息，默认等于BUILD
- `--target=`：生成的编译器程序生成的代码所允许的机器平台信息，默认等于HOST
- `--en/disable-{feature}`：开启/关闭某个特性
- `--with/without-{pacakge}`：包含/禁用某个软件包

gcc的源码编译过程中，通常会采用bootstrapping技术而分为三个阶段：

- stage1：用环境上现有的编译器编译gcc源码，生成new_gcc1
- stage2：用new_gcc1重写编译gcc源码，生成new_gcc2
- stage3：用new_gcc2重写编译gcc源码，生成new_gcc3
- 如果new_gcc2和new_gcc3相同，则编译成功，否则存在bug

## 从源码到AST/GENERIC

将高级程序源码转转换为目标机器汇编代码过程中，gcc主要使用了三种中间表示形式：

- 抽象语法树AST（Abstract Syntax Tree）
- GIMPLE
- 寄存器传输语言RTL（Register Transfer Language）

不同的程序设计语言，通过词法、语法分析后会得到不同形式的AST，而GENERIC是一种与前端无关的规范AST表示，由此将AST和GENERIC合称为AST/GENERIC。

### 树节点

gcc中用名为`tree_node`的联合体来表示AST节点：

- `gcc/tree.def`中声明了gcc中用到的所有树节点的基本信息，该声明由形如`#define DEFTREENODE(SYM, NAME, TYPE, LEN)`的宏定义来描述
  
  - SYM为节点的标识TREE_CODE，是树节点的语义描述，如`PLUS_EXPR`表示加法操作
  - NAME为树节点的名称，用于进行AST的中间结果表示
  - TYPE是TREE_CODE所属的类型（TREE_CODE TYPE，TCC），如`PLUS_EXPR`的TCC为双目运算`tcc_binary`
  - LEN描述了树节点所包含的操作数数目
  
- `gcc/tree-core.h`中定义了枚举类`tree_code`、`tree_code_class`来保存所有的SYM和TCC，数组`tree_code_type`、`tree_code_length`保存所有SYM对应的TYPE和LEN；`gcc/tree.cc`中定义了数组`tree_code_name`来保存所有SYM对应的NAME、数组`tree_code_class_strings`保存类型名称字符串

- `gcc/treestruct.def`中声明了`tree_node`中所有元素的枚举，而具体的`union tree_node`在`gcc/tree-core.h`中定义，它是不同类型树节点结构体的通用名称，且各类树节点结构体都在此头文件中定义：

  ```c
  union GTY ((ptr_alias (union lang_tree_node),
  	    desc ("tree_node_structure (&%h)"), variable_size)) tree_node {
    struct tree_base GTY ((tag ("TS_BASE"))) base; // 基类
    struct tree_typed GTY ((tag ("TS_TYPED"))) typed; // 类型节点
    struct tree_common GTY ((tag ("TS_COMMON"))) common; // 共有基本信息
    struct tree_int_cst GTY ((tag ("TS_INT_CST"))) int_cst; // 整型常量
    struct tree_poly_int_cst GTY ((tag ("TS_POLY_INT_CST"))) poly_int_cst;
    struct tree_real_cst GTY ((tag ("TS_REAL_CST"))) real_cst; // 实数常量
    struct tree_fixed_cst GTY ((tag ("TS_FIXED_CST"))) fixed_cst; // 定点数常量
    struct tree_vector GTY ((tag ("TS_VECTOR"))) vector; // 向量
    struct tree_string GTY ((tag ("TS_STRING"))) string; // 字符串
    struct tree_complex GTY ((tag ("TS_COMPLEX"))) complex; // 复数
    struct tree_identifier GTY ((tag ("TS_IDENTIFIER"))) identifier;
    struct tree_decl_minimal GTY((tag ("TS_DECL_MINIMAL"))) decl_minimal; // 声明节点基类
    struct tree_decl_common GTY ((tag ("TS_DECL_COMMON"))) decl_common;
    struct tree_decl_with_rtl GTY ((tag ("TS_DECL_WRTL"))) decl_with_rtl;
    struct tree_decl_non_common  GTY ((tag ("TS_DECL_NON_COMMON")))
      decl_non_common;
    struct tree_parm_decl  GTY  ((tag ("TS_PARM_DECL"))) parm_decl; // 参数声明
    struct tree_decl_with_vis GTY ((tag ("TS_DECL_WITH_VIS"))) decl_with_vis;
    struct tree_var_decl GTY ((tag ("TS_VAR_DECL"))) var_decl; // 变量声明
    struct tree_field_decl GTY ((tag ("TS_FIELD_DECL"))) field_decl; // struct/union声明
    struct tree_label_decl GTY ((tag ("TS_LABEL_DECL"))) label_decl; // 标号声明
    struct tree_result_decl GTY ((tag ("TS_RESULT_DECL"))) result_decl; // 返回值声明
    struct tree_const_decl GTY ((tag ("TS_CONST_DECL"))) const_decl; // 常量/枚举声明
    struct tree_type_decl GTY ((tag ("TS_TYPE_DECL"))) type_decl; // 类型声明
    struct tree_function_decl GTY ((tag ("TS_FUNCTION_DECL"))) function_decl; // 函数声明
    struct tree_translation_unit_decl GTY ((tag ("TS_TRANSLATION_UNIT_DECL")))
      translation_unit_decl;
    struct tree_type_common GTY ((tag ("TS_TYPE_COMMON"))) type_common;
    struct tree_type_with_lang_specific GTY ((tag ("TS_TYPE_WITH_LANG_SPECIFIC")))
      type_with_lang_specific;
    struct tree_type_non_common GTY ((tag ("TS_TYPE_NON_COMMON")))
      type_non_common;
    struct tree_list GTY ((tag ("TS_LIST"))) list; // 连接同类型节点，如函数参数
    struct tree_vec GTY ((tag ("TS_VEC"))) vec;
    struct tree_exp GTY ((tag ("TS_EXP"))) exp; // 表达式节点
    struct tree_ssa_name GTY ((tag ("TS_SSA_NAME"))) ssa_name;
    struct tree_block GTY ((tag ("TS_BLOCK"))) block;
    struct tree_binfo GTY ((tag ("TS_BINFO"))) binfo;
    struct tree_statement_list GTY ((tag ("TS_STATEMENT_LIST"))) stmt_list; // y
    struct tree_constructor GTY ((tag ("TS_CONSTRUCTOR"))) constructor;
    struct tree_omp_clause GTY ((tag ("TS_OMP_CLAUSE"))) omp_clause;
    struct tree_optimization_option GTY ((tag ("TS_OPTIMIZATION"))) optimization;
    struct tree_target_option GTY ((tag ("TS_TARGET_OPTION"))) target_option;
  };
  ```

  - 各类树节点之间呈一定继承关系，`tree_base`是所有树节点的基类
  
  - `tree_decl_minimal`是所有声明节点的基类，且声明节点间的继承关系如下图所示：
  
    ![声明节点继承关系](https://raw.githubusercontent.com/Hugsy19/Picbed/master/tech/gcc_decl_node_rela.jpg)

### 词法分析

对于C语言中的词法符号，gcc中用`struct c_token`来描述，该结构体在`gcc/c/c-parser.h`中被定义如下：

```c
struct GTY (()) c_token {
  /* The kind of token.  */
  ENUM_BITFIELD (cpp_ttype) type : 8; // 符号类型
  /* If this token is a CPP_NAME, this value indicates whether also
     declared as some kind of type.  Otherwise, it is C_ID_NONE.  */
  ENUM_BITFIELD (c_id_kind) id_kind : 8; // 标识符类型
  /* If this token is a keyword, this value indicates which keyword.
     Otherwise, this value is RID_MAX.  */
  ENUM_BITFIELD (rid) keyword : 8; // 关键字标识
  /* If this token is a CPP_PRAGMA, this indicates the pragma that
     was seen.  Otherwise it is PRAGMA_NONE.  */
  ENUM_BITFIELD (pragma_kind) pragma_kind : 8; // pragma类型
  /* The location at which this token was found.  */
  location_t location;  // 源码位置
  /* The value associated with this token, if any.  */
  tree value; // token值
  /* Token flags.  */
  unsigned char flags;

  source_range get_range () const
  {
    return get_range_from_loc (line_table, location);
  }

  location_t get_finish () const
  {
    return get_range ().m_finish;
  }
};
```

* 所有的符号类型`cpp_ttype`定义在`libcpp/include/cpplib.h`，主要由OP宏和TK宏给出符号及其类型值
* GCC对标识符类型进行了更详细的划分，具体定义在`gcc/c-parser.c`
* 所有的关键字标识定义在`gcc/c-family/c-common.h`中，由`enum rid`给出
* pragma类型由`gcc/c-family/c-pragma.h`的`enum pragma_kid`给出

使用gcc编译一个程序，进入编译器前端的函数调用栈如下：

- main函数定义在`gcc/main.cc`，其中创建了一个`toplev`对象，然后返回`toplev.main`
- `toplev`的声明和定义分别在`gcc/toplev.h`和`gcc/toplev.cc`
- `toplev.main`函数里面先进行了一些初始化步骤，特别注意其中的使用的`lang_hook`对象，它定义在`gcc/langhooks.h`中，是gcc编译器前端为了支持多编程语言而设计的，它没有用到cpp的虚函数机制，而是巧妙地使用了宏定义来实现接口和继承
  - `gcc/langhooks_def.h`中定义了很多宏，这些宏与`lang_hook`对象的成员函数进行了绑定，其他对象要继承它并重写这些函数，只需要先用`#undef`取消原来的宏定义，再用`#define`重新和那个宏绑定
- 执行到`toplev.main`中的`do_compile`函数，在来的其中的`compile_file`，这里调用`lang_hooks.parse_file`，来到了编译器的前端处理步骤，对于C语言，这个`parse_file`函数对应的宏在`gcc/c/c-objc-common.h`中被重定义，具体调用到的会是`gcc/c-family/c-opts.cc`中定义的`c_common_parse_file`

`gcc/gcc-main.cc`与`gcc/main.cc`的关系：

- `gcc/gcc-main.cc`包含gcc用户模式的`main`函数,用于生成`gcc`可执行文件。用户执行`gcc`编译命令实际运行的是`gcc/gcc-main.cc`中的 `main`函数
- `gcc/main.cc`包含gcc内部主函数,用于完成gcc的初始化工作和后续编译任务
- `gcc/gcc-main.cc`生成的`gcc`可执行文件被用户调用,从而间接调用到`gcc/main.cc`中的主函数


