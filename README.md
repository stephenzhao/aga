# vue 使用笔记


画马第一步

``` 
├── app
│   └── shaibei
│       ├── build.js
│       ├── img
│       │   ├── i_arr.png
│       │   ├── i_logo.png
│       │   ├── i_mi.png
│       │   ├── i_p.png
│       │   ├── i_password.png
│       │   ├── i_password_on.png
│       │   ├── i_rili.png
│       │   ├── i_search.png
│       │   ├── i_username.png
│       │   ├── i_username_on.png
│       │   ├── place_holder.png
│       │   ├── shaibei_backstage_0.5_z.png
│       │   └── user.jpg
│       ├── index.html
│       ├── page
│       │   └── app.js
│       ├── store
│       │   ├── cookies.js
│       │   └── store.js
│       ├── style
│       │   └── style.css
│       └── template
├── build
│   └── bundle.js
├── common
│   ├── fetch
│   │   └── fetch.js
│   ├── vue
│   │   └── vue.js
│   └── zpto
│       └── zpto.js
└── webpack.config.js

```
use webpack 

```
$ webpack
```
编译出bundel.js 

webpack 会自动根据入口文件 app.js 中的依赖关系来打包成单个 js 文件，输出到配置文件中指定的 output path 中。

这是一个初步的成果，我们就可以按照commonjs的方式来写代码了。


有时考虑类库代码的缓存, 我们会考虑打成多个包, 这样不难
比如下边的配置, 首先 entry 有多个属性, 对应多个 JavaScript 包,
然后 commonsPlugin 可以用于分析模块的共用代码, 单独打一个包出来:

```js
var webpack = require('webpack');
var CommonsChunkPlugin = webpack.optimize.CommonsChunkPlugin;

module.exports = {
  entry: './app/shaibei/page/app.js',
  output: {
    path: './build',
    filename: 'bundle.js'
  },
  plugins:[
    new CommonsChunkPlugin('vendor.js');
  ]
}

```













