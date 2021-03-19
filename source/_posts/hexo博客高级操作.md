---
title: hexo博客高级操作
date: 2018-12-25 13:03:00
tags: next 
categories: hexo
---

> 官网文档：[http://theme-next.iissnan.com](http://theme-next.iissnan.com/)
>
> hexo next高级操作：https://mp.weixin.qq.com/s/U7KPp3HsQQCCQz3z8o5q-Q

<!--more-->

## 修改``的样式

Next默认的主题样式是灰色的，不太显眼，颜色也不太好看，可以将其设置成自己喜欢的颜色，效果如下：

![图片](https://mmbiz.qpic.cn/mmbiz_png/19cc2hfD2rBB66CoumStyqbDTRmOCTP72RMvW1ABVYA7ZYyseLUtz2xQtlP5cWd1SAkEzHGLicoNItH1BpPiaMJg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

配置起来也是很简单，只需要在`\themes\next\source\css\_custom\custom.styl`文件中添加如下代码：

```
// Custom styles.
code {
    color: #ff7600;
    background: #fbf7f8;
    margin: 2px;
}
// 大代码块的自定义样式
.highlight, pre {
    margin: 5px 0;
    padding: 5px;
    border-radius: 3px;
}
.highlight, code, pre {
    border: 1px solid #d6d6d6;
}
```

> 以上的颜色可以配置自己喜欢的，比如效果图中的颜色是我个人比较喜欢的。