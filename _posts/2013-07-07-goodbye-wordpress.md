---
layout: post
title: "Goodbye Wordpress "
description: "又是一番折腾，花了两天总算迁移到了github page 了！"
category: 日志
tags: 
- 折腾
- jekyll
---
{% include JB/setup %}

算是暑假计划的一部分吧，前前后后花了三天把博客从wordpress迁移到了github page 上，使用`jekyll`生成。这个过程说多了都是泪啊，但是还是得说一说。

### wordpress 何错之有？

其实`wordpress`并非十恶不赦，相反，他依然是一个很优秀的博客系统。只是自带的富文本编辑器实在是不忍直视，贴代码的时候总是能够轻而易举地把代码搞得乱七八糟，如此下来，体验实在不佳。到了后期，我都是用`sublime text 2`写 `markdown`并生成html然后复制到wordpress中去，真是够折腾的。另外，虽然博客插件已经装得尽量少，同时也做了不少优化，但在某些时候还是体验不佳。所以，是时候换一种选择了。

### 艰苦的迁移过程

首先当然是对`jekyll`的工作原理有了大概的了解，再加上清楚地认识到自己的美工能力是幼儿园水平，所以选择了使用了比较亲切的`jekyll-Bootstrap`。按照 [网站](http://jekyllbootstrap.com/) 上的步骤一步一步搭建起jekyll 环境并不难，这里就不再多说。感觉一不小心已经在电脑了装了全套`ruby`的软件。当然也不一定要在自己的电脑上搭建jekyll环境，在本地装只是为了能够更好地预览，毕竟`github page` 不是实时更新的，如果要调整主题或者进行其他设置，那本地的预览环境还是非常必要的。

博客最重要的部分当然就是 **文章** ,所以要把我这一年来写的文章全部转换为`markdown`类型。按照官方文档里面的提供的方法，我没能成功，可能是因为文档没有能够跟着ruby的更新而及时更新，总之就是对不上号。后来找了另外一个`python`写的插件 [exitwp](https://github.com/thomasf/exitwp) ，按照说明，很容易就把从wordpress导出的xml文件转为了若干篇 markdown类型的文章。程序不可能做到十全十美，所以还是得人肉进一步加工。大部分的问题都是缩进或是其他一些html转换过程中的错误判断，一篇篇预览再更改，纯粹就是体力活T_T。回头看才发现自己写了这么多这么水的文章，可是毕竟是自己写出来的，不舍得删（安慰自己，这是成长过程的记录）。另外一个工作就是图片的迁移，在前面的一些文章中，图片是直接上传到 wordpress的uploads目录的，而后面的文章中的图片就都是挂在新浪图床上的。现在统一都把图片传到新浪上，这又是一个浩大的工程，不过想想以后就不用痛苦了，还是硬着头皮一篇篇改完了。

再配置一下 `_config.yml` ,博客总算能够跑起来了！

### 细节调整

更换了模板，同时首页参考了 [zyzhang](http://zyzhang.github.io/) 的布局，同时调整了页面宽度以及字体大小等其他细节，使其看起来更加舒服。同时加上了代码高亮，这里使用了[google-code-prettify](https://google-code-prettify.googlecode.com/) ,看起来简洁一些。再把原来博客里面的评论迁移到了`disqus`，这样大部分工作算是完成了。

更改域名指向，几分钟之后，大功告成！

