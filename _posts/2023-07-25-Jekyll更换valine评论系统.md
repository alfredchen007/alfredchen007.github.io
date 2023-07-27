---
layout: post

title: "Jekyll更换valine评论系统"

date: 2023-07-25 

description:  

cover: 'https://jc-1258611203.cos.ap-beijing.myqcloud.com/blog/2023-07-27-%E6%88%AA%E5%B1%8F2023-07-27%20%E4%B8%8B%E5%8D%884.56.44.png'

color: rgb(222,315,333)

tags: [Jekyll]
---

![](https://jc-1258611203.cos.ap-beijing.myqcloud.com/blog/2023-07-27-%E6%88%AA%E5%B1%8F2023-07-27%20%E4%B8%8B%E5%8D%884.30.17.png)
### Valine

最早支持的 disqus 因为 dddd 的原因已经不能用了，后来换成了来必力，一款韩国的评论系统，支持多个 SNS 登陆，用了好久一段时间，因为博客也没人看一般就是自用记录，所以就没有太关注。昨天网页端打开发现来必力的广告太难看了，所以准备换成无后端的 Valine 评论系统。

[ Valine 一款快速、简洁且高效的无后端评论系统。](https://valine.js.org/)

### 获取APP ID 和 APP Key

[可以参考 Valine 主页上的快速开始 ](https://valine.js.org/quickstart.html)

### 在 Jekyll 中集成配置

bing 了几篇 blog，发现配置有点过时了，跟现有的版本有点对不上了，参考官网记录一下最新配置。

#### 添加 html 片段

我使用的评论集成在了`_includes/comment.html`中，所以在这里增加 Valine 模块，如果评论是单独的 html 文件，可以在 _includes 下新建 valine.html，在 ` _layouts/post.html` 中合适地方引入即可

<img src="https://jc-1258611203.cos.ap-beijing.myqcloud.com/blog/2023-07-27-%E6%88%AA%E5%B1%8F2023-07-27%20%E4%B8%8B%E5%8D%883.55.03.png" style="zoom:150%;" />

```html
 //html代码段可能引起文章布局问题，图片代替
```

在 `comment.html` 中增加

![](https://jc-1258611203.cos.ap-beijing.myqcloud.com/blog/2023-07-27-%E6%88%AA%E5%B1%8F2023-07-27%20%E4%B8%8B%E5%8D%883.56.51.png)

#### config 配置 

如果要使用变量的话，需要在` _config.yml`中配置参数传递

```yaml
# Valine评论.
  # You can get your appid and appkey from https://leancloud.app
  # more info please open https://valine.js.org
valine: true
appid: 
appkey: 
avatar: # gravatar style
placeholder:  # comment box placeholder
pageSize: 10 # pagination size
recordIP: true # 是否记录评论者IP
enableQQ: false # 是否启用昵称框自动获取QQ昵称和QQ头像, 默认关闭
```

