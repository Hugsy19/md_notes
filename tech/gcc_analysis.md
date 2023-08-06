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
- 经过`pass_lower_cf`后，对高级GIMPLE进行数据、代码、返回语句合并，即将其转换为了低级GIMPLE，从而有利于生成更规整的后端代码，其中：
  - 词法范围被移除，如`GIMPLE_BIND`
  - if语句都被转化成then和else两个分支
  - `GIMPLE_TRY`、`GIMPLE_CATCH`语句都被转换为异常控制流
  - 多个相同的`GIMPLE_RETURN`语句被合并到一起


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

### 生成GIMPLE

`gcc/gimple.def`中用格式为`DEFGSCODE(GIMPLE_symbol, printable name, GSS_symbol)`的宏对各种GIMPLE语句进行了声明：

- `GIMPLE_symbol`：操作类型码
- `printable name`：打印名称
- `GSS_symbol`：由`gcc/gsstruct.def`中的`DEFGSSTRUCT`宏定义，用以计算GIMPLE语句存储结构中的偏移地址

`gcc/gimple.h`中定义的`struct gimple`是所有GIMPLE存储结构体的基类：

```c
struct GTY((desc ("gimple_statement_structure (&%h)"), tag ("GSS_BASE"),
	    chain_next ("%h.next"), variable_size))
  gimple
{
  /* [ WORD 1 ]
     Main identifying code for a tuple.  */
  ENUM_BITFIELD(gimple_code) code : 8;

  /* Nonzero if a warning should not be emitted on this tuple.  */
  unsigned int no_warning	: 1;

  /* Nonzero if this tuple has been visited.  Passes are responsible
     for clearing this bit before using it.  */
  unsigned int visited		: 1;

  /* Nonzero if this tuple represents a non-temporal move.  */
  unsigned int nontemporal_move	: 1;

  /* Pass local flags.  These flags are free for any pass to use as
     they see fit.  Passes should not assume that these flags contain
     any useful value when the pass starts.  Any initial state that
     the pass requires should be set on entry to the pass.  See
     gimple_set_plf and gimple_plf for usage.  */
  unsigned int plf		: 2;

  /* Nonzero if this statement has been modified and needs to have its
     operands rescanned.  */
  unsigned modified 		: 1;

  /* Nonzero if this statement contains volatile operands.  */
  unsigned has_volatile_ops 	: 1;

  /* Padding to get subcode to 16 bit alignment.  */
  unsigned pad			: 1;

  /* The SUBCODE field can be used for tuple-specific flags for tuples
     that do not require subcodes.  Note that SUBCODE should be at
     least as wide as tree codes, as several tuples store tree codes
     in there.  */
  unsigned int subcode		: 16;

  /* UID of this statement.  This is used by passes that want to
     assign IDs to statements.  It must be assigned and used by each
     pass.  By default it should be assumed to contain garbage.  */
  unsigned uid;

  /* [ WORD 2 ]
     Locus information for debug info.  */
  location_t location;

  /* Number of operands in this tuple.  */
  unsigned num_ops;

  /* [ WORD 3 ]
     Basic block holding this statement.  */
  basic_block bb;

  /* [ WORD 4-5 ]
     Linked lists of gimple statements.  The next pointers form
     a NULL terminated list, the prev pointers are a cyclic list.
     A gimple statement is hence also a double-ended list of
     statements, with the pointer itself being the first element,
     and the prev pointer being the last.  */
  gimple *next;
  gimple *GTY((skip)) prev;
};
```

不同类型的GIMPLE语句使用不同的结构体进行存储，`gcc/gsstrrut.def`中用`DEFGSSTRUCT(GSS enumeration value, structure name, has-tree-operands)`宏给出了所有`GSS_symbol`对应的结构体类型。

程序编译过程中，GIMPLE化过程中的函数调用栈如下：

- `compile_file`中的`lang_hooks.parse_file`执行完毕后，来到下面的`if (!in_lto_p)`判断，执行其中的`symtab->finalize_compilation_unit()`
- `symtab`为定义在`gcc/cgraph.h`中的类型名为`symbol_table *`的全局符号表，其中记录了整个编译过程中产生的所有函数和符号，函数的节点信息在其中用`cgraph_node`结构体表示，变量则用`varpool_node`结构体表示
- `finalize_compilation_unit`被定义在`gcc/cgraphunit.cc`中，其中的`analyze_functions`将遍历符号表中的所有节点，对于其中函数节点，通过`cnode->analyze()`完成其GIMPLE化过程，对变量节点则用`vnode->analyze()`进行对齐操作
- `cgraph_node::analyze`也定义在`gcc/cgraphunit.cc`中，高端GIMPLE化过程通过调用`gcc/gimplify.cc`中的定义的`gimplify_function_tree`来实现，低端GIMPLE化则通过`execute_pass_list (cfun, g->get_passes ()->all_lowering_passes); `,执行名为`all_lowering_passes`的pass来完成

### GIMPLE Pass

为了便于管理，gcc将诸如GIMPLE低端化、GIMPLE优化及RTL生成等优化处理过程都组织为Pass，每个Pass作为一个单独的处理过程完成一种特定的处理，然后将其输出结果作为下一个Pass的输入。

Pass的核心数据结构定义在`gcc/tree-pass.h`中：

```c
struct pass_data
{
  /* Optimization pass type.  */
  enum opt_pass_type type; // Pass类型

  /* Terse name of the pass used as a fragment of the dump file
     name.  If the name starts with a star, no dump happens. */
  const char *name;

  /* The -fopt-info optimization group flags as defined in dumpfile.h. */
  optgroup_flags_t optinfo_flags;

  /* The timevar id associated with this pass.  */
  /* ??? Ideally would be dynamically assigned.  */
  timevar_id_t tv_id;

  /* Sets of properties input and output from this pass.  */
  unsigned int properties_required; // 执行Pass需要满足的条件
  unsigned int properties_provided; // 执行Pass所提供的属性
  unsigned int properties_destroyed; // 执行Pass所破坏的属性

  /* Flags indicating common sets things to do before and after.  */
  unsigned int todo_flags_start;  // 执行Pass前需要执行的标准动作
  unsigned int todo_flags_finish; // 执后Pass前需要执行的标准动作
};
```

Pass共有四大类：

- GIMPLE_PASS：以GIMPLE为处理对象
- RTL_PASS：以RTL为处理对象
- SIMPLE_IPA_PASS：以GIMPLE为处理对象，进行**过程间分析（IPA，Inter-Procedural Analysis）**，及函数间变量的传递及参数传递
- IPA_PASS：同上

执行Pass的各种条件/属性由`gcc/tree-pass.h`中以`PROP_`开头的宏定义，执行Pass前后所要执行的标准动作则由`TODO_`开头的宏定义。

`opt_pass`继承自`pass_data`：

```c++
class opt_pass : public pass_data
{
public:
  virtual ~opt_pass () { }

  /* Create a copy of this pass.

     Passes that can have multiple instances must provide their own
     implementation of this, to ensure that any sharing of state between
     this instance and the copy is "wired up" correctly.

     The default implementation prints an error message and aborts.  */
  virtual opt_pass *clone ();
  virtual void set_pass_param (unsigned int, bool);

  /* This pass and all sub-passes are executed only if the function returns
     true.  The default implementation returns true.  */
  virtual bool gate (function *fun);

  /* This is the code to run.  If this is not overridden, then there should
     be sub-passes otherwise this pass does nothing.
     The return value contains TODOs to execute in addition to those in
     TODO_flags_finish.   */
  virtual unsigned int execute (function *fun);

protected:
  opt_pass (const pass_data&, gcc::context *);

public:
  /* A list of sub-passes to run, dependent on gate predicate.  */
  opt_pass *sub;

  /* Next in the list of passes to run, independent of gate predicate.  */
  opt_pass *next;

  /* Static pass number, used as a fragment of the dump file name.  */
  int static_pass_number;

protected:
  gcc::context *m_ctxt;
};
```

各种不同的Pass通过其中的`next`字段组成链表，且每个Pass的子Pass也可以将不同的Pass组织成子链表。`gcc/passses.cc`中定义的`INSERT_PASSES_AFTER(PASS)`、`NEXT_PASS (PASS)`等宏可将Pass快速得组织起来，且gcc中预定义的所有Pass都通过这些宏组织并写在了`gcc/passes.def`中，其中GIMPLE低端化相关的pass链表内容如下：

```
INSERT_PASSES_AFTER (all_lowering_passes)
  NEXT_PASS (pass_warn_unused_result); // 处理warn_unused_result选项
  NEXT_PASS (pass_diagnose_omp_blocks);
  NEXT_PASS (pass_diagnose_tm_blocks);
  NEXT_PASS (pass_omp_oacc_kernels_decompose);
  NEXT_PASS (pass_lower_omp);
  NEXT_PASS (pass_lower_cf); // gimple低端化的主要内容（去除gbind节点,将所有greturn节点放到函数最后）
  NEXT_PASS (pass_lower_tm);
  NEXT_PASS (pass_refactor_eh);
  NEXT_PASS (pass_lower_eh);
  NEXT_PASS (pass_coroutine_lower_builtins);
  NEXT_PASS (pass_build_cfg); // 将低端化后的指令序列转换为函数的CFG
  NEXT_PASS (pass_warn_function_return);
  NEXT_PASS (pass_coroutine_early_expand_ifns);
  NEXT_PASS (pass_expand_omp);
  NEXT_PASS (pass_build_cgraph_edges); // 构建函数间的调用关系图（Call Graph）
TERMINATE_PASS_LIST (all_lowering_passes)
```

- pass_lower_cf：定义在`gcc/gimple-low.cc`中，主要的功能函数为`lower_function_body`
- pass_build_cfg：定义在`gcc/tree-cfg.cc`中，主要的功能函数为`build_gimple_cfg`
- pass_build_cgraph_edges：定义在`gcc/cgraphbuild.cc`

执行Pass时，函数的信息都保存在全局变量`struct function *cfun`中，其数据类型`function`定义在`gcc/function.h`。

#### 控制流图（CFG）

**控制流图（Control Flow Graph, CFG）**是GIMPLE处理中的重要概念，其描述了函数中的处理流程，在一个CFG中，节点为程序的基本块（Basic Block），边则为借本块之间的跳转关系。gcc中的`pass_build_cfg`过程会对函数的GIMPLE序列进行分析，完成基本块的划分，并根据GIMPLE语义构造基本块之间的跳转关系。

用以保存CFG的数据类型为`gcc/cfg.h`中定义的`control_flow_graph`：

```c
struct GTY(()) control_flow_graph {
  /* Block pointers for the exit and entry of a function.
     These are always the head and tail of the basic block list.  */
  basic_block x_entry_block_ptr; // 函数入口块
  basic_block x_exit_block_ptr; // 函数出口块

  /* Index by basic block number, get basic block struct info.  */
  vec<basic_block, va_gc> *x_basic_block_info; // BB块信息

  /* Number of basic blocks in this flow graph.  */
  int x_n_basic_blocks; // BB块个数

  /* Number of edges in this flow graph.  */
  int x_n_edges; // 边条数

  /* The first free basic block number.  */
  int x_last_basic_block;

  /* UIDs for LABEL_DECLs.  */
  int last_label_uid;

  /* Mapping of labels to their associated blocks.  At present
     only used for the gimple CFG.  */
  vec<basic_block, va_gc> *x_label_to_block_map;

  enum profile_status_d x_profile_status;

  /* Whether the dominators and the postdominators are available.  */
  enum dom_state x_dom_computed[2];

  /* Number of basic blocks in the dominance tree.  */
  unsigned x_n_bbs_in_dom_tree[2];

  /* Maximal number of entities in the single jumptable.  Used to estimate
     final flowgraph size.  */
  int max_jumptable_ents;

  /* Maximal count of BB in function.  */
  profile_count count_max;

  /* Dynamically allocated edge/bb flags.  */
  int edge_flags_allocated;
  int bb_flags_allocated;
};
```

且`build_gimple_cfg`的具体实现如下:

```c
static void
build_gimple_cfg (gimple_seq seq)
{
  /* Register specific gimple functions.  */
  gimple_register_cfg_hooks ();

  memset ((void *) &cfg_stats, 0, sizeof (cfg_stats));

  init_empty_tree_cfg (); // 初始化cfg指针

  make_blocks (seq); // 创建基本块

  /* Make sure there is always at least one block, even if it's empty.  */
  if (n_basic_blocks_for_fn (cfun) == NUM_FIXED_BLOCKS)
    create_empty_bb (ENTRY_BLOCK_PTR_FOR_FN (cfun));

  /* Adjust the size of the array.  */
  if (basic_block_info_for_fn (cfun)->length ()
      < (size_t) n_basic_blocks_for_fn (cfun))
    vec_safe_grow_cleared (basic_block_info_for_fn (cfun),
			   n_basic_blocks_for_fn (cfun));

  /* To speed up statement iterator walks, we first purge dead labels.  */
  cleanup_dead_labels ();

  /* Group case nodes to reduce the number of edges.
     We do this after cleaning up dead labels because otherwise we miss
     a lot of obvious case merging opportunities.  */
  group_case_labels ();

  /* Create the edges of the flowgraph.  */
  discriminator_per_locus = new hash_table<locus_discrim_hasher> (13);
  make_edges (); // 创建边
  assign_discriminators ();
  cleanup_dead_labels ();
  delete discriminator_per_locus;
  discriminator_per_locus = NULL;
}
```

- CFG初始化过程组要包括构造函数初始块（Entry Block）和出口块（Exit Block），并将其链接起来，同时初始化一些BB块、边的数目等信息
- 创建BB块的过程中：
  - 一般一个`GIMPLE_LABLE`语句就对应一个BB块的开始，连续的多个`GIMPLE_LABLE`语句可进行合并
  - 出现`GIMPLE_COND`、`GIMPLE_SWITCH`、`GIMPLE_GOTO`、`GIMPLE_RETURN`等改变控制流的语句，则标志着一个BB块的结束

对于前面的C程序，生成的CFG dump文件如下：

```
;; Function main (main, funcdef_no=0, decl_uid=2738, cgraph_uid=1, symbol_order=0)

Removing basic block 6
;; 2 loops found
;;
;; Loop 0
;;  header 0, latch 1
;;  depth 0, outer -1
;;  nodes: 0 1 2 3 4 5 6
;;
;; Loop 1
;;  header 4, latch 3
;;  depth 1, outer 0
;;  nodes: 4 3
;; 2 succs { 4 }
;; 3 succs { 4 }
;; 4 succs { 3 5 }
;; 5 succs { 6 }
;; 6 succs { 1 }
int main ()
{
  int sum;
  int i;
  int D.2747;

  <bb 2> :
  i = 0;
  sum = 0;
  goto <bb 4>; [INV]

  <bb 3> :
  sum = sum + i;
  i = i + 1;

  <bb 4> :
  if (i <= 9)
    goto <bb 3>; [INV]
  else
    goto <bb 5>; [INV]

  <bb 5> :
  D.2747 = sum;

  <bb 6> :
<L3>:
  return D.2747;

}
```

#### 调用关系图（Cgraph）

**调用关系图（Call Graph）**描述了程序中各函数之间的调用关系，该关系一般用有向图（Directed Graph）进行描述，图中的节点代表函数，有向边则代表其间的调用关系。

其相关的数据结构`cgraph_node`和`cgraph_edge`都定义在`gcc/cgraph.h`。

### IPA Pass

`finalize_compilation_unit`中`analyze_functions`完成程序的gimplify后，之后的编译过程在`symbol_table::compile`中完成，其中不仅包含IPA Pass的执行，还有GIMPLE到RTL的转换，以及RTL到最终汇编代码的输出。

IPA Pass包含了all_small_ipa_passes、all_regular_ipa_passes以及all_late_ipa_passes，其作用是过程间优化，不针对具体的函数。

#### 静态单赋值（SSA）

**静态单赋值（Static Single Assignment）**指的是每个变量都只能被赋值一次，gcc中由all_small_ipa_passes中名为pass_build_ssa_passes的子Pass来完成这一过程，将GIMPLE转换为SSA形式后，在此基础上还会进行一些基本优化。

SSA有几个相关的概念：

- dominance frontier：指函数中BB块之间的控制流程关系，对两个BB块A和B，如果从初始块开始所有到B的流程都需要进行A，则称A为B的dominance frontier（支配前导块）

- immediate dominator：如果A是离B最近的dominance frontier，则称A是B的immediate dominator

- PHI（$\phi$）节点：SSA形式要求每个变量只能被赋值一次，当遇到如下所示的情况时：

  ```c
  a = 1;
  if (v < 10)
      a = 2;
  b = a;
  ```

  满足if条件的情况下，a将被二次赋值，为了满足SSA约束，就要把两个a进行区分，同时在给b赋值时，通过一个PHI节点：

  ```c
  a1 = 1;
  if (v < 10)
      a2 = 2;
  b = PHI(a1, a2);
  ```

​	PHI节点会根据控制流是从哪个BB块到达的，来决定使用哪个版本的a

该Pass在`gcc/tree-into-ssa.cc`中实现，主要包含以下步骤：

- 计算dominance frontier和immediate dominator，并根据需要插入PHI节点
- 查找并标记所有变量定义的块信息
- 在dominance frontier插入PHI节点
- 重命名块和语句

对前面的程序，生成的SSA dump如下：

```
;; Function main (main, funcdef_no=0, decl_uid=2738, cgraph_uid=1, symbol_order=0)

int main ()
{
  int sum;
  int i;
  int D.2747;
  int _5;

  <bb 2> :
  i_3 = 0;
  sum_4 = 0;
  goto <bb 4>; [INV]

  <bb 3> :
  sum_7 = sum_2 + i_1;
  i_8 = i_1 + 1;

  <bb 4> :
  # i_1 = PHI <i_3(2), i_8(3)>
  # sum_2 = PHI <sum_4(2), sum_7(3)>
  if (i_1 <= 9)
    goto <bb 3>; [INV]
  else
    goto <bb 5>; [INV]

  <bb 5> :
  _5 = sum_2;

  <bb 6> :
<L3>:
  return _5;

}
```

## 从GIMPLE到RTL

### RTL

为了将与机器无关的GIMPLE中间表示转换为与机器相关的汇编语言，GCC中引入了寄存器传输语言RTL，它采用了类型LISP语言的列表形式，描述了每一条指令的语义动作，根据其作用可分为两大类：

- 内部格式
- 文本格式

### 机器描述文件

