---
title: chrome开发者工具
tags:
- 前端
- 工具
- chrome
categories:
- 前端
- 工具
- chrome
---



# Elements（元素）

查看元素的代码：点击左上角的箭头图标（或按快捷键`Ctrl+Shift+C`）进入选择元素模式，然后从页面中选择需要查看的元素，然后可以在开发者工具元素（Elements）一栏中定位到该元素源代码的具体位置。
1. Styles：样式表，可以在此进行相关样式操作
2. Computed：盒子模型参数，更好的配合Styles。
3. Event Listeners：事件监听，定位事件代码
4. DOM Breakpoints：断点
5. Properties：选中元素的所有属性
6. Accessibility
7. 原生截屏工具：首先选中要截屏的元素，然后快捷键`ctrl + shift +  p` 显示命令行输入框。输入`node sceenshot`，回车可以截屏



# Console（控制台）

浏览器都自带了一些API，比如 $ 用法和jquery基本一致
1. 打印日志     


```js
console.log(1);     
console.info(2);    
console.warn(3);    
console.error(4);   
console.table([{name:'张三', age: 23}, {name:'李四', age: 26}]);    
```
2. 重写事件监听，用于调试或者绕过原网站做的js限制

```js
$('li').off('click').on('click', function(){
	location.href = 'https://www.baidu.com'
});
```

3. 数学运算



# Network（网络）

用于分析请求资源的状态、类型、大小、所用时间、瀑布流、以便于性能优化     



# Sources（源码）

提供了所有网站前端的静态资源及目录，通过断点进行调试



# Application（应用）

1. LocalStorage
2. SessionStorage
3. Cookies（XSS攻击）



# Performance（性能调优）

前身就是timeline，chrome 57之后变成了performance
这个面板前期可能用得不多，主要用于后期优化
1. 控制面板（Controls）

2. 概括（Overview）

3. Flame Chart

4. 详细信息（Detail）
    
    - 蓝色(Loading)：网络请求和HTML解析 
    
    - 黄色(Scripting)：JavaScript编译和执行
    - 紫色(Rendering)：样式解析，计算，渲染。
    - 绿色(Painting)：重排，重绘 
    - 灰色(other)：其它资源花费加载的时间
    - 白色(Idle)：空闲等待时间



# Security（安全）

证书安全相关



# Memory（内存分析）



# Audits



# 参考链接

[参考链接](https://segmentfault.com/a/1190000011868916)