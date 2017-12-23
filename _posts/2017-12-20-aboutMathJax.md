---
layout:     post
title:      "MathJax使用的坑点"
date:       2017-12-21 17:00:00
author:     "Huper"
tags:
    其他
---

一般来说，想在网页内渲染公式的话，直接在指定页面添加脚本就行了：

```html
<script 
        src="//cdn.bootcss.com/mathjax/2.7.0/MathJax.js?config=TeX-AMS-MML_HTMLorMML">
</script>
```

这个镜像地址是比较快的，其中也指定了渲染公式的默认配置，但是这样有个问题，就是默认情况下，行内公式只接受`\(  \)` 的标记，而且实际在输入公式的时候连`\`都要转义（类似java里的正则表达式），所以就成了这个样子`\\(  \\)`这个与一般的`markdown`编辑器语法根本不兼容，所以很麻烦。大多数`markdown`编辑器使用`$`作为行公式标记，但是`$`的行内标记在英文环境中容易引起歧义，所以默认配置里没有它。可以在页里添加如下脚本，动态修改`mathjax`的配置：
```html
<script type="text/x-mathjax-config">
MathJax.Hub.Config({
    tex2jax: {
        inlineMath: [ ['$','$'], ["\\(","\\)"] ],
        displayMath: [ ['$$','$$'], ["\\[","\\]"] ]
    }
});
</script>
```
注意这些一定要放在引入`mathjax`的脚本之前。
另外，由于要结合使用`mathjax`和`markdown`，所以要注意两者的保留标记冲突，比如虽然`|`在`mathjax`里不用转义，但是在`markdow`是表格的标记，所以一定要转义，并且只用`\|`  就行了。 再比如`{`在`mathjax`里是保留标记，但是在`markdown`里不是，所以也要转义，并且必须使用`\\{   \\}`。