
Wechat is a middleware and SDK of Wechat Official Account Admin Platform (mp.weixin.qq.com).

## Installation

Until I register it properly as a module please just clone it into the node_modules folder into node-wechat.

## Use with Express

NOTE: Body parsing done by this library. So no body parsing must be done before the WeChat handler. This is still legacy and I might make this optional.

The way I got around this was:

```
var router = express();
var bodyParser = require('body-parser');

var js = bodyParser.json();
var urlEnc = bodyParser.urlencoded({ extended: true });


router.use(function(req, res, next) {
  if (/^\/wechat/.test(req.url)) return next();
  
  js(req, res, function(err) {
    if (err) next(err);
    else urlEnc(req, res, next);
  });
});
```


```
var wechat = require('node-wechat');

app.use('/wechat', wechat('some token', function (req, res, next) {
  // message is located in req.weixin
  var message = req.weixin;
  if (message.Content === 'test') {
    // reply with text
    res.reply('This is a node-wechat test');
  } else {
    res.reply('You sent: ' + message.Content);
  }
}));
```

### Media Items

When sending media items you have to upload the item first before sending. TODO Provide example...

### Reply Messages

Auto reply a message when your followers send a message to you. also text, image, videos, music, thumbnails posts are supported. details API goes [here (official documents)](http://mp.weixin.qq.com/wiki/index.php?title=发送被动响应消息)

#### Reply with text
```
res.reply('Hello world!');
// or
res.reply({type: "text", content: 'Hello world!'});
```
#### Reply with Image
```
res.reply({
  type: "image",
  content: {
    mediaId: 'mediaId'
  }
});
```
#### Reply with voice
```
res.reply({
  type: "voice",
  content: {
    mediaId: 'mediaId'
  }
});
```
#### Reply with Video
```
res.reply({
  type: "video",
  content: {
    mediaId: 'mediaId',
    thumbMediaId: 'thumbMediaId'
  }
});
```
#### Reply with Music
```
res.reply({
  title: "Just some music",
  description: "I have nothing to lose",
  musicUrl: "http://mp3.com/xx.mp3",
  hqMusicUrl: "http://mp3.com/xx.mp3"
});
```
#### Reply with Thumbnails posts
```
res.reply([
  {
    title: 'Come to fetch me',
    description: 'or you want to play in another way ?',
    picurl: 'http://nodeapi.cloudfoundry.com/qrcode.jpg',
    url: 'http://nodeapi.cloudfoundry.com/'
  }
]);
```

### WXSession

Wechat messages are not communicate like traditional C/S model, therefore nothing Cookies will be store in Wechat client. this WXSession is designed to support access user's infomation via `req.wxsession`, with `connect.session` backed.

It's a simple demo:

```
app.use(connect.cookieParser());
app.use(connect.session({secret: 'keyboard cat', cookie: {maxAge: 60000}}));
app.use('/wechat', wechat('some token', wechat.text(function (info, req, res, next) {
  if (info.Content === '=') {
    var exp = req.wxsession.text.join('');
    req.wxsession.text = '';
    res.reply(exp);
  } else {
    req.wxsession.text = req.wxsession.text || [];
    req.wxsession.text.push(info.Content);
    res.reply('Message got ' + info.Content);
  }
})));
```

`req.wxsession` and `req.session` shares same store. width `redis` as persistence database, across processes sharing are supportd.

### Reply Waiter

a reply waiter is seems like a telephone menu system. it must be setup before activation. this function is supported upon WXSession. 

```
var List = require('wechat').List;
List.add('view', [
  ['reply {a}', function (info, req, res) {
    res.reply('Im Answer A');
  }],
  ['reply {b}', function (info, req, res) {
    res.reply('Im Answer B');
  }],
  ['reply {c}', 'Im Answer C (the shorthand method)']
]);
```

active the reply waiter we setuped before:

```
var app = connect();
app.use(connect.query());
app.use(connect.cookieParser());
app.use(connect.session({secret: 'keyboard cat', cookie: {maxAge: 60000}}));
app.use('/wechat', wechat('some token', wechat.text(function (info, req, res, next) {
  if (info.Content === 'list') {
    res.wait('view'); // view is the very waiter we setuped before.
  } else {
    res.reply('hehe');
    // or stop the waiter and quit.
    // res.nowait('hehe');
  }
})));
```

if waiter `view` actived, user will receive messages below:

```
reply a
reply b
reply c
```

reply waiter acquires both function and text as a `callback` action

```
List.add('view', [
  ['reply {a}', function (info, req, res, next) {
    // we callback as a function
    res.reply('Answer A');
  }],
  // or text as shorthand
  ['reply {c}', 'Answer C']
]);
```

if user's message is not in waiter's trigger texts. this message will be processd in the `else` way and can be stoped by `res.nowait()`, `res.nowait` method actions like `reply` method.

## Show cases
### Auto-reply robot based on Node.js

![Node.js API Auto-reply robot](http://nodeapi.diveintonode.org/assets/qrcode.jpg)

Codes here <https://github.com/JacksonTian/api-doc-service>

robots can be setup in PAASs like [CloudFoundry](http://www.cloudfoundry.com/), [appfog](https://www.appfog.com/) or [BAE](http://developer.baidu.com/wiki/index.php?title=docs/cplat/rt/node.js).

## API details
official document locates here [Messages API Guide (in Chinese)](http://mp.weixin.qq.com/wiki/index.php?title=消息接口指南)。

wachat 0.6.x supports shorthand methods below:

```
app.use('/wechat', wechat('some token', wechat.text(function (message, req, res, next) {
  // reply with text
  // { ToUserName: 'gh_d3e07d51b513',
  // FromUserName: 'oPKu7jgOibOA-De4u8J2RuNKpZRw',
  // CreateTime: '1359125035',
  // MsgType: 'text',
  // Content: 'http',
  // MsgId: '5837397576500011341' }
}).image(function (message, req, res, next) {
  // message为图片内容
  // { ToUserName: 'gh_d3e07d51b513',
  // FromUserName: 'oPKu7jgOibOA-De4u8J2RuNKpZRw',
  // CreateTime: '1359124971',
  // MsgType: 'image',
  // PicUrl: 'http://mmsns.qpic.cn/mmsns/bfc815ygvIWcaaZlEXJV7NzhmA3Y2fc4eBOxLjpPI60Q1Q6ibYicwg/0',
  // MediaId: 'media_id',
  // MsgId: '5837397301622104395' }
}).voice(function (message, req, res, next) {
  // Reply with Voice
  // { ToUserName: 'gh_d3e07d51b513',
  // FromUserName: 'oPKu7jgOibOA-De4u8J2RuNKpZRw',
  // CreateTime: '1359125022',
  // MsgType: 'voice',
  // MediaId: 'OMYnpghh8fRfzHL8obuboDN9rmLig4s0xdpoNT6a5BoFZWufbE6srbCKc_bxduzS',
  // Format: 'amr',
  // MsgId: '5837397520665436492' }
}).video(function (message, req, res, next) {
  // Reply with Video
  // { ToUserName: 'gh_d3e07d51b513',
  // FromUserName: 'oPKu7jgOibOA-De4u8J2RuNKpZRw',
  // CreateTime: '1359125022',
  // MsgType: 'video',
  // MediaId: 'OMYnpghh8fRfzHL8obuboDN9rmLig4s0xdpoNT6a5BoFZWufbE6srbCKc_bxduzS',
  // ThumbMediaId: 'media_id',
  // MsgId: '5837397520665436492' }
}).location(function (message, req, res, next) {
  // Reply with Location (geo)
  // { ToUserName: 'gh_d3e07d51b513',
  // FromUserName: 'oPKu7jgOibOA-De4u8J2RuNKpZRw',
  // CreateTime: '1359125311',
  // MsgType: 'location',
  // Location_X: '30.283950',
  // Location_Y: '120.063139',
  // Scale: '15',
  // Label: {},
  // MsgId: '5837398761910985062' }
}).link(function (message, req, res, next) {
  // Reply with Link
  // { ToUserName: 'gh_d3e07d51b513',
  // FromUserName: 'oPKu7jgOibOA-De4u8J2RuNKpZRw',
  // CreateTime: '1359125022',
  // MsgType: 'link',
  // Title: 'A link',
  // Description: 'A link has its desc',
  // Url: 'http://1024.com/',
  // MsgId: '5837397520665436492' }
}).event(function (message, req, res, next) {
  // Reply with Event
  // { ToUserName: 'gh_d3e07d51b513',
  // FromUserName: 'oPKu7jgOibOA-De4u8J2RuNKpZRw',
  // CreateTime: '1359125022',
  // MsgType: 'event',
  // Event: 'LOCATION',
  // Latitude: '23.137466',
  // Longitude: '113.352425',
  // Precision: '119.385040',
  // MsgId: '5837397520665436492' }
})));
```

*Tips*: `text`, `image`, `voice`, `video`, `location`, `link`, `event` must be set at least one.

### More simple APIs

Supported in 0.3.x and above.

```
app.use('/wechat', wechat('some token').text(function (message, req, res, next) {
  // TODO
}).image(function (message, req, res, next) {
  // TODO
}).voice(function (message, req, res, next) {
  // TODO
}).video(function (message, req, res, next) {
  // TODO
}).location(function (message, req, res, next) {
  // TODO
}).link(function (message, req, res, next) {
  // TODO
}).event(function (message, req, res, next) {
  // TODO
}).middlewarify());
```

### Functions Graph
![graph](https://raw.github.com/node-webot/wechat/master/figures/wechat.png)

*Tips*: Business logic in blue lines.

## License
The MIT license.

## Donation
buy me a cup of coffee please.

[![donate wechat](https://img.alipay.com/sys/personalprod/style/mc/btn-index.png)](https://me.alipay.com/jacksontian)
