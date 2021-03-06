# 导入OSS数据

您可以将日志文件保存到OSS中，并通过数据导入方式将OSS数据导入到日志服务，实现日志数据的查询分析、数据加工等操作。

-   已创建OSS Bucket，并将待导入的日志文件存储到OSS Bucket中，详情请参见[上传文件](/cn.zh-CN/快速入门/上传文件.md)。
-   已创建Project和Logstore，详情请参见[创建Project和Logstore](/cn.zh-CN/快速入门/快速入门.md)。
-   已经完成[云资源访问授权](https://ram.console.aliyun.com/#/role/authorize?request=%7B%22Requests%22:%20%7B%22request1%22:%20%7B%22RoleName%22:%20%22AliyunLogImportOSSRole%22,%20%22TemplateId%22:%20%22ImportOSSRole%22%7D%7D,%20%22ReturnUrl%22:%20%22https:%2F%2Fsls.console.aliyun.com%2F%22,%20%22Service%22:%20%22Log%22%7D)，即已授权日志服务使用AliyunLogImportOSSRole角色访问您的OSS资源。

    如果您使用的是RAM用户，还需授予RAM用户PassRole权限，授权策略如下所示，操作步骤请参见[创建自定义策略](/cn.zh-CN/权限策略管理/自定义策略/创建自定义策略.md)和[为RAM用户授权](/cn.zh-CN/用户管理/为RAM用户授权.md)。

    ```
    {
       "Statement": [
        {
          "Effect": "Allow",
          "Action": "ram:PassRole",
          "Resource": "acs:ram:*:*:role/aliyunlogimportossrole"
        }
      ],
      "Version": "1"
    }        
    ```


日志服务不仅可以通过Logtail和API方式采集数据，您还可以选择数据导入方式将日志文件直接导入到日志服务中，实现日志数据的采集。当前支持导入的文件格式有：JSON、CSV、Parquet、TEXT。

## 创建数据导入配置

1.  登录[日志服务控制台](https://sls.console.aliyun.com)。

2.  在接入数据区域，选择**OSS-对象存储**。

3.  在选择日志空间页签中，选择目标Project和Logstore，单击**下一步**。

    您也可以单击**立即创建**，重新创建Project和Logstore，详情请参见[步骤1：创建Project和Logstore](/cn.zh-CN/快速入门/快速入门.md)。

4.  设置导入配置。

    1.  在数据源设置页签中，配置如下参数。

        |参数|说明|
        |--|--|
        |配置名称|设置配置的名称。|
        |OSS Region|待导入的OSS文件所在Bucket的地域。|
        |Bucket|待导入的OSS文件所在的Bucket。|
        |文件夹前缀|待导入的OSS文件所在文件夹的前缀。用于准确定位待导入的文件夹。例如文件完整路径为csv/convertcsv.csv，则可以指定前缀为csv/。|
        |正则过滤|用于过滤文件的正则表达式。只有文件名匹配该正则表达式的文件才会被导入。默认为空，表示不过滤。|
        |数据格式|文件的解析格式，如下所示。         -   CSV：分隔符分割的文本文件，支持指定文件中的首行为字段名称或手动指定字段名称。除字段名称外的每一行都会被解析为日志字段的值。
        -   JSON：逐行读取OSS文件，将每一行看做一个JSON对象进行解析。解析后，JSON对象中的各个字段对应为日志中的各个字段。
        -   Parquet：Parquet格式，无需任何配置，自动解析成日志格式。不支持预览。
        -   单行文本日志：将OSS文件中的每一行解析为一条日志。
        -   跨行文本日志：多行模式，支持指定行首或者行尾的正则表达式解析日志。 |
        |压缩格式|待导入的OSS文件的压缩格式，支持格式：Gzip、Bzip2、Snappy和未压缩，日志服务根据对应格式进行解压并读取数据。默认为**未压缩**。|
        |编码格式|待导入的OSS文件的编码格式。|
        |解冻归档文件|如果待导入的OSS文件为归档存储，则需要解冻后才能读取。开启此功能，则归档文件会自动解冻。|

    2.  单击**预览**，查看文件预览结果。

    3.  确认无误后，单击**下个配置**。

    4.  在数据格式配置页签中，配置如下参数。

        -   通用配置项

            |参数|说明|
            |--|--|
            |使用系统时间|            -   开启**使用系统时间**，则解析后的日志时间显示为文件导入时的系统时间。
            -   关闭**使用系统时间**，则需要手动配置时间字段和时间格式。
**说明：** 推荐开启**使用系统时间**，日志时间可作为普通字段建立索引，用于日志查询。在导入历史数据时，如果数据时间早于当前时间减去Logstore数据保存时间，例如保存7天，那么时间为7天前的日志，无法在控制台上查询。 |
            |时间字段|如果关闭**使用系统时间**，则需要指定一个字段为时间字段。|
            |时间格式|如果关闭**使用系统时间**，需要指定一个Java SimpleDateFormat语法的时间格式，用于解析时间字段。时间格式的语法详情请参见[Class SimpleDateFormat](https://docs.oracle.com/javase/7/docs/api/java/text/SimpleDateFormat.html)。 **说明：** Java SimpleDateFormat不支持Unix时间戳，如果您要使用Unix时间戳，时间格式指定为epoch。 |
            |时区|如果关闭**使用系统时间**，需要指定一个时区，用于解析日志时间的时区。如果日志格式中已经有时区信息，那此参数无效。|

        -   CSV特有配置项

            |参数|说明|
            |--|--|
            |分隔符|CSV文件分隔符，默认用逗号（,）。|
            |Quote|当字段内包含分隔符时，需要使用Quote包裹，默认用双引号（"）。|
            |转义符|CSV文件转义符，默认用反斜线（\\）|
            |跨行日志最大行数|当一条日志跨多行时，需要指定最大行数，默认为1。|
            |首行作为字段名称|开启**首行作为字段名称**后，将使用CSV文件中的首行作为字段名称。|
            |自定义字段列表|关闭**首行作为字段名称**后，请根据需求自定义字段名称，多个字段名称之间用逗号（,）隔开。|
            |跳过行数|在CSV文件中的开始位置跳过指定行数之后才开始读取数据，默认为0。|

        -   跨行文本日志特有配置项

            |参数|说明|
            |--|--|
            |正则匹配位置|            -   选择**首行正则**，则使用正则表示式去匹配一条日志的行首，未匹配部分为该条日志的一部分，直到达到最大行数。
            -   选择**尾行正则**，则使用正则表示式去匹配一条日志的行尾，未匹配部分为下一条日志的一部分，直到达到最大行数。 |
            |正则表达式|根据日志内容，配置正确的正则表达式，详情请参见[如何调试正则表达式]()。|
            |最大行数|一条日志的最大行数，默认为10。|

    5.  设置数据格式完成后，单击**测试**。

    6.  测试成功后，单击**下个配置**。

    7.  在调度间隔页签中，配置如下参数。

        |参数|说明|
        |--|--|
        |导入间隔|OSS文件导入日志服务的时间间隔。|
        |立即执行|开启**立即执行**，则立即执行一次导入操作。|

    8.  配置完成后，单击**下一步**。

5.  在**查询分析配置**页签中，设置索引。

    默认已设置索引，您也可以根据业务需求，重新设置索引，具体请参见[开启并配置索引](/cn.zh-CN/查询与分析/开启并配置索引.md)。

    **说明：**

    -   全文索引和字段索引属性必须至少启用一种。同时启用时，以字段索引属性为准。
    -   索引类型为long、double时，大小写敏感和分词符属性无效。
6.  单击**下一步**，完成导入配置。


## 查看导入配置

创建导入配置成功后，您可以在控制台中查看已创建的导入配置及生成的统计报表。

1.  单击目标Project。

2.  选择目标日志库下的**数据接入** \> **数据导入**，单击配置名称。

3.  在导入配置概览页面，查看导入配置的基本信息和统计报表。

    ![导入任务概览](https://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/zh-CN/9040559951/p74200.png)


## 相关操作

在配置的导入配置概览页面，您还可以进行如下操作。

-   修改配置

    单击**修改配置**，修改导入配置的相关信息，详情请参见[设置导入配置](#step_pz7_0v7_4pg)。

-   删除配置

    单击**删除配置**，删除该导入配置。

    **说明：** 删除后不可恢复，请谨慎操作。


## 附录

-   时间格式样例

    |日期格式|解析语法|解析后的值（单位：秒）|
    |----|----|-----------|
    |2020-05-02 17:30:30|yyyy-MM-dd HH:mm:ss|1588411830|
    |2020-05-02 17:30:30:123|yyyy-MM-dd HH:mm:ss:SSS|1588411830|
    |2020-05-02 17:30|yyyy-MM-dd HH:mm|1588411800|
    |2020-05-02 17|yyyy-MM-dd HH|1588410000|
    |20-05-02 17:30:30|yy-MM-dd HH:mm:ss|1588411830|
    |2020-05-02T17:30:30V|yyyy-MM-dd'T'HH:mm:ss'V'|1588411830|
    |Sat May 02 17:30:30 CST 2020|EEE MMM dd HH:mm:ss zzz yyyy|1588411830|

-   时间格式语法

    |字符|解析|示例|
    |--|--|--|
    |G|纪元标记|AD|
    |y|年份|2001|
    |M|月份|July或者07|
    |d|日期|10|
    |h|小时，取值：1~12（AM、PM）|12|
    |H|一天中的小时，取值：0~23|22|
    |m|分钟|30|
    |s|秒|55|
    |S|毫秒|234|
    |E|星期|Tuesday|
    |D|一年中的第几天|360|
    |F|一个月中第几周的周几|2 \(second Wed. in July\)|
    |w|一年中的第几周|40|
    |W|一个月中的第几周|1|
    |a|AM/PM|PM|
    |k|一天中的小时，取值：1~24|24|
    |k|小时，取值：0~11（AM、PM）|10|
    |z|时区|Eastern Standard Time|
    |'|文字定界符|Delimiter|
    |"|单引号|无|


