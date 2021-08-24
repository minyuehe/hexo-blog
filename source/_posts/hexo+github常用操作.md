---
title: hexo+github常用操作
date: 2020-07-04 18:47:58
categories: 网站部署
tags: 
- hexo
- Markdown
---

# 如何部署到网站

## 三步走：

### 1,1 hexo clean  

清理缓存文件(db.json)和已生成的静态文件(public)

```bash
$ hexo clean
```

[^特别是当换主题时！！！]:

### 1,2 hexo generate（hexo g）

生成静态文件

```bash
$ hexo g
```

| 选项 | 描述                   |
| ---- | ---------------------- |
| -d   | 文件生成后立即部署网站 |
| -w   | 监事文件变动           |
|      |                        |

### 1,3 hexo deploy (hexo d)

部署网站

```bash
$ hexo d
```

| 参数 | 描述                     |
| ---- | ------------------------ |
| -g   | 部署之前，先生成静态文件 |
|      |                          |

### 1.4gulp压缩代码

```bash
$ gulp build		//相当于hexo cl&&hexo g&&hexo d
```

hexo

## 补充：

其中也可以加入 hexo server （hexo s）启动本地服务器访问http://localhost:4000/.来测试静态界面是否符合预期

 	<http://xiaoming403.github.io>

# 文章书写规范

### 1，文章front-matter

- 建议至少填写title和date值

```
---
title: typora-vue-theme主题介绍
date: 2020-07-07 09:25:00
author: xiaoming403
img: /source/images/xxx.jpg
top: true
cover: true
coverImg: /images/1.jpg
password: 8d969eef6ecad3c29a3a629280e686cf0c3f5d5a86aff3ca12020c923adc6c92
toc: false
mathjax: false
summary: 这是你自定义的文章摘要内容，如果这个属性有值，文章卡片摘要就显示这段文字，否则程序会自动截取文章的部分内容作为摘要
categories: Markdown
tags:
  - Typora
  - Markdown
---
```

| 配置选项      | 默认值                     | 描述                                                         |
| ------------- | -------------------------- | ------------------------------------------------------------ |
| title         | Markdown 的文件标题        | 文章标题，强烈建议填写此选项                                 |
| date          | 文件创建时的日期时间       | 发布时间，强烈建议填写此选项，且最好保证全局唯一             |
| author        | 根 _config.yml 中的 author | 文章作者                                                     |
| img           | featureImages 中的某个值   | 文章特征图，推荐使用图床 (腾讯云、七牛云、又拍云等) 来做图片的路径。如: http://xxx.com/xxx.jpg |
| top           | true                       | 推荐文章（文章是否置顶），如果 top 值为 true，则会作为首页推荐文章 |
| cover         | false                      | v1.0.2 版本新增，表示该文章是否需要加入到首页轮播封面中      |
| password      | 无                         | 文章阅读密码，如果要对文章设置阅读验证密码的话，就可以设置 password 的值，该值必须是用 SHA256 加密后的密码，防止被他人识破。前提是在主题的 config.yml 中激活了 verifyPassword 选项 |
| mathjax       | false                      | 是否开启数学公式支持 ，本文章是否开启 mathjax，且需要在主题的 _config.yml 文件中也需要开启才行 |
| tags          | 无                         | 文章标签，一篇文章可以多个标签                               |
| summary       | 无                         | 文章摘要，自定义的文章摘要内容，如果这个属性有值，文章卡片摘要就显示这段文字，否则程序会自动截取文章的部分内容作为摘要 |
| categories    | 无                         | 文章分类，本主题的分类表示宏观上大的分类，只建议一篇文章一个分类 |
| reprintPolicy | cc_by                      | 文章转载规则， 可以是 cc_by, cc_by_nd, cc_by_sa, cc_by_nc, cc_by_nc_nd, cc_by_nc_sa, cc0, noreprint 或 pay 中的一个 |

- 注：如果要对文章设置阅读验证密码的功能，不仅要在 Front-matter 中设置采用了 SHA256 加密的 password 的值，还需要在主题的 _config.yml 中激活了配置。有些在线的 SHA256 加密的地址，可供你使用：开源中国在线工具、chahuo、站长工具。

[^以上front-matter内容：学习参考 <https://yafine-blog.cn/posts/4ab2.html>]: 

### 2,图片添加方式

- 需要在目录新建一个与.md相同名称的文件夹存放img
- 调用格式如下：

```
{% asset_img 图片名称和后缀 图片脚注 %}
```

- 范例：{% asset_img pic示例.png pic示例 %}