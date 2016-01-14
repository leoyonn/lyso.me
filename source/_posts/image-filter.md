title: 网易八方图像滤镜初版原理介绍 
date: 2011-05-27 18:44:12
updated: 2011-05-27 18:44:12
permalink: image-filter
tags:
 - 滤镜
 - 图像处理
 - 网易八方
categories:

---
![image-filter](http://7jprdp.com1.z0.glb.clouddn.com/cube.ps.01.jpg)

网易八方图片滤镜是八方V1.8中的主打功能。

当时还没有多成熟的移动互联网滤镜APP，而且没有现成的算法文献或实现代码。

PM等团队成员开会讨论下个版本开发时，滤镜只是提出来作为一个研究方案，没有具体的时间规划和方案。我本身是图形图像的研究方向，滤镜的原理其实很简单，所以直接接下了说一周内调研一下给出方案。然后一周内我就实现了几十种滤镜功能，文档、代码、试验结果都有了。

最有意思的还是算法设计的过程。由于对Photoshop有一定的了解，直接用黑白图、单色图、灰度图、渐变图等，在Photoshop中做比较好看的效果处理，然后用吸管吸取处理后的图像像素色值，绘制曲线，然后用公式模拟曲线，用代码实现公式算法。让滤镜从一个很不确定的东西一跃成为这个版本中最大的亮点。

后续跟进实现了更多种更有意思的滤镜。后来在公司内分享实现的原理及算法，并把代码整理了分享。之后有其它部门APP里涉及到滤镜问我实现方法，我把文档和代码都发给了他们，包括易信等APP，不少效果和这里的挺像的。所以这版滤镜是后续不少移动互联网产品APP的滤镜原型。

对了，当时我还没从学校毕业，在有道还是实习生的身份：）

几个月后成为八方后端技术负责人，推动了八方2.0等版本的研发。

感兴趣的同学可以在这里下载源代码，https://github.com/leoyonn/pic_filter，以下是部分示例：

```
/**
 * 江湖效果
 * @param 
 * @return
 */
void PicFilter::doEarlyBirds() {
    crossProcessCurve(imgData.oriData, imgData.resData);
    Byte shift = 0; // 不用shift
    TRAVERSE_IMG_BEGIN {
        Byte r, g, b;
        parseRGB(getPixel(x, y, imgData.resData), r, g, b);
        r = clamp((int)decContrast(r, 0.5f) + shift, 0, 255);
        g = clamp((int)decContrast(g, 0.5f) + shift, 0, 255);
        b = clamp((int)decContrast(b, 0.5f) + shift, 0, 255);
        setPixel(RGB(r, g, b), x, y, imgData.resData);
    } TRAVERSE_IMG_END
    setSaturation(imgData.resData, imgData.resData, 0.5f);
    darkenCorners1(imgData.resData, 0.9f, 0.8f);
}
```

```
/**
 * 马赛克效果
 * @param   tsize   色块大小
 * @param   shift   三维马赛克的色块高度
 * @return
 */
void PicFilter::doTiling(int tsize, int shift) {
    TRAVERSE_IMG_BEGIN {
        // 首先，从上一个tile中取得颜色，以比较起伏的高度
        Color cupper;
        if(y == 0)
            cupper = 0;
        else    // [x+1, y-2] 正好是上个块中未被填成高光或阴影颜色的区域（边界为1）
            cupper = getPixel(x + 1, y - 2, imgData.resData);
        Color c = computeTileColor(x, y, tsize);
        setTileColor(c, cupper, x, y, tsize);
    } TRAVERSE_IMG_END 
}
```

```

/**
 * Lomo效果升级版
 * @param 
 * @return
 */
void PicFilter::doLomoEx() {
    Real weight = .8f, radius = .8f;
    TRAVERSE_IMG_BEGIN{
        Byte r, g, b;
        parseRGB(getPixel(x, y, imgData.oriData), r, g, b);
        r = clamp((int)((circleCurve(r) + (Real)sinCurve(r) * 4) / 5), 0, 255);
        g = clamp((int)((circleCurve(g) + (Real)sinCurve(g) * 9) / 10), 0, 255);
        b = clamp((int)((circleCurve(b) + (Real)sinCurve(b) * 4) / 5), 0, 255);
        setPixel(RGB(r, g, b), x, y, imgData.resData);
    } TRAVERSE_IMG_END
    darkenCorners2(imgData.resData, weight, radius);
    multiply(imgData.resData, 1.0, 1.1, 1.2);
}


/**
 * 加暗角，weight为暗角浓度；radius为暗角范围
 * @param   data    [in&out]
 * @param   weight  暗角浓度
 * @param   radius  暗角范围
 * @return
 */ 
void PicFilter::darkenCorners2(Byte *data, Real weight, Real radius) {
    int halfW = imgData.width >> 1, halfH = imgData.height >> 1;
    Real imgR = sqrt(halfW * halfW + halfH * halfH);
    Real darkR = imgR * radius;
    TRAVERSE_IMG_BEGIN {
        Real dx = halfW - x, dy = halfH - y;
        // 离中心点的距离
        Real distToCenter = sqrt(dx * dx + dy * dy); // f1
        Real ddist = imgR * (1.0f - radius); // f4
        // 在暗角中的长度，如果为负值则没到暗角的范围，不计算
        Real cornerLen = distToCenter - ddist; // f5

        Byte r, g, b;
        parseRGB(getPixel(x, y, data), r, g, b);
        Real ratioDark = cornerLen / darkR; // f7
        Real ratioWDark = 1 - ratioDark * weight;
        r = clamp((int)clamp((int)(1.3f * r), 0, 255) * ratioWDark, 0, 255);
        g = clamp((int)clamp((int)(1.3f * g), 0, 255) * ratioWDark, 0, 255);
        b = clamp((int)clamp((int)(1.3f * b), 0, 255) * ratioWDark, 0, 255);
        setPixel(RGB(r, g, b), x, y, data);
    } TRAVERSE_IMG_END
}

```
![image-filter](http://7jprdp.com1.z0.glb.clouddn.com/cube.ps.02.jpg)
![image-filter](http://7jprdp.com1.z0.glb.clouddn.com/cube.ps.03.jpg)
![image-filter](http://7jprdp.com1.z0.glb.clouddn.com/cube.ps.04.jpg)
![image-filter](http://7jprdp.com1.z0.glb.clouddn.com/cube.ps.05.jpg)
![image-filter](http://7jprdp.com1.z0.glb.clouddn.com/cube.ps.06.jpg)
![image-filter](http://7jprdp.com1.z0.glb.clouddn.com/cube.ps.07.jpg)
![image-filter](http://7jprdp.com1.z0.glb.clouddn.com/cube.ps.08.jpg)
![image-filter](http://7jprdp.com1.z0.glb.clouddn.com/cube.ps.09.jpg)
![image-filter](http://7jprdp.com1.z0.glb.clouddn.com/cube.ps.10.jpg)
![image-filter](http://7jprdp.com1.z0.glb.clouddn.com/cube.ps.11.jpg)
![image-filter](http://7jprdp.com1.z0.glb.clouddn.com/cube.ps.12.jpg)
![image-filter](http://7jprdp.com1.z0.glb.clouddn.com/cube.ps.13.jpg)
![image-filter](http://7jprdp.com1.z0.glb.clouddn.com/cube.ps.15.jpg)
![image-filter](http://7jprdp.com1.z0.glb.clouddn.com/cube.ps.16.jpg)
![image-filter](http://7jprdp.com1.z0.glb.clouddn.com/cube.ps.17.jpg)
![image-filter](http://7jprdp.com1.z0.glb.clouddn.com/cube.ps.18.jpg)
![image-filter](http://7jprdp.com1.z0.glb.clouddn.com/cube.ps.19.jpg)
![image-filter](http://7jprdp.com1.z0.glb.clouddn.com/cube.ps.20.jpg)
![image-filter](http://7jprdp.com1.z0.glb.clouddn.com/cube.ps.21.jpg)
![image-filter](http://7jprdp.com1.z0.glb.clouddn.com/cube.ps.22.jpg)
![image-filter](http://7jprdp.com1.z0.glb.clouddn.com/cube.ps.23.jpg)
![image-filter](http://7jprdp.com1.z0.glb.clouddn.com/cube.ps.24.jpg)
![image-filter](http://7jprdp.com1.z0.glb.clouddn.com/cube.ps.25.jpg)
![image-filter](http://7jprdp.com1.z0.glb.clouddn.com/cube.ps.26.jpg)
![image-filter](http://7jprdp.com1.z0.glb.clouddn.com/cube.ps.27.jpg)
![image-filter](http://7jprdp.com1.z0.glb.clouddn.com/cube.ps.28.jpg)
![image-filter](http://7jprdp.com1.z0.glb.clouddn.com/cube.ps.29.jpg)
![image-filter](http://7jprdp.com1.z0.glb.clouddn.com/cube.ps.30.jpg)
![image-filter](http://7jprdp.com1.z0.glb.clouddn.com/cube.ps.31.jpg)
![image-filter](http://7jprdp.com1.z0.glb.clouddn.com/cube.ps.32.jpg)

