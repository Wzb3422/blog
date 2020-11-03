---
title: 📱移动端适配
keywords: Mobile Web
tags: Responsive Web
categories:
  - CSS
date: 2020-02-06 14:37:56
---

在前端开发中会遇到各种各样的单位，或许会让你感到十分恐惧？战胜恐惧的最好办法就是面对恐惧。来，把它们的来龙去脉捋捋清楚。

<!-- MORE -->

# 法律声明

**警告**：本作品遵循 **署名-非商业性使用-禁止演绎 4.0 未本地化版本（CC BY-NC-ND 4.0）** 协议发布。你应该明白与本文有关的一切行为都应该遵循此协议。

> [这是什么？](https://creativecommons.org/licenses/by-nc-nd/4.0/)


## 写在前面

本文将从现实到抽象，跟着 web 的发展帮你理解各种单位与移动端适配方案。

## 常见概念

### 英寸 inch

英寸一般用来描述屏幕的物理大小，如：`23英寸显示器`、`6.3英寸手机`。起源于人类拇指的宽度。

英寸与厘米的换算：`1 inch = 2.54 cm`

### 像素 Pixel

像素是影像显示的基本单位。译自英文 Pixel，`pix` 是单词 `pictures` 的缩写，加上 `el` (element)。就得到了 `pixel`，故像素表示影像元素之意。像素是在屏幕影像上显示的最小单位。

你能看见下图中被放大区域的像素块。



### 分辨率 Image resolution

分辨率泛指量测或显示系统对细节的分辨能力。我们在这里只讨论，显示分辨率与影像分辨率。

#### 图像分辨率

通常图像分辨率被表示为每一个方向上的像素数量，如 `640x480`。

也可以使用每英寸像素数及图像的高宽描述，如 `72 ppi & 8x6 inch`



#### 显示分辨率

对于屏幕，我们使用像素数量来衡量。如 `1920x1080`，这代表该屏幕水平方向上和竖直方向上分别有1920个和1080个像素。



#### PPI

Pixel Per Inch - 每英寸像素数。PPI 可以用来衡量显示器或者一张图片的质量。



上图 iPhone 的对比中，我们看到了三款手机的 PPI 参数。

计算方法分为两步：

首先算得屏幕对角线上的像素数量。

![](https://wzb-img-base.oss-cn-shanghai.aliyuncs.com/img/20200511142306.png)

然后除以屏幕尺寸（对角线长度）

![](https://wzb-img-base.oss-cn-shanghai.aliyuncs.com/img/20200511142317.png)

### 设备独立像素

上面我们描述的都是物理像素，即真实存在的物理单元。那什么是设备独立像素呢？

首先我们来了解一下设备独立像素是在怎样的背景下产生的。

在智能手机发展快速的时代，手机屏幕的分辨率也在不断的升级。几年前可能我们还在用着分辨率比较低的手机，如：480x320 分辨率。但随着科技的发展，更高分辨率的手机出现了。如iPhone 4：960x640（刚好是前者手机分辨率（在方向上的）两倍）如果以物理像素和影像像素数量一一对应的显示方式的话，相同大小的图片在iPhone 4 手机上看到的大小是前者低分辨率手机的二分之一。

这样，岂不是后面出现更高分辨率的手机，页面元素会变得越来越小吗？同时，这也没达到手机更高分辨率的目的——让视觉体验更加细腻。

在 WWDC 2010 上，Steve Jobs 发布了 iPhone 4 ，它是第一款采用 Retina Display 技术的产品。由四个像素为一组，输出原来屏幕一个像素显示的大小区域的图像。这样以来，用户看到的图标与文字大小与原来分辨率的显示屏相同，但是精细度是原来的四倍。

通过下图对比可以明显地观察到这种关系。

iPhone 3 GS

![](https://wzb-img-base.oss-cn-shanghai.aliyuncs.com/img/20200511142343.png)

iPhone 4 / 4s

![](https://wzb-img-base.oss-cn-shanghai.aliyuncs.com/img/20200511142402.png)

正式它解决了这样的问题，也使得它成为了一款跨时代的手机。

所以，我们必须以一种单位告诉不同分辨率的手机，页面元素在它们屏幕上显示元素的实际大小是多少，这个单位称为设备独立像素（Device Independent Pixels），简称 `DIP` 或者`DP`。

打开 Chrome Devtools ，可以模拟各个手机型号的显示情况。比如下图显示的 iPhone X 尺寸为 `375x812`

，我们知道 iPhone X 的实际显示像素会比这个值高很多。这里的 `375x812` 指的是设备独立像素。
![](https://wzb-img-base.oss-cn-shanghai.aliyuncs.com/img/20200511142423.png)

### 设备像素比

在真正弄清设备独立像素这个概念之前，还需要了解设备像素比。

设备像素比（Device Pixel Ratio），简称 DPR，描述的是物理像素和设备独立像素的比值。

在 web 浏览器中，我们可以通过 window.devicePixelRatio 来获取设备的 DPR。

### 视口 Viewport

viewport 在计算机图形学领域中指的是可见的多边形区域。在 Web 开发中，我们一般会讨论三种视口。

分别是：

- 布局视口 layout viewport
- 视觉视口 visual viewport
- 理想视口 ideal viewport

## References

- [Inch - Wikipedia](https://en.wikipedia.org/wiki/Inch)
- [Pixel - Wikipedia](https://en.wikipedia.org/wiki/Pixel)
