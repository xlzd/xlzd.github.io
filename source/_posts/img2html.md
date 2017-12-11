---
title: "img2html: 将图片转换成 HTML 页面"
date: 2017-04-02 22:22:14
tags: Python
---

闲得无聊，写了个程序，将一张图片转换成一个 HTML 页面。
如图，左边是原图，右边是一个 HTML 页面，根据文字颜色不同拼出了左边的图片：


原始图片             |  转换后
:-------------------------:|:-------------------------:
![](https://raw.githubusercontent.com/xlzd/img2html/master/demo/before.png)  |  ![](https://raw.githubusercontent.com/xlzd/img2html/master/demo/after.png)


（注意：右图可不是图片，而是一个 [HTML 页面][1]）

```
                                 ___      __          __                    ___
 __                            /'___`\   /\ \        /\ \__                /\_ \
/\_\     ___ ___       __     /\_\ /\ \  \ \ \___    \ \ ,_\    ___ ___    \//\ \
\/\ \  /' __` __`\   /'_ `\   \/_/// /__  \ \  _ `\   \ \ \/  /' __` __`\    \ \ \
 \ \ \ /\ \/\ \/\ \ /\ \L\ \     // /_\ \  \ \ \ \ \   \ \ \_ /\ \/\ \/\ \    \_\ \_
  \ \_\\ \_\ \_\ \_\\ \____ \   /\______/   \ \_\ \_\   \ \__\\ \_\ \_\ \_\   /\____\
   \/_/ \/_/\/_/\/_/ \/___L\ \  \/_____/     \/_/\/_/    \/__/ \/_/\/_/\/_/   \/____/
                       /\____/
                       \_/__/
```

我把这个程序叫做「img2html」，并上传到了 [PYPI](https://pypi.python.org/pypi/img2html)，所以，你可以直接这样安装：

```
pip install img2html
```

具体调用方式上，可以直接命令行调用，也可以通过代码调用，具体使用方式写在了 GitHub 的 README 上：[img2html](https://github.com/xlzd/img2html)。

代码逻辑非常简单，将图片每 N\*N 个像素合并成一个像素，并取这 N\*N 像素的平均值当做合成的像素的颜色，然后渲染为 HTML 页面中对应位置的文字颜色。代码中虽然使用了 4 个 for 语句，但是其实只是遍历了图片中每个像素一次。

[1]: http://old-blog.xlzd.me/hide/img2html/