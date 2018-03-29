---
layout: post
title: 如何阻止 jquery 事件触发多次的问题
category : [JS/JQuery]
tagline: "Supporting tagline"
tags : []
---
{% include JB/setup %}
# 如何阻止 jquery 事件触发多次的问题
--- 


> [如何阻止 jquery 事件触发多次的问题](https://site.douban.com/143011/widget/forum/6934620/discussion/60441740/) 


有时候jquery的事件，如click事件会触发多次，到时ga统计错误，或其他问题，怎么办呢？ 

最简单的做法是阻止冒泡，但是stopPropagation不一定有用，这时候可以试试stopImmediatePropagation 


<!--break-->  


还有其他方法： 
参考： http://www.gajotres.net/prevent-jquery-multiple-event-triggering/ 

我比较喜欢solution4: 
``` 
$(document).on('pagebeforeshow', '#index', function(){ 
$(document).on('click', '#test-button',function(e) { 
if(e.handled !== true) // This will prevent event triggering more then once 
{ 
alert('Clicked'); 
e.handled = true; 
} 
}); 
});
```


 