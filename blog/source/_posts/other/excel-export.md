---
title: node导出excel
date: 2018-02-01
categories: other
tags: work
---

之前我们写过[一篇文章](/other/excel/)，简单介绍了浏览器里如何使用get/post方法下载excel。本文我们介绍下如何利用node导出excel

我们使用 excel-export 这个库来完成

```javascript
const nodeExcel = require('excel-export');

const conf = {};
conf.name = 'sheet';

// 指定excel基本配置
conf.stylesXmlFile = path.resolve(__dirname, '../lib/styles.xml');

// 设置标题行
conf.cols = [
    { caption:'日期', type:'string', width:50 },
    { caption:'展示量', type:'number', width:50 },
    { caption:'点击量', type:'number', width:50 },
    { caption:'点击率', type:'number', width:50 },
    { caption:'平均点击花费', type:'number', width:50 },
    { caption:'千次点击花费', type:'number', width:50 },
    { caption:'花费', type:'number', width:50 }
];

// 设置内容区
conf.rows = [[...],[...]];

// 设置下载格式
ctx.type = 'application/vnd.openxmlformats';

// 设置文件名
ctx.attachment('demo.xlsx');

// 输出
ctx.body = new Buffer(nodeExcel.execute(conf), 'binary');
```

附录：styles.xml如下
```bash
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<styleSheet xmlns="http://schemas.openxmlformats.org/spreadsheetml/2006/main" xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006" mc:Ignorable="x14ac" xmlns:x14ac="http://schemas.microsoft.com/office/spreadsheetml/2009/9/ac"><fonts count="2" x14ac:knownFonts="1"><font><sz val="11"/><color theme="1"/><name val="Calibri"/><family val="2"/><scheme val="minor"/></font><font><sz val="11"/><color theme="1"/><name val="Calibri"/><family val="2"/><scheme val="minor"/></font></fonts><fills count="3"><fill><patternFill patternType="none"/></fill><fill><patternFill patternType="gray125"/></fill><fill><patternFill patternType="solid"><fgColor rgb="FFFFFFCC"/></patternFill></fill></fills><borders count="2"><border><left/><right/><top/><bottom/><diagonal/></border><border><left style="thin"><color rgb="FFB2B2B2"/></left><right style="thin"><color rgb="FFB2B2B2"/></right><top style="thin"><color rgb="FFB2B2B2"/></top><bottom style="thin"><color rgb="FFB2B2B2"/></bottom><diagonal/></border></borders><cellStyleXfs count="2"><xf numFmtId="0" fontId="0" fillId="0" borderId="0"/><xf numFmtId="0" fontId="1" fillId="2" borderId="1" applyNumberFormat="0" applyFont="0" applyAlignment="0" applyProtection="0"/></cellStyleXfs><cellXfs count="4"><xf numFmtId="0" fontId="0" fillId="0" borderId="0" xfId="0"/><xf numFmtId="14" fontId="0" fillId="0" borderId="0" xfId="0" applyNumberFormat="1"/><xf numFmtId="14" fontId="0" fillId="2" borderId="1" xfId="1" applyNumberFormat="1" applyFont="1"/><xf numFmtId="0" fontId="0" fillId="2" borderId="1" xfId="1" applyFont="1"/></cellXfs><cellStyles count="2"><cellStyle name="Normal" xfId="0" builtinId="0"/><cellStyle name="Note" xfId="1" builtinId="10"/></cellStyles><dxfs count="0"/><tableStyles count="0" defaultTableStyle="TableStyleMedium2" defaultPivotStyle="PivotStyleLight16"/><extLst><ext uri="{EB79DEF2-80B8-43e5-95BD-54CBDDF9020C}" xmlns:x14="http://schemas.microsoft.com/office/spreadsheetml/2009/9/main"><x14:slicerStyles defaultSlicerStyle="SlicerStyleLight1"/></ext></extLst></styleSheet>
```