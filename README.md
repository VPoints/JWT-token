# 关于如何不用session做认证（JWT-token认证）

#### 正题： 我们这里要用的是 `jwt-smiple`模块

```
npm install jwt-smiple
```
#### 这个安装方面的就不说了直接往下走   （这里用express做服务端）

```
let express = require('express'),
    app = express(),
    jwt = require('jwt-simple'),
    jwtSecret = require('./secret').key; // 这个地方是你保存的密匙
//为了我们方便调用 我们把它放在 app set里
app.set('jwtSecret',jwtSecret)
```
#### 好了   这个时候我们就已经把准备工作做好了，下面要开始走流程

基本流程是这样的：

用户登录  == 》  返回token  ==》 用户接收token  ==》 用户带token请求  ==》 服务端验证

这很容易让我们想到微信的 ```access_token``` 的验证方式，微信正是用了这种方式来对用户的登录授权进行了处理

#### 当用户发送了登录请求并且我们认证成功的情况下我们进行token返回： 这里需要知道几点

token的组成分成三部分

第一部分是告知加密方式 `HS256` 微信加密方式貌似是 `sha1`

```
{
  "typ" : "JWT",
  "alg" : "HS256"

}
```

这个iss是issuer的简写，表明请求的实体。通常意味着请求API的用户。exp是expires的简写，是用来限制token的生命周期。一旦加密，JSON token就像这样：

```
{
    "iss" : "test" ,//你可以设置用户的名字
    "exp" : 1234567981 , // 这里是个时间戳 Date.now() 可以看到当前的时间戳
    "http://example.com/is_root": true
}
```

第三个部分是JWT根据第一部分和第二部分的签名（Signature）。

三个部分都由点（.）区别开来像这样

```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.
eyJ1c2VyIjoi6aOf5p2C5bqX5LiH5LiH5bKBIiwiZXhwIjoxNDkxNjM2OTg4LCJpYXQiOjE0ODkwNDQ5ODh9.
HTGEbWqY0wOg_8gGl5O_CGTsabGKzyeR8uHmMpAljAQ
```
#### 好了现在就当用户已经登录了，现在我们要返回一个token给客户端 这里有个好的时间转换工具 moment
```
npm install moment --save
```
```
let timeOut = require('moment')().
    add('days', 7).
    valueOf(); // 得到一个7天后的时间戳
var token = jwt.encode({
        iss: user.id, // 这里是用户的id
        exp: timeOut // 这里把时间戳放进去
    }, app.get('jwtSecret')); // 第二个参数是我们的密匙

res.send({
  token : token  // 将token返回出去
});
```

#### 假设客户端现在拿到了这个token，请求的时候就会带上token（先不讨论他怎么带的  一般是三个地方（连接中，参数中，请求头））
我们现在是拿到了token
````
var token = (req.body && req.body.access_token) || (req.query && req.query.access_token) || req.headers['x-access-token'];
if(token){
    var code = jwt.decode(token, app.get('jwtSecret'));
    //当时我就把他给解密了不是？
        if(code.exp >= Date.now()){ // 比较过期
            decoded.iss  // 这里就拿到了用户的信息了 怎么操作它你看着办
        }
}else{
    //用户登录界面
}
````

验证就是这么简单：我也是走过山路的人- -  聊天室做到一般session认证搞不了  心碎
可以看看我的聊天室  希望大家给点建议  http://www.fetchs.cn
