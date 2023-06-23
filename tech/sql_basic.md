- 1. 数据库和SQL

  - 基础知识

    - 数据库管理系统（DataBase Management System, DBMS）：管理数据库的计算机系统
      - 常采用C/S结构

    - 种类：

      - 层次（Hierarchical）数据库（HDB）：把数据通过层次（树形）结构表现出来

      - 关系（Relational）数据库（RDB）：用行和列组成的二维表来管理数据

        - 典型的RDBMS：

          - Oracle Database

          - SQL Server（MS）

          - DB2（IBM）

          - PostgreSQL

          - MySQL

      - 面向对象数据库（OODB）：把数据及其操作集合起来以对象为单位进行管理

      - XML数据库（XMLDB）

      - 键值对数据库（Key-Value Store, KVS）

    - 表：

      - 表：管理数据的二维表

      - 字段：表的列，表示表中的数据项

      - 记录：表的行，表示一条数据
        - RDBMS必须以行为单位进行数据读写

    - SQL语句：一条语句由关键字、表名、列名等组合而成

      - 根据指令种类，可分为：

        - DDL（Data Definition Language）：对表对象进行增删改

          - CREATE 创建

          - DROP 删除

          - ALTER 修改

        - DML（Data Manipulation Language）：查询或变更表中记录（核心）

          - SELECT 查询

          - INSERT 插入

          - UPDATE 更新

          - DELETE 删除

        - DCL（Data Control Language）：确认或取消数据变更、用户权限

          - COMMIT 确认

          - ROLLBACK 回滚

          - GRANT 赋予

          - REVOKE 撤销

      - 规则：

        - 各语句以分号结尾

        - 不区分大小写

          - 关键字大写

          - 表名首字母大写

  - 表的操作

    - CREATE DATEBASE <database_name>; -- 创建数据库

    - CREATE TABLE <table_name> (  

      - <line_name_1> <data_type> <constraints>,

      - <line_name_n> <data_type> <constraints>); -- 创建表

        - table_name：半角英文字母、数字、下划线

        - data_type：

          - INTEGER：整型

          - CHAR：定长字符串，可指定长度

          - VARCHAR：可变字符串，可指定长度

          - DATE：日期

          - NUMERIC：数值，以全体位数+小数位数的形式定义

        - constraints：

          - PRIMARY KEY：主键

          - NOT NULL：非空

          - DEFAULT：默认值

    - DROP TABLE <table_name>; -- 删除表

    - ALTER TABLE <table_name> <ops>; -- 更新表的定义

      - ops：

        - ADD COLUMN <line_define>; -- 新增列

        - DROP COLUMN <line_name>; -- 删除列

        - RENAME TO <new_name>; -- 重命名（Oracle、PSQL）
          - RENAME TABLE <old> to <new>; （MySQL）

- 2. 查询基础

  - SELECT语句

    - SELECT <line_name> FROM <table_name>; -- 简单查询

    - SELECT <line_name/constant> AS <alias/‘别名’> FROM <table_name>; -- 取别名

    - SELECT DISTINCT <line_name> FROM <table_name>; -- 排除重复项

    - SELECT <line_name> FROM <table_name> WHERE <cond_expr>; -- 根据条件

  - 运算符

    - 可对<line_name>用算术运算符，以行为单位进行四则运算

    - 可在<cond_expr>用算术、比较、逻辑运算符，进行各种比较运算

      - 比较：
        - <>：不等于

      - 逻辑：

        - AND：与

        - OR：或

        - NOT：非

    - SQL中的逻辑运算为三值逻辑，除真假外还包含第三种值--UNKONWN

- 3. 聚合与排序

  - 聚合函数：

    - 只可在SELECT、HAVING、ORDER BY子句中使用

    - 常用：

      - COUNT：表中记录数

      - SUM：列合计值

      - AVG：列平均值

      - MAX/MIN：列最大/小值

    - 可用关键字：
      - DISTINCT：排除重复

  - 分组：

    - SELECT <line_name> FROM <table_name> GROUP BY <line_name>;
      - SELECT子句中不可出现聚合键（GROUP BY 子句指定的列名）以外的列名

    - SELECT <line_name> FROM <table_name>  WHERE <cond_expr> GROUP BY <line_name>; -- 依条件显示分组

      - 执行顺序：FROM->WHERE->GROUP BY->SELECT

      - 不可用别名作为聚合键

    - SELECT <line_name> FROM <table_name> GROUP BY <line_name> HAVING <cond_expr>; -- 依条件从分组结果中选择
      - 顺序：SELECT -> FROM -> WHERE -> GROUP BY -> HAVING

  - 排序：

    - SELECT <line_name> FROM <table_name> ORDER BY <reference_line> DESC/ASC; -- 降/升序排序

      - 顺序：FROM->WHERE->GROUP BY->HAVING->SELECT->ORDER BY

      - 排序键中包含NULL时，会在开头或末尾汇总

- 4. 数据更新

  - 增/删/改：

    - INSERT INTO <table_name> ( <line_name> ) VALUES ( <value> ); 

    - INSERT INTO ... SELECT ...; -- 依SELECT的执行结果作为插入值

    - DELETE FROM<table_name>; -- 清空表

    - DELETE FROM<table_name> WHERE<cond_expr>; --依条件删除

    - UPDATE <table_name> SET<line_name> = <expr>; -- 更新某字段

    - UPDATE <table_name> SET(<line_name>) = (<expr>) WHERE <cond_expr>; -- 依条件更新

  - 事务：需要在同一个处理单元中执行的一系列更新处理的集合

    - 语法：

      - 开始语句; -- BEGIN/START/空 TRANSCTION; (PostgreSQL/MySQL/Oracle)

        - DML语句1;

        - DML语句2;

        - DML语句3;

      - 结束语句; -- COMMIT/ROLLBACK; (提交/取消处理）

    - ACID特性：

      - 原子性（Atomicity）：事务结束时，其中包含的更新处理要么全部执行，要么全部不执行

      - 一致性（Consistency）：事务中包含的处理要满足数据库提前设置的约束

      - 隔离性（Isolation）：不同事务间互不干扰

      - 持久性（Durability）：事务结束后，DBMS能保证该时间点的数据被保存，即使发生故障也有恢复手段

- 5. 复杂查询

  - 视图：由常用的SELECT语句封装而成

    - 语法：

      - CREATE VIEW <view_name> ( <vline_name> ) AS <SELECT语句>; -- 创建

      - SELECT <vline_name> FROM <view_name>; -- 使用

      - DROP VIEW <view_name>; -- 删除

    - 限制：

      - 定义时不可用ORDER BY语句

      - 因视图应和表同时进行更新，故要对视图进行更新需满足一些条件
        - 对表汇总后所得视图无法更新

  - 子查询

    - SELECT <line_name> FROM <SELECT语句> AS <child_name>; 

    - 标量子查询：返回单个值的结果（一行一列）
      - 可在任何能用常数或列名的地方使用，如SELECT、WHERE、GROUP BY、HAVING、ORDER BY子句

    - 关联子查询：常用于在细分的组内进行比较
      - 例如：
        - SELECT product_type, product_name, sale_price FROM Product AS P1 WHERE sale_price > (SELECT AVG(sale_price) FROM Product AS P2 WHERE P1.product_type = P2.product_type GROUP BY Product_type);

- 6. 函数、谓词、CASE表达式

  - 函数：

    - 算术：

      - ABS：绝对值

      - MOD(m, n)：求余（m/n）

      - ROUND(m, n)：舍入（n-保留位数）

    - 字符串：

      - ||/CONCAT：拼接

      - LENGTH：长度

      - LOWER/UPPER：大/小写

      - REPLACE(str, m, n)：替换（m->n）

      - SUBSTRING(str FROM m FOR n）：截取（m位始取n个）

    - 日期：

      - CURRENT_DATE：当前日期

      - CURRENT_TIME：当前时间

      - CURRENT_TIMESTAMP：当前日期、时间

      - EXTRACT(m FROM n)：截取（n中截取m元素）

    - 转换：

      - CAST(m AS n)：类型转换（m转换为n）

      - COALESCE：将NULL转换为其他值，含有NULL时防止计算结果为NULL（可变参数）

  - 谓词：返回值为真的函数

    - LIKE：字符串部分一致查询

      - %：0个以上任意字符

      - _：1个任意字符

      - 例如：SELECT * FROM SampleLike WHERE strcol LIKE '%ab_';

    - BETWEEN：范围查询
      - 例如：SELECT product_name, sale_price FROM Product WHERE sale_price BETWEEN 100 AND 1000;

    - IS (NOT) NULL：是否为空

    - (NOT) IN：是否包含

    - (NOT) EXISTS：可替代以上
      - 例如：SELECT product_name, sale_price FROM Product AS P WHERE EXISTS (SELECT * FROM ShopProduct AS SP WHERE SP. shop_id = '000C' AND SP.product_id = P.product_id);

  - CASE表达式：

    - 搜索CASE：

      - CASE WHEN <value_expr> THEN <expr>

        - WHEN <value_expr> THEN <expr>

        - ELSE <expr>

      - END

    - 简单CASE：

      - CASE <expr>

        -  WHEN <value_expr> THEN <expr>

        - ELSE <expr>

      - END

- 7. 集合运算

  - 表的运算：

    - SELECT ... FROM <table_name_1> UNION (ALL) SELECT ... FROM <table_name_2>; -- 并集（是否去重）

    - SELECT ... FROM <table_name_1>  INTERSECT (ALL) SELECT ... FROM <table_name_2>; -- 交集

    - SELECT ... FROM <table_name_1>  EXCEPT (ALL) SELECT ... FROM <table_name_2>; -- 差集

  - 联结：

    - SELECT <line_name> FROM <table_name_1> INNER JOIN <table_ame_2> ON <expr>; -- 依条件（联结键）内联

    - SELECT <line_name> FROM <table_name_1> (LEFT/RIGHT) OUTER JOIN <table_ame_2> ON <expr>; -- 依条件（以左/右表为主表）外联

    - SELECT <line_name> FROM <table_name_1> CROSS JOIN <table_ame_2>; -- 交叉联结（笛卡尔积）

- 8. SQL高级处理

  - 窗口（OLAP，OnLine Analytical Processing）函数

    - 语法：<olap_func> OVER ( [PARTITION BY <line_list>] ORDER BY <ord_line_list>)

      - <olap_func>：一部分聚合函数分专用的窗口函数

      - PARTITION BY：横向上分组为窗口（范围）

      - ORDER BY：纵向上排序

    - 专用窗口函数：

      - RANK：排序次位，存在相同次位时跳过

      - DENSE_RANK：存在相同次位时不跳过

      - ROW_NUMBER：唯一连续次位

    - 窗口函数只能在SELECT子句中使用

  - GROUPING运算符：

    - ROLLUP：计算合计与小计（超级分组）
      - 例如：
        - SELECT product_type, regist_date, SUM(sale_price) AS sum_price FROM Product GROUP BY ROLLUP(product_type, regist_date);

    - GROUPING：参数列中的值为超级分组产生的NULL时返回1，其他返回0

      - 例如：

        - SELECT CASE WHEN GROUPING(product_type) = 1 

          - THEN '商品种类 合计'

          - ELSE product_type END AS product_type, 

        - CASE WHEN GROUPING(regist_date) = 1

          - THEN '登记日期 合计'

          - ELSE CAST(regist_date AS VARCHAR(16)) END AS regist_date,

        - SUM(sale_price) AS sum_price FROM Product

        - GROUP BY ROLLUP(product_type, regist_date);

    - CUBE：将聚合键的所有可能组合进行汇总

    - GROUPING SETS：从ROLLUP或CUBE的结果中取出部分