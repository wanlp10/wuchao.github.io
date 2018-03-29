---
layout: post
title: JavaScript 以 POST 方式打开新页面
category : [JS/JQuery]
tagline: "Supporting tagline"
tags : []
---
{% include JB/setup %}
# JavaScript 以 POST 方式打开新页面 
--- 

<!--break-->  


> [JavaScript 以 POST 方式打开新页面](https://www.cnblogs.com/lideqiang/p/5667576.html) 


场景：前置的查询页面，选择查询条件后提交到另一个页面。

方式很多，列出我知道的几种

1.window.open.

2.Response.Redirect.

3.Server.Transfer.

方法一和方法二都存在同样的问题，因为是get方式提交的，所以提交的数据都会显示URL中，一个是安全问题，另外一个是URL长度限制，在IE中，URL最大长度为2083.所以数据量过多时会导致数据丢失。

于是考虑到通过POST方式传递参数。

``` 
/*
 * 功能： JS跳转页面，并已POST方式提交数据
 * 参数： URL 跳转地址 PARAMTERS 参数
 * 返回值：
 * 创建时间：20160713
 * 创建人： 
*/
function ShowReport_Click() {

    var parames = new Array();
    parames.push({ name: "param1", value: "param1"});
    parames.push({ name: "param2", value: "param2"});

    Post("SupplierReportPreview.aspx", parames);

    return false;
}
  
/*
 * 功能： 模拟form表单的提交
 * 参数： URL 跳转地址 PARAMTERS 参数
 * 返回值：
 * 创建时间：20160713
 * 创建人： 
*/
function Post(url, params) {
    //创建form表单
    var temp_form = document.createElement("form");
    temp_form.action = url;
    //如需打开新窗口，form的target属性要设置为'_blank'
    temp_form.target = "_self";
    temp_form.method = "post";
    temp_form.style.display = "none";
    //添加参数
    for (var item in params) {
        var opt = document.createElement("textarea");
        opt.name = params[item].name;
        opt.value = params[item].value;
        temp_form.appendChild(opt);
    }
    document.body.appendChild(temp_form);
    //提交数据
    temp_form.submit();
}
```


 