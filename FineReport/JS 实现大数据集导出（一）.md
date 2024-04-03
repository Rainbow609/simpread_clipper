> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [zhuanlan.zhihu.com](https://zhuanlan.zhihu.com/p/586744827)

**前言**

**帆软 FineReport 大批量数据导出**的时候，会对服务器、网络传输、数据库造成一定的压力。为了防止这样的风险，FineReport 11.0 新增了**「大数据集导出」**的功能，可直接**根据数据集结果进行导出。**

**1. 接口简介与注意事项**

**1.1 接口简介**

大数据集导出接口：directExportToExcel: function (dsName, fileName, params, colNames，forMat，enCoding)

<table data-draft-node="block" data-draft-type="table" data-size="normal" data-row-style="normal"><tbody><tr><th>Key</th><th>Value</th><th>举例</th></tr><tr><td>sessionID</td><td>具体 sessionID</td><td>-</td></tr><tr><td>dsName</td><td>数据集名称</td><td>ds1</td></tr><tr><td>fileName</td><td>导出文件名称，不指定使用默认的「模板名 - 数据集名称. xlsx」</td><td>销量</td></tr><tr><td>params</td><td>数据集参数 JSON，参数名: 参数值</td><td>{id: '9527',name:'Stephen'}</td></tr><tr><td>colNames</td><td>列名称用逗号分割，不指定使用数据集所有字段</td><td>col1,col2,...</td></tr><tr><td>forMat 注：11.0.10 及以后新增</td><td>导出的格式只支持 xlsx 或 csv，可以不写，默认导出为 Excel</td><td>"xlsx" 注：若不设置，则为 ""</td></tr><tr><td>enCoding 注：11.0.10 及以后新增</td><td>编码格式值可以为 UTF-8 或 GBK，导出格式为 csv 时生效，默认为 UTF-8</td><td>"UTF-8" 注：若不设置，则为 ""</td></tr></tbody></table>

接口示例如下：

// 接口为 directExportToExcel: function (dsName, fileName, params, colNames)

// 注意参数中的特殊字符需要进行 url 编码，比如大括号，冒号等。

var paramStr = encodeURIComponent("{param1:1,param2:\"21','22\",param3:\"text\",...}")

// 数据集传参，字符串参数建议写成格式 \"text\"

var colNames = encodeURIComponent("col1, col2, col3,...")

// 指定导出的数据列，导出字段按此顺序排列，为空默认导出所有

_g().directExportToExcel("数据集名称", "导出文件名称", paramStr, colNames, forMat, enCoding)

**1.2 注意事项**

1）此功能只支持关系型数据库。且 SQL Server 数据库需要把游标设置为服务器游标。

2）Oracle 数据库，SQL 查询时默认返回的列名是大写，所以 JS 传入 colNames 参数时，列名大小写要与其保持一致。

3）此功能无法直接导出 date/datetime 型空值，需要在 JDBC 数据连接的 URL 后添加 zeroDateTimeBehavior=convertToNull 参数。

4）建议导出的数据量不超过「1000W 行 * 20 列」，数据量超大可能会导致仅导出部分数据。

5）导出的 Excel 是通过 SQL 语句直接从数据库中获取的数据，并非报表中的数据，因此报表中设置的数据格式等无法被导出。

6）此功能不支持移动端。

7）导出超过 5s 时显示进度条。

**2. 示例：大数据集导出固定参数值**

**2.1 新建模板**

**2.1.1 新建数据集**

新建普通报表，新建数据集，SQL 语句如下：

ds1：SELECT * FROM 销量 where 1=1 and 地区 in ('${area}') and 销售员 in ('${stuff}')

ds2：SELECT 销售员 FROM 销量 where 地区 in ('${area}')

![](https://pic1.zhimg.com/v2-c59e7f82144b76f009fbc086d4968a98_r.jpg)

**2.1.2 设计报表**

报表主体样式如下图所示：

![](https://pic2.zhimg.com/v2-2b67c5c24df3e4db1f9320b78a6c0f69_r.jpg)

**2.2 设置查询控件**

编辑参数面板，新增两个「标签控件」，两个「下拉复选框控件」，一个「查询控件」。如下图所示：

![](https://pic2.zhimg.com/v2-95d4b3adf9bdd1ce5cf68b9af2604021_r.jpg)

**2.2.1 地区控件**

选中第一个「下拉复选框控件」，设置控件名称为「area」，标签名称为「地区：」，数据字典设置为数据库 FRDemo 中「销量」表中的「地区」，返回值类型为「字符串」，分隔符为','，如下图所示：

![](https://pic1.zhimg.com/v2-778b1ac46228cbf271f5fabcc49a78b4_r.jpg)

**2.2.2 销售员控件**

选中第二个「下拉复选框控件」，设置控件名称为「stuff」，标签名称为「销售员：」，数据字典设置为数据集「ds2」中的「销售员」，返回值类型为「字符串」，分隔符为','，如下图所示：

![](https://pic3.zhimg.com/v2-6c0c60f83b8c22ae695fec55577f3f82_r.jpg)

**2.3 设置导出控件**

**2.3.1 新增控件**

新增一个「按钮控件」，点击「控件设置 > 属性」，设置按钮名字为「大数据集导出」，如下图所示：

![](https://pic1.zhimg.com/v2-565501baf6b96cd8de1109a686a67048_r.jpg)

**2.3.2 设置点击事件**

选中「按钮控件」，点击「控件设置 > 属性」，新增「点击事件」，输入 JavaScript 语句，如下图所示：

![](https://pic2.zhimg.com/v2-04fa8f3dd6a794ebad8010399b318265_r.jpg)

JavaScript 代码如下：

// 接口为 directExportToExcel: function (dsName, fileName, params, colNames)

// 注意参数中的特殊字符需要进行 url 编码，比如大括号，冒号等。

var paramStr = encodeURIComponent("{area:\" 华北','华东 \",stuff:\" 孙林','王伟 \"}")

// 数据集传参

var colNames = encodeURIComponent("地区, 销售员, 产品类型, 产品, 销量")

// 指定导出的数据列，导出字段按此顺序排列，为空默认导出所有

_g().directExportToExcel("ds1", "销量", paramStr, colNames, "excel", " ")

注 1：这里的参数名是指模板参数名，而不是数据集中的数据列名。

注 2：在 SQL 参数值的前后需要加上 \ ，防止被解析。

**2.4 效果预览**

保存模板，点击「分页预览」。点击「大数据集导出按钮」，可以将「地区为华北、华东且销售员为孙伟、王林」的 ds1 数据集导出为 Excel 文件。导出内容与查询内容无关，如下图所示：

![](https://pic1.zhimg.com/v2-1c5af178264b58f67b5fbf22ffe86974_r.jpg)

注：不支持移动端。

**总结**

[帆软 FineReport](https://link.zhihu.com/?target=https%3A//www.finereport.com/%3Futm_source%3Dseo%26utm_medium%3Dseo%26utm_campaign%3Dfr-paper%26utm_term%3Dzhihu)「大数据集导出」是一种占用资源少且速度快的 Excel 导出方式，无需前台数[帆软数据表格](https://link.zhihu.com/?target=https%3A//www.finereport.com/reporttool%3Futm_source%3Dseo%26utm_medium%3Dseo%26utm_campaign%3Dfr-paper%26utm_term%3Dzhihu)后台流式导出。

该功能主要是针对明细表进行[决策平台搭建](https://link.zhihu.com/?target=https%3A//www.finereport.com/product/functions/system%3Futm_source%3Dseo%26utm_medium%3Dseo%26utm_campaign%3Dfr-paper%26utm_term%3Dzhihu)，用户通过自定义 JavaScrpit 代码调用接口，实现跳过报表计算直接取数导出。实现原理如下：

1）使用 SXSSFWorkbook 流式行导出，速度快。

2）使用生产者消费者模式，一个线程用于取数，把数据行存在队列中，另一线程读取行导出。