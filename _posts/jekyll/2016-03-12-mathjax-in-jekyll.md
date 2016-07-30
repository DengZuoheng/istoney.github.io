---
layout: post
title: 在Jekyll中配置MathJax数学公式
category: jekyll
tags: [jekyll, mathjax]
---
{% include JB/setup %}

[MathJax](http://www.mathjax.org/)是一款基于 JavaScript 的开源数学公式显示引擎，利用它可在各种当下流行的浏览器中“完美”渲染并显示 LaTax、MathML 以及 AsciiMath 数学公式。

但是，在 Jekyll 中使用 MathJax 时会出现严重问题，原因在于一些 LaTex 公式通常被作为 Markdown 标记处理从而影响公式的正常输出显示，因此需要一些特殊的配置。

### 配置并链接MathJax库
在`<head>`中添加javascript引用：

```js
<script type="text/x-mathjax-config">
  MathJax.Hub.Config({
    tex2jax: {
      skipTags: ['script', 'noscript', 'style', 'textarea', 'pre']
    }
  });

  MathJax.Hub.Queue(function() {
    var all = MathJax.Hub.getAllJax(), i;
    for(i=0; i < all.length; i += 1) {
      all[i].SourceElement().parentNode.className += ' has-jax';
    }
  });
</script>      
<script type="text/javascript" src="http://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML"> </script>
```

在style.css中添加`has-jax`类，方便进行样式控制：

```css
code.has-jax {font: inherit; font-size: 100%; background: inherit; border: inherit;}
```

数学公式插入格式为：

段间公式：

> \`\[
> LaTax 公式内容
> \]`

行内公式：

> \`\( LaTax 公式内容 \)`

将数学公式置于反引号之内，是因为反引号用于代码控制，使输入的代码不受Markdown语法干扰而原样输出，这样 MathJax 即可正确渲染解析这些公式。
