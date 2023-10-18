# 升级{{nebula.name}}  3.x 至 {{nebula.release}}  

本文介绍如何升级{{nebula.name}} 3.x 至 {{nebula.release}}。

## 升级说明

- 不支持轮转热升级，需完全停止整个集群服务。
- 未提供升级脚本，需手动在每台服务器上依次执行。
- 必须在原服务器上原地升级，不能修改原机器的 IP 地址、配置文件，不可更改集群拓扑。
- 数据目录不要使用软连接切换，避免失效。
- 部分升级操作需要有 sudo 权限。
- 机器硬盘剩余空间至少需为原数据目录的 1.5 倍。
- 升级时不会自动备份原有数据。务必手动备份数据，防止丢失。

## 升级影响

<!-- - 数据膨胀
  
  {{nebula.name}} 3.x 版本扩展了原有的数据格式，每个点多出一个 key，所以升级后数据会占用更大的空间。
  
  新增 key 的格式为： Type 字段（1 字节）+ Partition ID 字段（3 字节）+ VID（大小根据类型而定）。key 的 value 为空。多占用的空间可以根据点的数量和 VID 的数据类型计算。例如，数据集中有 1 亿个点，且 VID 为 INT64，则升级后这个 key 会占用 1 亿 * （1 + 3 + 8）= 12 亿字节，约等于 1.2 GB。 -->

- 客户端兼容

  升级后旧版本客户端将无法连接{{nebula.name}}，需将所有客户端都升级到兼容{{nebula.name}} {{nebula.release}} 的版本。

- 配置变化

  少数配置参数发生改变，详情参考版本发布说明和参数文档。

- 语法兼容

  nGQL 语法有部分不兼容：

  - 禁用`YIELD`子句返回自定义变量。

  - `FETCH`、`GO`、`LOOKUP`、`FIND PATH`、`GET SUBGRAPH`语句中必须添加`YIELD`子句。

  - MATCH 语句中获取点属性时，必须指定 Tag，例如从`return v.name`变为`return v.player.name`。

- 全文索引

  在升级部署了全文索引的{{nebula.name}}前，需要手动删除 Elasticsearch (ES) 中的全文索引。在升级后需要重新使用`SIGN IN`语句登录 ES 并重新创建全文索引。用户可通过 cURL 命令手动删除 ES 中全文索引。命令为`curl -XDELETE -u <es_username>:<es_password> '<es_access_ip>:<port>/<fullindex_name>'`，例如`curl -XDELETE -u elastic:elastic 'http://192.168.8.223:9200/nebula_index_2534'`。如果 ES 没有设置用户名及密码，则无需指定`-u`选项。 

  !!! note

    升级 3.5.0 及之后版本至 {{nebula.release}}，不需要手动删除全文索引。<!-- 因为3.5.0企业版中，全文索引进行了重构。 -->


## 升级 3.4.0 及以上版本至 {{nebula.release}}
<!--因为企业版 {{nebula.name}} 3.4 中一个分片对应一个 RocksDB 实例不同于 3.4 之前的一个图空间对应一个 RocksDB 实例。因此企业版3.4.0和之前版本数据格式不兼容，但是和3.4之后的版本数据兼容--> 

1. 停止所有{{nebula.name}}服务。

  ```
  <install_path>/scripts/nebula.service stop all
  ```

  `install_path`代表待升级{{nebula.name}}的安装目录。

  `storaged` 进程 flush 数据要等待约 1 分钟。运行命令后可继续运行`nebula.service status all`命令以确认所有服务都已停止。启动和停止服务的详细说明参见[管理服务](../manage-service.md)。

  !!! note

        如果超过 20 分钟不能停止服务，放弃本次升级，并联系售后人员。

  !!! caution

        从 3.0.0 开始，支持插入无 Tag 的点。如果用户需要保留无 Tag 的点，在集群内所有 Graph 服务的配置文件（`nebula-graphd.conf`）中新增`--graph_use_vertex_key=true`；在所有 Storage 服务的配置文件（`nebula-storaged.conf`）中新增`--use_vertex_key=true`。

2. 准备{{nebula.name}} {{nebula.release}} 版本的安装包并解压，可指定任一安装目录。

3. 在 {{nebula.release}} 版本安装目录下，用其`bin`目录中的新版二进制文件替换{{nebula.name}}安装路径下`bin`目录中的旧版二进制文件。

  !!! note
        每台部署了{{nebula.name}}服务的机器上都要更新相应服务的二进制文件。

<!-- 在3.0.0后可忽略该步骤，因为3.0.0及之后配置文件中改了该字段的默认值。
3. 编辑所有 Graph 服务的配置文件，修改以下参数以适应新版本的取值范围。如参数值已在规定范围内，忽略该步骤。
   
  - 为`session_idle_timeout_secs`参数设置一个在 [1,604800] 区间的值，推荐值为 28800。
  - 为`client_idle_timeout_secs`参数设置一个在 [1,604800] 区间的值，推荐值为 28800。

  这些参数在 2.x 版本中的默认值不在新版本的取值范围内，如不修改会升级失败。详细参数说明参见[Graph 服务配置](../../5.configurations-and-logs/1.configurations/3.graph-config.md)。 -->

4. 在 {{nebula.name}} 的配置文件`nebula-metad.conf`中添加`license_manager_url`参数，指定 LM 的路径以启用 License 校验。

  LM 用于校验{{nebula.name}}的授权信息，详情参见[LM 配置](../../9.about-license/2.license-management-suite/3.license-manager.md)。

  !!! note
        3.5.0 及之后版本，启用{{nebula.name}}会进行 License 校验，需要提前安装和配置 LM。

5. 启动所有 Meta 服务。

  ```
  <nebula_install_path>/scripts/nebula-metad.service start
  ```

  启动后，Meta 服务选举 leader。该过程耗时数秒。

  启动后可以任意启动一个 Graph 服务节点，使用{{nebula.name}}连接该节点并运行[`SHOW HOSTS meta`](../../3.ngql-guide/7.general-query-statements/6.show/6.show-hosts.md)和[`SHOW META LEADER`](../../3.ngql-guide/7.general-query-statements/6.show/19.show-meta-leader.md)，如果能够正常返回 Meta 节点的状态，则 Meta 服务启动成功。

  !!! note

        如果启动异常，放弃本次升级，并联系售后人员。

6. 启动所有 Graph 和 Storage 服务。

  !!! note

        如果启动异常，放弃本次升级，并联系售后人员。

7. 连接新版{{nebula.name}}，验证服务是否可用、数据是否正常。连接方法参见[连接服务](../connect-to-nebula-graph.md)。

  测试升级成功的参考命令如下：

  ```ngql
  nebula> SHOW HOSTS;
  nebula> SHOW HOSTS storage;
  nebula> SHOW SPACES;
  nebula> USE <space_name>
  nebula> SHOW PARTS;
  nebula> SUBMIT JOB STATS;
  nebula> SHOW STATS;
  nebula> MATCH (v) RETURN v LIMIT 5;
  ```

## 升级 3.x（x < 4）至 {{nebula.release}}

<!--因为企业版 {{nebula.name}} 3.4 中一个分片对应一个 RocksDB 实例不同于 3.4 之前的一个图空间对应一个 RocksDB 实例。因此企业版3.4.0和之前版本数据格式不兼容，但是和3.4之后的版本数据兼容--> 


1. [联系我们获取](https://yueshu.com.cn/contact){{nebula.name}} v{{nebula.release}} 的安装包并安装。

  !!! note

        不同安装包的升级步骤相同。本文以 RPM 包且安装目录为`/usr/local/yueshu-{{nebula.release}}`为例。具体操作请参见[安装 RPM 包](../2.compile-and-install-nebula-graph/2.install-nebula-graph-by-rpm-or-deb.md)。 
   
  !!! caution

        请确保 {{nebula.release}} 集群的 Meta 服务和 Storage 服务的配置文件中的`--data_path`参数设置的存储路径数量与 3.x 集群的配置文件中的`--data_path`参数配置的路径数量相同。否则，升级后的集群无法启动。

2. 备份{{nebula.name}} 3.x 版本的数据目录和`bin`目录下的二进制文件.

  !!! note

        请务必备份数据，防止丢失。

3. 停止{{nebula.name}} v3.x 服务。详情请参见[管理{{nebula.name}}服务](../../2.quick-start/3.quick-start-on-premise/5.start-stop-service.md)。
  运行命令后可继续运行`nebula.service status all`命令以确认所有服务都已停止。

4. 在{{nebula.name}} v{{nebula.release}} 安装目录的子目录`etc`中，更新配置文件（如果之前有配置更新的话）。

  !!! note

        如果之前没有配置更新，可跳过此步骤。


5. 在{{nebula.name}} v{{nebula.release}} 的`nebula-metad.conf`文件中设置`license_manager_url`为 LM 的访问路径。

  LM 用于校验{{nebula.name}}的授权信息，详情参见[LM 配置](../../9.about-license/2.license-management-suite/3.license-manager.md)。

  !!! note
        3.5.0及之后版本，企业版开启 License 校验，需要安装和配置 LM。
   
6. 在{{nebula.name}} v{{nebula.release}} 的安装目录下，分别执行以下命令以升级 Storage 和 Meta 服务。<!-- 不需要事先创建`data`目录 -->

  - 升级 Storage 服务：

    命令：

    ```bash
    sudo ./bin/db_upgrader  --max_concurrent_parts=<num> --src_db_path=<source_storage_data_path> --dst_db_path=<destination_storage_data_path>
    ```

    | 参数            | 说明                         |
    | :-------------- | :--------------------------- |
    | `--max_concurrent_parts` | 指定同时升级的分片数量，默认值为 `1`。<br/>建议根据磁盘性能适当调大。 |
    | `--src_db_path` | 指定源数据目录的绝对路径。下述示例源数据的目录为`/usr/local/yueshu-3.x.0/data/storage`。  |
    | `--dst_db_path` | 指定目标数据目录的绝对路径。本文示例的目标数据目录为`/usr/local/yueshu-{{nebula.release}}/data/storage`。|

    示例：

    ```bash
    sudo ./bin/db_upgrader --max_concurrent_parts=20 --src_db_path=/usr/local/yueshu-3.x.0/data/storage --dst_db_path=/usr/local/yueshu-{{nebula.release}}/data/storage
    ```

    如果有多个源数据目录，请分别指定不同的源数据目录和目标数据目录并执行命令。例如，有两个源数据目录`/usr/local/yueshu-3.x.0/data/storage`和`/usr/local/yueshu-3.x.0/data2/storage`，则执行以下命令：

    ```bash
    sudo ./bin/db_upgrader --src_db_path=/usr/local/yueshu-3.x.0/data/storage --dst_db_path=/usr/local/yueshu-{{nebula.release}}/data/storage

    sudo ./bin/db_upgrader --src_db_path=/usr/local/yueshu-3.x.0/data2/storage --dst_db_path=/usr/local/yueshu-{{nebula.release}}/data2/storage
    ```

  - 升级 Meta 服务：

    命令：

    ```bash
    sudo ./bin/meta_upgrader --src_meta_path=<source_meta_data_path> --dst_meta_path=<destination_meta_data_path>
    ```

    | 参数            | 说明                         |
    | :-------------- | :--------------------------- |
    | `--src_meta_path` | 指定源 Meta 数据目录的绝对路径。下述示例源数据的目录为`/usr/local/yueshu-3.x.0/data/meta`。  |
    | `--dst_meta_path` | 指定目标 Meta 数据目录的绝对路径。本文示例的目标数据目录为`/usr/local/yueshu-{{nebula.release}}/data/meta`。|

    示例：

    ```bash
    sudo ./bin/meta_upgrader --src_meta_path=/usr/local/yueshu-3.x.0/data/meta --dst_meta_path=/usr/local/yueshu-{{nebula.release}}/data/meta
    ```

    如果有多个源 Meta 数据目录，请指定不同的源 Meta 数据目录和目标 Meta 数据目录并分别执行命令。

  服务升级完成后，会在 v{{nebula.release}} 的安装目录下生成`data`目录，其中包含升级后的数据文件。


7. 启动和连接{{nebula.name}} v{{nebula.release}} 服务后，验证数据是否正确。参考命令如下：

  ```
  nebula> SHOW HOSTS;
  nebula> SHOW HOSTS storage;
  nebula> SHOW SPACES;
  nebula> USE <space_name>
  nebula> SHOW PARTS;
  nebula> SUBMIT JOB STATS;
  nebula> SHOW STATS;
  nebula> MATCH (v) RETURN v LIMIT 5;
  ```

## 升级历史版本至 {{nebula.release}}

如果用户{{nebula.name}}版本低于 3.0.0，升级到 {{nebula.release}} 的操作步骤，可参见上文**升级 3.x（x < 4）至 {{nebula.release}}**。
