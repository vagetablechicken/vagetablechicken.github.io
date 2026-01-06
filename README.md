# My Blog

本仓库为个人博客源码，基于 [Jekyll](https://jekyllrb.com/) + [Chirpy 主题](https://github.com/cotes2020/jekyll-theme-chirpy)。

---

## 快速开始

1. 安装 Ruby 环境（推荐用 Homebrew 或 rvm）：
	```bash
	# rvm install 3.0.0
	# rvm use 3.0.0
	# 或 macOS:
	brew install ruby@3.2
	echo 'export PATH="/opt/homebrew/opt/ruby@3.2/bin:$PATH"' >> ~/.zshrc
	source ~/.zshrc
	ruby -v
	```
2. 安装依赖：
	```bash
	gem install bundler
	bundle install
	```
3. 启动开发服务器：
	```bash
	bundle exec jekyll serve
	```

---

## 文章与写作

- 文件命名格式：`YEAR-MONTH-DAY-title.MARKUP`
- 文章资源建议放在 `/assets/` 或同名文件夹下
- Admonition 语法见：[jekyll-gfm-admonitions](https://github.com/Helveg/jekyll-gfm-admonitions)——暂时没用
- 更多写作规范见 [Jekyll 官方文档](https://jekyllrb.com/docs/posts/#creating-posts)

---

## 主题与功能

- 本站采用 [Chirpy](https://github.com/cotes2020/jekyll-theme-chirpy) 主题，支持目录、PWA、评论、SEO 等功能
	- 主题官方文档详见 [Chirpy Wiki](https://github.com/cotes2020/jekyll-theme-chirpy/wiki)
	- 主题支持的语法见 [Chirpy 语法支持](https://github.com/cotes2020/jekyll-theme-chirpy/blob/master/_posts/2019-08-08-text-and-typography.md)，渲染效果见[Chirpy 演示](https://chirpy.cotes.page/posts/text-and-typography/)
- 如需了解主题原版说明，见 [README_theme.md](README_theme.md)

---

## 其它

- ToC 目录在页面较窄时会显示在顶部，宽屏时显示在右侧
- 有问题欢迎 issue 或 PR
