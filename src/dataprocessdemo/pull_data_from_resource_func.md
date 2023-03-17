# 使用资源函数增量获取数据
使用增量拉取模式获取数据时，日志服务每次只拉取新增或更新的数据，效率高。本文介绍如何使用res_rds_mysql函数增量获取RDS MySQL数据库中的数据。

## 前提条件
* 日志服务
  * 已上传数据到日志服务源Logstore中。具体操作，请参见[数据采集](https://help.aliyun.com/document_detail/28981.htm?spm=a2c4g.11186623.0.0.9cd930efWPocMA#concept-ikm-ql5-vdb)。
  * 已创建日志服务目标Logstore。具体操作，请参见[创建Logstore](https://help.aliyun.com/document_detail/48990.htm?spm=a2c4g.11186623.0.0.9cd97c6fCEr5Iy#section-v52-2jx-ndb)。
  * 如果您使用的是RAM用户，则需要先授予RAM用户数据加工操作权限。具体操作，请参见[授予RAM用户数据加工操作权限](https://help.aliyun.com/document_detail/125779.htm?spm=a2c4g.11186623.0.0.9cd9793421KP9y#task-2005445)。
  * 已配置源Logstore和目标Logstore的索引。具体操作，请参见[配置索引](https://help.aliyun.com/document_detail/90732.htm?spm=a2c4g.11186623.0.0.9cd91550ECLMvG#task-jqz-v55-cfb)。
  数据加工任务不依赖索引；但不配置索引，您无法执行查询和分析操作。

* RDS
  * 已创建RDS数据库和账号。具体操作，请参见[创建数据库和账号](https://help.aliyun.com/document_detail/87038.htm?spm=a2c4g.11186623.0.0.9cd91de3LO6lRm#concept-jyq-tc5-q2b)。
  * 已上传数据到RDS MySQL数据库表。
  * 已配置RDS MySQL白名单。具体操作，请参见[通过客户端、命令行连接RDS MySQL实例](https://help.aliyun.com/document_detail/87038.htm?spm=a2c4g.11186623.0.0.9cd91de3LO6lRm#concept-jyq-tc5-q2b)。
  **注意** 使用res_rds_mysql函数从RDS MySQL数据库中拉取数据时，需先在RDS MySQL中配置白名单为0.0.0.0，使所有IP地址都可访问数据库，但是这种方式会增加数据库的风险。如果您需要设置指定的日志服务IP地址，可以提[工单](https://smartservice.console.aliyun.com/service/create-ticket)进行配置。

## 背景信息
某科技公司将客户信息保存在云数据库RDS MySQL数据库中，将客户维护记录保存在日志服务的Logstore中，两种数据都在持续更新。现在该公司希望将两种数据JOIN后保存到一个新的Logstore中。

日志服务提供res_rds_mysql函数，可从RDS MySQL数据库中拉取数据，保存到目标Logstore中。增量拉取模式下，在加工任务运行过程中，日志服务根据数据库中的时间戳字段增量拉取数据，只拉取更新或新增的数据，效率高。适用于数据库中的数据量大、更新频繁、删除数据较少、数据加工实时性要求高的场景。

其他拉取模式以及增量拉取的更多信息，请参见[res_rds_mysql](https://help.aliyun.com/document_detail/129401.htm?spm=a2c4g.11186623.0.0.9cd9576cKy7ZTm#section-49h-ufh-ptu)。

## 资源信息与数据示例
* 日志服务资源
  * Project：client-project
  * 源Logstore：client-log
    数据样例如下所示：
    ![数据样例1](/img/dataprocessdemo/数据富化/数据样例1.png)
  * 目标Logstore：client-information
* RDS资源
  * 数据库：client-db
  * 数据库表：client
    数据样例如下所示：
    ![数据样例2](/img/dataprocessdemo/数据富化/数据样例2.png)

  * 数据库用户名和密码：test/test1234@@
  * 数据库外网地址：rm-bp1k****tp8o.mysql.rds.aliyuncs.com
## 操作步骤
1. 登录[日志服务控制台](https://sls.console.aliyun.com/?spm=a2c4g.11186623.0.0.9cd93c05OdrePh)。
2. 进入数据加工页面。
  a. 在Project列表中，单击目标Project（client-project）。
  b. 在**日志存储** > **日志库**页签中，**单击**目标Logstore（client-log）。
  c. 在查询和分析页面，单击**数据加工**。
3. 在页面右上角，选择数据的时间范围。
  请确保在**原始日志**页签中有日志数据。
4. 在编辑框中，输入数据加工语句。
  详细的参数说明，请参见[res_rds_mysql](https://help.aliyun.com/document_detail/129401.htm?spm=a2c4g.11186623.0.0.9cd91cf8bLH92K#section-49h-ufh-ptu)。
    ```python
    e_table_map(
        res_rds_mysql(
            "rm-bp1k****tp8o.mysql.rds.aliyuncs.com",
            "test",
            "test1234@@",
            "client-db",
            table="client",
            fields=["c_id", "name", "telephone", "update_time"],
            refresh_interval=1,
            primary_keys="c_id",
            update_time_key="update_time",
            deleted_flag_key=None,
        ),
        "c_id",
        ["name", "telephone"],
    )
    ```
5. 快速预览数据。
  使用快速预览，确保加工语句填写正确。更多信息，请参见[快速预览](https://help.aliyun.com/document_detail/263336.htm?spm=a2c4g.11186623.0.0.9cd93c11K1aIHY#task-2089290)。
  a. 单击**快速**。
  b. 在**测试数据** > **数据**页签中输入如下内容。
      ```
      {
        "__source__": "192.0.2.0",
        "__time__": 1624956516,
        "__topic__": "log",
        "__tag__:__client_ip__":"192.0.2.2",
        "c_id": "1",
        "staff_id": "002",
        "status": "持续跟进",
        "tag": "二次回访",
      }
      ```
    c. 在**测试数据** > **数据**页签中输入如下内容。
    ```
    c_id,name,telephone,update_time,delete_flag
    1,maki,010-123,1606358931,false
    ```
    d. 单击**预览数据**。
      查看预览结果。快速预览结果
      ![预览结果1](/img/dataprocessdemo/数据富化/预览结果1.png)
6. 高级预览数据。
  使用高级预览，确保日志服务连接RDS MySQL数据库成功。更多信息，请参见**高级预览**。
    a. 单击**高级**。
    b. 单击**预览数据**。
    c. 在添加预览配置面板中，配置授权方式，然后单击**确定**。
    ![预览结果2](/img/dataprocessdemo/数据富化/预览结果2.png)
    d. 查看预览结果。
    ![预览结果3](/img/dataprocessdemo/数据富化/预览结果3.png)
    如果提示运行出错，请参见[获取RDS MySQL数据语法错误](https://help.aliyun.com/document_detail/135597.htm?spm=a2c4g.11186623.0.0.9cd93c05OdrePh#concept-2070603)。

7. 创建数据加工任务。
  a. 单击**保存数据加工**。
  b. 在**创建数据加工规则**面板中，配置相关信息，然后单击**确定**。
    具体参数说明，请参见[创建数据加工任务](https://help.aliyun.com/document_detail/125615.htm?spm=a2c4g.11186623.0.0.9cd93d89eMoN40#task-1181217)。
    ![数据加工1](/img/dataprocessdemo/数据富化/数据加工1.png)
    创建数据加工任务后，您可以在目标Logstore中查看已加工的日志。加工结果
    ![数据加工2](/img/dataprocessdemo/数据富化/数据加工2.png)
## 常见问题
如何使用增量更新模式中的删除功能？
增量更新模式下，日志服务会根据数据库表中的主键和时间，只拉取更新或新增的数据。如果您在数据库表中将某条数据标记为删除（例如delete_flag=true），日志服务依旧会拉取该数据继续加工。针对该情况时，日志服务在res_rds_mysql函数中新增了deleted_flag_key参数。在加工语句中配置deleted_flag_key参数后，加工任务在获取数据库表更新的数据后，会将读取到的delete_flag为true的数据行从加工任务的内存维表中删除（不影响数据库表的内容）。在加工任务内存维表中被删除的数据行将不再参与后续与日志数据JOIN的操作。

**注意**
  * delete_flag设置为true、1等值时，都表示删除。更多信息，请参见[res_rds_mysql](https://help.aliyun.com/document_detail/129401.htm?spm=a2c4g.11186623.0.0.9cd95b1bV2L9z1#section-49h-ufh-ptu)。
  * 如果在加工语句中配置了deleted_flag_key参数，则还需配置update_time_key参数。

例如在RDS MySQL数据库表中新增两条数据（name=mia，name=tom），其中name=mia这条数据被标记为删除，即设置delete_flag为true。则日志服务内存维表更新时，name=mia这条数据在维表中被删除，不会被加工。
![数据加工3](/img/dataprocessdemo/数据富化/数据加工3.png)
加工语句示例如下：
```python
e_table_map(
    res_rds_mysql(
        "rm-bp1****l3tp.mysql.rds.aliyuncs.com",
        "test",
        "test1234@@",
        "client-db",
        table="client",
        fields=["c_id", "name", "telephone", "update_time"],
        refresh_interval=1,
        primary_keys="c_id",
        update_time_key="update_time",
        deleted_flag_key="delete_flag",
    ),
    "c_id",
    ["name", "telephone"],
)
```