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

![gcc目录结构](https://raw.githubusercontent.com/Hugsy19/Picbed/master/tech/gcc_struct.png)

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
  
    ![声明节点继承关系](https://raw.githubusercontent.com/Hugsy19/Picbed/master/tech/gcc_decl_node_rela.png)

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
* gcc对标识符类型进行了更详细的划分，由`enum c_id_kind`定义
* 所有的关键字标识定义在`gcc/c-family/c-common.h`中，由`enum rid`给出
* pragma类型由`gcc/c-family/c-pragma.h`的`enum pragma_kid`给出

使用gcc编译一个程序，进入编译器前端的函数调用栈如下：

- main函数定义在`gcc/main.cc`，其中创建了一个`toplev`对象，然后返回`toplev.main`
- `toplev`的声明和定义分别在`gcc/toplev.h`和`gcc/toplev.cc`
- `toplev.main`函数里面先进行了一些初始化步骤，特别注意其中的使用的`lang_hook`对象，它定义在`gcc/langhooks.h`中，是gcc编译器前端为了支持多编程语言而设计的，它没有用到cpp的虚函数机制，而是巧妙地使用了宏定义来实现接口和继承
  - `gcc/langhooks_def.h`中定义了很多宏，这些宏与`lang_hook`对象的成员函数进行了绑定，其他对象要继承它并重写这些函数，只需要先用`#undef`取消原来的宏定义，再用`#define`重新和那个宏绑定
- 执行到`toplev.main`中的`do_compile`函数，再来到其中的`compile_file`，这里调用`lang_hooks.parse_file`，来到了编译器的前端处理步骤，对于C语言，这个`parse_file`函数对应的宏在`gcc/c/c-objc-common.h`中被重定义，具体调用到的会是`gcc/c-family/c-opts.cc`中定义的`c_common_parse_file`，再来到`gcc/c/c-parser.cc`的`c_parse_file`
- 从`c_parse_file -> c_parser_translation_unit -> c_parser_external_declaration -> c_parser_peek_token`的层层调用，会来到关键的词法分析步骤`c_lex_one_token`，其中会调用C家族语言共用的手写词法分析程序`gcc/c-family/c-lex.cc`
- `c_lex_with_flags`是`c-lex.cc`中的关键函数，函数中会转而调用`gcc/libgcc/lex.cc`中的`_cpp_lex_token`，`_cpp_lex_token`转而调用了`_cpp_lex_direct`，这个才是最底层的C语言词法分析函数

`gcc/gcc-main.cc`与`gcc/main.cc`的关系：

- `gcc/gcc-main.cc`包含gcc用户模式的`main`函数,用于生成`gcc`可执行文件。用户执行`gcc`编译命令实际运行的是`gcc/gcc-main.cc`中的 `main`函数
- `gcc/main.cc`包含gcc内部主函数,用于完成gcc的初始化工作和后续编译任务
- `gcc/gcc-main.cc`生成的`gcc`可执行文件被用户调用,从而间接调用到`gcc/main.cc`中的主函数

### 语法分析

为了进行“最多提前预读四个词法符号的自顶向下的语法分析“推导过程，gcc将读取到的词法符号的相关信息保存在`struct c_parser`中，它在`gcc/c/c-parser.cc`中定义如下：

```c
struct GTY(()) c_parser {
  /* The look-ahead tokens.  */
  c_token * GTY((skip)) tokens;
  /* Buffer for look-ahead tokens.  */
  c_token tokens_buf[4]; // 预读词法符号
  /* How many look-ahead tokens are available (0 - 4, or
     more if parsing from pre-lexed tokens).  */
  unsigned int tokens_avail; // 可用数目
  /* Raw look-ahead tokens, used only for checking in Objective-C
     whether '[[' starts attributes.  */
  vec<c_token, va_gc> *raw_tokens;
  /* The number of raw look-ahead tokens that have since been fully
     lexed.  */
  unsigned int raw_tokens_used;
  /* True if a syntax error is being recovered from; false otherwise.
     c_parser_error sets this flag.  It should clear this flag when
     enough tokens have been consumed to recover from the error.  */
  BOOL_BITFIELD error : 1;
  /* True if we're processing a pragma, and shouldn't automatically
     consume CPP_PRAGMA_EOL.  */
  BOOL_BITFIELD in_pragma : 1;
  /* True if we're parsing the outermost block of an if statement.  */
  BOOL_BITFIELD in_if_block : 1;
  /* True if we want to lex a translated, joined string (for an
     initial #pragma pch_preprocess).  Otherwise the parser is
     responsible for concatenating strings and translating to the
     execution character set as needed.  */
  BOOL_BITFIELD lex_joined_string : 1;
  /* True if, when the parser is concatenating string literals, it
     should translate them to the execution character set (false
     inside attributes).  */
  BOOL_BITFIELD translate_strings_p : 1;

  /* Objective-C specific parser/lexer information.  */

  /* True if we are in a context where the Objective-C "PQ" keywords
     are considered keywords.  */
  BOOL_BITFIELD objc_pq_context : 1;
  /* True if we are parsing a (potential) Objective-C foreach
     statement.  This is set to true after we parsed 'for (' and while
     we wait for 'in' or ';' to decide if it's a standard C for loop or an
     Objective-C foreach loop.  */
  BOOL_BITFIELD objc_could_be_foreach_context : 1;
  /* The following flag is needed to contextualize Objective-C lexical
     analysis.  In some cases (e.g., 'int NSObject;'), it is
     undesirable to bind an identifier to an Objective-C class, even
     if a class with that name exists.  */
  BOOL_BITFIELD objc_need_raw_identifier : 1;
  /* Nonzero if we're processing a __transaction statement.  The value
     is 1 | TM_STMT_ATTR_*.  */
  unsigned int in_transaction : 4;
  /* True if we are in a context where the Objective-C "Property attribute"
     keywords are valid.  */
  BOOL_BITFIELD objc_property_attr_context : 1;

  /* Whether we have just seen/constructed a string-literal.  Set when
     returning a string-literal from c_parser_string_literal.  Reset
     in consume_token.  Useful when we get a parse error and see an
     unknown token, which could have been a string-literal constant
     macro.  */
  BOOL_BITFIELD seen_string_literal : 1;

  /* Location of the last consumed token.  */
  location_t last_token_location;
};
```

gcc语法分析的入口函数为`gcc/c/c-parser.cc`中的`c_parse_file`:

```c
void
c_parse_file (void)
{
  /* Use local storage to begin.  If the first token is a pragma, parse it.
     If it is #pragma GCC pch_preprocess, then this will load a PCH file
     which will cause garbage collection.  */
  c_parser tparser;

  memset (&tparser, 0, sizeof tparser);
  tparser.translate_strings_p = true;
  tparser.tokens = &tparser.tokens_buf[0];
  the_parser = &tparser;

  if (c_parser_peek_token (&tparser)->pragma_kind == PRAGMA_GCC_PCH_PREPROCESS) // 预读一个词法符号，判断是否为预处理符号
    c_parser_pragma_pch_preprocess (&tparser); // 进行预处理
  else
    c_common_no_more_pch ();

  the_parser = ggc_alloc<c_parser> ();
  *the_parser = tparser;
  if (tparser.tokens == &tparser.tokens_buf[0])
    the_parser->tokens = &the_parser->tokens_buf[0];

  /* Initialize EH, if we've been told to do so.  */
  if (flag_exceptions)
    using_eh_for_cleanups ();

  c_parser_translation_unit (the_parser); // 进行语法推导
  the_parser = NULL;
}
```

gcc编译的最大单位是源文件，统称为编译单元（translation_unit），其语法分析过程由以下函数完成：

- `c_parser_translation_unit`：解析整个编译单元,并为每个外部声明在全局符号表中为其生成对应的函数/变量节点
- `c_parser_external_declaration`：进行外部变量的语法分析，如全局变量、函数声明/定义语句等
- `c_parser_declaration_or_fndef`：进行函数声明/定义的语法分析
- `c_parser_declarator`：进行声明说明符的语法分析

## 从AST/GENERIC到GIMPLE

### GIMPLE

为了处理不同的前端语言及其相应的AST/GENERIC，gcc引入了名为GIMPLE的与前端语言无关的中间表示：

- AST是树状结构，GIMPLE则是线性的中间表示序列
- GIMPLE中通过引入临时变量保存中间结果，将AST表达式才分为不超过三个操作数的元组
- AST中if-else等控制结构，在GIMPLE中被转换为条件跳转语句

AST转为GIMPLE的过程中，会先后经历高级GIMPLE（High-Level GIMPLE）和低级GIMPLE（Low-Level GIMPLE）两个阶段：

- 高级GIMPLE中会有`GIMPLE_BIND`等表示作用域的语句
- 经过`pass_lower_cf`后高级GIMPLE即被转换为了低级，`GIMPLE_BIND`、`GIMPLE_TRY`等语句都会被移除

如对于以下代码：

```c
int main() {
    int i = 0;
    int sum = 0;
    for (i; i < 10; i++) {
        sum = sum + i;
    }
    return sum;
}
```

使用`-fdump-tree-gimple-raw`选项编译，得到的高级GIMPLE内容如下：

```
int main ()
gimple_bind <
  int D.2747;

  gimple_bind <
    int i;
    int sum;

    gimple_assign <integer_cst, i, 0, NULL, NULL>
    gimple_assign <integer_cst, sum, 0, NULL, NULL>
    gimple_goto <<D.2745>>
    gimple_label <<D.2744>>
    gimple_assign <plus_expr, sum, sum, i, NULL>
    gimple_assign <plus_expr, i, i, 1, NULL>
    gimple_label <<D.2745>>
    gimple_cond <le_expr, i, 9, <D.2744>, <D.2742>>
    gimple_label <<D.2742>>
    gimple_assign <var_decl, D.2747, sum, NULL, NULL>
    gimple_return <D.2747>
  >
  gimple_assign <integer_cst, D.2747, 0, NULL, NULL>
  gimple_return <D.2747>
>
```

用`-fdumo-tree-lower-raw`得到的低级GIMPLE则为：

```
;; Function main (main, funcdef_no=0, decl_uid=2738, cgraph_uid=1, symbol_order=0)

int main ()
{
  int sum;
  int i;
  int D.2747;

  gimple_assign <integer_cst, i, 0, NULL, NULL>
  gimple_assign <integer_cst, sum, 0, NULL, NULL>
  gimple_goto <<D.2745>>
  gimple_label <<D.2744>>
  gimple_assign <plus_expr, sum, sum, i, NULL>
  gimple_assign <plus_expr, i, i, 1, NULL>
  gimple_label <<D.2745>>
  gimple_cond <le_expr, i, 9, <D.2744>, <D.2742>>
  gimple_label <<D.2742>>
  gimple_assign <var_decl, D.2747, sum, NULL, NULL>
  gimple_goto <<D.2748>>
  gimple_assign <integer_cst, D.2747, 0, NULL, NULL>
  gimple_goto <<D.2748>>
  gimple_label <<D.2748>>
  gimple_return <D.2747>
}
```

`gcc/gimple.def`中用格式为`DEFGSCODE(GIMPLE_symbol, printable name, GSS_symbol)`的宏对各种GIMPLE语句进行了声明：

- `GIMPLE_symbol`：操作类型码
- `printable name`：打印名称
- `GSS_symbol`：由`gcc/gsstruct.def`中的`DEFGSSTRUCT`宏定义，用以计算GIMPLE语句存储结构中的偏移地址

程序编译过程中，GIMPLE化过程中的函数调用栈如下：

- `compile_file`中的`lang_hooks.parse_file`执行完毕后，来到下面的`if (!in_lto_p)`判断，执行其中的`symtab->finalize_compilation_unit()`
- `symtab`为定义在`gcc/cgraph.h`中的类型名为`symbol_table *`的全局符号表，其中记录了整个编译过程中产生的所有函数和符号，函数的节点信息在其中用`cgraph_node`结构体表示，变量则用`varpool_node`结构体表示
- `finalize_compilation_unit`被定义在`gcc/cgraphunit.cc`中，其中的`analyze_functions`将遍历符号表中的所有节点，对于其中函数节点，通过`cnode->analyze()`完成其GIMPLE化过程，对变量节点则用`vnode->analyze()`进行对齐操作
- `cgraph_node::analyze`也定义在`gcc/cgraphunit.cc`中，高端GIMPLE化过程通过调用`gcc/gimplify.cc`中的定义的`gimplify_function_tree`来实现，低端GIMPLE化则通过执行名为`all_lowering_passes`的pass来完成



