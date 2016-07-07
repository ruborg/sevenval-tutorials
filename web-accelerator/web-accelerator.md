# Web Accelerator

Here you'll learn how you can use FIT to increase both the real and the perceived performance of your website. At this point we assume that you already have [integrated your website, or our test site, with FIT as outlined before](https://developer.sevenval.com/start/tutorials/integration.html).

If you haven't integrated your own website, you can follow through on our test website, [http://example-backend.sevenval.com](http://example-backend.sevenval.com), which is shown below, on wide (desktop), and narrow (mobile) viewports:

<img src="https://raw.githubusercontent.com/ruborg/sevenval-tutorials/master/web-accelerator/images/fit-demo-site.jpg" width="100%" style="max-width:1000px">

## Image Optimization

Images are generally the largest contributor to page weight. Image optimization therefore can often yield significant payload savings which drastically improves performance: Fewer bytes have to be transferred and the rendering is faster. Moreover, sending large, unoptimized images to mobile devices can impact battery life too.

In FIT, there are two sides to image optimization: image scaling, and image compression. Since FIT version 14.3.1 it is possible to configure image scaling and compression separately.

### Image Scaling

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

Using Chrome dev tools, we can immediately see the effect this has when we reload our optimized web page (`http://local14.sevenval-fit.com`). Note that in the original page, the background image is a 1274x1274 pixel JPEG. When we use a smaller viewport, the image is scaled down to 480x480 pixels, and (in Chrome) it now uses the more efficient WebP format. The background image size has been reduced from 605KB to just 20.1KB!

![FIT image scaling](https://raw.githubusercontent.com/ruborg/sevenval-tutorials/master/web-accelerator/images/fit-image-scaling.jpg "FIT image scaling")

A full list of image scaling parameters can be found [on the image scaling documentation page](https://developer.sevenval.com/docs/current/ress/Image_Scaling.html).

### Image Compression

As well as scaling images, Web Accelerator also provides separate options for image compression. Global image compression can be enabled in your application using the `image-compression` element in your `config.xml` file:

```xml
<config>
  <acceleration>
    <image-compression [ quality="70" ] />
  </acceleration>
</config>
```

#### Quality

The optional `quality` parameter allows you to set the global compression level. The default compression value is 70. A value of -1 indicates lossless compression for PNG images, and maximum quality for JPEGs.

> Note that the `quality` value specified here will override any value specified in the `image scaling` configuration.

If we reload our optimized site with the config above, we can see there is little *discernible* change to the images, but the file size difference is significant: the background image size has been reduced from 605KB to 123KB, and using `webpagetest.org` to analyze the page, we can see the total size for all images has been reduced from 892KB to 239KB.

![Chart of total images size before](https://raw.githubusercontent.com/ruborg/sevenval-tutorials/master/web-accelerator/images/images-size-before.png "Total size for all images before optimization") ![Chart of total images size after](https://raw.githubusercontent.com/ruborg/sevenval-tutorials/master/web-accelerator/images/images-size-after.png "Total size for all images after optimization")

We can push these savings further using a lower quality value. The image below shows the image with quality 10. The file size has been reduced to 34.8KB, but compression artifacts are now visible in image when displayed at full size. File size will obviously vary with quality: a lower quality value results in lower file size, so there is a trade-off here. The default of 70 is often a good option!


![Highly compressed image](https://raw.githubusercontent.com/ruborg/sevenval-tutorials/master/web-accelerator/images/compressed-image.jpg "Compression quality of 10 results in visible artifacts in displayed at full resolution")



Note too that the site will score better now in external tools such as Google's [PageSpeed Insights](https://developers.google.com/speed/pagespeed/insights/) which has noticed the optimized images. With this one configuration change, the score for our demo site goes from 77 to 83 (inset), and PageSpeed Insights can find little to complain about in our optimized images:

![PageSpeed Insights before and after](https://raw.githubusercontent.com/ruborg/sevenval-tutorials/master/web-accelerator/images/pagespeedinsights-before-after.jpg "PageSpeed Insights score goes from 77 to 83 with image compression")

More details are available on the [Image Compression documentation page](https://developer.sevenval.com/docs/current/web-accelerator/Image_Compression.html).

#### Image Formats

FIT Web Accelerator supports many different image formats, and it will convert your images to the best supported format for any device or browser. The formats supported by FIT include JPEG, PNG, WebP, GIF, SVG. Read more about supported formats in the [FIT image format documentation](https://developer.sevenval.com/docs/current/ress/Image_Scaling.html#image-formats).

The WebP format is [on average between 25%-34% smaller for the same JPEG image](https://developers.google.com/speed/webp/docs/webp_study#experiment_2_ssim_vs_bpp_plots_for_webp_and_jpeg). Therefore it is preferred where it is supported.

We can see this in action below, where a Chrome client receives WebP images, while Firefox receives JPEGs since WebP is not supported. Web Accelerator knows which image format is the best for which browser, and will make this decision automatically for you, so that your visitors will always receive the optimal image format for their browser.

Firefox developer tools network panel:

![Firefox dev tools network panel](https://raw.githubusercontent.com/ruborg/sevenval-tutorials/master/web-accelerator/images/firefox-dev-tools-network.jpg "Network panel of Firefox dev tools showing JPEG images")

Chrome developer tools network panel:

![Chrome dev tools network panel](https://raw.githubusercontent.com/ruborg/sevenval-tutorials/master/web-accelerator/images/chrome-dev-tools-network.jpg "Network panel of Chrome dev tools showing WebP images")


#### JPEG Chroma Subsampling

<!-- TODO: Too much detail here, just mention subsampling in passing -->

Chroma subsampling is a feature of JPEG image compression based on the fact that humans are better at detecting variations in luminance (lighting) than in color. Thus, color information in images can be compressed without causing any detectable degradation in image quality.

Web Accelerator automatically performs chroma subsampling on JPEG images to produce smaller image files. 

<!--
Using [ImageMagick](http://www.imagemagick.org/script/index.php)'s `identify` command to analyze the background image in our example, we can see the original shows no chroma subsampling: `1x1,1x1,1x1` (`4:4:4`), while the optimized image reports `2x2,1x1,1x1` (`4:2:0`) subsampling.

```
$ identify -format "%[jpeg:sampling-factor]" bg.jpg
1x1,1x1,1x1
```

Running the same command on the optimized version of the image (now at URL path: `/;m=is;f=jpg;pass;q=70/images/bg.jpg`):

```
$ identify -format "%[jpeg:sampling-factor]" bg.jpg
2x1,1x1,1x1
```
-->

#### PNG Quantization

Another optimization FIT performs is quantization of PNG images. FIT will automatically reduce 24-bit truecolor PNG images to 8-bit PNG images to reduce PNG image file size.


#### Image Compression Parameters

Image compression can also be controlled more precisely on a per-image basis using the `ai-compress` attribute:

```xml
<img alt="compress-me" src="image.jpg" />
<img alt="no-compress-me" ai-compress="false" src="image.png" />
```

### Delayed Images

One further optimization FIT can perform is *image delaying* (also known as *lazy loading*). By removing image source URLs from image elements, the `load` event is triggered earlier, and the web page becomes responsive more quickly.

This feature is enabled by adding the `<image-delaying />` element to the `config.xml` file: 

```xml
<config>
  <acceleration>
    <image-delaying prioritization="visibility" />
  </acceleration>
</config>
```

Note the optional `prioritization="visibility"` parameter. With this option, all images will be loaded when they enter the viewport. You may also specify that images should be loaded when they are *within a certain pixel distance of the viewport*. This makes it more likely that an image will already be loaded by the time it actually enters the viewport. The `visibility-offset-x` and `visibility-offset-y` attributes can be used to specify the desired pixel distances.

In the image below, we can see that only the two visible images from the *Cologne in pictures* section of the web page have been loaded (`01.jpg.webp` and `02.jpg.webp`), and the ones that are not visible have not yet been loaded (`03.jpg.webp` to `06.jpg.web`). If you scroll down the page, and keep an eye on the network requests, you should be able to see the images being fetched as they scroll into the viewport.

![Network panel of Chrome dev tools showing delayed image loading](https://raw.githubusercontent.com/ruborg/sevenval-tutorials/master/web-accelerator/images/image-delaying.jpg "Network panel of Chrome dev tools showing delayed image loading")


Prioritization can also be based on a simple ordering by providing an integer value for the `ai-priority` attribute of an image element in the HTML document e.g.

```html
  <img src="logo.png" ai-priority="1">
  <img src="image.jpg" ai-priority="3">
```

The value must be in the range 0-3, where 3 is the highest priority. So in the example above, the second image, `image.jpg`, will be loaded before the first one.

> Note: setting a `prioritization` value of `visibility` in the config file will override any numeric `ai-priority` value in the HTML document.  

See the [Delayed Images documentation](https://developer.sevenval.com/docs/current/web-accelerator/Image_Delay.html) for more detail about priorities and how the original markup is manipulated to facilitate image delaying.

## Image Optimization Summary
Now that we've seen some of the image optimizations that FIT can carry out, let's see how the performance of our web page has improved.

|            | Requests | Page size KB    | PageSpeed Insights (mobile) | PageSpeed Insights (desktop)
|     ---    | ---:      |     ---:| ---:                         | ---:
|**Before**  |    25    |  1157      | 57                         |  69 
|**After**   |    26    |  242       | 61                          |  84  


The configuration file `config.xml` that was used to achieve this is listed below:

```xml
<config>
  <ress>
    <image-scaling viewport-fitting="current" />
    <detection-page title="FIT14 Detection Page"/>
  </ress>
  <acceleration>
    <image-compression />
    <image-delaying prioritization="visibility" />
  </acceleration>
</config>
```
