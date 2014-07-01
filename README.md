======
微信公共平台消息接口服务中间件与API SDK
forked from [node-webot/wechat](https://github.com/node-webot/wechat)

wechat-pro是wechat的一个修改版，修改涉及：

##### 用request替换urllib

##### 提供proxy功能

```
var api = new API('appid', 'secret');
```

is changed to:

```
var api = new API('appid', 'secret', 'proxy');
```

and

```
var api = new OAuth('appid', 'secret');
```

is changed to:

```
var api = new OAuth('appid', 'secret', 'proxy');
```

======
原作README在[这里](./README-origin.md)

See wechat README English version [here](./README-origin.en.md)

原作API文档在[这里](http://node-webot.github.io/wechat/api.html)

Note: test failure is a known issue.