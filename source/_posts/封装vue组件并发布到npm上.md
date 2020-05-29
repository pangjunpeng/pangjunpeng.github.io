---
title: 封装vue组件并发布到npm上
date: 2019-03-02 20:50:30
tags:
    - js
    - vue
---
# 准备
准备一个项目用来开发我们的组件
```bash
> vue init webpack-simple <project-name>
```
根据自己需要选择配置项，准备工作完成  

目录结构如下
```
├── src/                            // 源码目录
│   ├── toast/                      // 组件
│   │   ├── toast.vue               // 组件实现代码
│   ├── switch/                     // 组件
│   │   ├── switch.vue              // 组件实现代码
│   ├── lazyLoad/                     // 组件
│   │   ├── lazyLoad.vue              // 组件实现代码
│   ├── App.vue                     // 页面入口
│   ├── main.js                     // 程序入口
│   ├── index.js                    // （所有）插件入口
├── index.html                      // 入口html文件
```
# 写*.vue组件
<!-- more -->
```
<template>
    <div class="lazy-load-warpper" :style="{'background-image': `url(${defaultImg})`}">
        <img v-if="loading && loadingStatus===1" :src="loading" alt="加载中">
        <img v-else-if="loadingStatus===0" :src="src" alt="图片" class="target-img">
        <slot></slot>
    </div>
</template>
<script>
export default {
    name: 'lazyLoad',
    props: {
        defaultImg: String,
        loading: String,
        src: {
            type: String,
            required: true
        },
        beforeLoad: Function,
        onLoad: Function,
        onError: Function
    },
    data(){
        return {
            loadingStatus: 1,
            showImg: this.loading
        }
    },
    created(){
        let img = new Image()
        img.onload = () => {
            this.onLoad && this.onLoad()
            this.loadingStatus = 0
        }
        img.onerror = () => {
            this.onError && this.onError()
            this.loadingStatus = -1
        }
        this.beforeLoad && this.beforeLoad()
        img.src = this.src || this.loading
    },
}
</script>
<style scoped>
    .lazy-load-warpper {
        display: flex;
        justify-content: center;
        align-items: center;
        background-repeat: no-repeat;
        background-size: cover;
    }
    .target-img {
        width: 100%;
        height: 100%;
    }
</style>
```

# 导出
在src下新建`index.js`文件
```javascript
import Toast from './toast/toast';
import Switch from './toast/switch';
import lazyLoad from './lazyLoad/lazyLoad';
const components = [
    Toast,
    Switch,
    lazyLoad
]
if (typeof window !== 'undefined' && window.Vue) {
    components.map(component => {
        Vue.component(component.name, component)
    })
}
/* 如果要做的不止这一件事（比如还有过滤器指令什么的）。可以给组件添加install方法。因为Vue.use会调用传入的组件的install方法 */
// Toast.install = function(Vue){
//   Vue.component(Toast.name, Toast)
// };
/* 如果想让别人直接script标签引入就能用，那就手动use一下 */
// if (typeof window !== 'undefined' && window.Vue) {
//   window.Vue.use(Toast)
// }

export default {
    Toast,
    Switch,
    lazyLoad
};
```
# 测试
我们的webpack-simple就可以启动一个本地服务，`npm run dev`
然后在你的`main.js`中import`index.js`即可像一个普通包一样应用到APP.vue或其他页面测试

# 打包
修改`webpack.config.js`文件
```javascript
module.exports = {
    entry: './src/index.js',
    output: {
        // 修改打包出口，在最外级目录打包出一个 index.js 文件，我们 import 默认会指向这个文件
        path: path.resolve(__dirname, './dist'),
        publicPath: '/dist/',
        filename: 'my-components.js',
        library: 'window-component', // 如果script引入方式 会在window下生成这个变量
        libraryTarget: 'umd', // libraryTarget会生成不同umd的代码,可以只是commonjs标准的，也可以是指amd标准的，也可以只是通过script标签引入的
    },
}
```
# 编写Readme
readme的友好程度决定了这个包的使用量  
readme参考地址  
[知乎：如何写好Github中的readme？](https://www.zhihu.com/question/29100816)  
[github: README-Template.md](https://gist.github.com/PurpleBooth/109311bb0361f32d87a2#project-title)
# 发布
### 打开`package.json`文件，填写重要信息
```js
{
    "name": "my-test-component",
    "description": "A Vue.js component for test",
    "version": "1.0.0",
    "author": "pangjunpeng <pangjunpeng@baijiahulian.com>",
    "license": "MIT",
    "private": false,           // 发布开源需要改为false
    "main": "./dist/test.js",   // 这是别人import你的包时指向的文件
    "keywords": ["test"],       // 让别人搜到你
    "repository": {             // 如果在git上放着，填写项目地址
        "type": "git",
        "url": "git+https://github.com/gaotu-tech-fe/my-test-component.git"
    }
}
```
### 发布前注意几个重点
1. 打开`.gitignore`文件，去掉`dist`（如果你的main指向dist的话），不然打包发上去别人import你的会`Module Not Found`
2. `package.json`中`name`不能和别人重复
3. 每次改动后，都需要改`package.json`中`version`，不然发布不成功

### 打开命令行 
1. 设置npm源: `npm config set registry https://registry.npmjs.org/`
2. 登录
    `npm login` 输入username, password, email。登陆成功后
3. 发布
    `npm publish`
4. 可以去[npm官网](https://www.npmjs.com/)看看可否能搜到你的插件（根据发布地址自己决定）

# 使用
安装
```bash
> npm i my-test-component
```
在`main.js`中import进来，像别的包一样去使用吧，当然怎么用你自己最清楚。所以，`readme`一定要写好！

main.js中  
![](/images/11264410-08ed71fdb510e2c5.png)    

页面内使用  
![](/images/11264410-1cd4a1fa597997bb.png)  

# 完结，撒花
对了，如果想撤销发布的包，可以`unpublish`（发布之后24小时内可操作）
```bash
> npm unpublish my-test-components
```