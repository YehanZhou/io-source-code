---
title: JS中Math.random()的使用和扩展
date: 2016-09-21 10:31:41
tags: js基础
categories: 学习笔记
---
Math.random()方法返回大于等于 0 小于 1 的一个随机数。对于某些站点来说，这个方法非常实用，因为可以利用它来随机显示一些名人名言和新闻事件。
## 在连续整数中取得一个随机数
```
值 = Math.floor(Math.random() * 可能值的总数 + 第一个可能的值)
```
例：产生1-10的随机数
```javascript
var rand1 = Math.floor(Math.random() * 10 + 1);
```

编写产生startNumber至endNumber随机数的函数
```javascript
function selectFrom(startNumber, endNumber) {
    var choice = endNumber - startNumber + 1;
    return Math.floor(Math.random() * choice + startNumber)
}
var rand2 = selectFrom(2,8);//产生2至8的随机数
```
## 在不相邻整数中取得一个随机数
### 在不相邻的两个整数中取得一个随机数
例：随机产生2或4中的一个数
```javascript
var rand3 = Math.random() < 0.5 ? 2 : 4;
```
### 在不相邻的多个整数中产生一个随机数
结合函数参数数组，可编写在不相邻的多个整数中产生一个随机值的函数
```javascript
function selectFromMess() {
    return arguments[Math.floor(Math.random() * arguments.length)]
}
//随机产生1、6、8中的一个数
var rand4 = selectFromMess(1, 6, 8);

//也可随机产生文本
var randomTxt1 = selectFromMess("安慰奖", "二等奖", "一等奖");
```
每次要输入这么多参数比较麻烦，可以改写一下函数
```javascript
function selectFromMessArray(arr) {
    return arr[Math.floor(Math.random() * arr.length)]
}
var arrayTxt=["一","二","三","四","五"];
var randTxt2 = selectFromMessArray(arrayTxt);
```
或者不改变原有方法，可以利用apply()这个方法传递数组参数
```javascript
var randTxt3 = selectFromMess.apply(null,arrayTxt);
```
关于apply方法的使用可以[看这里](http://www.cnblogs.com/delin/archive/2010/06/17/1759695.html)