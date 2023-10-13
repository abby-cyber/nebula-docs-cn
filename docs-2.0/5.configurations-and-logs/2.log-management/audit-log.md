# 审计日志

{{nebula.name}}的审计日志功能可以将 Graph 服务接受到的所有操作进行分类存储，然后提供给终端用户查看，终端用户可以根据需要，追踪指定类型的操作。

!!! enterpriseonly

    仅企业版支持本功能。

## 日志类别

|类别|语句|说明|
|:--|:--|:--|
|`login` |-| 客户端尝试连接 Graph 服务时，记录相关信息。 |
|`exit`  |-| 断开与 Graph 服务的连接时，记录相关信息。 |
|`ddl` |`CREATE SPACE`、`DROP SPACE`、`CREATE TAG`、`DROP TAG`、`ALTER TAG`、`DELETE TAG`、`CREATE EDGE`、`DROP EDGE`、`ALTER EDGE`、`CREATE INDEX`、`DROP INDEX`、`CREATE FULLTEXT INDEX`、`DROP FULLTEXT INDEX`|记录 DDL 语句的信息。 |
|`dql` |`MATCH`、`LOOKUP`、`GO`、`FETCH`、`GET SUBGRAPH`、`FIND PATH`、`UNWIND`、`GROUP BY`、`ORDER BY`、`YIELD`、`LIMIT`、`RETURN`、`REBUILD INDEX`、`REBUILD FULLTEXT INDEX`|记录 DQL 语句的信息。|
|`dml` |`INSERT VERTEX`、`DELETE VERTEX`、`UPDATE VERTEX`、`UPSERT VERTEX`、`INSERT EDGE`、`DELETE EDGE`、`UPDATE EDGE`、`UPSERT EDGE`|记录 DML 语句的信息。 |
|`dcl`|`CREATE USER`、`GRANT ROLE`、`REVOKE ROLE`、`CHANGE PASSWORD`、`ALTER USER`、`DROP USER`、`CREATE SNAPSHOT`、`DROP SNAPSHOT`、`ADD LISTENER`、`REMOVE LISTENER`、`BALANCE`、`SUBMIT JOB`、`STOP JOB`、`RECOVER JOB`、`ADD DRAINER`、`REMOVE DRAINER`、`SIGN IN DRAINER SERVICE`、`SIGN OUT DRAINER SERVICE`、`DOWNLOAD HDFS`、`INGEST`|记录 DCL 语句的信息。|
|`util`|`SHOW HOSTS`、`SHOW USERS`、`SHOW ROLES`、`SHOW SNAPSHOTS`、`SHOW SPACES`、`SHOW PARTS`、`SHOW TAGS`、`SHOW EDGES`、`SHOW INDEXES`、`SHOW CREATE SPACE`、`SHOW CREATE TAG/EDGE`、`SHOW CREATE INDEX`、`SHOW INDEX STATUS`、`SHOW LISTENER`、`SHOW TEXT SEARCH CLIENTS`、`SHOW DRAINER CLIENTS`、`SHOW FULLTEXT INDEXES`、`SHOW CONFIGS`、`SHOW CHARSET`、`SHOW COLLATION`、`SHOW STATS`、`SHOW SESSIONS`、`SHOW META LEADER`、`SHOW DRAINERS`、`SHOW QUERIES`、`SHOW JOB`、`SHOW JOBS`、`DESCRIBE INDEX`、`DESCRIBE EDGE`、`DESCRIBE TAG`、`DESCRIBE SPACE`、`DESCRIBE USER`、`USE SPACE`、`SIGN IN TEXT SERVICE`、`SIGN OUT TEXT SERVICE`、`EXPLAIN`、`PROFILE`、`KILL QUERY`|记录工具类语句的信息。 |
|`unknown`|-|记录未能识别的语句。|

## 设置审计日志

使用审计日志需要修改集群内的所有 Graph 服务的配置（`nebula-graphd.conf`），默认路径为`/usr/local/nebula/etc/nebula-graphd.conf`。

!!! note

    修改配置后，需要重启 Graph 服务才能生效。

与审计日志相关的参数说明如下。

|参数|预设值|说明|
|:--|:--|:--|
| `enable_audit` | `false` | 是否开启审计日志。 |
| `audit_log_handler` | `file` | 审计日志的存储方案。可选值为`file`（本地文件）和`es`（Elasticsearch），支持的 Elasticsearch 版本为 7.x 和 8.x。|
| `audit_log_file` | `./logs/audit/audit.log` | 仅在`audit_log_handler=file`时生效。审计日志的存储路径，支持相对路径或绝对路径。 |
| `audit_log_strategy` | `synchronous` | 仅在`audit_log_handler=file`时生效。审计日志的同步方案。可选值为`asynchronous`和`synchronous`。设置为`asynchronous`时，日志事件使用内存缓冲，不会阻塞主线程，但是可能会因为缓存不够而导致日志缺失；设置为`synchronous`时，日志事件每次都刷新并同步到文件中。 |
| `audit_log_max_buffer_size` | `1048576` |仅在`audit_log_handler=file`、`audit_log_strategy=asynchronous`时生效。审计日志的缓存大小。单位：字节。  |
| `audit_log_format` | `xml` | 仅在`audit_log_handler=file`时生效。审计日志的格式。可选值为`xml`、`json`和`csv`。 |
| `audit_log_es_address` | - | 仅在`audit_log_handler=es`时生效。Elasticsearch 服务器的地址。格式为`IP1:port1, IP2:port2, ...`。 |
| `audit_log_es_user` | - | 仅在`audit_log_handler=es`时生效。登录 Elasticsearch 服务器的用户名。 |
| `audit_log_es_password`     | -  | 仅在`audit_log_handler=es`时生效。Elasticsearch 用户名对应的密码。  |
| `audit_log_es_batch_size`      | `1000`  | 仅在`audit_log_handler=es`时生效。每次发送至 Elasticsearch 服务器的日志条数。  |
| `audit_log_exclude_spaces`      | -  | 不需要记录日志的图空间列表。多个图空间用英文逗号（,）分隔。  |
| `audit_log_categories`      | `login,exit`  | 需要记录日志的分类列表。多个类别用英文逗号（,）分隔。  |

## 审计日志格式

以默认路径（`/usr/local/nebula/logs/audit/audit.log`）和默认 XML 格式为例说明各个字段的含义。

!!! note

    如果在{{nebula.name}}运行过程中删除审计日志目录，日志不会继续打印，但是不会影响程序运行。重启服务审计日志打印可以恢复正常。

```bash
<AUDIT_RECORD
  CATEGORY="util"
  TIMESTAMP="2022-04-07 02:31:38"
  TERMINAL=""
  CONNECTION_ID="1649298693144580"
  CONNECTION_STATUS="0"
  CONNECTION_MESSAGE=""
  USER="root"
  CLIENT_HOST="127.0.0.1"
  HOST="192.168.8.111"
  SPACE=""
  QUERY="use basketballplayer1"
  QUERY_STATUS="-1005"
  QUERY_MESSAGE="SpaceNotFound: "
/>
<AUDIT_RECORD
  CATEGORY="util"
  TIMESTAMP="2022-04-07 02:31:39"
  TERMINAL=""
  CONNECTION_ID="1649298693144580"
  CONNECTION_STATUS="0"
  CONNECTION_MESSAGE=""
  USER="root"
  CLIENT_HOST="127.0.0.1"
  HOST="192.168.8.111"
  SPACE=""
  QUERY="use basketballplayer"
  QUERY_STATUS="0"
  QUERY_MESSAGE=""
/>
```

|字段|说明|
|:--|:--|
|`CATEGORY`| 日志类别。|
|`TIMESTAMP`| 日志生成时间。 |
|`TERMINAL`| 保留字段，暂不支持。|
|`CONNECTION_ID`| 连接的会话ID。 |
|`CONNECTION_STATUS`| 连接的状态码。`0`表示成功，其他数字代表不同的错误信息。|
|`CONNECTION_MESSAGE`| 如果连接出错，会显示报错信息。|
|`USER`| 连接的用户名。 |
|`CLIENT_HOST`| 客户端的 IP 地址。 |
|`HOST`| 连接的机器的 IP 地址。 |
|`SPACE`| 执行查询的图空间。|
|`QUERY`| 查询语句。|
|`QUERY_STATUS`| 查询状态。`0`表示成功，其他数字代表不同的错误信息。|
|`QUERY_MESSAGE`| 如果查询出错，会显示报错信息。|

## 使用 logrotate 进行审计日志轮转

用户可以使用 Linux 系统中的 [logrotate](https://github.com/logrotate/logrotate) 工具，对审计日志进行轮转以定期归档和销毁审计日志，避免日志文件过大。

以下为使用`logrotate`来定时清理{{nebula.name}}审计日志的操作步骤：

!!! note

    需要使用 root 用户或者具有 sudo 权限的用户来安装 logrotate 或者运行 logrotate。

1. 安装 logrotate。
   
  - Debian/Ubuntu：

    ```bash
    sudo apt-get install logrotate
    ```

  - CentOS/RHEL：

    ```bash
    sudo yum install logrotate
    ```

2. 创建 logrotate 配置文件。

  在`/etc/logrotate.d`目录下，为审计日志创建一个新的 logrotate 配置文件，例如创建一个名为`audit`的文件并其中添加以下内容：

  ```bash
  # 创建 audit 文件
  sudo vim /etc/logrotate.d/audit
  ```

  ```bash
  # 在 audit 文件中添加配置以设置日志轮转规则
  /usr/local/nebula/logs/audit/audit.log {
      daily
      rotate 5
      copytruncate
      nocompress
      missingok
      notifempty
      create 644 root root
      dateext
      dateformat .%Y-%m-%d-%s
      maxsize 1k
  }
  ```
  
  示例中的`/usr/local/nebula/logs/audit/audit.log`为{{nebula.name}}的默认审计日志文件（`audit.log`）的路径，如果日志路径不同，需根据实际情况修改配置文件中的路径。以下为示例配置文件中各个参数的解释：

  |参数|说明|
  |:--|:--|
  |`daily`| 每天轮转日志。可用的时间单位有：`hourly`、`daily`、`weekly`、`monthly`、`yearly`。|
  |`rotate 5`| 在删除前日志文件前，其被轮转的次数。即保留最近生成的 5 个日志文件。|
  |`copytruncate`| 将当前日志文件复制一份，然后清空当前日志文件。|
  |`nocompress`| 不压缩旧的日志文件。|
  |`missingok`| 如果日志文件丢失，不报告错误。|
  |`notifempty`| 如果日志文件为空，不进行轮转。|
  |`create 644 root root`| 创建新的日志文件，并设置适当的权限和所有者。|
  |`dateext`| 在日志文件名中添加日期后缀。<br/>默认是当前日期。默认是`-%Y%m%d`的后缀。可用`dateformat`选项扩展配置。|
  |`dateformat .%Y-%m-%d-%s`| 必须配合`dateext`使用，紧跟在下一行出现，定义文件切割后的文件名。<br/>在V3.9.0 之前，只支持`%Y`、`%m`、`%d`、`%s`参数。在V3.9.0 及之后，支持 %H 参数。|
  |`maxsize 1k`| 当日志文件大小超过`1`千字节（`1024`字节）或者超过设定的周期（如`daily`）时，进行日志轮转。可用的大小单位有：`k`、`M`，默认单位为字节。|

  用户可以根据实际需求修改配置文件中的参数。更多关于参数的配置及解释，参见 [logrotate](https://man7.org/linux/man-pages/man8/logrotate.8.html)。

3. 测试 logrotate 配置。

  为了验证 logrotate 的配置是否正确，可以使用以下命令来进行测试：

  ```bash
  sudo logrotate --debug /etc/logrotate.d/audit
  ```

4. 运行 logrotate。

  尽管`logrotate`通常由 Cron 作业自动执行，但也可以手动执行以下命令，以立即进行日志轮转：

  ```bash
  sudo logrotate -fv /etc/logrotate.d/audit
  ```

  `-fv`：`f`表示强制执行，`v`表示打印详细信息。

5. 查看日志轮转结果。

  日志轮转后，会在`/usr/local/nebula/logs/audit`目录下看到新的日志文件，例如`audit.log.2022-04-07-1649298693`。原始日志内容会被清空，但文件会被保留，新日志继续写入。当日志数量超过`rotate`值时，最旧的日志将被删除。

  当日志文件数量超过`rotate`设置的值时，会删除最旧的日志文件。例如，`rotate 5`表示保留最近生成的 5 个日志文件，当日志文件数量超过 5 个时，会删除最旧的日志文件。

  ```bash
  [test@test audit]$ ll
  -rw-r--r-- 1 root root    0 10月 12 11:15 audit.log
  -rw-r--r-- 1 root root 1436 10月 11 19:38 audit.log-202310111697024305 # 保留的日志文件中最旧的一个，当日志文件数量超过设定数量 5 时，会删除该文件。
  -rw-r--r-- 1 root root  286 10月 12 11:05 audit.log-202310121697079901
  -rw-r--r-- 1 root root  571 10月 12 11:05 audit.log-202310121697079940
  -rw-r--r-- 1 root root  571 10月 12 11:14 audit.log-202310121697080478
  -rw-r--r-- 1 root root  571 10月 12 11:15 audit.log-202310121697080536
  [test@test audit]$ ll
  -rw-r--r-- 1 root root 571 10月 12 11:18 audit.log
  -rw-r--r-- 1 root root 286 10月 12 11:05 audit.log-202310121697079901
  -rw-r--r-- 1 root root 571 10月 12 11:05 audit.log-202310121697079940
  -rw-r--r-- 1 root root 571 10月 12 11:14 audit.log-202310121697080478
  -rw-r--r-- 1 root root 571 10月 12 11:15 audit.log-202310121697080536
  -rw-r--r-- 1 root root 571 10月 12 11:17 audit.log-202310121697080677 # 新生成的日志文件。
  ```

## 视频

* [{{nebula.name}}的审计日志](https://www.bilibili.com/video/BV17F41157JB)（3 分 53 秒）
<iframe src="//player.bilibili.com/player.html?aid=299493340&bvid=BV17F41157JB&cid=731096973&page=1&high_quality=1" scrolling="no" border="0" frameborder="no" framespacing="0" allowfullscreen="true" width="720px" height="480px"> </iframe>
