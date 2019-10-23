---
title: 返回顶部按钮实现
date: 2016-04-06 11:08:19
tags:
- jquery
- js
---

返回顶部按钮常常出现在各大博客网站、购物网站、新闻门户网站等右下角，方便用户快速浏览。IOS设备上点击状态栏也会返回顶部。有两种方式实现返回顶部，一种使用 `JavaScript` 实现，一种是在页面顶部增加`id`为 `top` 的标签，赋值返回顶部按钮 `a` 标签 `href='#top'` 属性。

<!--more-->

```html
<div id="totop">
    <a title="返回顶部"><img src="/img/scrollup.png"/></a>
</div>
```
![totop](/img/scrollup.png)

css样式

```css
#totop {
  position: fixed;
  bottom: 10%;
  right: 50px;
  cursor: pointer;
  z-index: 10000;
}
```
## jQuery实现
jQuery实现很简单，封装的方法兼容性好。
`totop.js`

```js
(function($) {
  // 按钮出现的高度  
  var upperLimit = $(document).height() * 0.3;
  var scrollElem = $('#totop');
  // 返回顶部的过度速度
  var scrollSpeed = 700;
  // 隐藏按钮  
  scrollElem.hide();
  // 监听窗口滚动条滚动事件
  $(window).scroll(function() {
    var scrollTop = $(document).scrollTop();
    if (scrollTop > upperLimit) {
      $(scrollElem).fadeIn(500);//展示速度
    } else {
      $(scrollElem).fadeOut(500);//淡出速度
    }
  });
  // 鼠标点击事件
  $(scrollElem).click(function() {
    $('html, body').animate({
      scrollTop: 0
    },scrollSpeed);
    return false;
  });
})(jQuery);
```

## js原生实现
原生js实现也很简单，不过没有了动态效果

```js
window.onload = function() {
  var oTop = document.getElementById("top");
  var screenw = document.documentElement.clientWidth || document.body.clientWidth;
  var screenh = document.documentElement.clientHeight || document.body.clientHeight;
  oTop.style.left = screenw - oTop.offsetWidth + "px";
  oTop.style.top = screenh - oTop.offsetHeight + "px";
  window.onscroll = function() {
    var scrolltop = document.documentElement.scrollTop || document.body.scrollTop;
    oTop.style.top = screenh - oTop.offsetHeight + scrolltop + "px";
  }
  oTop.onclick = function() {
    document.documentElement.scrollTop = document.body.scrollTop = 0;
  }
} 
```
