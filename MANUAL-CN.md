# Ora2Pg 中文操作手册

## 简介

Ora2Pg 是一个免费工具，用于将 Oracle 数据库迁移到 PostgreSQL 兼容的模式。它连接到您的 Oracle 数据库，自动扫描并提取数据库结构或数据，然后生成可以导入到 PostgreSQL 数据库的 SQL 脚本。

Ora2Pg 应用场景广泛，从 Oracle 数据库逆向工程到大型企业数据库迁移，或者仅仅是将部分 Oracle 数据复制到 PostgreSQL 数据库。它使用简单，除了提供连接 Oracle 数据库所需的参数外，无需任何 Oracle 数据库专业知识。

## 特性

Ora2Pg 由一个 Perl 脚本 (`ora2pg`) 和一个 Perl 模块 (`Ora2Pg.pm`) 组成。您只需要修改配置文件 `ora2pg.conf`，设置连接 Oracle 数据库的 DSN，并可选择指定模式名称。完成后，只需设置所需的导出类型：TABLE（含约束）、VIEW、MVIEW、TABLESPACE、SEQUENCE、INDEXES、TRIGGER、GRANT、FUNCTION、PROCEDURE、PACKAGE、PARTITION、TYPE、INSERT 或 COPY、FDW、QUERY、KETTLE、SYNONYM。

默认情况下，Ora2Pg 导出到文件，您可以使用 `psql` 客户端将其加载到 PostgreSQL 中。您也可以通过在配置文件中设置 PostgreSQL 的 DSN，直接导入到 PostgreSQL 数据库。通过 `ora2pg.conf` 的各种配置选项，您可以完全控制导出内容和导出方式。

**主要功能包括：**

*   导出完整的数据库模式（表、视图、序列、索引），包括唯一约束、主键、外键和检查约束
*   导出用户和组的权限/授权
*   导出范围/列表分区和子分区
*   导出指定的表（通过表名选择）
*   将 Oracle 模式导出为 PostgreSQL 8.4+ 模式
*   导出预定义的函数、触发器、存储过程、包和包体
*   导出全部数据或根据 WHERE 条件导出部分数据
*   完全支持将 Oracle BLOB 对象转换为 PostgreSQL BYTEA
*   将 Oracle 视图导出为 PostgreSQL 表
*   导出 Oracle 用户自定义类型
*   提供 PL/SQL 到 PL/pgSQL 的基本自动代码转换
*   跨平台支持
*   将 Oracle 表导出为外部数据包装器（FDW）表
*   导出物化视图
*   生成 Oracle 数据库内容报告
*   评估 Oracle 数据库迁移成本
*   评估 Oracle 数据库迁移难度等级
*   评估文件中 PL/SQL 代码的迁移成本
*   评估文件中 Oracle SQL 查询的迁移成本
*   生成用于 Pentaho Data Integrator (Kettle) 的 XML ktr 文件
*   将 Oracle Locator 和空间几何数据导出到 PostGIS
*   将 DBLINK 导出为 Oracle FDW
*   将 SYNONYM 导出为视图
*   将 DIRECTORY 导出为外部表或 `external_file` 扩展目录
*   在多个 PostgreSQL 连接上分发 SQL 命令
*   在 Oracle 和 PostgreSQL 数据库之间执行差异对比（用于测试）
*   支持 MySQL/MariaDB 和 Microsoft SQL Server 迁移

## 安装

### 依赖要求

**必需组件：**

*   **Oracle Instant Client 或完整 Oracle 安装**：系统必须安装。可从 Oracle 官方下载中心获取
*   **Perl**：需要 Perl 5.10 或更高版本
*   **DBI**：Perl 模块 `DBI` 版本需 > 1.614
*   **DBD::Oracle**：用于连接 Oracle 数据库的 Perl 模块
*   **DBD::MySQL**（可选）：如需迁移 MySQL 数据库
*   **DBD::ODBC**（可选）：如需迁移 SQL Server 数据库

使用 CPAN 安装所需的 Perl 模块：

```bash
perl -MCPAN -e 'install DBD::Oracle'
perl -MCPAN -e 'install DBD::MySQL'
perl -MCPAN -e 'install DBD::ODBC'
```

**可选组件：**

*   **DBD::Pg**：如需直接将数据导入 PostgreSQL
*   **Compress::Zlib**：如需使用 gzip 压缩输出文件

### 源码安装

按照以下步骤从源码安装 Ora2Pg：

```bash
tar xjf ora2pg-x.x.tar.bz2
cd ora2pg-x.x/
perl Makefile.PL
make && make install
```

这将把 `Ora2Pg.pm` 安装到 Perl 站点库目录，`ora2pg` 安装到 `/usr/local/bin/`，`ora2pg.conf` 安装到 `/etc/ora2pg/`。

在 **Windows** 系统上，使用 `gmake` 替代 `make`。

安装到自定义目录：

```bash
perl Makefile.PL PREFIX=<安装目录>
make && make install
```

### 打包构建

源代码的 `packaging/` 目录包含为各种 Linux 发行版（RPM、Debian、Slackware）构建二进制包所需的文件和说明。

*   **RPM (RedHat/Fedora)：**
    1.  将源码包复制到 `~/rpmbuild/SOURCES/`
    2.  运行 `rpmbuild -bb ora2pg.spec`

*   **Debian：**
    1.  在 `debian/` 目录下运行 `sh create-deb-tree.sh` 创建包目录结构
    2.  运行 `dpkg -b ora2pg ora2pg.deb` 生成 deb 包

*   **Slackware：**
    1.  将源码包复制到 `slackbuild/` 目录
    2.  运行 `sh Ora2Pg.SlackBuild`

## 配置

Ora2Pg 的所有行为通过配置文件 `ora2pg.conf` 控制。默认位置为 `/etc/ora2pg/`。

### Oracle 数据库连接

连接 Oracle 数据库的关键配置参数：

*   `ORACLE_HOME`：设置 Oracle 主目录环境变量
*   `ORACLE_DSN`：设置数据源名称（DSN）。例如：`dbi:Oracle:host=oradb.host.com;sid=DB_SID;port=1521`
*   `ORACLE_USER`：数据库连接用户名
*   `ORACLE_PWD`：数据库连接密码

### 模式导出

*   `SCHEMA`：指定要导出的 Oracle 模式名称
*   `EXPORT_SCHEMA`：设为 `1` 在 PostgreSQL 中创建同名模式
*   `CREATE_SCHEMA`：设为 `1` 在输出文件中包含 `CREATE SCHEMA` 语句

## 命令行使用

默认情况下，Ora2Pg 查找 `/etc/ora2pg/ora2pg.conf` 配置文件。直接运行：

```bash
ora2pg
```

使用不同的配置文件，使用 `-c` 参数：

```bash
ora2pg -c /path/to/your/ora2pg.conf
```

### 常用参数

*   `-a | --allow str`：允许导出的对象列表（逗号分隔）
*   `-b | --basedir dir`：设置默认输出目录
*   `-c | --conf file`：指定替代配置文件
*   `-d | --debug`：启用详细输出
*   `-e | --exclude str`：从导出中排除的对象列表（逗号分隔）
*   `-h | --help`：显示帮助信息
*   `-i | --input file`：指定包含 PL/SQL 代码的文件进行转换，无需连接数据库
*   `-j | --jobs num`：并行向 PostgreSQL 发送数据的进程数
*   `-l | --log file`：设置日志文件
*   `-n | --namespace schema`：设置要提取的 Oracle 模式
*   `-o | --out file`：设置输出文件名，默认为 `output.sql`
*   `-p | --plsql`：启用 PL/SQL 到 PL/pgSQL 的代码转换
*   `-q | --quiet`：禁用进度条
*   `-t | --type export`：设置导出类型（如 `TABLE`、`COPY`、`VIEW`）。覆盖配置文件中的 `TYPE` 设置
*   `-u | --user name`：设置 Oracle 数据库连接用户
*   `-v | --version`：显示 Ora2Pg 版本并退出
*   `-w | --password pwd`：设置 Oracle 数据库用户密码
*   `--estimate_cost`：激活迁移成本评估（与 `SHOW_REPORT` 配合使用）
*   `--init_project name`：初始化典型的 ora2pg 项目目录结构

### 使用示例

*   **测试连接并显示 Oracle 版本：**
    ```bash
    ora2pg -t SHOW_VERSION -c config/ora2pg.conf
    ```

*   **导出指定表结构：**
    ```bash
    ora2pg -c ora2pg.conf -t TABLE -a MY_TABLE -o my_table.sql
    ```

*   **以 COPY 格式导出指定表数据：**
    ```bash
    ora2pg -c ora2pg.conf -t COPY -a MY_TABLE -o my_table_data.sql
    ```

*   **创建迁移项目模板：**
    ```bash
    ora2pg --project_base /app/migration/ --init_project test_project
    ```

## 开发

Ora2Pg 是一个 Perl 项目。开发者可以添加自定义选项。`ora2pg.conf` 中的任何配置指令都可以作为小写参数传递给新的 `Ora2Pg` 对象实例。请参考 `ora2pg` 脚本代码了解如何添加自定义选项。
