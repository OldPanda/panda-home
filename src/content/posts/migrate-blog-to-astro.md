---
title: 将博客转移到了 Astro
pubDate: 2025-02-08 11:20:28 -08:00
categories: ["随便聊聊"]
tags: Astro, Blog, Cloudflare
heroImage: /images/blog/a-dog-in-moving-box.jpg
heroImageDescription: Photo by Erda Estremera on Unsplash

draft: true
---

大约在[七年前](https://old-panda.com/posts/hello-world)将博客转移到了 WordPress ，在这期间使用体验良好，也攒下了一堆文章，收获了各种 RSS 平台上的若干订阅。但随着近年家庭支出飙升，曾经眼中的一点小钱也变成了沉重的负担，看着一个月将近 20 块钱的服务器费用，不得不开始“降本增效”，所以今年年初花了将近一个月的工夫把博客迁移到了 Astro 。

# 为什么选择 Astro ？
因为我不想花钱，并且还希望保留自己的域名，那么最流行的选择基本只剩下了 [GitHub Pages](https://pages.github.com/) 和 [Cloudflare Pages](https://pages.cloudflare.com/) ，由于之前搭建 [Douban Book+ 主页](https://doubanbook.plus/)和[小地瓜主页](https://xiaodigua.app/)的经验，所以毫不犹豫地选择了 Cloudflare ，这样的学习成本最低。

那么下一步就是挑选主页搭建框架了，我听说 [Astro](https://astro.build/) 还是去年看到 X 站（前推特）上有很多网友讨论这个框架，并且很多网友利用它做出了非常漂亮的网站，并有一系列建站教程出现，比如说[这一篇](https://godruoyi.com/posts/how-to-build-your-blog/)就包含了非常详细的步骤，虽然没有那么多时间精力去详细学习了解一个新东西，但我还是想尝尝鲜。

于是在一系列拍脑袋的决策下，我决定在 Cloudflare Pages 上用 Astro 运行我的博客。

在此特别感谢 [Moeyua](https://blog.moeyua.com/) 开发的这款 [Typography 主题](https://github.com/Moeyua/astro-theme-typography)，简洁清爽，使读者可以沉浸于文字，而且在代码技术相关的文章方面也有不俗的表现力。

# 如何迁移？
WordPress 提供文章导出的功能， Astro 支持 Markdown 语法，两者格式并不兼容。幸运的是， Astro 官网提供了详细的[迁移教程](https://docs.astro.build/en/guides/migrate-to-astro/from-wordpress/)。成功将 WordPress 文章转换成 Markdown 格式之后，迁移的准备工作就算完成了。

在直接复制粘贴之前，还有几件事要做。

## 社交媒体图标
Typography 自带许多社交媒体图标，但还是无法满足我的需求，因为我同时活跃在微博、 X 站、小红书、蓝天、长毛象等各种平台，所以我需要一个[容量更大的图标库](https://github.com/OldPanda/panda-home/pull/1/commits/16bfd8884f394a3de6a68521cdd423072e7664f2)。

## 文章标签和图片
我的每篇文章几乎都带有若干个标签，这样容易归类查找，并且会配上一张或贴题或随意的图片，所以我需要一个专门的页面显示所有标签及某个标签下所有文章的列表，同时打开每篇文章时最开始需要显示图片，比如说下面这样

![](/images/blog/Screenshot_2025-02-12_at_19.49.04.png)

但这个主题没有提供这个功能，那么我只好[自己动手丰衣足食](https://github.com/OldPanda/panda-home/pull/1/commits/8f1047c0bf1850a963a38a14c74d11bf17bf24e8)了。

## 博客配置
这个没什么可讲的，就是把自己需要的信息[填进去](https://github.com/OldPanda/panda-home/pull/1/commits/c5903491895ced04a006183d99bb8b023c12c77b)即可。

## 文章迁移
[这块](https://github.com/OldPanda/panda-home/pull/1/commits/4a57525d86b84e5d1e3adabb43567969abdd4abc)是我花时间最多的一部分，包括所有图片都迁移和文章格式的整理，因为有些内容并没有那么完美地转换成 Markdown ，总共大几十篇吧。不敢想象假如哪天文章多了之后，再次面对迁移需求应该如何是好，估计到时候就得开发一款趁手的工具了，希望这是最后一次吧。

# 部署
完成了上述操作，就可以把内容发布到 Cloudflare Pages 上了，具体步骤我就不在这里重复了，感兴趣的读者可以参阅[这篇文章](https://godruoyi.com/posts/how-to-build-your-blog/)的“如何部署”部分。

# 参考资料
* [如何快速搭建自己的博客网站](https://godruoyi.com/posts/how-to-build-your-blog/)
* [Astro 官网](https://astro.build/)
