# Ora2Pg 中文操作手册

## 简介

Ora2Pg 是一个免费工具，用于将 Oracle 数据库迁移到与 PostgreSQL 兼容的模式。它会连接到您的 Oracle 数据库，自动扫描并提取其结构或数据，然后生成可以加载到 PostgreSQL 数据库的 SQL 脚本。

Ora2Pg 的用途广泛，从逆向工程 Oracle 数据库，到大型企业数据库迁移，或仅仅是将某些 Oracle 数据复制到 PostgreSQL 数据库。它非常易于使用，除了提供连接 Oracle 数据库所需的参数外，不需要任何 Oracle 数据库知识。

## 特性

Ora2Pg 由一个 Perl 脚本 (`ora2pg`) 和一个 Perl 模块 (`Ora2Pg.pm`) 组成。您唯一需要修改的是配置文件 `ora2pg.conf`，通过设置 DSN 连接到 Oracle 数据库，并可选择性地指定模式名称。完成后，您只需设置所需的导出类型：TABLE（带约束）、VIEW、MVIEW、TABLESPACE、SEQUENCE、INDEXES、TRIGGER、GRANT、FUNCTION、PROCEDURE、PACKAGE、PARTITION、TYPE、INSERT 或 COPY、FDW、QUERY、KETTLE、SYNONYM。

默认情况下，Ora2Pg 导出到一个文件，您可以使用 `psql` 客户端加载到 PostgreSQL 中。但您也可以通过在配置文件中设置其 DSN，直接导入到 PostgreSQL 数据库。通过 `ora2pg.conf` 的所有配置选项，您可以完全控制导出的内容和方式。

**主要功能包括：**

*   导出完整的数据库模式（表、视图、序列、索引），包括唯一、主键、外键和检查约束。
*   导出用户和组的授权/权限。
*   导出范围/列表分区和子分区。
*   导出选定的表（通过指定表名）。
*   将 Oracle 模式导出为 PostgreSQL 8.4+ 模式。
*   导出预定义的函数、触发器、过程、包和包体。
*   导出全部数据或根据 WHERE 子句导出部分数据。
*   完全支持将 Oracle BLOB 对象作为 PG BYTEA。
*   将 Oracle 视图导出为 PG 表。
*   导出 Oracle 用户自定义类型。
*   提供一些从 PL/SQL 到 PL/PGSQL 的基本自动代码转换。
*   跨平台工作。
*   将 Oracle 表导出为外部数据包装器（FDW）表。
*   导出物化视图。
*   显示 Oracle 数据库内容的报告。
*   评估 Oracle 数据库的迁移成本。
*   评估 Oracle 数据库的迁移难度级别。
*   评估文件中 PL/SQL 代码的迁移成本。
*   评估文件中存储的 Oracle SQL 查询的迁移成本。
*   生成用于 Pentaho Data Integrator (Kettle) 的 XML ktr 文件。
*   将 Oracle locator 和空间几何图形导出到 PostGIS。
*   将 DBLINK 导出为 Oracle FDW。
*   将 SYNONYMS 导出为视图。
*   将 DIRECTORY 导出为外部表或 `external_file` 扩展的目录。
*   在多个 PostgreSQL 连接上分派 SQL 命令列表。
*   为测试目的，在 Oracle 和 PostgreSQL 数据库之间执行差异比较。
*   支持 MySQL/MariaDB 和 Microsoft SQL Server 迁移。

## 安装

### 依赖

**必需：**

*   **Oracle Instant Client 或完整的 Oracle 安装**：必须在系统上安装。您可以从 Oracle 下载中心下载。
*   **Perl**: 需要 Perl 5.10 或更高版本。
*   **DBI**: Perl 模块 `DBI` 版本需 > 1.614。
*   **DBD::Oracle**: 用于连接 Oracle 的 Perl 模块。
*   **DBD::MySQL** (可选): 如果您计划迁移 MySQL 数据库。
*   **DBD::ODBC** (可选): 如果您计划迁移 SQL Server 数据库。

您可以使用 CPAN 安装所需的 Perl 模块：

```bash
perl -MCPAN -e 'install DBD::Oracle'
perl -MCPAN -e 'install DBD::MySQL'
perl -MCPAN -e 'install DBD::ODBC'
```

**可选：**

*   **DBD::Pg**: 如果您想直接将数据导入 PostgreSQL。
*   **Compress::Zlib**: 如果您想使用 gzip 压缩输出文件。

### 从源码安装

您可以按照以下步骤从源码安装 Ora2Pg：

```bash
tar xjf ora2pg-x.x.tar.bz2
cd ora2pg-x.x/
perl Makefile.PL
make && make install
```

这会将 `Ora2Pg.pm` 安装到您的 Perl 站点仓库，`ora2pg` 安装到 `/usr/local/bin/`，`ora2pg.conf` 安装到 `/etc/ora2pg/`。

在 **Windows** 上，您可以使用 `gmake` 代替 `make`。

要安装到自定义目录，请使用：

```bash
perl Makefile.PL PREFIX=<your_install_dir>
make && make install
```

### 打包

源代码的 `packaging/` 目录包含了为各种 Linux 发行版（RPM, Debian, Slackware）构建二进制包所需的文件和说明。

*   **RPM (RedHat/Fedora):**
    1.  将源码包复制到 `~/rpmbuild/SOURCES/`。
    2.  运行 `rpmbuild -bb ora2pg.spec`。

*   **Debian:**
    1.  在 `debian/` 目录下，运行 `sh create-deb-tree.sh` 创建包目录结构。
    2.  运行 `dpkg -b ora2pg ora2pg.deb` 生成 deb 包。

*   **Slackware:**
    1.  将源码包复制到 `slackbuild/` 目录。
    2.  运行 `sh Ora2Pg.SlackBuild`。

## 配置

Ora2Pg 的所有行为都通过一个名为 `ora2pg.conf` 的配置文件控制。默认情况下，它位于 `/etc/ora2pg/`。

### Oracle 数据库连接

以下是连接 Oracle 数据库的关键配置指令：

*   `ORACLE_HOME`: 设置 Oracle 主目录的环境变量。
*   `ORACLE_DSN`: 设置数据源名称（DSN）。例如：`dbi:Oracle:host=oradb.host.com;sid=DB_SID;port=1521`。
*   `ORACLE_USER`: 连接数据库的用户名。
*   `ORACLE_PWD`: 连接数据库的密码。

### 模式导出

*   `SCHEMA`: 指定要导出的 Oracle 模式名称。
*   `EXPORT_SCHEMA`: 设为 `1` 以在 PostgreSQL 中创建同名模式。
*   `CREATE_SCHEMA`: 设为 `1` 以在输出文件中包含 `CREATE SCHEMA` 语句。

## 命令行 (CLI) 用法

默认情况下，Ora2Pg 会查找 `/etc/ora2pg/ora2pg.conf` 配置文件。您可以直接运行：

```bash
ora2pg
```

要使用不同的配置文件，请使用 `-c` 参数：

```bash
ora2pg -c /path/to/your/ora2pg.conf
```

### 常用参数

*   `-a | --allow str`: 允许导出的对象列表（逗号分隔）。
*   `-b | --basedir dir`: 设置默认输出目录。
*   `-c | --conf file`: 指定备用配置文件。
*   `-d | --debug`: 启用详细输出。
*   `-e | --exclude str`: 从导出中排除的对象列表（逗号分隔）。
*   `-h | --help`: 打印帮助信息。
*   `-i | --input file`: 指定包含 PL/SQL 代码的文件进行转换，不连接数据库。
*   `-j | --jobs num`: 并行向 PostgreSQL 发送数据的进程数。
*   `-l | --log file`: 设置日志文件。
*   `-n | --namespace schema`: 设置要从中提取的 Oracle 模式。
*   `-o | --out file`: 设置输出文件名，默认为 `output.sql`。
*   `-p | --plsql`: 启用 PL/SQL 到 PL/PGSQL 的代码转换。
*   `-q | --quiet`: 禁用进度条。
*   `-t | --type export`: 设置导出类型（例如 `TABLE`, `COPY`, `VIEW`）。这将覆盖配置文件中的 `TYPE` 设置。
*   `-u | --user name`: 设置 Oracle 数据库连接用户。
*   `-v | --version`: 显示 Ora2Pg 版本并退出。
*   `-w | --password pwd`: 设置 Oracle 数据库用户密码。
*   `--estimate_cost`: 激活迁移成本评估（与 `SHOW_REPORT` 一起使用）。
*   `--init_project name`: 初始化一个典型的 ora2pg 项目目录结构。

### 示例

*   **测试连接并显示 Oracle 版本：**
    ```bash
    ora2pg -t SHOW_VERSION -c config/ora2pg.conf
    ```

*   **导出一个特定的表结构：**
    ```bash
    ora2pg -c ora2pg.conf -t TABLE -a MY_TABLE -o my_table.sql
    ```

*   **以 COPY 格式导出特定表的数据：**
    ```bash
    ora2pg -c ora2pg.conf -t COPY -a MY_TABLE -o my_table_data.sql
    ```

*   **创建一个迁移项目模板：**
    ```bash
    ora2pg --project_base /app/migration/ --init_project test_project
    ```

## 开发

Ora2Pg 是一个 Perl 项目。开发者可以添加自定义选项。任何在 `ora2pg.conf` 中的配置指令都可以作为小写参数传递给新的 `Ora2Pg` 对象实例。请参阅 `ora2pg` 脚本代码了解如何添加您自己的选项。
