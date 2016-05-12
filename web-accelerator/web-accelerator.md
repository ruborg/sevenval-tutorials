# Web Accelerator

Here you'll learn how you can use FIT to increase both the real and the perceived performance of your website. At this point we assume that you already have [integrated your website, or our test site, with FIT as outlined before](https://developer.sevenval.com/start/tutorials/integration.html).

If you haven't integrated your own website, you can follow through on our test website, which is shown below, on wide (desktop), and narrow (mobile) viewports:

<img src="https://raw.githubusercontent.com/ruborg/sevenval-tutorials/master/web-accelerator/images/fit-demo-site.jpg" width="100%" style="max-width:1000px">

## Image Optimization

Images are genereally the largest contributor to page weight. Image optimization therefore can often yield significant payload savings which drastically improves performance: Fewer bytes have to be transferred and the rendering is faster. Moreover, sending large, unoptimized images to mobile devices can impact battery life too.

In FIT, there are two sides to image optimization: image scaling, and image compression. Since FIT version 14.3.1 it is possible to configure image scaling and compression separately.

### Image scaling

The `image-scaling` option in your `conf/config.xml` enables scaling of images according to the client's display dimensions.

To provide browsers with optimal content, you should also enable the [JavaScript client feature detection](https://developer.sevenval.com/docs/current/ress/DC_detectionPage.html) (`detection-page`); this will provide FIT with information about what features are supported by the client:

```xml
<config>
  <ress>
    <image-scaling viewport-fitting="current" />
    <detection-page title="FIT14 Detection Page"/>
  </ress>
</config>
```

Using Chrome dev tools, we can immediately see the effect this has when we reload our optimized web page (`http://local14.sevenval-fit.com`). Note that in the original page, the background image is a 1274x1274 pixel JPEG. When we use a smaller viewport, the image is scaled down to 480x480 pixels, and it now uses the more efficient WebP format. The background image size has been reduced from 605KB to just 20.1KB!

IMAGES

A full list of image scaling parameters can be found [on the image scaling documentation page](https://developer.sevenval.com/docs/current/ress/Image_Scaling.html).

### Image compression

As well as scaling images, Web Accelerator also provides separate options for image compression. Global image compression can be enabled in your application using the `image-compression` element in your `config.xml` file:

```xml
<config>
  <acceleration>
    <image-compression quality="70"/>
  </acceleration>
</config>
```

#### Quality

The optional `quality` parameter allows you to set the global compression level. The default compression value is 70. A value of -1 indicates lossless compression for PNG images, and maximum quality for JPEGs.

If we reload our optimized site with the config above, we can see there is little *discernible* change to the images, but the file size difference is significant: the background image size has been reduced from 605KB to 123KB, and using `webpagetest.org` to analyze the page, we can see the total size for all images has been reduced from 892KB to 239KB.

![Chart of total images size before](https://raw.githubusercontent.com/ruborg/sevenval-tutorials/master/web-accelerator/images/images-size-before.png "Total size for all images before optimization") ![Chart of total images size after](https://raw.githubusercontent.com/ruborg/sevenval-tutorials/master/web-accelerator/images/images-size-after.png "Total size for all images after optimization")

We can push these savings further using a lower quality value. The image below shows the image with quality 10. The file size has been reduced to 34.8KB, but compression artifacts are now visible in image when displayed at full size. File size will obviously vary with quality: a lower quality value results in lower file size, so there is a tradeoff here. The default of 70 is often a good option!

Notice too that the site will score better now in external tools such as Google's PageSpeed Insights which has noticed the optimized images.

> Note that the `quality` value specifed here will override any value specified in the `image scaling` configuration.


#### Image formats

FIT Web Accelerator supports many different image formats, and it will convert your images to the best supported format for any device or browser. The formats supported by FIT include JPEG, WebP, GIF, SVG... https://developer.sevenval.com/docs/current/ress/Image_Scaling.html#image-formats

The WebP format is [on average between 25%-34% smaller for the same JPEG image](https://developers.google.com/speed/webp/docs/webp_study#experiment_2_ssim_vs_bpp_plots_for_webp_and_jpeg). Therefore it is preferred where it is supported.

We can see this in action below, where a Chrome client receives WebP images, while Firefox receives JPEGs since WebP is not supported. Web Accelerator knows which image format is the best for which browser, and will make this decision automatically for you, so that your visitors will always recieve the optimal image format for their browser.

IMAGE FF/Chrome dev tools


#### JPEG Chroma Subsampling

Chroma subsampling is a feature of JPEG image compression based on the fact that humans are better at detecting variations in luminance (lighting) than in colour. Thus, colour information in images can be compressed without causing any detectable degradation in image quality.

Web Accelerator automatically peforms chroma subsampling on JPEG images to produce smaller image sizes. Using Imagemagick's `identify` command to analyze the background image in our example, we can see the original shows no chroma subsampling: `1x1,1x1,1x1` (`4:4:4`), while the optimized image reports `2x2,1x1,1x1` (`4:2:0`) subsampling.

```
$ identify -format "%[jpeg:sampling-factor]" bg.jpg
1x1,1x1,1x1
```

Running the same command on the optimized version of the image (now at URL path: `/;m=is;f=jpg;pass;q=70/images/bg.jpg`):

```
$ identify -format "%[jpeg:sampling-factor]" bg.jpg
2x1,1x1,1x1
```

#### PNG Quantization

Web Accelerator also reduces 24-bit truecolor PNG images to 8-bit PNG images. We can see the effect of this below, reducing the file size by XXX

IMAGE BEFORE/AFTER


#### Image Compression Parameters

Image compression can also be controlled more precisely on a per-image basis using the `ai-compress` attribute:

```xml
<img alt="compress-me" src="image.jpg" />
<img alt="no-compress-me" ai-compress="false" src="image.png" />
```

