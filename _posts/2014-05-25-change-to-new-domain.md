---
layout: post
title: "切换到新域名 gracece.com "
description: "换域名更改记录"
category: life
tags: 
- 域名
---
{% include JB/setup %}

### 关于域名
对于`gracece`，实在想不到什么合适的了，先这样凑合着吧。对于后缀，为什么当时会脑子抽风选了.net呢？其实很简单，因为当时
godaddy注册的时候，.net的价格比.com低。但是上次续费的时候发现godaddy真是坑爹，续费的时候两个域名几乎是一样贵的，上次续费花
了9美元，换域名太麻烦，所以忍忍就过了。今年再看续费价格真是逆天了，几乎找不到能用的优惠码，找到几个鸡肋优惠码之后还是要十多美元。
无奈，那就借着机会换回.com吧。

国内万网新域名注册只要￥49，好诱惑，但是国内的还是算了，没有安全感。经过一番搜寻，锁定使用[namesilo](https://www.namesilo.com/),
注册和续费都不到9美元，首年还送域名保护，还算比较划算。据说新用户注册可以用个优惠码便宜一美元，试了好一会，找的好几个码都提示
使用次数达到限制，无奈 不想再浪费时间，开虚拟机准备用paypal跳网银支付，随手再打了一次优惠码 `GFW`，居然就可以了！然后就愉快
地$7.99拿下`gracece.com`。剩下的就是改dns到dnspod,具体过程不表。


### 切换域名
- 旧域名指向vps，使用nginx 301跳转到新域名

    ```
server {
    server_name gracece.net;
    rewrite ^(.*) http://gracece.com$1 permanent;
}
    ```
- 新域名指向github page,具体可以看[Setting up a custom domain with GitHub Pages](https://help.github.com/articles/setting-up-a-custom-domain-with-github-pages#step-2-configure-dns-records)
和[Faster, More Awesome GitHub Pages](https://github.com/blog/1715-faster-more-awesome-github-pages),
域名切换生效可能需要一段时间，可以把电脑
的DNS改成`8.8.8.8`生效比较快一些。

- Github Page 相关配置文件更改
    - `CNAME` 域名更改
    - `_config.yml` 中更改相关跟域名有关的配置

- disqus 评论迁移

- 通知搜索引擎。处于对互联网负责的态度（其实估计也没多少人会来看我这小博客），需要告知搜索引擎域名更改。对于Google，可以
参考[webmasters](https://support.google.com/webmasters/answer/83106?hl=zh-Hans)，对于百度，参考[zhangzhan](http://zhanzhang.baidu.com/rewrite/index)。
估计不会那么快见效，慢慢观察吧。

### 遗留问题
有一段蛋疼的时间，还在维护旧的wordpress博客，因为上面挂了几个链接广告（这也就导致我博客的百度收录寥寥无几了）。虽然手里还是有vps，
接下来还是放弃这个旧站，统统301定向到github page 的博客吧。












