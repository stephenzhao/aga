# 生成电子书


这本书使用 Gitbook 撰写并生成网站，请查看 package.json 中的 scripts 配置和 /scripts 目录中的脚本来了解这本书的构建和发布过程。



```js
// 初始化 nodejs 依赖
$ npm install

// 安装 gitbook 插件
$ npm install gitbook-cli -g
$ gitbook install ./content

// 启动 gitbook 服务开始撰写工作
$ npm run serve-gitbook

// 生成 gitbook
$ npm run generate-gitbook

// 生成 wiki
$ npm run generate-wiki

// 发布到 gh-pages 分支
$ npm run deploy-gitbook

// 发布到 wiki
$ npm run deploy-wiki

// 生成并发布，是上面4条命令的快捷方式，通常编辑内容后只需要进行这个操作
$ npm run generate-and-deploy
```