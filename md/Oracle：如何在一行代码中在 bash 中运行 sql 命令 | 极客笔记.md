> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [deepinout.com](https://deepinout.com/oracle/oracle-questions/961_oracle_sqlplus_oracle_how_can_i_run_sql_command_on_bash_in_1_line.html)

> Oracle：如何在一行代码中在 bash 中运行 sql 命令 在本文中，我们将介绍如何在一行代码中在 bash 中运行 Oracle SQL 命令。

Oracle：如何在一行代码中在 bash 中运行 sql 命令
================================

在本文中，我们将介绍如何在一行代码中在 bash 中运行 Oracle SQL 命令。使用 Oracle Sqlplus 工具可以在命令行界面中执行 SQL 语句。这是与 Oracle 数据库连接，并运行 SQL 命令的主要工具之一。我们将使用 Sqlplus 命令来演示在 bash 中运行 SQL 命令的几种方法。

**阅读更多：[Oracle 教程](https://deepinout.com/oracle)**

方法 1：使用 - e 选项
--------------

使用 Sqlplus 的 - e 选项可以在一行中运行 SQL 命令。该选项可以让我们直接在命令行中传递 SQL 语句，而无需通过脚本或文件来执行。

以下是在 bash 中使用 Sqlplus 的 - e 选项运行 SQL 命令的示例：

```
sqlplus -s username/password@database_server:port/service_name <<EOF
SQL语句
EOF

```

SQLCopy

其中，-s 选项用于禁止 Sqlplus 显示横幅信息，以及登录成功和退出成功的提示。

示例：

```
sqlplus -s scott/tiger@testdb <<EOF
SELECT * FROM employees;
EOF

```

SQLCopy

上述示例中，将通过用户名、密码和数据库连接字符串连接到名为 testdb 的数据库中，并运行 SELECT * FROM employees; 查询。

方法 2：使用 here 文档和管道
------------------

另一种在一行中运行 SQL 命令的方法是使用 here 文档结合管道。这种方法利用了 bash 中的管道功能，将 SQL 命令作为纯文本传递给 Sqlplus。

以下是使用 here 文档和管道在 bash 中运行 SQL 命令的示例：

```
echo "SQL语句" | sqlplus -S username/password@database_server:port/service_name

```

SQLCopy

其中，-S 选项用于安静模式运行 Sqlplus，即禁止显示横幅信息。

示例：

```
echo "SELECT * FROM employees;" | sqlplus -S scott/tiger@testdb


```

SQLCopy

上述示例中，使用 echo 命令将 SQL 语句传递给 Sqlplus，并将结果输出到命令行。

方法 3：使用 sql 脚本文件
----------------

除了在一行中直接运行 SQL 命令外，还可以通过运行包含 SQL 语句的脚本文件来执行。这对于包含多个 SQL 语句或复杂 SQL 逻辑的情况非常有用。

以下是使用 bash 运行 SQL 脚本文件的示例：

```
sqlplus -s username/password@database_server:port/service_name @script.sql


```

SQLCopy

其中，@符号用于指定要运行的脚本文件。

示例：

```
sqlplus -s scott/tiger@testdb @query.sql


```

SQLCopy

上述示例中，将通过用户名、密码和数据库连接字符串连接到名为 testdb 的数据库中，并执行 query.sql 脚本文件中的 SQL 语句。

总结
--

本文介绍了在一行中在 bash 中运行 Oracle SQL 命令的几种方法。我们可以使用 Sqlplus 工具的 - e 选项直接传递 SQL 语句，或者使用 here 文档和管道将 SQL 命令作为纯文本传递给 Sqlplus。此外，还可以通过运行包含 SQL 语句的脚本文件来执行复杂的 SQL 逻辑。根据需求选择适合的方法，可以在命令行中方便地运行 Oracle SQL 命令。