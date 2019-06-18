---
title: sentry-在复杂多变的用户环境中监控页面报错信息
date: 2019-06-18 11:50:36
tags: js
---

### 思考场景：开发测试环境一帆风顺，可是线上频频有用户反馈这有问题那有问题，测试还复现不出来，怎么办？怎么办！

线上环境多种多样，无论你代码写的再完美，也总有考虑不到的和测试测不到的地方。尤其是前端，ie，firefox，chrome，移动端微信下，系统自带浏览器下，还有webview下，抖音头条等等等等，测试肯定不可能面面俱到。

`sentry`就是用来解决这一痛点，让开发人员能快速准确地定位到问题的根源所在，快速修复。减少排查问题时间上的浪费
<!--more-->
一般公司内部都有搭建好的sentry，接下来说说我们在项目中如何接入  

### 安装  

一. 首先，Sentry 接入需要 DSN验证（别念成DNS），在项目设置中可以得到。新建项目时设置向导也会告诉你，把这个存下来，等下会用到  

二. 引入js的sdk，`raven-js`  
    ```bash
        > npm install raven-js --save
    ```

三. 在项目中配置Sentry  
```js
    import Vue from 'vue';
    import Raven from 'raven-js';
    import RavenVue from 'raven-js/plugins/vue';

    Raven
        .config('https://xxxxxx@sentry.io/123456', {}) // 第一步得到的DSN
        .addPlugin(RavenVue, Vue)
        .install()
```
默认会捕获window.onerror中的错误，添加`RavenVue`和`Vue`之后，Raven会自动注入到vue的`Vue.config.errorHandler`中，以便捕获vue的错误

### 在项目中使用  
按理说配置到这块就已经可以捕获到项目运行中的错误了，但是比较僵硬。有时候对于某些错误，我们需要自己传一些参数来让我们自己知道一些有用的信息，方便我们排查问题。  
`Raven.config`的第二个参数，可以传一些配置，如：
+ Raven默认会限制上报的字符串为250位，我们可以在config中添加
    ```
        {
            maxUrlLength: 0
        }
    ```
来解除限制
+ 对于接口的错误，我们更需要的是我们向接口传的参数，这个时候可以
```
{
    extra: {
        username: '',
        userId: '',
        ...
    }
}
```
+ 还有其余的配置，如 `release`, `logger`, `ignoreErrors`, `ignoreUrls`, `whitelistUrls`, `includePaths`, `headers`, `collectWindowErrors`, `captureUnhandledRejections`, `maxMessageLength`, `stackTraceLimit`, `autoBreadcrumbs`, `instrument`, `sampleRate`, `sanitizeKeys`  

### 手动抛错  
+ 手动throw new Error('xxxxx')。但在raven没有初始化完成时，这样就不顶用，可以加setTimeout。【缺点：代码量大且丑陋，且会在控制台报红】  

+ `import * as Sentry from '@sentry/browser';`
    - `Sentry.captureException(new Error('xxx'))`
    【Sentry.captureException传入一个Error或者Error实例（如catch的err）】
    优点：携带错误堆栈信息。
    缺点：参数僵硬，想传入自定义数据必须使用new Error(JSON.stringify({msg: '11111'}))，如果数据量大的话，sentry页面上也会非常难看
    - `Sentry. captureMessage('自定义消息', level)`
    【上报一个消息，可以设置level等级，不能传入自定义消息】  

+ Raven.captureMessage(message, options)
    message传入自定义消息(最终会被转为字符串)，options内可自定义上报内容，options内有个字段叫extra，可添加附加内容，如：
    ```js
    Raven.captureMessage('一个参数缺失错误的例子', {
        extra: {
            api: 'xxx/xxx/xxx.do',
            params: params,
            errMsg: res.msg,
            url: location.href
        }
    })
    ```
    `extra`内可以传需要的任何数据，这样在sentry页面就会多一条叫【附加数据】的条目，附加的参数会被完整有序的列出来，便于查看并完整的复现问题
    `options`字段内有`stacktrace`字段，设为true，便可附加错误堆栈  

+ 还有一个是Raven.captureException(err, options)，接收第一个参数为Error类或实例，源码中对err处理之后下一步还是调用了captureMessage。  

所以还是直接用Raven.captureMessage方便一些。
**可绑定到Vue原型上，方便调用，Vue.prototype.$sentryReport = Raven.captureMessage.bind(Raven)【注意绑定this】**

### 上传sourceMap
一般线上都是打包后的文件，减小文件体积，但是这个时候报的错也是压缩后的了，非常不容易定位问题。sourceMaop就是用来建立压缩代码与源码之间的映射关系。但我们又不希望sourceMap被带到线上，一是会增加打包后的体积，二是别人也可以看到代码。
针对这一问题，Sentry给出了解决方案：把压缩后的代码和对应的 .map 文件上传到对应 Project 的 Release 下。这样线上并没有sourceMap，我们还能定位到错误的具体某一行。  

1. 安装`@sentry/webpack-plugin`  
```bash
> npm i @sentry/webpack-plugin -D
```
2. 配置  
+ 在`webpack.prod.config.js`中
```js
// webpack.config.js

const config = require('../config')
const SentryCliPlugin = require('@sentry/webpack-plugin');
const pkg = require('./package');

module.exports = {
    // 其他配置项
    // ...
    plugins: [
        // 其他插件
        // ...
        new SentryCliPlugin({
            include: config.build.assetsRoot, // 打包后存放的路径
            ignoreFile: '.gitignore',
            ignore: ['node_modules'],
            configFile: '.sentryclirc', // 
            urlPrefix: '~/yourDirName', // 上线后路径，~为根路径/
            release: pkg.version
        }),
        // 其他插件
        // ...
    ],
    // 其他配置项
    // ...
};
```
+ 在项目根目录下建立`.sentryclirc`文件
```yml
[defaults]
url=https://sentry.yoursite.com/
org=web      ; # 组织名
project=test ; # 项目名

[auth]
token=xxxxxx ; # 授权码。在sentry页面右下角有个API，点进去可以生成
```
token需要在sentry页面右下角有个`API`点进去生成

## 至此，sentry就添加好了，可以在sentry的设置页开启邮件提醒，没有每天就会受到一堆邮件（哈哈哈）。如果项目特别多，可以在邮箱内设置规则，只收自己关心的项目