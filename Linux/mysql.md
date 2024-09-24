### mysql

#### mysql安装

```bash
sudo apt install mysql-server

# 查看mysql状态
# Active: active (running) 表示MySQL当前正在运行。
# Loaded: loaded (/lib/systemd/system/mysql.service; enabled) 表示MySQL已经设置为开机自启，enabled状态代表开机启动
systemctl status mysql

# -u root：-u表示指定要登录的用户名，root是你要使用的MySQL用户名。在这里你选择用MySQL的root用户登录数据库
# -p：告诉MySQL客户端在连接时提示你输入密码。执行该命令后，终端会提示你输入root用户的密码
sudo mysql -u root -p
```

#### 常用指令

```mysql
# 查询MySQL用户列表
# plugin（插件）用于控制用户的身份验证方式（authentication method），即MySQL如何验证某个用户是否可以登录和访问数据库
# mysql_native_password	基于用户名和密码进行验证
# caching_sha2_password	使用SHA-256算法来加密密码
# auth_socket	通过本地操作系统的身份验证（如Linux上的unix_socket）来登录，而不能通过TCP/IP远程登录
select user, host, plugin from mysql.user where user = 'root';

# 查询用户权限
show grants for 'username'@'hostname';
# 要查看root用户在localhost（本地）的权限
show grants for 'root'@'localhost'

# 查询数据库列表
show databases;

# 创建数据库并使用UTF-8编码
create database test_db character set utf8mb4 collate utf8mb4_general_ci;

# 查询数据库的创建语句
show create database test_db;
# | test_db  | CREATE DATABASE test_db /*!40100 DEFAULT CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci */ /*!80016 DEFAULT ENCRYPTION='N' */ |
# /*!...*/ 这样的语法是版本控制注释（versioned comments），用于在不同的MySQL服务器版本之间保持兼容性
# /*!40100 ... */：这个表示在MySQL 4.1.0及以后版本中，执行其中的语句或选项

# 使用数据库
use test_db;

# 查询当前使用的数据库
select database();

# 查询当前登录的用户
select user();

# 查看数据库中所有表项
show tables;

# 查看表的结构
describe table_name;
show columns from table_name;
```

#### 新建用户

```mysql
# 创建用户admin，允许其从任何主机（%）登录
# 将'%'替换为特定的IP地址或主机名来限制登录来源，例如'admin'@'192.168.1.100'
# IDENTIFIED BY 'your_password'：设置用户的密码
create user 'admin'@'%' identified by '06305715';

# 授予admin用户有所有数据库的所有权限
# *.*：表示所有数据库的所有表
# 允许admin用户将权限授予其他用户
grant all privileges ON *.* TO 'admin'@'%' with grant option;

# 刷新权限
flush privileges;

# 查询用户权限
show grants for 'admin'@'%';
```

#### 数据类型

##### 数值类型

| 类型         | 大小       | 范围（有符号）                                          | 范围（无符号）                  | 用途            |
| :----------- | :--------- | :------------------------------------------------------ | :------------------------------ | :-------------- |
| TINYINT      | 1 Bytes    | (-128，127)                                             | (0，255)                        | 小整数值        |
| SMALLINT     | 2 Bytes    | (-32 768，32 767)                                       | (0，65 535)                     | 大整数值        |
| MEDIUMINT    | 3 Bytes    | (-8 388 608，8 388 607)                                 | (0，16 777 215)                 | 大整数值        |
| INT或INTEGER | 4 Bytes    | (-2 147 483 648，2 147 483 647)                         | (0，4 294 967 295)              | 大整数值        |
| BIGINT       | 8 Bytes    | (-9,223,372,036,854,775,808，9 223 372 036 854 775 807) | (0，18 446 744 073 709 551 615) | 极大整数值      |
| FLOAT        | 4 Bytes    | (-3.40 E+38，-1.18 E-38)，0，(1.18 E-38，3.40 E+38)     | 0，(1.18 E-38，3.40 E+38)       | 单精度 浮点数值 |
| DOUBLE       | 8 Bytes    | (-1.80 E+308，-2.23 E-308)，0，(2.23 E-308，1.80 E+308) | 0，(2.23 E-308，1.80 E+308)     | 双精度 浮点数值 |
| DECIMAL      | max(M,D)+2 | 依赖于M和D的值                                          | 依赖于M和D的值                  | 小数值          |

##### 日期和时间类型

| 类型      | 大小 ( bytes) | 范围                                                         | 格式                | 用途                     |
| :-------- | :------------ | :----------------------------------------------------------- | :------------------ | :----------------------- |
| DATE      | 3             | 1000-01-01/9999-12-31                                        | YYYY-MM-DD          | 日期值                   |
| TIME      | 3             | '-838:59:59'/'838:59:59'                                     | HH:MM:SS            | 时间值或持续时间         |
| YEAR      | 1             | 1901/2155                                                    | YYYY                | 年份值                   |
| DATETIME  | 8             | '1000-01-01 00:00:00' 到 '9999-12-31 23:59:59'               | YYYY-MM-DD hh:mm:ss | 混合日期和时间值         |
| TIMESTAMP | 4             | '1970-01-01 00:00:01' UTC 到 '2038-01-19 03:14:07' UTC结束时间是第 **2147483647** 秒，北京时间 **2038-1-19 11:14:07**，格林尼治时间 2038年1月19日 凌晨 03:14:07 | YYYY-MM-DD hh:mm:ss | 混合日期和时间值，时间戳 |

##### 字符串类型

| 类型       | 大小                  | 用途                            |
| :--------- | :-------------------- | :------------------------------ |
| CHAR       | 0-255 bytes           | 定长字符串                      |
| VARCHAR    | 0-65535 bytes         | 变长字符串                      |
| TINYBLOB   | 0-255 bytes           | 不超过 255 个字符的二进制字符串 |
| TINYTEXT   | 0-255 bytes           | 短文本字符串                    |
| BLOB       | 0-65 535 bytes        | 二进制形式的长文本数据          |
| TEXT       | 0-65 535 bytes        | 长文本数据                      |
| MEDIUMBLOB | 0-16 777 215 bytes    | 二进制形式的中等长度文本数据    |
| MEDIUMTEXT | 0-16 777 215 bytes    | 中等长度文本数据                |
| LONGBLOB   | 0-4 294 967 295 bytes | 二进制形式的极大文本数据        |
| LONGTEXT   | 0-4 294 967 295 bytes | 极大文本数据                    |

#### 建表

```mysql
CREATE TABLE table_name (
    column1 datatype constraints,
    column2 datatype constraints,
    ...
);

# AUTO_INCREMENT：自动递增
# PRIMARY KEY：主键
# NOT NULL：不能为空
CREATE TABLE employees (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    position VARCHAR(50),
    hire_date DATE,
    salary DECIMAL(10, 2)
);
```

### c操作mysql

[第25章：API和库_MySQL 中文文档 (mysqlzh.com)](https://www.mysqlzh.com/doc/194.html)

#### 安装mysql开发库

```bash
sudo apt-get install libmysqlclient-dev
```

#### 常用结构

```c
typedef struct MYSQL {
  NET net;                     /* 通信参数 */
  unsigned char *connector_fd; /* SSL连接的文件描述符 */
  char *host;                  /* 数据库服务器的主机名或IP地址 */
  char *user;                  /* 数据库连接的用户名 */
  char *passwd;                /* 数据库连接的密码 */
  char *unix_socket;           /* Unix socket连接的路径 */
  char *server_version;        /* MySQL服务器版本信息 */
  char *host_info;             /* 关于主机的信息 */
  char *info;                  /* 当前连接的信息字符串 */
  char *db;                    /* 当前使用的数据库名称 */
  struct CHARSET_INFO *charset;/* 当前字符集信息 */
  MYSQL_FIELD *fields;         /* 查询结果中的字段信息 */
  struct MEM_ROOT *field_alloc;/* 字段内存管理结构 */
  uint64_t affected_rows;      /* 受影响的行数（例如，UPDATE后） */
  uint64_t insert_id;          /* 最后插入的行ID */
  unsigned long thread_id;     /* 在服务器中的连接线程ID */
  unsigned long packet_length;  /* 当前数据包的长度 */
  unsigned int port;           /* 数据库服务器的端口号 */
  unsigned long client_flag;   /* 客户端连接的标志位，指示特性 */
  unsigned long server_capabilities; /* 服务器的能力标志 */
  unsigned int protocol_version;/* MySQL协议版本 */
  unsigned int field_count;    /* 查询返回的字段数量 */
  unsigned int server_status;   /* 服务器状态标志 */
  unsigned int warning_count;   /* 警告计数 */
  struct st_mysql_options options; /* 客户端选项设置 */
  enum mysql_status status;     /* 当前连接的状态 */
  bool free_me;                /* 指示是否在关闭连接时释放该结构体 */
  bool reconnect;              /* 指示是否允许自动重连 */

  char scramble[SCRAMBLE_LENGTH + 1]; /* 用于认证的随机字符串 */

  LIST *stmts;                 /* 存储所有语句的列表 */
  const struct MYSQL_METHODS *methods; /* 支持的操作方法 */
  void *thd;                   /* 当前线程的指针 */

  bool *unbuffered_fetch_owner;/* 控制结果集的标志 */
  void *extension;             /* 扩展指针 */
} MYSQL;
```

```c
typedef struct MYSQL_RES {
  uint64_t row_count;                  /* 行数 */
  MYSQL_FIELD *fields;                 /* 字段信息指针 */
  struct MYSQL_DATA *data;             /* 数据指针 */
  MYSQL_ROWS *data_cursor;             /* 数据游标 */
  unsigned long *lengths;              /* 当前行的列长度 */
  MYSQL *handle;                       /* 用于未缓冲读取的连接句柄 */
  const struct MYSQL_METHODS *methods; /* 支持的方法 */
  MYSQL_ROW row;                       /* 如果是未缓冲读取，当前行数据 */
  MYSQL_ROW current_row;               /* 当前行的缓冲数据 */
  struct MEM_ROOT *field_alloc;        /* 字段内存管理结构 */
  unsigned int field_count;            /* 字段数量 */
  unsigned int current_field;          /* 当前字段索引 */
  bool eof;                            /* 是否到达结果集末尾 */
  bool unbuffered_fetch_cancelled;     /* 未缓冲读取被取消标志 */
  enum enum_resultset_metadata metadata; /* 结果集元数据类型 */
  void *extension;                     /* 扩展指针 */
} MYSQL_RES;
```

```c
typedef char **MYSQL_ROW;		// 定义一个新的类型 MYSQL_ROW，它是一个指向字符串数组的指针
```

```c
/**
 * @brief 描述 MySQL 查询结果集中每个字段的元数据，即列
 */
typedef struct MYSQL_FIELD {
    char *name;               /* 字段名称 */
    char *org_name;           /* 原始字段名称（如果是别名） */
    char *table;              /* 字段所属的表名 */
    char *org_table;          /* 原始表名（如果表是别名） */
    char *db;                 /* 字段所在数据库 */
    char *catalog;            /* 字段的目录 */
    char *def;                /* 默认值（由 mysql_list_fields 设置） */
    unsigned long length;     /* 字段创建时的宽度 */
    unsigned long max_length; /* 选定集的最大宽度 */
    unsigned int name_length; /* 字段名称的长度 */
    unsigned int org_name_length; /* 原始名称的长度 */
    unsigned int table_length; /* 表名的长度 */
    unsigned int org_table_length; /* 原始表名的长度 */
    unsigned int db_length;   /* 数据库名称的长度 */
    unsigned int catalog_length; /* 目录名称的长度 */
    unsigned int def_length;  /* 默认值的长度 */
    unsigned int flags;       /* 字段的标志位 */
    unsigned int decimals;    /* 字段的小数位数 */
    unsigned int charsetnr;   /* 字符集编号 */
    enum enum_field_types type; /* 字段类型（见 mysql_com.h 中的类型定义） */
    void *extension;          /* 供扩展使用的指针 */
} MYSQL_FIELD;
```

```c
/* 预处理语句句柄 */
typedef struct MYSQL_STMT
{
	struct MEM_ROOT *mem_root; /* 用于根内存分配的指针，管理内存分配 */
	LIST list;				   /* 用于跟踪所有准备语句的链表 */
	MYSQL *mysql;			   /* 连接句柄，指向当前 MySQL 连接 */
	MYSQL_BIND *params;		   /* 输入参数的绑定数组 */
	MYSQL_BIND *bind;		   /* 输出参数的绑定数组 */
	MYSQL_FIELD *fields;	   /* 结果集的元数据，描述结果集中的列 */
	MYSQL_DATA result;		   /* 缓存的结果集数据 */
	MYSQL_ROWS *data_cursor;   /* 当前行在缓存结果集中的位置指针 */
	/*
	  mysql_stmt_fetch() 调用此函数以获取一行数据。此函数
	  针对不同的取回方式（缓冲、非缓冲和游标）有所不同。
	*/
	int (*read_row_func)(struct MYSQL_STMT *stmt, unsigned char **row);
	/* 执行语句后受影响的行数的副本 */
	uint64_t affected_rows;
	uint64_t insert_id;			 /* 复制的插入 ID，来自 mysql->insert_id */
	unsigned long stmt_id;		 /* 准备语句的唯一标识符 */
	unsigned long flags;		 /* 游标的类型或其他标志 */
	unsigned long prefetch_rows; /* 每次 COM_FETCH 命令预取的行数 */
	/*
	  从 mysql->server_status 复制，表示执行/获取后的
	  服务器端游标状态。
	*/
	unsigned int server_status;
	unsigned int last_errno;			/* 最后一次操作的错误代码 */
	unsigned int param_count;			/* 输入参数的数量 */
	unsigned int field_count;			/* 结果集中的列数 */
	enum enum_mysql_stmt_state state;	/* 当前语句的状态 */
	char last_error[MYSQL_ERRMSG_SIZE]; /* 最后一次错误的消息 */
	char sqlstate[SQLSTATE_LENGTH + 1]; /* SQL 错误状态代码 */
	/* 指示要发送到服务器的输入参数类型 */
	bool send_types_to_server;
	bool bind_param_done; /* 指示输入缓冲区是否已提供 */
	unsigned char bind_result_done; /* 指示输出缓冲区是否已提供 */
	/* mysql_stmt_close() 需要取消此结果 */
	bool unbuffered_fetch_cancelled;
	/*
	  如果在执行 mysql_stmt_store_result 时需要计算字段的
	  max_length，则此标志设置为 true。
	*/
	bool update_max_length;
	struct MYSQL_STMT_EXT *extension; /* 用于扩展功能的指针 */
} MYSQL_STMT;
```

```c
/**
 * @struct MYSQL_BIND
 * @brief 用于绑定参数和结果集的结构体。
 *
 * 此结构体在执行预处理语句时用于传递参数和接收结果数据。
 */
typedef struct MYSQL_BIND
{
	unsigned long *length; /* 输出长度指针，指向存储结果数据长度的变量 */
	bool *is_null;		   /* 指向 NULL 指示符的指针，指示字段是否为 NULL */
	void *buffer;		   /* 数据的缓冲区，用于存储输入或输出的数据 */
	/* 如果希望跟踪提取过程中发生的数据截断，请设置此指针 */
	bool *error; /* 指向错误指示符的指针，指示是否发生错误 */
	unsigned char *row_ptr; /* 当前数据位置的指针，用于访问结果集中的行数据 */
	/* 用于存储参数的函数指针 */
	void (*store_param_func)(NET *net, struct MYSQL_BIND *param);
	/* 用于提取结果的函数指针 */
	void (*fetch_result)(struct MYSQL_BIND *, MYSQL_FIELD *, unsigned char **row);
	/* 跳过结果的函数指针 */
	void (*skip_result)(struct MYSQL_BIND *, MYSQL_FIELD *, unsigned char **row);
	/* 输出缓冲区长度，提取字符串/二进制数据时必须设置 */
	unsigned long buffer_length; /* 用于指示缓冲区的长度 */
	unsigned long offset; /* 字符串/二进制数据提取的偏移位置 */
	unsigned long length_value; /* 当 length 为 0 时使用的长度值 */
	unsigned int param_number; /* 参数编号，用于 NULL 计数和错误消息 */
	unsigned int pack_length; /* 用于打包数据的内部长度 */
	enum enum_field_types buffer_type; /* 缓冲区的数据类型 */
	bool error_value; /* 如果 error 为 0，则使用此值指示错误状态 */
	bool is_unsigned; /* 如果整数类型为无符号，则设置此标志 */
	bool long_data_used; /* 如果与 mysql_send_long_data 一起使用，则设置此标志 */
	bool is_null_value; /* 如果 is_null 为 0，则使用此值指示 NULL 状态 */
	void *extension; /* 扩展指针，用于附加信息或功能 */
} MYSQL_BIND;
```

#### 常用函数

```c
/**
 * @brief 初始化一个 MYSQL 对象。
 *
 * @param mysql 指向现有 MYSQL 对象的指针，如果传入 NULL，则会创建一个新的对象。
 * @return 返回指向初始化后的 MYSQL 对象的指针，如果失败则返回 NULL。
 */
MYSQL *STDCALL mysql_init(MYSQL *mysql);
```

```c
/**
 * 连接到 MySQL 数据库。
 *
 * @param mysql 指向已初始化的 MYSQL 对象的指针。
 * @param host 数据库主机的名称或 IP 地址。
 * @param user 连接数据库的用户名。
 * @param passwd 连接数据库的密码。
 * @param db 要连接的数据库名称。
 * @param port 数据库服务的端口号（通常为 3306），置0选择默认端口3306。
 * @param unix_socket UNIX 套接字路径（如果适用）。
 * @param clientflag 客户端连接标志（可选）。
 * @return 如果连接成功，返回指向 MYSQL 对象的指针，即mysql；否则返回 NULL。
 *
 * 此函数用于建立与 MySQL 数据库的连接，在执行其他数据库操作前必须调用。
 */
<<<<<<< HEAD
MYSQL *STDCALL mysql_real_connect(MYSQL *mysql, const char *host, const char *user, const char *passwd, const char *db, unsigned int port, const char *unix_socket, unsigned long clientflag);
=======
MYSQL *STDCALL mysql_real_connect(MYSQL *mysql, const char *host,
const char *user, const char *passwd,
const char *db, unsigned int port,
const char *unix_socket, unsigned long clientflag);
>>>>>>> d1af7d29b72d9b1dc2869af4ffbb412f0bb97e32
```

```c
/**
 * 执行一条 SQL 语句，可以写成字符串的任意 SQL 语句
 *
 * @param mysql 指向已连接的 MYSQL 对象的指针。
 * @param q 要执行的 SQL 查询字符串。
 * @return 如果查询成功，返回 0；如果出错，返回非零值。
 *
 * 此函数用于执行不返回结果集的 SQL 语句（如 INSERT、UPDATE、DELETE 等），或者准备好返回结果集的 SELECT 语句。
 * 在执行 SELECT 查询后，应使用 mysql_store_result() 或 mysql_use_result() 获取结果集。
 */
int STDCALL mysql_query(MYSQL *mysql, const char *q);
```

```c
/**
 * @brief 获取最近一次操作所影响的行数。
 *
 * 此函数返回与最近一次执行的 INSERT、UPDATE 或 DELETE 操作相关的行数。
 * 如果最近一次操作是 SELECT 查询，则返回值为 -1，表示没有受影响的行。
 *
 * @param mysql 指向 MYSQL 对象的指针，该对象必须是有效的且处于连接状态。
 * @return 返回最近一次影响的行数
 */
uint64_t mysql_affected_rows(MYSQL *mysql);
```

```c
/**
 * @brief 从当前连接中存储所有查询结果。
 *
 * 此函数会将最近执行的查询结果完全存储在内存中。
 * 适用于需要多次访问结果集的场景，支持随机访问
 *
 * @param mysql 指向 MYSQL 对象的指针，表示当前的数据库连接。
 * @return 返回指向 MYSQL_RES 对象的指针，包含查询的结果集。
 *         如果查询失败或没有结果，返回 NULL。
 */
MYSQL_RES *STDCALL mysql_store_result(MYSQL *mysql);

/**
 * @brief 使用流式方式获取查询结果。
 *
 * 此函数返回一个指向查询结果的指针，允许逐行处理结果集。
 * 适用于处理大型结果集，避免占用过多内存，仅支持顺序访问
 *
 * @param mysql 指向 MYSQL 对象的指针，表示当前的数据库连接。
 * @return 返回指向 MYSQL_RES 对象的指针，包含查询的结果集。
 *         如果查询失败或没有结果，返回 NULL。
 */
MYSQL_RES *STDCALL mysql_use_result(MYSQL *mysql);
```

```c
/**
 * @brief 获取当前结果集中的字段数量，即结果集中的列数
 *
 * @param mysql 指向 MYSQL 对象的指针，该对象必须是有效的且处于连接状态。
 * @return 返回当前结果集中的字段数量。如果没有结果集，返回 0。
 */
unsigned int mysql_field_count(MYSQL *mysql);

/**
 * @brief 获取结果集中的字段数量。
 *
 * @param res 指向 MYSQL_RES 对象的指针，该对象包含查询的结果集。
 * @return 返回结果集中的字段数量。如果结果集为空，返回 0。
 */
unsigned int mysql_num_fields(MYSQL_RES *res);

/**
 * @brief 获取结果集的列名。
 *
 * 此函数返回一个指向 MYSQL_FIELD 结构体数组的指针，
 * 该数组包含结果集中每个字段的元数据，例如名称、类型、大小等。
 *
 * @param res 指向 MYSQL_RES 对象的指针，该对象包含查询的结果集。
 * @return 返回指向 MYSQL_FIELD 结构体数组的指针。如果结果集为空或查询失败，返回 NULL。
 */
MYSQL_FIELD *STDCALL mysql_fetch_fields(MYSQL_RES *res);
```

```c
/**
 * @brief 获取结果集中的行数。
 *
 * @param res 指向 MYSQL_RES 对象的指针，该对象包含查询的结果集。
 * @return 返回结果集中的行数。如果结果集为空或查询失败，返回 0。
 */
uint64_t mysql_num_rows(MYSQL_RES *res);

/**
 * @brief 获取结果集中的下一行数据。
 *
 * 此函数返回查询结果集中的下一行数据，以 MYSQL_ROW 形式表示。
 * 数据以数组的形式返回，每个元素对应结果集中的一列。
 *
 * @param result 指向 MYSQL_RES 对象的指针，该对象包含查询的结果集。
 * @return 返回指向 MYSQL_ROW 的指针，包含当前行的数据。如果没有更多行可供返回，返回 NULL。
 */
MYSQL_ROW STDCALL mysql_fetch_row(MYSQL_RES *result);
```

```c
/**
 * @brief 释放结果集所占用的内存。
 *
 * 此函数用于释放由 mysql_store_result 或 mysql_use_result 分配的结果集内存。
 * 在不再使用结果集后调用此函数，以防止内存泄漏。
 *
 * @param result 指向 MYSQL_RES 对象的指针，该对象表示查询的结果集。
 */
void STDCALL mysql_free_result(MYSQL_RES *result);

/**
 * @brief 关闭与 MySQL 服务器的连接。
 *
 * 此函数用于关闭一个已经打开的 MySQL 连接，并释放与之相关的资源。
 * 在不再需要与数据库交互时调用此函数，以确保资源得到适当释放。
 *
 * @param sock 指向 MYSQL 对象的指针，该对象表示要关闭的数据库连接。
 */
void STDCALL mysql_close(MYSQL *sock);
```

<<<<<<< HEAD
```c
/**
 * @brief 初始化一个预处理语句句柄。
 *
 * @param mysql 连接到数据库的 MYSQL 句柄。
 * @return 返回一个指向 MYSQL_STMT 的指针，如果初始化失败，则返回 NULL。
 */
MYSQL_STMT *STDCALL mysql_stmt_init(MYSQL *mysql);

/**
 * @brief 准备一个预处理语句。
 *
 * @param stmt 指向 MYSQL_STMT 的指针，表示要准备的语句。
 * @param query 要准备的 SQL 查询字符串。
 * @param length 查询字符串的长度。
 * @return 如果成功，返回 0；如果失败，返回非零值。
 */
int STDCALL mysql_stmt_prepare(MYSQL_STMT *stmt, const char *query, unsigned long length);

/**
 * @brief 绑定参数到预处理语句。
 *
 * @param stmt 指向要绑定参数的预处理语句的 MYSQL_STMT 指针。
 * @param bnd 指向 MYSQL_BIND 结构的指针，包含要绑定的参数信息。
 * @return 如果成功绑定参数，返回 true；如果失败，返回 false。
 *
 * 该函数用于将客户端提供的参数与预处理语句关联，以便在执行时使用这些参数。
 */
bool STDCALL mysql_stmt_bind_param(MYSQL_STMT *stmt, MYSQL_BIND *bnd);

/**
 * @brief 执行一个预处理语句。
 *
 * @param stmt 指向 MYSQL_STMT 的指针，表示要执行的语句。
 * @return 如果成功，返回 0；如果失败，返回非零值。
 */
int STDCALL mysql_stmt_execute(MYSQL_STMT *stmt);

/**
 * @brief 获取预处理语句的结果集元数据，不包括任何行数据
 *
 * @param stmt 指向已准备好的预处理语句的 MYSQL_STMT 指针。
 * @return 返回指向 MYSQL_RES 的指针，包含结果集的元数据；如果失败，返回 NULL。
 *
 * 此函数用于获取与预处理语句关联的结果集的列信息，包括列的数量、类型和名称等。
 * 需要在使用完后调用 mysql_free_result() 释放返回的结果集资源。
 */
MYSQL_RES *STDCALL mysql_stmt_result_metadata(MYSQL_STMT *stmt);

/**
 * @brief 绑定结果集到预处理语句的输出缓冲区。
 *
 * @param stmt 指向已准备好的预处理语句的 MYSQL_STMT 指针。
 * @param bnd 指向 MYSQL_BIND 结构的指针，包含输出缓冲区的信息。
 * @return 如果成功绑定结果，返回 0；如果失败，返回非零值。
 *
 * 此函数用于将结果集的列与指定的缓冲区关联，以便在执行查询后提取结果。
 * 绑定的缓冲区需要在执行 mysql_stmt_fetch() 时使用，以获取查询的结果数据。
 */
bool STDCALL mysql_stmt_bind_result(MYSQL_STMT *stmt, MYSQL_BIND *bnd);

/**
 * @brief 获取预处理语句的下一行数据。
 *
 * @param stmt 指向 MYSQL_STMT 的指针，表示要获取数据的语句。
 * @return 如果成功获取到数据，返回 0；如果没有更多行，返回 1；如果发生错误，返回 -1。
 */
int STDCALL mysql_stmt_fetch(MYSQL_STMT *stmt);

/**
 * @brief 关闭预处理语句并释放相关资源。
 *
 * @param stmt 指向要关闭的预处理语句的 MYSQL_STMT 指针。
 * @return 如果成功关闭语句，返回 0；如果失败，返回非零值。
 *
 * 此函数用于释放与预处理语句相关的所有资源，包括内部数据结构。
 * 在调用此函数后，该语句的指针将变得无效，不能再被使用。
 */
bool STDCALL mysql_stmt_close(MYSQL_STMT *stmt);
```

#### 连接数据库

test_table表

| id   | name   |
| ---- | ------ |
| 2    | data 0 |
| 3    | data 1 |

```c
#include <stdio.h>
#include <stdlib.h>
#include <mysql/mysql.h>

void finish_with_error(MYSQL *conn)
{
	fprintf(stderr, "%s\n", mysql_error(conn));
	mysql_close(conn);
	exit(1);
}

int main()
{
	MYSQL *conn = mysql_init(NULL);
	if (conn == NULL)
	{
		fprintf(stderr, "mysql_init() failed\n");
		return EXIT_FAILURE;
	}

	if (mysql_real_connect(conn, "localhost", "admin", "06305715", "test_db", 0, NULL, 0) == NULL)
	{
		finish_with_error(conn);
	}

	char query[256];	 // 增加缓冲区大小以防溢出
	char *id = "2";
	char *name = "data 0";

	// 使用安全的格式化函数
	snprintf(query, sizeof(query), "SELECT * FROM test_table WHERE id = '%s' AND name = '%s';", id, name);

	// 执行查询并检查返回值
	if (mysql_query(conn, query))
	{
		finish_with_error(conn);
	}

	MYSQL_RES *res = mysql_store_result(conn);
	if (NULL == res)
	{
		finish_with_error(conn);
	}

	int fields_num = mysql_num_fields(res);

	// 处理列名
	MYSQL_FIELD *fields = mysql_fetch_fields(res);
	for (int i = 0; i < fields_num; ++i)
	{
		printf("%s\t\t", fields[i].name);
	}
	printf("\n");

	// 处理行数据
	MYSQL_ROW row;
	while ((row = mysql_fetch_row(res)))
	{
		for (int i = 0; i < fields_num; ++i)
		{
			printf("%s\t\t", row[i] ? row[i] : "NULL"); // 处理NULL值
		}
		printf("\n");
	}

	mysql_free_result(res);
	mysql_close(conn);

	return 0;
}
```

#### sql注入攻击

```c
#include <stdio.h>
#include <stdlib.h>
#include <mysql/mysql.h>

void finish_with_error(MYSQL *conn)
{
	fprintf(stderr, "%s\n", mysql_error(conn));
	mysql_close(conn);
	exit(1);
}

int main()
{
	MYSQL *conn = mysql_init(NULL);
	if (conn == NULL)
	{
		fprintf(stderr, "mysql_init() failed\n");
		return EXIT_FAILURE;
	}

	if (mysql_real_connect(conn, "localhost", "admin", "06305715", "test_db", 0, NULL, 0) == NULL)
	{
		finish_with_error(conn);
	}

	char query[256]; // 增加缓冲区大小以防溢出
	char *id = "3';-- "; // -- 为sql中的注释，--后必须接空格；这样只要输入id就可以获取数据了
	char *name = "data 0";

	// 使用安全的格式化函数
	snprintf(query, sizeof(query), "SELECT * FROM test_table WHERE id = '%s' AND name = '%s';", id, name);

	// 执行查询并检查返回值
	if (mysql_query(conn, query))
	{
		finish_with_error(conn);
	}

	MYSQL_RES *res = mysql_store_result(conn);
	if (NULL == res)
	{
		finish_with_error(conn);
	}

	int fields_num = mysql_num_fields(res);

	// 处理列名
	MYSQL_FIELD *fields = mysql_fetch_fields(res);
	for (int i = 0; i < fields_num; ++i)
	{
		printf("%s\t\t", fields[i].name);
	}
	printf("\n");

	// 处理行数据
	MYSQL_ROW row;
	while ((row = mysql_fetch_row(res)))
	{
		for (int i = 0; i < fields_num; ++i)
		{
			printf("%s\t\t", row[i] ? row[i] : "NULL"); // 处理NULL值
		}
		printf("\n");
	}

	mysql_free_result(res);
	mysql_close(conn);

	return 0;
}
```

#### sql注入防御方法

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <mysql/mysql.h>

void finish_with_error(MYSQL *conn)
{
	fprintf(stderr, "%s\n", mysql_error(conn));
	mysql_close(conn);
	exit(1);
}

int main()
{
	MYSQL *conn = mysql_init(NULL);
	if (conn == NULL)
	{
		fprintf(stderr, "mysql_init() failed\n");
		return EXIT_FAILURE;
	}

	if (mysql_real_connect(conn, "localhost", "admin", "06305715", "test_db", 0, NULL, 0) == NULL)
	{
		finish_with_error(conn);
	}

	// 使用预处理语句
	MYSQL_STMT *stmt;
	const char *query = "SELECT * FROM test_table WHERE id = ? AND name = ?";
	stmt = mysql_stmt_init(conn);
	if (!stmt)
	{
		finish_with_error(conn);
	}

	if (mysql_stmt_prepare(stmt, query, strlen(query)))
	{
		finish_with_error(conn);
	}

	// 设置参数
	MYSQL_BIND bind[2];
	memset(bind, 0, sizeof(bind));

	char *id = "3';-- ";
	char *name = "data 0";

	bind[0].buffer_type = MYSQL_TYPE_STRING;
	bind[0].buffer = id;
	bind[0].buffer_length = strlen(id);

	bind[1].buffer_type = MYSQL_TYPE_STRING;
	bind[1].buffer = name;
	bind[1].buffer_length = strlen(name);

	if (mysql_stmt_bind_param(stmt, bind))
	{
		finish_with_error(conn);
	}

	// 执行查询
	if (mysql_stmt_execute(stmt))
	{
		finish_with_error(conn);
	}

	// 获取结果集元数据
	MYSQL_RES *result_metadata = mysql_stmt_result_metadata(stmt);
	if (!result_metadata)
	{
		finish_with_error(conn);
	}

	// 打印列名
	int num_fields = mysql_num_fields(result_metadata);
	MYSQL_FIELD *fields = mysql_fetch_fields(result_metadata);
	for (int i = 0; i < num_fields; i++)
	{
		printf("%s\t\t", fields[i].name);
	}
	printf("\n");

	// 绑定结果
	char result_id[32] = {0};
	char result_name[32] = {0};
	MYSQL_BIND result_bind[2];
	memset(result_bind, 0, sizeof(result_bind));
	result_bind[0].buffer_type = MYSQL_TYPE_STRING;
	result_bind[0].buffer = result_id;
	result_bind[0].buffer_length = sizeof(result_id);
	result_bind[1].buffer_type = MYSQL_TYPE_STRING;
	result_bind[1].buffer = result_name;
	result_bind[1].buffer_length = sizeof(result_name);

	if (mysql_stmt_bind_result(stmt, result_bind) != 0)
	{
		finish_with_error(conn);
	}

	// 打印结果
	while (mysql_stmt_fetch(stmt) == 0)
	{
		printf("%s\t\t%s\n", result_id, result_name);
	}

	// 清理
	mysql_stmt_close(stmt);
	mysql_free_result(result_metadata);
	mysql_close(conn);

	return 0;
}
```
