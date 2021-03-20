---
layout: single
title:  "这个博客是怎么建成的"
date:   2020-08-15 11:22:26 +0800
toc: true
toc_sticky: true
---



## 文档

官方文档

- https://jekyllrb.com/docs/installation/
- https://jekyllcn.com/docs/



GitHub Pages

- https://docs.github.com/en/github/working-with-github-pages/getting-started-with-github-pages



主题

- http://jekyllthemes.org/
- https://github.com/topics/jekyll-theme
- https://github.com/mmistakes/minimal-mistakes
- https://hydejack.com/showcase/

minimal-mistakes 样本示例

- https://mmistakes.github.io/minimal-mistakes/docs/layouts/



https://hexo.io/

## 博客主题 minimal-mistakes

https://mmistakes.github.io/minimal-mistakes/docs/quick-start-guide/

下载好 minimal-mistakes 之后，进入根目录

- 按这个操作 https://github.com/mmistakes/minimal-mistakes



```bash
[root@localhost minimal-mistakes-4.22.0]# bundle
Fetching gem metadata from https://rubygems.org/..........
Resolving dependencies.....
Fetching rake 13.0.3
Installing rake 13.0.3
...

bundle exec jekyll serve --host 0.0.0.0 --port 80


jekyll serve --host 0.0.0.0 --port 80 --incremental
```

--incremental

### 配置相关选项

https://mmistakes.github.io/minimal-mistakes/docs/configuration/

- _data文件夹里存的是可以配置的导航栏
- _site目录中存放生成的 html 文件，会缓存的，值修改配置，貌似不会生效，直接删掉？

- 右侧边栏 ```toc: true```

## 备忘录

安装 Ruby gem jekyll  等基础环境支持

实现在本地预览：

```bash
//安装jekyll
gem install  bundler


bundle

jekyll new testblog

cd testblog

jekyll serve --host 0.0.0.0 --port 80

```

在浏览器输入 http://127.0.0.1:80/  即可浏览刚刚创建的 blog



## 问题记录及解决

- 怎么添加目录
- 怎么加载图片, 如何 让 md 与 博客兼容，没法同时显示？经过一番爬坑，终于好了。
- 怎么添加右侧文章边栏，文件开头加 `toc: true`。
- 怎么添加左侧大目录，data 文件下配置。
- 怎么调整字体，自己修改样式文件？
-  _post 中文件名：必须是 `YEAR-MONTH-DAY-title.md` 格式吗？ 这个有点麻烦。
- 调整边栏宽度，自己修改 css，但是会产生兼容问题，尤其是是 Safari，一些场景，容易样式变形。
- 绑定第三方域名。

