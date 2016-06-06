

---
title: Webpack的静态资源版本管理方案
date: 2016-06-06 13:36:09
tags:
    - webpack
    - 翻译
    - 前端工程化
---

> [原文](https://medium.com/@okonetchnikov/long-term-caching-of-static-assets-with-webpack-1ecb139adb95#.skj8j8h4u)


Webpack is a great way to package all your static resources such as JavaScript, CSS or even images, but to effectively use generated assets in production one should leverage long-term caching. Documentation on this topic is scattered across different resources and it is not quite easy to get it right. The aim of this article is to guide front-end developers through the complete setup.

## tl;dr
To enable long-term caching of static resources produced by webpack:
1. Use [chunkhash] to add a content-dependent cache-buster to each file.
2. Use compiler stats to get the file names when requiring resources in HTML.
3. Generate the chunk-manifest JSON and inline it into the HTML page before loading resources.
4. Ensure that the entry point chunk containing the bootstrapping code doesn’t change its hash over time for the same set of dependencies.
5. Profit!


## The problem
Each time something needs to be updated in our code, it has to be re-deployed on the server and then re-downloaded by all clients. This is clearly very inefficient since fetching resources over the network can be slow. This is why browsers cache static resources. But the way it works has a pitfall: if we don’t change filenames of our resources when deploying a new version, browser might think it hasn’t been updated and client will get a cached version of it.

Probably the simplest way to tell the browser to download a newer version is to alter asset’s file name. In a pre-webpack era we used to add a build number to the filenames as a parameter and then increment it:
```
application.js?build=1
application.css?build=1
```
It is even easier to do with webpack: each webpack build generates a unique hash which can be used to compose a filename. The following example config will generate 2 files (1 per entry point) with a hash in filenames:

```js

// webpack.config.js
module.exports = {
  entry: {
    vendor: './src/vendor.js',
    main: './src/index.js'
  },
  output: {
    path: path.join(__dirname, 'build'),
    filename: '[name].[hash].js'
  }
};
```
Running webpack with this config will produce the following output:
```
Hash: 55e783391098c2496a8f
Version: webpack 1.10.1
Time: 58ms
Asset Size Chunks Chunk Names
main.55e783391098c2496a8f.js 1.43 kB 0 [emitted] main
vendor.55e783391098c2496a8f.js 1.43 kB 1 [emitted] vendor
[0] ./src/index.js 46 bytes {0} [built]
[0] ./src/vendor.js 40 bytes {1} [built]

```
But the problem here is that, each time we create a new build, all filenames will get altered and clients will have to re-download the whole application code again.

Not exactly what we wanted, huh? So how can we guarantee that clients always get the latest versions of assets without re-downloading all of them?

## Separate hash per file
What if we could produce the same filename if the contents of the file did not change between builds? For example, the file where we put all our libraries and other vendor stuff does not change that often.

*Pro Tip!*

>Separate your vendor and application code with **[CommonsChunkPlugin](http://webpack.github.io/docs/list-of-plugins.html#2-explicit-vendor-chunk)** and create an explicit vendor chunk to prevent it from changing too often.

Webpack allows you to generate hashes depending on the file contents. Here is the updated config (excerpt):

```js
// webpack.config.js
module.exports = {
  ...
  output: {
    ...
    filename: '[name].[chunkhash].js'
  }
}
```
This config will also create 2 files, but in this case, each file will get its own unique hash.

```
main.155567618f4367cd1cb8.js 1.43 kB 0 [emitted] main
vendor.c2330c22cd2decb5da5a.js 1.43 kB 1 [emitted] vendor
```
>Don’t use [**chunkhash**] in development since this will increase compilation time. Separate development and production configs and use [**name**].js for development and [**name**].[**chunkhash**].js in production.


## Get filenames from
When working in development mode, you just reference a JavaScript file by entry point name in your HTML.
```js
<script src="main.js"></script>
```

## Deterministic hashes
## Conclusion
Webpack is very modular and allows lots of optimizations that aren’t enabled by default. The flexibility Webpack provides makes it possible to use it with any setup imaginable, but keeping in mind that long-term caching is a good general practice, I hope next versions will get better defaults to make things easier. Here is a sample Github repository with an example used in this article.