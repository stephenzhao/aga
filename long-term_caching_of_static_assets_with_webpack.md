

---
title: Webpack的静态资源版本管理方案
date: 2016-06-06 13:36:09
tags:
    - webpack
    - 翻译
    - 前端工程化
---
Webpack is a great way to package all your static resources such as JavaScript, CSS or even images, but to effectively use generated assets in production one should leverage long-term caching. Documentation on this topic is scattered across different resources and it is not quite easy to get it right. The aim of this article is to guide front-end developers through the complete setup.

## tl;dr
To enable long-term caching of static resources produced by webpack:
1. Use [chunkhash] to add a content-dependent cache-buster to each file.
2. Use compiler stats to get the file names when requiring resources in HTML.
3. Generate the chunk-manifest JSON and inline it into the HTML page before loading resources.
4. Ensure that the entry point chunk containing the bootstrapping code doesn’t change its hash over time for the same set of dependencies.
5. Profit!


## The problem
## Separate hash per file
## Get filenames from webpack compilation stats
## Deterministic hashes
## Conclusion
Webpack is very modular and allows lots of optimizations that aren’t enabled by default. The flexibility Webpack provides makes it possible to use it with any setup imaginable, but keeping in mind that long-term caching is a good general practice, I hope next versions will get better defaults to make things easier. Here is a sample Github repository with an example used in this article.