---
title: Vue.js 学习笔记
tags:
  - Vue
categories: 白科技
abbrlink: f34a88b2
date: 2019-05-19 14:02:24
---
本文是关于 Vue.js 的学习笔记
<!--more-->

## 模板语法

### 插值

#### 文本

数据绑定最常见的方式是“Mustache”语法（双大括号）：

```html
<span>Message: {{ msg }}</span>
```

此时的 `msg` 是可以修改的，如果希望值不会被更新，可以使用 `v-once` 指令：

```html
<span v-once>This message would not be changed: {{ msg }}</span>
```

#### 原始HTML

双大括号会将数据解释为普通文本，而非HTML代码。若要输出真正的HTML代码，需要使用 `v-html` 指令：

```html
<div id="vm">
    <p id="p1">Using mustaches: {{ rawHtml }}</p>
    <p id="p2">Using v-html directive: <span v-html="rawHtml"></span></p>
</div>
```

```js
var vm = new Vue({
    el: '#app3',
    data: {
        rawHtml: '<span style="color: red">This should be red.</span>'
    }
});

```

最终 `p2` 将输出真正的HTML。

#### 特性

Mustache 语法不能作用在 HTML 特性上，此时可以采用 `v-bind` 指令:

```html
<span v-bind:id="dynamicId"></span>
```

此时 `dynamicId` 可以通过 JS 文件进行赋值。
对于布尔值来说......

#### 使用Javascript表达式

Vue.js 支持 Javascript 表达式：

```html
{{ number + 1 }}

{{ ok ? 'YES' : 'NO' }}

{{ message.split('').reverse().join('') }}

<div v-bind:id="'list-' + id"></div>
```

### 指令

指令是带有 `v-` 前缀的特殊特性。指令特性的值预期是**单个 Javascript 表达式**（`v-for` 例外）。指令的职责是，当表达式的值改变时，将其产生的连带影响，响应式地作用于DOM。例如：

```html
<p v-if="seen">Now, you can see me!</p>
```

这里， `v-if` 指令将根据 `seen` 的值的真假来插入/移除 `<p>` 元素。

#### 参数

一些指令能够接收一个“参数”，在指令名称之后以冒号表示。例如，`v-bind` 指令可以用于响应式地更新 HTML 特性：

```html
<a v-bind:href="url">...</a>
```

这里的 `href` 是参数，告知 `v-bind` 指令将该元素的 `href` 特性与表达式 `url` 的值绑定。

另一个例子是 `v-on` 指令，它用于监听 DOM 事件：

```html
<a v-on:click="doSomething">...</a>
```

在这里参数（click）是监听的事件名。

### 缩写

#### `v-bind` 缩写

```html
<!-- 完整语法 -->
<a v-bind:href="url">...</a>

<!-- 缩写 -->
<a :href="url">...</a>
```

#### `v-on` 缩写

```html
<!-- 完整语法 -->
<a v-on:click="doSomething">...</a>

<!-- 缩写 -->
<a @click="doSomething">...</a>
```

## 计算属性和侦听器
