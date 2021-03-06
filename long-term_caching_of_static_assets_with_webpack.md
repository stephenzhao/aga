

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
In order to reference a correct file, we’ll need some information about our build. This can be extracted from webpack compilation stats by using this simple plugin:
```js
// webpack.config.js
module.exports = {
  ...
  plugins: [
    function() {
      this.plugin("done", function(stats) {
        require("fs").writeFileSync(
          path.join(__dirname, "...", "stats.json"),
          JSON.stringify(stats.toJson()));
      });
    }
  ]
}
```
Or you can just use one of these plugins to export JSON files:
- **[webpack-manifest-plugin](https://www.npmjs.com/package/webpack-manifest-plugin ) **
- **[assets-webpack-plugin](https://www.npmjs.com/package/assets-webpack-plugin) **

A sample output of webpack-manifest-plugin for our config looks like:
```
{
  “main.js”: “main.155567618f4367cd1cb8.js”,
  “vendor.js”: “vendor.c2330c22cd2decb5da5a.js”
}
```
The rest depends on your server setup but I believe is pretty straightforward. There is a [nice walk through](http://clarkdave.net/2015/01/how-to-use-webpack-with-rails/#including-precompiled-assets-in-views) if you’re using Rails.

We’re done, you might think. Well, almost.

## Deterministic hashes
To minimise the size of generated files, webpack uses identifiers instead of module names. During compilation identifiers are generated, mapped to chunk filenames and then put into a JavaScript object called chunk manifest. It is (along with some bootstrapping code) then placed into the entry chunk and it is crucial for webpack-packaged code to work.

The problem with this is the same as before: whenever we change any part of the code, it will, even if the rest of its contents wasn’t altered, update our entry chunk to include the new manifest. This, in turn, will lead to a new hash and dismiss the long-term caching.
To fix that, we should use [chunk-manifest-webpack-plugin](https://github.com/diurnalist/chunk-manifest-webpack-plugin) which will extract that manifest to a separate JSON file. Here is an updated webpack.config.js which will produce chunk-manifest.json in our build directory
```js
// webpack.config.js
var ChunkManifestPlugin = require('chunk-manifest-webpack-plugin');

module.exports = {
  // your config values here
  plugins: [
    new ChunkManifestPlugin({
      filename: "chunk-manifest.json",
      manifestVariable: "webpackManifest"
    })
  ]
};
```

Since we removed the manifest from entry chunk, now it’s our responsibility to provide webpack with it. You have probably noticed the manifestVariable option in the example above. This is the name of the global variable where webpack will look for the manifest JSON and this is why it should be defined before we require our bundle in HTML. This is easy as inlining the contents of the JSON in HTML. Our HTML head section should look like this:
```
<html>
  <head>
    <script>
    //<![CDATA[
    window.webpackManifest = {"0":"main.3d038f325b02fdee5724.js","1":"1.c4116058de00860e5aa8.js"}
    //]]>
    </script>
  </head>
  <body>
  </body>
</html>
```
The second issue is how webpack requires modules: by default the order of modules in the bundle isn’t deterministic for the same set of dependencies. This means: modules can get different IDs from build to build, resulting in a slightly different content and thus different hashes. There is [an issue on Github](https://github.com/webpack/webpack/issues/950) which suggests to use [OccurenceOrderPlugin](http://webpack.github.io/docs/list-of-plugins.html#occurenceorderplugin) to work around this.

So the final webpack.config.js should look like this:
```js
var path = require('path');
var webpack = require('webpack');
var ManifestPlugin = require('webpack-manifest-plugin');
var ChunkManifestPlugin = require('chunk-manifest-webpack-plugin');

module.exports = {
  entry: {
    vendor: './src/vendor.js',
    main: './src/index.js'
  },
  output: {
    path: path.join(__dirname, 'build'),
    filename: '[name].[chunkhash].js',
    chunkFilename: '[name].[chunkhash].js'
  },
  plugins: [
    new webpack.optimize.CommonsChunkPlugin({
      name: "vendor",
      minChunks: Infinity,
    }),
    new ManifestPlugin(),
    new ChunkManifestPlugin({
      filename: "chunk-manifest.json",
      manifestVariable: "webpackManifest"
    }),
    new webpack.optimize.OccurenceOrderPlugin()
  ]
};
```
Using this config the vendor chunk should not be changing its hash unless you change its code or dependencies. Here is a sample output for 2 runs with moduleB.js being changed between the runs:
```
> webpack

Hash: 92670583f688a262fdad
Version: webpack 1.10.1
Time: 65ms

Asset Size Chunks Chunk Names
chunk-manifest.json 68 bytes [emitted]
vendor.6d107863983028982ef4.js 3.71 kB 0 [emitted] vendor
1.c4116058de00860e5aa8.js 107 bytes 1 [emitted]
main.5e17f4dff47bc1a007c0.js 373 bytes 2 [emitted] main

[0] ./src/index.js 186 bytes {2} [built]
[0] ./src/vendor.js 40 bytes {0} [built]
[1] ./src/moduleA.js 28 bytes {2} [built]
[2] ./src/moduleB.js 28 bytes {1} [built]

> webpack

Hash: a9ee1d1e46a538469d7f
Version: webpack 1.10.1
Time: 67ms

Asset Size Chunks Chunk Names
chunk-manifest.json 68 bytes [emitted]
vendor.6d107863983028982ef4.js 3.71 kB 0 [emitted] vendor
1.2883246944b1147092b1.js 107 bytes 1 [emitted]
main.5e17f4dff47bc1a007c0.js 373 bytes 2 [emitted] main

[0] ./src/index.js 186 bytes {2} [built]
[0] ./src/vendor.js 40 bytes {0} [built]
[1] ./src/moduleA.js 28 bytes {2} [built]
[2] ./src/moduleB.js 28 bytes {1} [built]
```
Notice that vendor chunk has the same filename. Exactly what we wanted!
## Conclusion
Webpack is very modular and allows lots of optimizations that aren’t enabled by default. The flexibility Webpack provides makes it possible to use it with any setup imaginable, but keeping in mind that long-term caching is a good general practice, I hope next versions will get better defaults to make things easier. Here is a sample [Github repository](https://github.com/okonet/webpack-long-term-cache-demo) with an example used in this article.