---
title: Hexo
date: 2021-01-06 11:18:37
tags: hexo
categories: build
---

# Hexo

Hexo的创建使用不多赘述。

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
