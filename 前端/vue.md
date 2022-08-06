官网地址：

# 一、vue安装









## Vue.set的使用









## 计算属性vs方法



计算属性有缓存







# Vue脚手架



`npm config set registry https://registry.npm.taobao.org`

`npm install -g @vue/cli`

![image-20211203154838998](https://gitee.com/jobim/blogimage/raw/master/img/20211203154839.png)





vue.runtime.esm.js 缺少模板解析器

## 默认配置



# 浏览器的本地存储

# 消息订阅与发布

pubsub-js 消息订阅与发布

![image-20211207145919847](https://gitee.com/jobim/blogimage/raw/master/img/20211207145927.png)



## 动画效果

集成第三方动画

![image-20211207161929572](https://gitee.com/jobim/blogimage/raw/master/img/20211207161929.png)





## axios

![image-20211207163501984](https://gitee.com/jobim/blogimage/raw/master/img/20211207163502.png)



# vue-resource

![image-20211208093022796](https://gitee.com/jobim/blogimage/raw/master/img/20211208093022.png)



## vuex



![image-20211208105310976](https://gitee.com/jobim/blogimage/raw/master/img/20211208105311.png)

## 路由



![image-20211221105309121](https://gitee.com/jobim/blogimage/raw/master/img/20211221105316.png)







# element-ui中表单校验规则

[vue element-ui的form表单校验总结，动态校验、自定义校验（用正则校验手机号码、数字、url、中文）_新林。的博客-CSDN博客_element form 手机校验](https://blog.csdn.net/qq_21187515/article/details/103543721)



# axios的使用

[axios在vue中的使用_前端报刊的博客-CSDN博客_vue使用axios](https://blog.csdn.net/qq_36995521/article/details/115223647)



# router

1. `this.$router.push`：跳转到指定url路径，并在history栈中添加一个记录，点击后退会返回到上一个页面。
2. `this.$router.replace`：跳转到指定url路径，但是history栈中不会有记录，点击返回会跳转到上上个页面。
3. `his.$router.go(n)`：向前或者向后跳转n个页面，n可为正整数或负整数。
   如果n为0的话，将刷新当前页面。

[vue-router（路由）详细教程_Yin_Xiaobo的博客-CSDN博客_vue 路由](https://blog.csdn.net/wulala_hei/article/details/80488727)

[vue3如何使用vue-router_周小盗的博客-CSDN博客_vue3使用vuerouter](https://blog.csdn.net/weixin_42581303/article/details/117951449)

[官网介绍 | Vue Router (vuejs.org)](https://router.vuejs.org/zh/introduction.html)





# vue .prettierrc文件

[vue .prettierrc文件常见配置， 以及配置 Prettier - Code formatter 插件 格式化_Liang_Cheng_Jiu的博客-CSDN博客](https://blog.csdn.net/weixin_47988564/article/details/107900245)





# vue-table-with-tree-grid的使用

[vue-table-with-tree-grid的使用（黑马笔记）_笑道三千的博客-CSDN博客](https://blog.csdn.net/weixin_42349568/article/details/109577844)

[vue-table-with-tree-grid文档及用法_Sky_fy_1314的博客-CSDN博客_vue-table-with-tree-grid 文档](https://blog.csdn.net/Sky_fy_1314/article/details/107383695)



# VueX

[VueX（Vue状态管理模式) - 简书 (jianshu.com)](https://www.jianshu.com/p/2e5973fe1223)





# 跨域请求问题







# babel-plugin-transform-remove-console

可以移除所有console.*的调用

# 项目发布

使用CDN优化打包





npm init -y

npm i express -S

编写app.js

```js
const express = require('express')
const app = express()


app.use(express.static('./dist'))

app.listen(80, () => {
    console.log('server running at http://127.0.0.1')
  })
```







# 复制数组





# vue.config.js



# 插槽相关（待学习）

