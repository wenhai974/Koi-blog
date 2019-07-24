---
title: 用vue做微信公众号开发踩坑合集
date: 2018-10-09
categories:
- [VUE]
tags:
- [微信公众号]
---

## 准备工作
作为个人开发者，可以申请微信订阅号，但是很多借口权限并不能获得，如网页授权登录等，所以需要申请一个测试账号，具体位置：微信公众平台==>开发者工具==>公众平台测试号
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190125163249230.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzM2OTUyMQ==,size_16,color_FFFFFF,t_70)
获取到测试号之后，将appId 配置到自己项目中，并将appsecret和appId一并给后台同学，使用自己的微信号关注该测试号，并配置JS接口安全域名（用作jssdk签名），和网页授权获取用户信息回调域名。
##### 注意此处的域名须与你自己本地前端项目所跑的域名统一，且不能是locahost，具体方法此处就不细讲了
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190125163528802.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzM2OTUyMQ==,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190125164425457.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzM2OTUyMQ==,size_16,color_FFFFFF,t_70)
## 用户授权登录
官方文档参考此处：
https://mp.weixin.qq.com/wiki?t=resource/res_main&id=mp1421140842
这里分为如下几步，而前端同学需要做的只有第一步，引导用户同意授权，然后获取携带code的URL，并取得code，这个code即为当次用户授权的唯一凭证，并且只能使用一次
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190125165012454.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzM2OTUyMQ==,size_16,color_FFFFFF,t_70)
获取到code之后，将它通过自己的业务逻辑传给后台同学，获取userToken即可。
引导用户授权代码在项目的index.html中即可：

```js
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width,initial-scale=1,user-scalable=0">
    <title></title>
    <script src="https://res2.wx.qq.com/open/js/jweixin-1.4.0.js"></script>
    <script>
      // 网页授权之获取code
      const code = getUrlParam("code");
      if (!code) {
        const appid = "你自己的appId";
        const url = encodeURIComponent(location.href.split('#')[0]);
        window.location.href = 'https://open.weixin.qq.com/connect/oauth2/authorize?appid=' + appid + '&redirect_uri=' + url + '&response_type=code&scope=snsapi_userinfo&state=STATE#wechat_redirect'
      }
      //获取url中参数
      function getUrlParam(name) {
        var reg = new RegExp("(^|&)" + name + "=([^&]*)(&|$)");
        var r = window.location.search.substr(1).match(reg);
        if (r != null) return decodeURI(r[2]);
        return null;
      }
    </script>
  </head>
  <body style="font-size: 14px;">
    <div id="app"></div>
    <!-- built files will be auto injected -->
  </body>
</html>
```
**main.js**
```js
// 微信登录成功后再挂载Vue
Login().then(res => {
  new Vue({
    router,
    store,
    render: h => h(App),
    created () {}
  }).$mount('#app')
}).catch(e => {
  console.log('登录失败')
})
```
**Login.js**

```js
import axios from 'axios'
import Vue from 'vue'
import { ToastPlugin, LoadingPlugin } from 'vux'
import tools from './assets/js/tools'
Vue.use(ToastPlugin)
Vue.use(LoadingPlugin)

// 创建axios实例
const loginService = axios.create({
  baseURL: process.env.BASE_API, // api的base_url
  timeout: 8000 // 请求超时时间
})

export default function () {
  return new Promise((resolve, reject) => {
    var wxCode = tools.getUrlParam('code')
    var bonusCode = tools.getUrlParam('bonusCode')
    if (bonusCode) {
      // 携带红包id
      window.localStorage.setItem('bonusCode', bonusCode)
    } else {
      window.localStorage.removeItem('bonusCode')
    }
    if (wxCode) {
      Vue.$vux.loading.show()
      loginService({
        url: '/user/v1/login',
        method: 'post',
        data: {
          wxCode
        }
      }).then(res => {
        Vue.$vux.loading.hide()
        if (res.data.errCode === '0') { // 成功
          window.localStorage.setItem('userToken', res.data.data.userToken)
          window.localStorage.setItem('userInfo', JSON.stringify(res.data.data.userInfo))
          resolve(res)
        } else if (res.data.errCode === '600001') { // 600001: 获取accessToken失败-code过期
          window.location.href = tools.DelUrlParam('code') // 删除URL中的code参数，并刷新-重新登录
          reject(res)
        } else {
          reject(res)
        }
      }).catch(e => {
        reject(e)
      })
    }
  })
}
```
以上代码逻辑是每次用户进入都将重新登录，可根据自己项目需求在入口中判断userToken的存在与否即可避免每次登录，当然main.js中的登录逻辑也得响应修改。

## jssdk微信分享
官方文档：https://mp.weixin.qq.com/wiki?t=resource/res_main&id=mp1421141115
微信分享需要注意的是，因为是spa单页面应用，所以在调用接口之前都应该根据当前前端url重新config一次sdk。
>所有需要使用JS-SDK的页面必须先注入配置信息，否则将无法调用（同一个url仅需调用一次，对于变化url的SPA的web app可在每次url变化时进行调用,目前Android微信客户端不支持pushState的H5新特性，所以使用pushState来实现web app的页面会导致签名失败，此问题会在Android6.2中修复）。

需要注意的是，为了避免某些奇怪的问题，建议所有能使用https的地方，都统一使用https
以下为分享相关的封装：

```js
// wx-js-sdk授权
/* eslint-disable */
import userService from '../../service/userApi'

//获取jssdk需要的签名等参数
export default {
    initConfig: (reqUrl, callback) => {
        userService.getWxSgin(reqUrl).then((resp) => {
          if (resp.data.errCode === '0') {
            const { signature, nonceStr, timestamp } = resp.data.data;
            const appId = '你自己的appId'
            wx.config({
              signature, nonceStr, timestamp, appId, debug: false, jsApiList: [ "updateTimelineShareData", "updateAppMessageShareData", "onMenuShareTimeline", "onMenuShareAppMessage"]
            })
            wx.ready(() => {
              console.log('微信JSSDK初始化成功')
              if (callback) {
                callback()
              }
            })
            wx.error((e) => {
              console.log(e)
              console.log('微信JSSDK初始化失败')
            })
          }
        })
    },
    setShare: (params, callback) => {
      const origin = location.origin;
      //“分享给朋友”
      wx.onMenuShareAppMessage({
        title: params.title,
        desc: params.desc,
        imgUrl: params.imgUrl,
        link: params.link || origin,
        success: function () {
          //这里是回调函数 
          if (callback) {
            callback()
          }
        },
        cancel: function () {
          console.log('取消分享')
        }
      })
      //“分享到朋友圈”
      wx.onMenuShareTimeline({
        title: params.title,
        desc: params.desc,
        imgUrl: params.imgUrl,
        link: params.link || origin,
        success: function () {
          //这里是回调函数 
          if (callback) {
            callback()
          }
        },
        cancel: function () {
        	console.log('取消分享')
        }
      });
    }
}
```
 代码中的userService.getWxSgin，即为通过后台获取config相关参数，并初始化。使用时，只需要在initConfig的回调中再调用setShare即可。

#### 分享中遇到的坑
这里只说我做的时候遇到的坑，不排除还没有遇到的。
IOS分享的时候，在微信开发工具上一切正常，但到真机上之后，IOS自定义分享并不生效，排除掉分享图片大小，url合法性等问题之后。一个主要原因是：
**IOS微信浏览器记录的是第一次打开页面时的URL，也就是说，前端路由跳转时，IOS的url地址栏并不会根据前端路由改变，所以，当调用wx.config()时，你所注册的当前页面url还是第一次进入页面的url，所以获取到的时间戳，签名等参数，并不是当前前端路由所需要的，也就导致分享失败**
###### 解决办法：
在需要分享的页面中加入如下代码：

```js
 beforeRouteEnter (to, from, next) {
    next(vm => {
      var isiOS = !!navigator.userAgent.match(/\(i[^;]+;( U;)? CPU.+Mac OS X/)
      if (!window.localStorage.getItem('resultReload') && isiOS) {
        window.localStorage.setItem('resultReload', window.location.href)
        // 微信分享需要重新设置URL
        window.location.href = window.location.href
      }
      instance.setShareInfo()
    })
  },

```
在main.js中加入如下代码：

```js
router.beforeEach((to, from, next) => {
  if (to.meta.title) {
    document.title = to.meta.title
  }
  if (to.name !== 'bonusResult') {
    window.localStorage.removeItem('resultReload')
  }
  next()
})
```
**说明：**
在进入需要设置分享的页面时，判断是否为IOS，如果是，将url强制更改为前端路由对应的url，当然，还需要在路由变化时，删除掉用于做分享页面强制刷新的标识。然后再刷新之后再重新调用wx.config（）以及设置分享信息等。

###### PS：
在做分享的时候，请求微信的一切东西，能用https的地方，绝不用http，如下两个：

```js
<script src="https://res2.wx.qq.com/open/js/jweixin-1.4.0.js"></script>
window.location.href = 'https://open.weixin.qq.com/connect/oauth2/authorize?appid=' + appid + '&redirect_uri=' + url + ......
```
#### 总结
因为此项目工期较短，很多东西还未深究，以前东西仅为个人记录和参考，需要优化的地方还很多，比如登录逻辑，欢迎交流。