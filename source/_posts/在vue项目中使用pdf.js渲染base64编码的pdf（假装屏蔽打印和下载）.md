---
title: 在vue项目中使用pdf.js渲染base64编码的pdf（假装屏蔽打印和下载）
date: 2018-10-09
categories:
- [VUE]
tags:
- [pdf.js]
---

  先说一下业务场景吧，项目要求查看的pdf只能在网页上查看，不能下载或者打印，开始直接使用pdf的格式预览，然后网上找的一些前端代码屏蔽下载和打印等功能（找度娘，很多），但并不能阻止有心的用户找到源文件。所以考虑使用base64的方式，避免用户简单的找到源文件。
首先，下载[pdf.js](https://github.com/mozilla/pdf.js/releases),下载下来之后，直接解压放到vue-cli项目的static目录下，目录中会有一个viewer.html，也就是官方提供的集成好各种功能的demo。如图
![在这里插入图片描述](https://img-blog.csdn.net/20181009175826161?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzM2OTUyMQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
其中，viewer.css为该页面的样式，viwer.js为各种事件绑定等，而真正起作用的是build文件夹下的pdf.js和pdf-worder.js。如果只是做简单的pdf查看，只需要在viewer.js中找到DEFAULT_URL这个变量，大概在10060行左右。将路径修改为你存放pdf的路径。建议将pdf.worder.js的引入方式修改为绝对路径如图：
![在这里插入图片描述](https://img-blog.csdn.net/20181009180657653?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzM2OTUyMQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
这样做的好处是可以将你预览pdf的html放在任意地方也可以正确的引入必要的文件，否则只能将预览页面放在pdfjs目录下面，原因是pdfjs打包出来的文件有自己的一套内部模块加载机制，当你讲预览页面放到pdf后访问的build目录与vue项目里面的build目录冲突。
#### 重点
预览base64编码的pdf，只需要将你的pdf转码成base64,然后再通过下面这段代码解码成浏览器能识别的数组模式即可。

```javascript
<script src="/static/pdf/baseData/pdf-Paper_cn.js"></script>
<script>
    var BASE64_MARKER = ';base64,';
    function convertDataURIToBinary(dataURI) {
      var base64Index = dataURI.indexOf(BASE64_MARKER) + BASE64_MARKER.length;
      var base64 = dataURI.substring(base64Index).replace(/[\r\n]/g, '');
      var raw = window.atob(base64);
      var rawLength = raw.length;
      var array = new Uint8Array(new ArrayBuffer(rawLength));
    
      for(var i = 0; i < rawLength; i++) {
        array[i] = raw.charCodeAt(i);
      }
      return array;
    }
    var pdfAsDataUri = BASE_DATA;
    var DEFAULT_URL = convertDataURIToBinary(pdfAsDataUri);
</script>

```
我的做法是在viewer.js中将DEFAULT_URL注掉后在viewer.html中重新定义，然后将转码后的base64存放在一个单独的pdf0Paper_cn.js文件中,通过暴露出来的变量名为BASE_DATA。最后在vue项目中直接使用 域名+/static/pdfjs/web/viewer.html访问即可。
#### 最后
这样并不能做到绝对的防止用户下载和打印，只是稍微增加了难度。目前还没有找到绝对可以防止的方法，有过相关经验的大佬欢迎交流。