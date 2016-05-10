### 简介

当APP打开一个WebView时，会往页面中注入一个GJNativeAPI的对象，该对象有一个方法（send）。JavaScript通过调用该方法向Native发送'消息'。当WebView中的JS准备完成时，会在GJNativeAPI上注册一个onMessage回调，Native需要发送消息给JS时，会调用该方法。

下简称API

### 协议

Native与JS的通信消息协议是jsonrpc 2.0协议

### API封装

NG_sta中对GJNativeAPI进行了封装，相关的模块为：app/hybrid/lib/native/native.js。 使用实例：

```js
var NativeAPI = require('app/hybrid/lib/native/native.js');
NativeAPI.invoke('updateTitle',{
{
 text:'学习学习'
},
function (err, data) {
  console.log(data)
}
});

```

### API 列表
***

***getUserInfo***

版本:1.0

 描述: 获取当前的用户信息，如果用户未登录，则返回错误。 

``` json 

method: getUserInfo

 params: 
	▪	无
result: 
    - "user_id" 用户id
    - "user_name" 用户名
    - "phone" 手机号
    - "level" 用户等级
    - "score" 用户积分
    - "avatar"用户头像
    - "qq_openid"
    - "weixin_openid"
    - "sinaweibo_openid"
    - "token"


error
	▪	code:
	1.	-32000: 未知错误
	2.	-32001: 未登录

DEMO
→ { method: ‘getUserInfo’, params: null, id: 1}
← { result: { username: ‘zhaofei’, user_id: '123231', email: 'zhaofei@youwe.com', …}, id: 1}  //成功时
← { error: { code: -32001, message: '请登录后再试'}, id: 1}  // 需要登录

```

***updateHeaderRightBtn***

版本：1.0
描述：更新顶部条的样式

```
method: updateHeaderRightBtn

 params: 
    ▪ {
        action: 'show',
        icon: 'author',
        text: '楼主',
        data: {
           is_louzhu：0/1
        }
      }
result: 
    ▪   null
error
    ▪   code:
    1.  -32000: 未知错误

 params: 
    ▪ {
        action: 'show',
        icon: 'mark',
        text: '收藏',
        data: {
            has_marked: 0/1
        }
      }
result: 
    ▪   null
error
    ▪   code:
    1.  -32000: 未知错误

分享
    params: 
    ▪ {
        action: 'show',
        icon: 'share',
        text: '分享',
        data: {
            title:标题,
            weibo_title:微博标题,
            desc: 分享描述,
            img_url: 分享图片链接,
            url: 分享链接
        }
      }
result: 
    ▪   null
error
    ▪   code:
    1.  -32000: 未知错误

```

***bottomFixedBar***

版本:1.4

 描述: 注册一个底部浮条的点击事件

``` json
method: bottomFixedBar

 params: 
    ▪   {type:'forumDetail', controls:[{
            name:'like',
            data:{
                like_num: 20,
                has_liked:0 / 1
            },
            action: show
        },
        {
            name:'comment',
            data:{
            },
            action:'show'
        },
        {
            name:'page',
            data:{
                page_count:10,
                page:1
            },
            action:'show'
        }
        ]}
result: 
    ▪   null
error
    ▪   code:
    1.  -32000: 未知错误

```

***closeWebview***

版本:1.4

 描述: 关闭当前的webview 相当于点击顶部的返回键

``` json
method: closeWebview

 params: 
    ▪  null
result: 
    ▪   null
error
    ▪   code:
    1.  -32000: 未知错误

```



***forumLike***

版本:1.4

 描述:调起native论坛点赞

``` json
method: forumLike

 params: 
    ▪   {
        type_id: 1主贴喜欢，2子贴喜欢
        from_id: 被喜欢的帖子ID,
        for_like:false/true,
    }
result: 
    ▪   null
error
    ▪   code:
    1.  -32000: 未知错误
    2.  -32001: 未登陆

```

***forumReply***


版本:1.4

 描述:调起native论坛回复

``` json
method: forumReply

 params: 
    ▪   {
        tid: 帖子ID
        atc_title: 标题，可不填
        atc_content: 内容
        pid: 被回复的帖子ID
        reply_user_name:'damon'
    }
result: 
    ▪   {
        "data": {
            "subject": "我是标题aaa",
            "content": "测试发帖子测试发帖子测试发帖子测试发帖子测试发帖子测试发帖子",
            "reply_notice": 1,
            "created_userid": 194,
            "created_username": "光与影的交替",
            "created_ip": "192.168.0.150",
            "ipfrom": "Unknown",
            "fid": 8,
            "created_time": 1443443618,
            "disabled": 0,
            "ischeck": 1,
            "lastpost_userid": 194,
            "lastpost_username": "光与影的交替",
            "lastpost_time": 1443443618,
            "word_version": 11,
            "reminds": false,
            "useubb": 0
        },
        "tid": "1130",
        "fid": 8
    }
error
    ▪   code:
    1.  -32000: 未知错误
    2.  -32001: 未登陆

```

***forumAdd***

版本:1.4

 描述:调起native论坛发帖

``` json
method: forumAdd

 params: 
    ▪   {
        fid: 论坛板块ID
        atc_title: 标题
        atc_content: 内容
        subforumtype: 论坛fid
        flashatt: 图片的aid列表
        special: 可不填，默认为default
        reply_notice: 可不填，默认为1
    }
result: 
    ▪    {
        "data": {
            "subject": "我是标题aaa",
            "content": "测试发帖子测试发帖子测试发帖子测试发帖子测试发帖子测试发帖子",
            "reply_notice": 1,
            "created_userid": 194,
            "created_username": "光与影的交替",
            "created_ip": "192.168.0.150",
            "ipfrom": "Unknown",
            "fid": 8,
            "created_time": 1443443618,
            "disabled": 0,
            "ischeck": 1,
            "lastpost_userid": 194,
            "lastpost_username": "光与影的交替",
            "lastpost_time": 1443443618,
            "word_version": 11,
            "reminds": false,
            "useubb": 0
        },
        "tid": "1130",
        "fid": 8
    }
error
    ▪   code:
    1.  -32000: 未知错误
    2.  -32001: 未登陆

```

***forumPage***

版本:1.4

 描述:调起native评论列表分页

``` js
method: forumPage

 params: 
    ▪   {
        page: 当前页，
        page_count:总页数
    }
result: 
    ▪ null
error
    ▪   code:
    1.  -32000: 未知错误
    2.  -32001: 未登陆

```


***getDeviceInfo***

版本:1.2

 描述:获取设备情况

``` json
method: getDeviceInfo

 params: 
    ▪   null
result: 
    ▪ {
        user_id: 1130,
        token:'xxxx',
        sign:'xxxx',
        time: 1443443618,
        mid: 194,
        network_type:
    }
error
    ▪   code:
    1.  -32000: 未知错误

```


***back***

版本:1.2

 描述:返回 当开启一个webview时监听native的返回键

``` js
method: back

 params: 
    ▪   null
result: null
error
    ▪   code:
    1.  -32000: 未知错误

```


***createWebview***

版本:1.4

 描述:根据提供的url打开一个新的webView，顶栏有一个返回按钮。createWebView会根据参数决定是否显示搜索框等。
注意: 如果controls为空, 那么webview应该默认带一个title控件, 并且这个title控件的text为空, 以备后续updateTitle接口使用.

``` js
method: createWebView
params:
 - url: url有可能是相对的，如果是相对路径的话就根据当前webView的url进行计算
- controls: Object 可选 包含一个控件的配置。
    例如显示搜索框: control: [{ type: 'searchBox', text: '关键字', placeholder: '搜索' }]
result:

- 无

```


##JS API 

***bottomFixedBarClick***

当客户端，底部浮条被点击时

``` js
    
    method: bottomFixedBarClick

     params:返回的参数列表 
        ▪   {
                fn_index:0,
                name:'page',
                data:{
                    page: 翻到的页数
                }
            }
    result: 
        ▪   null


```
***checkBigPicture***

描述:打开app查看大图

``` js
method: checkBigPicture

 params: 
    ▪ {
        images: ["http://bbsimg.dajia365.com/bbs/1506/thread/4_276_ac47eac23317eae.jpg-thumb","http://bbsimg.dajia365.com/bbs/1506/thread/4_276_ac47eac23317eae.jpg-thumb"],
        current_index:2
    }
result: null

```
