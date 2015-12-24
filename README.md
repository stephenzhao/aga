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


打包多个文件入口
``` js
$ webpack p1=./page1 p2=./page2 p3=./page3 [name].entry-chunk.js

```
```

module.exports = {
    entry: {
        p1: "./page1",
        p2: "./page2",
        p3: "./page3"
    },
    output: {
        filename: "[name].entry.chunk.js"
    }
}
```
这样我们就可以得到三个入口文件来分别引用
```
p1.entry.chunk.js, p2.entry.chunk.js and p3.entry.chunk.js
```

但是问题在与共用了的chunk没有被分离出来。

所以我加入了 CommonsChunkPlugin

```js
var CommonsChunkPlugin = require("webpack/lib/optimize/CommonsChunkPlugin");
module.exports = {
    entry: {
        p1: "./page1",
        p2: "./page2",
        p3: "./page3"
    },
    output: {
        filename: "[name].entry.chunk.js"
    },
    plugins: [
        new CommonsChunkPlugin("commons.chunk.js")
    ]
}
```

我们也可以指定分析某几个入口文件的以来来踢去共有部分
```js
var CommonsChunkPlugin = require("webpack/lib/optimize/CommonsChunkPlugin");
module.exports = {
    entry: {
        p1: "./page1",
        p2: "./page2",
        p3: "./page3",
        ap1: "./admin/page1",
        ap2: "./admin/page2"
    },
    output: {
        filename: "[name].js"
    },
    plugins: [
        new CommonsChunkPlugin("admin-commons.js", ["ap1", "ap2"]),
        new CommonsChunkPlugin("commons.js", ["p1", "p2", "admin-commons.js"])
    ]
};
```






