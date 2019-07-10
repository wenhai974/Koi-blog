# Koi-blog
A little Koi's blog repository

### 项目框架

[**Hexo**](https://hexo.io/zh-cn/docs/)

### 主题

[**hexo-theme-next**](https://github.com/theme-next/hexo-theme-next)

#### 主题配置

```yml
# Menu configuration.
menu:
  home: /
  archives: /archives

# Favicon
favicon: /favicon.ico

# Avatar (put the image into next/source/images/)
# can be any image format supported by web browsers (JPEG,PNG,GIF,SVG,..)
avatar: /default_avatar.png

# Code highlight theme
# available: normal | night | night eighties | night blue | night bright
highlight_theme: normal

# Fancybox for image gallery
fancybox: true

# Specify the date when the site was setup
since: 2013

```



### 发布目录 ：docs

### 发布流程

1.新建文章：hexo new [layout] <title>

2.生成静态文件：hexo g

3.开启本地调试：hexo s

4.发布到网站: hexo d
