# weixin-signatrue-server

> 自从微信6.5+版本之后，如果分享到朋友圈需要自定义内容，都需要接入微信JS-SDK。当只需要自定义一个缩略图就要接入SDK的话，流程有些繁琐。而且需要在线调试，调用次数不能触发频率限制。故分享一个的经过验证的NODE实现的版本。

## 构建

``` bash
# 安装依赖
npm install

# 启动服务
node server.js

```

## 接入微信JS-SDK

参考文档[https://mp.weixin.qq.com/wiki?t=resource/res_main&id=mp1421141115](https://mp.weixin.qq.com/wiki?t=resource/res_main&id=mp1421141115)

* 登录微信公众平台进入“公众号设置”的“功能设置”里填写“JS接口安全域名”
* 在“微信公众平台-开发-基本配置”页中获得AppID和AppSecret，为生成签名使用。
* 请将server.js中的AppID和AppSecret修改为实际值。

## 代码目录

```
├── node_modules      --- node_modules
├── third_party       --- 微信提供的签名算法nodejs实现
    ├── sign.js
├── package.json      --- package.json
├── README.md         --- README
├── server.js         --- 实现签名的后端服务

```

## 前端调用

``` javascript
window.onload = wxjssdk();

function wxjssdk() {
    window.wxShareData = {
        // 分享标题，根据情况修改
        title: 'XXX',
        // 分享描述，根据情况修改
        desc: 'XXX',
        // 分享链接，必须与当前页面对应的公众号JS安全域名一致
        link: 'http://XXX.XXX.com',
        // 分享缩略图参考尺寸300*300，根据情况修改
        imgUrl: 'XXX.png'
    };

    $.ajax({
        // 与nginx配置的接收路径一致即可
        url: '/wxjssdk/getSignature',
        type: 'get',
        // 必须不能缓存，签名验证会失效
        cache: false,
        dataType: 'json',
        contentType: 'application/x-www-form-urlencoded; charset=utf-8',
        // 微信要求url动态获取
        data: {
            'url': location.href.split('#')[0]
        },
        success: function (data) {
            wx.config({
                // 可以开启debug模式，页面会alert出错误信息
                debug: false,
                appId: data.appId,
                timestamp: data.timestamp,
                nonceStr: data.noncestr,
                signature: data.signature,
                // 配置所需的API列表
                jsApiList: ['checkJsApi',
                        'onMenuShareTimeline',
                        'onMenuShareAppMessage']
            });
        }
    });

    wx.ready(function () {
        // 分享到朋友圈
        wx.onMenuShareTimeline({
            title: wxShareData.title,
            link: wxShareData.link,
            imgUrl: wxShareData.imgUrl,
            success: function () {
                // 用户确认分享后执行的回调函数
            },
            cancel: function () {
                // 用户取消分享后执行的回调函数
            }
        });
        // 分享给朋友
        wx.onMenuShareAppMessage({
            title: wxShareData.title,
            desc: wxShareData.desc,
            link: wxShareData.link,
            imgUrl: wxShareData.imgUrl,
            success: function () {
                // 用户确认分享后执行的回调函数
            },
            cancel: function () {
                // 用户取消分享后执行的回调函数
            }
        });
    });
}
```

## nginx配置

``` bash
server {
    # 配置前端页面访问端口，根据情况修改
    listen       8788;

    # 配置前端页面路径，根据情况修改
    location / {
        alias  /Users/XXX/workspace/h5/output/;
    }

    # 配置后端服务路径，与页面Ajax访问路径一致即可
    location /wxjssdk/getSignature {

        # 配置后端服务访问端口，与server.js中的监听端口一致即可
        proxy_pass http://localhost:8787;
    }
}
```
