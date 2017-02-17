---
title: 在cli快速生成二维码
date: 2017-02-14 10:35:57
tags:
---

``` bash
brew install qrencode
```
如其名，qrencode 是一个生成二维码的工具。

看一下help:

``` bash
-t {PNG,EPS,SVG,ANSI,ANSI256,ASCII,ASCIIi,UTF8,ANSIUTF8}
           specify the type of the generated image. (default=PNG)
-o FILENAME  write image to FILENAME. If '-' is specified, the result
           will be output to standard output. If -S is given, structured
           symbols are written to FILENAME-01.png, FILENAME-02.png, ...
           (suffix is removed from FILENAME, if specified)
```
试一下：

``` bash
qrencode -t ANSI256 https://www.google.com
```

{% asset_img qr1.png %}

效果拔群。

简化一下

``` bash
alias qr='qrencode -o - -t ANSI256'
```

{% asset_img qr2.png %}


