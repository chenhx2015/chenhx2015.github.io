---
title: JavaScript代码的解释和执行
date: 2016-04-20 21:09:11
tags:
categories: javascript
---

当函数执行的时候，执行上下文栈、变量对象、作用域链如何变化。

<!-- more -->

&emsp;&emsp;比较两段代码

```js
var foo = function() {
  console.log("foo1");
};

foo(); // foo1

var foo = function() {
  console.log("foo2");
};

foo(); // foo2
```

&emsp;&emsp;下面一段代码：

```js
function foo() {
  console.log("foo1");
}

foo(); // foo2

function foo() {
  console.log("foo2");
}

foo(); // foo2
```

打印的结果却是两个 `foo2`。

JavaScript 引擎并非一行一行地分析解释和执行代码，而是先把整段代码分析解释完成后再执行。分析代码时，如果有重复函数的定义，前面的定义会被后面的覆盖。上面的代码中，第一段代码函数定义采用匿名方式，第二段代码命名函数 foo 的定义，前者被后者覆盖。
