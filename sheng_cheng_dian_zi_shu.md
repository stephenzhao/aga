##构建书籍

首先，使用 gitbook build 将书籍内容输出到默认目录，也就是当前目录下的 _book 目录。

```zsh
$ gitbook build
Starting build ...
Successfully built!

$ ls _book
GLOSSARY.html       chapter1            chapter2            gitbook             glossary_index.json index.html          search_index.json
```
##创建 gh-pages 分支

执行如下命令来创建分支，并且删除不需要的文件：
```zsh
$ git checkout --orphan gh-pages
$ git rm --cached -r .
$ git clean -df
$ rm -rf *~
```
现在，目录下应该只剩下 _book 目录了，首先，忽略一些文件：

```zsh
$ echo "*~" > .gitignore
$ echo "_book" >> .gitignore
$ git add .gitignore
$ git commit -m "Ignore some files"
```
然后，加入 _book 下的内容到分支中：

``` zsh
$ cp -r _book/* .
$ git add .
$ git commit -m "Publish book"
```