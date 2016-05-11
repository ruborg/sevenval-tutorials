# Web Accelerator

Here you'll learn how you can use FIT to increase both the real and the perceived performance of your website. At this point we assume that you already have [integrated your website, or our test site, with FIT as outlined before](https://developer.sevenval.com/start/tutorials/integration.html).

If you haven't intergrated your own website, you can follow through on our test website, which is shown below, on wide (desktop), and narrow (mobile) viewports:

IMAGES

## Image Optimization

Images usually account for the major part of the page size. Image optimization therefore can often yield significant payload savings which drastically improves the performance: Fewer bytes have to be transferred and the rendering is faster. Moreover, sending large, unoptimized images to mobile devices can even impact battery life too.

In FIT, there are two sides to image optimization: image scaling, and image compression. Since FIT version 4.4.4 it is possible to configure image scaling and compression separately.

### Image scaling

The `image-scaling` option in your `conf/config.xml` enables scaling of images according to the client's display dimensions.

To provide browsers with optimal content, you should also enable the [JavaScript client feature detection](https://developer.sevenval.com/docs/current/ress/DC_detectionPage.html) (`detection-page`):

```xml
<config>
  <ress>
    <image-scaling viewport-fitting="current" />
    <detection-page title="FIT14 Detection Page"/>
  </ress>
</config>
```

Using Chrome dev tools, we can immediately see the effect this has on our web page at `http://local14.sevenval-fit.com`. Note in the original page, the background image size is a 1274x1274 pixel JPEG. When we use a smaller viewport, FIT has scaled image down to 600x600 pixels, and it now uses the more efficient WebP format. The image size has been reduced from 164KB to just 7.2KB!

IMAGES

A full list of image scaling parameters can be found [on the image scaling documentation page](https://developer.sevenval.com/docs/current/ress/Image_Scaling.html).

### Image compression



