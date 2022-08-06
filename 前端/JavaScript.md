# async/await

[JavaScript异步函数async/await - 掘金 (juejin.cn)](https://juejin.cn/post/6996253288102363143)





# Promise的使用





[JavaScript Promise | 菜鸟教程 (runoob.com)](https://www.runoob.com/js/js-promise.html)



```js
setTimeout(function () {
    console.log("First");
    setTimeout(function () {
        console.log("Second");
        setTimeout(function () {
            console.log("Third");
        }, 3000);
    }, 4000);
}, 1000);
```

```js
new Promise(function (resolve, reject) {
    setTimeout(function () {
        console.log("First");
        resolve();
    }, 1000);
}).then(function () {
    return new Promise(function (resolve, reject) {
        setTimeout(function () {
            console.log("Second");
            resolve();
        }, 4000);
    });
}).then(function () {
    setTimeout(function () {
        console.log("Third");
    }, 3000);
});
```





# 箭头函数





[=> js 中箭头函数使用总结_yangxiaodong88的博客-CSDN博客_js 箭头函数](https://blog.csdn.net/yangxiaodong88/article/details/80460332#:~:text=js中 的 箭头函数 ES6标准新增了一种新的 函数 %3AArrow Function (,x%3B } 箭头函数 相当于匿名 函数 %2C并且简化了 函数 定义。.)





# 解构

[JavaScript中的解构及数组对象操作_带着梦想飞翔的博客-CSDN博客_js解构数组](https://blog.csdn.net/u013008795/article/details/94745984)

[JS对象解构 - 毁梦 - 博客园 (cnblogs.com)](https://www.cnblogs.com/cheng-du-lang-wo1/p/7259343.html)