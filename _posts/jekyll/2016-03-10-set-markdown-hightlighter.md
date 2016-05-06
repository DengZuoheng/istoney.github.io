---
layout: post
title: Jekyll配置Markdown代码高亮
catergory: jekyll
tags: [jekyll, highlighter]
---
{% include JB/setup %}

Github Pages在今年宣布开始使用Jekyll 3.0, 导致的变化有：

 - 将只支持kramdown作为Markdown引擎
 - 将只支持Rouge作为Markdown代码语法高亮
 - 大幅提升构建速度
 - 不在支持相对Permalinks和Textile
对于已经托管在Github Pages上Markdown高亮代码出现了对应的问题，当提交包含Markdown代码高亮的Post时，就会收到Warning：

```
The page build completed successfully, but returned the following warning:

You are attempting to use the 'pygments' highlighter, which is currently unsupported on GitHub Pages. Your site will use 'rouge' for highlighting instead. To suppress this warning, change the 'highlighter' value to 'rouge' in your '_config.yml' and ensure the 'pygments' key is unset. For more information, see https://help.github.com/articles/page-build-failed-config-file-error/#fixing-highlighting-errors.

GitHub Pages was recently upgraded to Jekyll 3.0. It may help to confirm you're using the correct dependencies:

  https://github.com/blog/2100-github-pages-now-faster-and-simpler-with-jekyll-3-0
```
所以，需要对Jekyll的相应配置进行修改，来适应这种变化。

### # 修改配置文件
修改Jekyll的配置文件`_config.yml`，将markdown选项改为kramdown，highlighter和syntax_highlighter都改为rouge.

```
highlighter: rouge
markdown: kramdown
kramdown:
  input: GFM
  syntax_highlighter: rouge
```

### # 配置高亮样式
Github一直比较推荐Backtick风格的代码块高亮，可以通过rouge附带的rougify工具生成不同高亮主题的css，然后添加到html中即可。

首先安装rouge，然后生成对应主题的css文件。

```
$ gem install rouge
$ rougify style monokai.sublime > assets/css/syntax.css
```
将这个css文件放到html中即可。

```html
<link rel="stylesheet" href="/assets/css/syntax.css">
```
如果需要修改代码块的背景颜色的话，修改syntax.css中的背景配色即可。Sublime的背景色为#272822，Atom的背景色为#282C34.

```css
pre[class='highlight'] {background-color:#272822;}
```
