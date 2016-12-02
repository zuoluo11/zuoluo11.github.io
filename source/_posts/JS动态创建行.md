---
title: JS动态创建行
date: 2016-12-02 13:56:30
categories: 
- 前端基础
tags:
- javascript 
- easyui
---
JS在前台动态创建表格行，行的数量按照用户来定，行内有select下拉框、文本框或者是easyui-combobox组合框。
文本框增加数字验证。
<!--more-->
需要做如图的报表

![报表图片][1]
  [1]: http://p1.bqimg.com/4851/7b01293160fe36b3.png

1、js代码
```javascript
//添加空行
        function addRow() {
            var rowNum = $(".firmRow").length;
            //增加第一行
            var $newRow = $(".firmRow:first").clone();
            $newRow.removeClass("hideFirmRow");
            //修改名称
//          $newRow.find("select").attr("name", "cFirmGuid" + rowNum); //普通select下拉框
            //easyui-combobox组合框控件
            $newRow.find("td:eq(0)").html("<select class='easyui-combobox' name='cFirmGuid" + rowNum + "' style='width:100%;'></select>"); 
            $newRow.find("input:eq(0)").attr("name", "cContactAdd" + rowNum);
            $newRow.find("input:eq(1)").attr("name", "cIllegality" + rowNum);
            $newRow.find("input:eq(2)").attr("name", "iIssueNum" + rowNum);
            $newRow.find("input:eq(3)").attr("name", "cSituation" + rowNum);
            $("#JDJLK").before($newRow);
            $newRow.find(".easyui-combobox").combobox({
                valueField: 'cEnterpriseGUID',
                textField: 'cEnterpriseName',
                data: res,
                onChange: function (newValue, oldValue) {
                    //获得企业地址
                    changeEnterprise(newValue, rowNum);//修改下拉框值后，触发事件
                }
            });

        }
```

2、HTML代码
```html						
	<table>
	<tr class="firmRow hideFirmRow">
	    <td colspan="2">
		<select class="easyui-combobox" style="width:100%;">
		</select>
	    </td>
	    <td colspan="2">
		<asp:TextBox runat="server" />
	    </td>
	    <td colspan="4">
		<asp:TextBox runat="server" />
	    </td>
	    <td>
	      
		<asp:TextBox runat="server" onkeyup="if(this.value.length==1){this.value=this.value.replace(/[^1-9]/g,'')}else{this.value=this.value.replace(/\D/g,'')}"
		    onafterpaste="if(this.value.length==1){this.value=this.value.replace(/[^1-9]/g,'')}else{this.value=this.value.replace(/\D/g,'')}" />
	    </td>
	    <td colspan="3">
		<asp:TextBox runat="server" />
	    </td>
	</tr>
	</table>
```
3、firmRow表示是第一行，hideFirmRow用来将第一行隐藏
当需要创建行时，直接使用jquery的Clone，将firmRow克隆然后设置值就好了，方便操作。
```
 var $newRow = $(".firmRow:first").clone();
```
4、其中文本控件增加了JS验证，只能输入数字
```
<asp:TextBox runat="server" onkeyup="if(this.value.length==1){this.value=this.value.replace(/[^1-9]/g,'')}else{this.value=this.value.replace(/\D/g,'')}"
onafterpaste="if(this.value.length==1){this.value=this.value.replace(/[^1-9]/g,'')}else{this.value=this.value.replace(/\D/g,'')}" />
```

5、easyui-combobox 绑定初始化
```
	    $(".easyui-combobox").combobox({
		valueField: '值列名',
		textField: '文本列名',
		data: res //res是JSON对象
	    });
```