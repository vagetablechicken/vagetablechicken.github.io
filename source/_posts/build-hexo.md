---
title: Hexo
date: 2021-01-06 11:18:37
tags: hexo
categories: build
---

# Hexo

Hexo的创建使用不多赘述。注意，先搞清楚hexo是否满足书写要求，可能你需要的是sphinx。

## With github

Hexo通过`hexo d`来发布到github，因此github中deploy到的分支是网站目录。也就是deploy后在本地可以看到的`.deloy_git`。
我个人使用了另一个branch来保存hexo项目的原始文件，也就是_config.yml配置，md文档等等。

## Q&A

名字解释：
`<root>`: 指的网页根目录，或者说是hexo deploy到的repo分支根目录。

Q: 遇到网站repo中路径是存在的，比如`<root>/categories/index.html`确实存在，但是浏览器点击却是404？
A: 可以自行访问尝试下`<root>/categories/`或者全路径，看看`index.html`是不是有问题。如果这样就能访问到正常页面，那么问题大概就是缓存了。你可以换个浏览器快速检查下，或清除该页面缓存重试下。

### 附赠-chrome清除单个页面缓存

chrome的`开发者工具-setting`中`Network-Disable cache(while DevTools is open)`，此选项打开后，在你想要调试的页面，打开开发者工具，就不会出现一些奇怪的缓存现象了。

## Hexo cmds

```
npm install -g hexo-cli
cd <blog-source>
npm install
# if ERROR Package xxx is not installed
npm install xxx

hexo new draft <draft_name> # just the name, no .md
# writing, writing, ...
hexo publish <draft_name>
```

```
hexo clean && hexo d
```

If you want to delete a post, just delete it in the source folder.

Localhost debug way:
```
hexo s --debug
```

## Hexo writing

### link to another post

Ref [Include Posts](https://hexo.io/docs/tag-plugins.html#Include-Posts).

```
{% post_link 要跳转文章md文件名(不要后缀) %}
```

### pics

hexo默认的图片格式不是markdown的语法，想要写md时预览，可能需要vscode安装插件Hexo Utils。
但有个方法可以继续用md的image格式（https://hexo.io/docs/asset-folders#Embedding-an-image-using-markdown）：
```
npm i --save hexo-renderer-marked
npm i --save hexo-asset-link
```
然后确保_config.yml中有（一般都有）：
```
post_asset_folder: true
marked:
  prependRoot: true
  postAsset: true
```

## NPM install

如果`apt install npm`遇到gcc update-alternatives slave问题，就先把update-alternatives清理了，再添加也方便，[清除参考写法](https://gist.github.com/ArseniyShestakov/a458b96a354014f80ab8d95676100c03)。

```
# cleanup
update-alternatives --remove-all gcc
# set one g++
update-alternatives --install /usr/bin/g++ g++ /usr/bin/g++-9 10
# recover
update-alternatives --remove-all cc
update-alternatives --remove-all c++
update-alternatives --remove-all gcc 
update-alternatives --remove-all g++
update-alternatives --remove-all clang
update-alternatives --remove-all clang++
update-alternatives --remove-all icc
update-alternatives --remove-all icc++
update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-8 80 --slave /usr/bin/g++ g++ /usr/bin/g++-8 --slave /usr/bin/gcov gcov /usr/bin/gcov-8 --slave /usr/bin/c++ c++ /usr/bin/g++
```
slave有个好处是不怕个别版本被修改，还是建议维持这个样子。

但ubuntu版本早的话，node.js版本可能太低了，hexo没法用，报错：
```
TypeError: line.matchAll is not a function
       at res.value.res.value.split.map.line (/root/vagetablechicken.github.io/node_modules/hexo-util/lib/highlight.js:121:26
```
比如，我ubuntu20.04装的node就是10.19.0，需要12及以上的。
升级：
```
sudo apt -y install curl dirmngr apt-transport-https lsb-release ca-certificates
curl -sL https://deb.nodesource.com/setup_12.x | sudo -E bash -
sudo apt -y install nodejs
```

`hexo version`可以查到相关version。

然后又可能遇到`npm rebuild node-sass`很慢的问题，卡在github上下载node-sass了。可以换下源试试，`npm i node-sass --sass_binary_site=https://npm.taobao.org/mirrors/node-sass/`。socks5代理会hang up，不知道是不是暂时的不稳定。

node版本再升高，https://github.com/nodesource/distributions#debian-and-ubuntu-based-distributions 。

hexo-renderer-sass有点旧，node版本高了不行，升高sass版本就没有问题了。

我想要个block admonition，但没有插件能做到解析'```{note}```'这种。只能改用现有语法，缺陷是vscode没法preview，只能deploy/server后看效果。最终还是决定使用hexo next主题自带的。preview的问题不大，用的频率不会太多。
切换sphinx有点麻烦，hexo配reStructuredText插件，例如hexo-renderer-pandoc，没成功，据说这个插件也比较大，deploy会慢。

## Next Dark

@media (prefers-color-scheme: dark) {}删了，只保留内层。否则，会考察OS或浏览器的主题，可能不能dark，强制弄黑，保护眼睛。

