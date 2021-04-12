---
title: ES6入门 
date: 2020-01-10 10:49:12
categories: [前端]
tags: [理论,前端]
---

# let命令

​	let与var在的区别:

1. let只在所定义的代码块中有用，var则是在整个函数中都阔以用<span style="color:red">（自己理解，还没验证）</span>。

2. let不存在变量提升的问题，变量提升就是，先使用再定义，不报错，只是会提示undefinde。 

   ``` javascript
   // var情况
   console.log(foo); // 输出undefinde
   var foo=2;
   // let情况
   console.log(foo); // 报错ReferencrError
   let foo=2;
   ```

3. 和var一样在同一作用域不可以重复声明。

# 区块作用域   

​	在es5中区块中不可定义函数而es6中可以。但是并不鼓励这么使用....

    ``` javascript
if(true){
    function f(){} // es5中是会报错的
}
    ```

避免不了可以使用函数表达式

   ``` javascript
if(true){
   let f = function f(){} // es5中是会报错的
}
   ```

