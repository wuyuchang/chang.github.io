---
title: 微信JSSDK测试接口的Node实现
date: 2017-07-13 16:30:56
tags: [Wechat, JSSDK, Node.js, Chinese]
---

做了一个网站，放到线上，用微信打开，点击分享，可是分享后发给朋友的链接卡片是微信默认自带的，如下：
{% asset_img default_share.png 默认分享 %}
<br/>
这标题，描述以及图片是默认自带的，丑不说，分享给别人还以为是盗号网站呢，而接入微信的JSSDK后，分享可以自定义内容，如下：
{% asset_img jssdk_share.png 接口分享 %}
<br/>
我承认，虽然这分享的标题和内容也并不正经，但这不妨碍我表达**我们可以通过微信JSSDK定义分享内容**，接下来我们将一步一步从零实现JSSDK从后端Node.js的接入。

## 成为测试公众号开发者
### 登录测试公众号后台
首先我们需要在[微信公众平台](https://mp.weixin.qq.com/debug/cgi-bin/sandbox?t=sandbox/login)申请测试接口，地址：https://mp.weixin.qq.com/debug/cgi-bin/sandbox?t=sandbox/login
使用微信扫描登录后，即可来微信公众平台测试账号系统。

### 成为测试公众号开发者
其次在微信公众平台测试账号中，扫描测试号二维码，成为测试公众号的开发者


## 接口配置信息
### 修改接口配置信息
1. URL地址必须是你服务器上的地址，此地址要能通过浏览器的地址栏访问到（没有服务器？没关系，一会儿我们搭建一个）
假设我这里填写的服务器地址是"http://www.your_server_name.com/wxJssdk"
2. Token你可以随意填写，用作生成签名，（不知道签名？没关系，一会儿会用到这东西的）
假设我这里填写的Token是"[jegfjaeghfuccawegfgjdbh]()"

此时点击提交是会提示配置失败的，因为在提交的时候，微信是会请求你的服务器地址，而你的当前配置的地址并不能访问，所以会提示配置失败。不过别急，我们先来搭建一个简单的Node服务器，让微信能够访问该服务器。

### 搭建简单的Node服务器
我们需要在http://www.your_server_name.com 这个域名上搭建一个服务器，并且曝出一个接口为`/wxJssdk`
``` javascript
const express = require('express')
const app = express()

app.get('/wxJssdk', (req, res) => {
  res.send('请求成功了了了了')
})

app.listen(80, err => {
  if(!err) console.log('connect succeed')
})
```
现在我们在地址栏中访问http://www.your_server_name.com/wxJssdk ，如果页面显示“请求成功了了了了”，则进入到下一步，如果没有成功的话，检查一下你的服务器是否开启Node服务器，如：`node index.js`

此时保存微信测试公众号后台的接口配置信息，仍然会提示配置失败，这是因为我们没有按照它的要求返回。

### 根据微信测试公众号请求信息返回对应内容
根据[微信公众号开发文档接入指南](https://mp.weixin.qq.com/wiki?t=resource/res_main&id=mp1421135319)，微信在请求我们配置的接口时，会带上如下信息

| 参数 | 描述 |
| :--- | :--- |
|signature|微信加密签名，signature结合了开发者填写的token参数和请求中的timestamp参数、nonce参数。|
|timestamp|时间戳|
|nonce|随机数|
|echostr|随机字符串|
微信服务器会通过GET请求，来请求我们所配置的接口，并带上以上表格的信息，而我们必须按照以下要求，将微信发送的信息进行要求校验，以确保是微信发送的信息，其中校验流程如下：
  > 1）将token、timestamp、nonce三个参数进行字典序排序
2）将三个参数字符串拼接成一个字符串进行sha1加密
3）开发者获得加密后的字符串可与signature对比，标识该请求来源于微信

``` javascript
const express = require('express')
const app = express()
const sha1 = require('sha1')

app.get('/wxJssdk', (req, res) => {
  let wx = req.query

  let token = 'jegfjaeghfuccawegfgjdbh'
  let timestamp = wx.timestamp
  let nonce = wx.nonce

  // 1）将token、timestamp、nonce三个参数进行字典序排序
  let list = [token, timestamp, nonce].sort()

  // 2）将三个参数字符串拼接成一个字符串进行sha1加密
  let str = list.join('')
  let result = sha1(str)

  // 3）开发者获得加密后的字符串可与signature对比，标识该请求来源于微信
  if (result === wx.signature) {
    res.send(wx.echostr) // 返回微信传来的echostr，表示校验成功，此处不能返回其它
  } else {
    res.send(false)
  }
})
```
此时我们重启Node服务器，再次保存接口配置信息即可配置成功。

## 微信JSSDK使用步骤
根据[微信JSSDK说明文档](https://mp.weixin.qq.com/wiki?t=resource/res_main&id=mp1421141115)，我们需要完成如下：
### 填写安全域名
登录微信公众平台进入“公众号设置”的“功能设置”里填写“JS接口安全域名”，即要调用接口的域名，不包含协议
### 前端引入JS
在需要调用JS接口的页面引入此JS文件，（支持https）：http://res.wx.qq.com/open/js/jweixin-1.2.0.js
### 填写接口的配置信息
``` javascript
wx.config({
  debug: true, // 开启调试模式,调用的所有api的返回值会在客户端alert出来，若要查看传入的参数，可以在pc端打开，参数信息会通过log打出，仅在pc端时才会打印。
  appId: '', // 必填，公众号的唯一标识
  timestamp: , // 必填，生成签名的时间戳
  nonceStr: '', // 必填，生成签名的随机串
  signature: '',// 必填，签名
  jsApiList: [] // 必填，需要使用的JS接口列表，所有JS接口列表见附录2
});
```
### 调用接口
做你前端该做的，调用微信分享接口，或微信提供的其它接口，whatever you need，当然，这并不是我们所要讲的重点，我们接下来要看一下微信的配置信息从哪获取

## 在Node服务器中生成jssdk所需要的配置信息
从上一节可以看到，调用微信JSSDK需要以下信息
1. appId
2. timestamp
3. nonceStr
4. signature
5. jsApiList

其中：
1. 第1项appId是测试公众号后台的appId，我们已知
2. 第2项时间戳我们也可以自己生成
3. 第3项nonceStr可以随意填写，你可以理解为密钥
4. 第4项signature则需要我们按要求生成
5. 第5项是所需要接口的接口名

### 生成signature
> 生成签名之前必须先了解一下jsapi_ticket，jsapi_ticket是公众号用于调用微信JS接口的临时票据。正常情况下，jsapi_ticket的有效期为7200秒，通过access_token来获取。由于获取jsapi_ticket的api调用次数非常有限，频繁刷新jsapi_ticket会导致api调用受限，影响自身业务，开发者必须在自己的服务全局缓存jsapi_ticket 。

为了保证我们appid，appsecret，nonceStr等信息不在前端曝露，我们以下步骤将在服务器上进行操作，以免他人盗用信息获取（注：微信请求有每日次数限制，一旦超出，则无法使用，具体请求次数限制在微信公众号后台中可查看）

#### 生成access_token
根据微信开发文档[获取access_token文档说明]，我们需要将微信测试公众号后台的appid和和appsecret以GET的请求方式向https://api.weixin.qq.com/cgi-bin/token?grant_type=client_credential&appid=APPID&secret=APPSECRET 发起请求获取token，请求成功后我们会获得下返回JSON转化的字符串
``` JSON
{"access_token":"ACCESS_TOKEN","expires_in":7200}
```
具体请求代码如下：
``` javascript
const request = require('request')

const grant_type = 'client_credential'
const appid = 'your app id'
const secret = 'your app secret'

request('https://api.weixin.qq.com/cgi-bin/token?grant_type=' + grant_type + '&appid=' + appid + '&secret=' + secret, (err, response, body) => {
  let access_toekn = JSON.parse(body).access_token
})
```
#### 获取jsapi_ticket
``` javascript
const request = require('request')

const grant_type = 'client_credential'
const appid = 'your app id'
const secret = 'your app secret'

request('https://api.weixin.qq.com/cgi-bin/token?grant_type=' + grant_type + '&appid=' + appid + '&secret=' + secret, (err, response, body) => {
  let access_toekn = JSON.parse(body).access_token

  request('https://api.weixin.qq.com/cgi-bin/ticket/getticket?access_token=' + access_token + '&type=jsapi', (err, response, body) => {
     let jsapi_ticket = JSON.parse(body).ticket
  })
})
```

#### 生成签名
生成签名的步骤和最开始的`/wxJssdk`的算法是一致的，具体如下：
``` javascript
let jsapi_ticket = jsapi_ticket  // 上一步从获取的jsapi_ticket
let nonce_str = '123456'    // 密钥，字符串任意，可以随机生成
let timestamp = new Date().getTime()  // 时间戳
let url = req.query.url   // 使用接口的url链接，不包含#后的内容

// 将请求以上字符串，先按字典排序，再以'&'拼接，如下：其中j > n > t > u，此处直接手动排序
let str = 'jsapi_ticket=' + jsapi_ticket + '&noncestr=' + nonce_str + '&timestamp=' + timestamp + '&url=' + url

// 用sha1加密
let signature = sha1(str)
```

连接后的代码为：
``` javascript
const request = require('request')

const grant_type = 'client_credential'
const appid = 'your app id'
const secret = 'your app secret'

request('https://api.weixin.qq.com/cgi-bin/token?grant_type=' + grant_type + '&appid=' + appid + '&secret=' + secret, (err, response, body) => {
  let access_toekn = JSON.parse(body).access_token

  request('https://api.weixin.qq.com/cgi-bin/ticket/getticket?access_token=' + access_token + '&type=jsapi', (err, response, body) => {
     let jsapi_ticket = JSON.parse(body).ticket
     let nonce_str = '123456'    // 密钥，字符串任意，可以随机生成
     let timestamp = new Date().getTime()  // 时间戳
     let url = req.query.url   // 使用接口的url链接，不包含#后的内容

     // 将请求以上字符串，先按字典排序，再以'&'拼接，如下：其中j > n > t > u，此处直接手动排序
     let str = 'jsapi_ticket=' + jsapi_ticket + '&noncestr=' + nonce_str + '&timestamp=' + timestamp + '&url=' + url

     // 用sha1加密
     let signature = sha1(str)
  })
})
```

### 曝露接口，返回给前端
``` javascript
app.post('/wxJssdk/getJssdk', (req, res) => {
  const request = require('request')

  const grant_type = 'client_credential'
  const appid = 'your app id'
  const secret = 'your app secret'

  request('https://api.weixin.qq.com/cgi-bin/token?grant_type=' + grant_type + '&appid=' + appid + '&secret=' + secret, (err, response, body) => {
    let access_toekn = JSON.parse(body).access_token

    request('https://api.weixin.qq.com/cgi-bin/ticket/getticket?access_token=' + access_token + '&type=jsapi', (err, response, body) => {
       let jsapi_ticket = JSON.parse(body).ticket
       let nonce_str = '123456'    // 密钥，字符串任意，可以随机生成
       let timestamp = new Date().getTime()  // 时间戳
       let url = req.query.url   // 使用接口的url链接，不包含#后的内容

       // 将请求以上字符串，先按字典排序，再以'&'拼接，如下：其中j > n > t > u，此处直接手动排序
       let str = 'jsapi_ticket=' + jsapi_ticket + '&noncestr=' + nonce_str + '&timestamp=' + timestamp + '&url=' + url

       // 用sha1加密
       let signature = sha1(str)

       res.send({
         appId: appid,
         timestamp: timpstamp,
         nonceStr: nonce_str,
         signature: signature,
       })
    })
  })
})
```

## 前端请求后端接口，获取配置信息
### 获取配置
``` javascript
axios.post('/wxJssdk/getJssdk', {url: location.href}).then((response) => {
  var data = response.data

  wx.config({
    debug: false, // 开启调试模式,调用的所有api的返回值会在客户端alert出来，若要查看传入的参数，可以在pc端打开，参数信息会通过log打出，仅在pc端时才会打印。
    appId: data.appId, // 必填，公众号的唯一标识
    timestamp: data.timestamp, // 必填，生成签名的时间戳
    nonceStr: data.nonceStr, // 必填，生成签名的随机串
    signature: data.signature,// 必填，签名，见附录1
    jsApiList: ['onMenuShareTimeline', 'onMenuShareAppMessage'] // 必填，需要使用的JS接口列表，所有JS接口列表见附录2
  });

})

```
### 做你想做的，比如，自定义分享
``` javascript
if (wx) {
  axios.post('/wxJssdk/getJssdk', {url: location.href}).then((response) => {
    var data = response.data

    wx.config({
      debug: false, // 开启调试模式,调用的所有api的返回值会在客户端alert出来，若要查看传入的参数，可以在pc端打开，参数信息会通过log打出，仅在pc端时才会打印。
      appId: data.appId, // 必填，公众号的唯一标识
      timestamp: data.timestamp, // 必填，生成签名的时间戳
      nonceStr: data.nonceStr, // 必填，生成签名的随机串
      signature: data.signature,// 必填，签名，见附录1
      jsApiList: ['onMenuShareTimeline', 'onMenuShareAppMessage'] // 必填，需要使用的JS接口列表，所有JS接口列表见附录2
    });

    wx.ready(function () {
      wx.onMenuShareTimeline({
      title: wxShare.title,
      desc: wxShare.desc,
      link: wxShare.link,
      imgUrl: wxShare.imgUrl
      });

      wx.onMenuShareAppMessage({
      title: wxShare.title,
      desc: wxShare.desc,
      link: wxShare.link,
      imgUrl: wxShare.imgUrl
    });
  })

    wx.error(function (res) {
       // config信息验证失败会执行error函数，如签名过期导致验证失败，具体错误信息可以打开config的debug模式查看，也可以在返回的res参数中查看，对于SPA可以在这里更新签名。
    })
  })

}

```
至此，后端配置好了，我们已经能够正常使用微信的接口了，但是微信每日接口请求是有上限的，通过2000次/天，因此如果网站上线后，一量当天访问量超过2000次你的接口将失效，而且每次都请求微信接口两次，造成请求时间浪费，所以我们需要将以上获取信息缓存在后端，避免造成接口失效以及多次请求微信后台。

## 缓存access_token及jsapi_ticket
此处直接上代码，利用node_cache包进行缓存
``` javascript
const request = require('request')
const express = require('express')
const app = express()
const sha1 = require('sha1')
const waterfall = require('async/waterfall')
const NodeCache = require('node-cache')
const cache = new NodeCache({stdTTL: 3600, checkperiod: 3600}) //3600秒后过过期

app.get('/wxJssdk', (req, res) => {
  let wx = req.query

  // 1）将token、timestamp、nonce三个参数进行字典序排序
  let token = 'jegfjaeghfuyawegfgjdbh'
  let timestamp = wx.timestamp
  let nonce = wx.nonce

  // 2）将三个参数字符串拼接成一个字符串进行sha1加密
  let list = [token, timestamp, nonce]
  let result = sha1(list.sort().join(''))

  // 3）开发者获得加密后的字符串可与signature对比，标识该请求来源于微信
  if (result === wx.signature) {
    res.send(wx.echostr)
  } else {
    res.send(false)
  }
})

app.get('/wxJssdk/getJssdk', (req, res) => {
  let grant_type = 'client_credential'
  let appid = 'your app id'
  let secret = 'your app secret' // appscret

  let steps = []

  // 第一步，获取access_token
  steps.push((cb) => {

  let steps1 = []

    // 第1.1步，从缓存中读取access_token
    steps1.push((cb1) => {
      let access_token = cache.get('access_token', (err, access_token) => {
        cb1(err, access_token)
      })
    })

    // 第1.2步，缓存中有access_token则直接返回，如果没有，则从服务器中读取access_token
    steps1.push((access_token, cb1) => {
      if (access_token) {
        cb1(null, access_token, 'from_cache')
      } else {
        request('https://api.weixin.qq.com/cgi-bin/token?grant_type=' + grant_type + '&appid=' + appid + '&secret=' + secret, (err, response, body) => {
          cb1(err, JSON.parse(body).access_token, 'from_server')
        })
      }
    })

    // 第1.3步，如果是新从服务器取的access_token，则缓存起来，否则直接返回
    steps1.push((access_token, from_where, cb1) => {
      if (from_where === 'from_cache') {
        console.log(' === 成功从缓存中读取access_token: ' + access_token + ' ===')
        cb1(null, access_token)
      } else if (from_where === 'from_server') {
        cache.set('access_token', access_token, (err, success) => {
          if (!err && success) {
            console.log(' === 缓存已过期，从服务器中读取access_token: ' + access_token + ' ===')
            cb1(null, access_token)
          } else {
            cb1(err || 'cache设置access_token时，出现未知错误')
          }
        })
      } else {
        cb1('1.3获取from_where时，from_where值为空')
      }
    })



    waterfall(steps1, (err, access_token) => {
      cb(err, access_token)
    })
  })


  // 第二步，获取ticket
  steps.push((access_token, cb) => {
    let steps1 = []

    // 第2.1步，从缓存中读取ticket
    steps1.push((cb1) => {
      let ticket = cache.get('ticket', (err, ticket) => {
        cb1(err, ticket)
      })
    })

    // 第2.2步，缓存中有ticket则直接返回，如果没有，则从服务器中读取ticket
    steps1.push((ticket, cb1) => {
      if (ticket) {
        cb1(null, ticket, 'from_cache')
      } else {
        request('https://api.weixin.qq.com/cgi-bin/ticket/getticket?access_token=' + access_token + '&type=jsapi', (err, response, body) => {
          cb1(err, JSON.parse(body).ticket, 'from_server')
        })
      }
    })

    // 第2.3步，如果新从服务器取的ticket，则缓存起来，否则直接返回
    steps1.push((ticket, from_where, cb1) => {
      if (from_where === 'from_cache') {
        console.log(' === 成功从缓存中读取ticket: ' + ticket + ' ===')
        cb1(null, ticket)
      } else if (from_where === 'from_server') {
        cache.set('ticket', ticket, (err, success) => {
          if (!err && success) {
            console.log(' === 缓存已过期，从服务器中读取ticket: ' + ticket + ' ===');
            cb1(null, ticket)
          } else {
            cb1(err || 'cache设置ticket时，出现未知错误')
          }
        })
      } else {
        cb1('2.3获取from_where时，from_where值为空')
      }
    })

    waterfall(steps1, (err, ticket) => {
      cb(err, ticket)
    })
  })


  // 第三步，生成签名
  steps.push((ticket, cb) => {
    let jsapi_ticket = ticket
    let nonce_str = '123456'
    let timestamp = new Date().getTime()
    let url = req.query.url

    let str = 'jsapi_ticket=' + jsapi_ticket + '&noncestr=' + nonce_str + '&timestamp=' + timestamp + '&url=' + url
    let signature = sha1(str)

    cb(null, {
      appId: appid,
      timestamp: timestamp,
      nonceStr: nonce_str,
      signature: signature,
      ticket: ticket
    })
  })

  waterfall(steps, (err, data) => {
    if (err) {
      res.send({status: 'error', data: err})
    } else {
      res.send({status: 'success', data: data})
    }
  })
})

app.use('/wxJssdk/public', express.static('public'))

app.listen(80, err => {
  if(!err) console.log('connect succeed')
})

```
