> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [zhuanlan.zhihu.com](https://zhuanlan.zhihu.com/p/586745234)

1.  **示例：大数据集导出动态参数值**  
    **1.1 新建模板**  
    **1.1.1 新建数据集**  
    新建普通报表，新建数据集，SQL 语句如下：  
    ds1：SELECT * FROM 销量 where 1=1 and 地区 in ('${area}') and 销售员 in ('${stuff}')  
    ds2：SELECT 销售员 FROM 销量 where 地区 in ('${area}')  
    

![](https://pic1.zhimg.com/v2-c59e7f82144b76f009fbc086d4968a98_r.jpg)

  
**1.1.2 设计报表**  
报表主体样式如下图所示：  

![](https://pic2.zhimg.com/v2-2b67c5c24df3e4db1f9320b78a6c0f69_r.jpg)

  
**1.2 设置查询控件**  
编辑参数面板，新增两个「标签控件」，两个「下拉复选框控件」，一个「查询控件」。如下图所示：  

![](https://pic2.zhimg.com/v2-95d4b3adf9bdd1ce5cf68b9af2604021_r.jpg)

  
**1.2.1 地区控件**  
选中第一个「下拉复选框控件」，设置控件名称为「area」，标签名称为「地区：」，数据字典设置为数据库 FRDemo 中「销量」表中的「地区」，返回值类型为「字符串」，分隔符为','，如下图所示：  

![](https://pic1.zhimg.com/v2-778b1ac46228cbf271f5fabcc49a78b4_r.jpg)

  
**1.2.2 销售员控件**  
选中第二个「下拉复选框」控件，设置控件名称为「stuff」，标签名称为「销售员：」，数据字典设置为数据集 ds2 中的「销售员」，返回值类型为「字符串」，分隔符为','，如下图所示：  

![](https://pic3.zhimg.com/v2-6c0c60f83b8c22ae695fec55577f3f82_r.jpg)

  
**1.3 设置动态导出列**  
新增一个「标签控件」，一个「下拉复选框控件」。选中「下拉复选框控件」，点击「控件设置 > 属性」，设置控件名称为「col」，标签名称为「导出列」，数据字典类型为「公式」，实际值为：TABLEDATAFIELDS("ds1")，如下图所示：  

![](https://pic2.zhimg.com/v2-d178df86bb8a29b2585ed8e7c450ec09_r.jpg)

  
**1.4 设置导出控件**  
**1.4.1 新增控件**  
新增一个「按钮控件」，点击「控件设置 > 属性」，设置按钮名字为「大数据集导出」，如下图所示：  

![](https://pic2.zhimg.com/v2-c5f8e2af70ad513ff9d67a0f4f6576b5_r.jpg)

  
**1.4.2 设置点击事件**  
选中「按钮控件」，点击「控件设置 > 属性」，新增「点击事件」，输入 JavaScript 语句，如下图所示：  

![](https://pic3.zhimg.com/v2-39ebdceb44de8efe997edd86ccb9ae2e_r.jpg)

  
JavaScript 代码如下：  
var widgetNames = ['area', 'stuff']; // 定义数组存放控件名称  
function getWidgetValueByName(name) {  
var widget = _g().parameterEl.getWidgetByName(name); // 根据控件名获取控件值  
if (widget == undefined) return;  
var obj = {};  
obj[name] = widget.getValue();  
return obj; // 返回控件值组成的数组  
}  
var paramJson = widgetNames.map(getWidgetValueByName).reduce(function(a, b) {  
return Object.assign(a, b)  
});  
var paramJsonStr = JSON.stringify(paramJson); // 将 json 数据转换为字符串  
var col = _g().getParameterContainer().getWidgetByName("col").getValue();  
//alert(col);  
// 参数进行 url 编码  
var colNames = encodeURIComponent(col)  
//var colNames = encodeURIComponent("地区, 销售员, 产品类型, 产品, 销量") // 指定导出的数据列，导出字段按此顺序排列，为空默认导出所有  
// 调用导出接口  
//console.log(paramJsonStr);  
//console.log(colNames);  
_g().directExportToExcel("ds1", "销量", encodeURIComponent(paramJsonStr), colNames,""," ");  
显示代码  
**1.5 效果预览**  
保存模板，点击「分页预览」。查询「地区」、「销售员」，选择需要导出的数据列，点击「大数据集导出按钮」，导出内容与查询内容一致，且仅导出指定的数据列。如下图所示：  

![](https://pic3.zhimg.com/v2-1a3162196dd7e768fc73c3bac8a4a40a_r.jpg)

  
注：不支持移动端。

1.  **已完成模板**

**2.1 示例一**

已完成模板可参见：%FR_HOME%\webapps\webroot\WEB-INF\reportlets\doc\Parameter \ 大数据集导出固定参数值. cpt

**2.2 示例二**

已完成模板可参见：%FR_HOME%\webapps\webroot\WEB-INF\reportlets\doc\Parameter \ 大数据集导出动态参数值. cpt

**总结**

[帆软 FineReport](https://link.zhihu.com/?target=https%3A//www.finereport.com/%3Futm_source%3Dseo%26utm_medium%3Dseo%26utm_campaign%3Dfr-paper%26utm_term%3Dzhihu) 大数据集导出时连接的是 Presto 数据库，如果导出失败且有如下报错信息：

警告: 14:39:00 EventThread ， 帆软的人性化操作做得好，也是一套相对完整的[报表平台制作表格软件](https://link.zhihu.com/?target=https%3A//www.finereport.com/reporttool%3Futm_source%3Dseo%26utm_medium%3Dseo%26utm_campaign%3Dfr-paper%26utm_term%3Dzhihu)，也可进行[全链路数据建设](https://link.zhihu.com/?target=https%3A//www.finereport.com/product/functions/designer%3Futm_source%3Dseo%26utm_medium%3Dseo%26utm_campaign%3Dfr-paper%26utm_term%3Dzhihu)。中文用户界面用户更容易上手，都提供官方技术支持、论坛、交流小组等，可以帮助用户快速解决问题。ERROR [standard] com.facebook.presto.jdbc.NotImplementedException: Method Connection.prepareStatement is not yet implemented

![](https://pic1.zhimg.com/v2-995f88d74ef33cfb29601a304fab9f24_r.jpg)

只需要将驱动升级到 presto-jdbc-339.jar 即可，、进入[官网](https://link.zhihu.com/?target=https%3A//prestosql.io/download.html) ，下载驱动的方法如下图所示：

![](https://pic4.zhimg.com/v2-40f873d4383a9b6241eec21a5478e7f3_r.jpg)