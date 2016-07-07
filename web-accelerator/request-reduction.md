# Request Reduction

Now that the assets have been minified, let's take a look at how FIT can reduce the number of requests. Since there is an overhead for each HTTP request, having too many requests is bad for performance. FIT can reduce the number of requests by inlining small external resources such as [images](https://developer.sevenval.com/docs/current/web-accelerator/Image_Inlining.html), [scripts, and stylesheets](https://developer.sevenval.com/docs/current/web-accelerator/JsCssInlining.html).

> The optimizations described here are, at this time, mostly HTTP 1.1 transport optimizations. Thus, image inlining, script and style inlining, and Script Manager optimizations will automatically be deactivated when the Delivery Context Property `request/http2` is set, because the measures taken can be counterproductive in HTTP 2. To force inlining of resources with `ai-inline="true"` set even if `request/http2` is true, use the `force-explicit="true"` attribute for the respective inlining option. Please note that this is not recommended!

## Image Inlining
Image files can be automatically inlined into the main HTML document by FIT if they are smaller than a certain threshold KB size. FIT can inline up to 50KB of images per page. However, inlining may also be forced or prevented on a per element basis directly in your HTML documents.

Image inlining will only be applied to images in the FIT cache. Images are added to the FIT cache when image scaling or compression are applied, or when they have been explicitly cached. This means that you will need to enable `image scaling` or `image-compression` to see the effects of image inlining.

Only images that are less than 2KB in size (after FIT scaling and/or compression) will be inlined.

Image inlining is enabled by adding the `<image-inlining />` element to your `config.xml`.

```xml
<config>
  <acceleration>
    <image-inlining [ force-explicit="true" ] />
  </acceleration>
</config>
```

If you open up your developer tools, and compare the original example site with the optimized version, you will see that the smaller images have been converted into data URIs and are now inlined. This has saved us five network requests!

![FIT image inlining - before](https://raw.githubusercontent.com/ruborg/sevenval-tutorials/master/web-accelerator/images/image-inlining-before.png "FIT image inlining - before") ![FIT image inlining - after](https://raw.githubusercontent.com/ruborg/sevenval-tutorials/master/web-accelerator/images/image-inlining-after.png "FIT image inlining - after")


To force or prevent inlining on a per image basis, use the `ai-inline` attribute on an individual `img` element:

```html
<img ai-inline="true" src="big-photo.jpg" width="2000" height="2000">
<img ai-inline="false" src="tiny-icon.png" width="10" height="10">
```

Setting `ai-inline="true"` will force inlining, and `ai-inline="false"` will disable inlining on an individual image, regardless of its size.

See the [Image Inlining documentation](https://developer.sevenval.com/docs/current/web-accelerator/Image_Inlining.html) for further details.


## JavaScript Inlining

Small JavaScript resources can be inlined by FIT by adding `<script-inlining />` to your `config.xml` file:

```xml
<config>
  <acceleration>
    <script-inlining [ force-explicit="true" ] />
  </acceleration>
</config>
```

As with image inlining, script inlining can be configured separately for each included resource in the HTML document, using the `ai-inline="true"` attribute.

To illustrate JavaScript inlining, the HTML source of our example site has a small script referenced in the HTML head:

```html
<script src="assets/js/dummy.js" type="text/javascript"></script>
```

Now, with `<script-inlining />` added to your config, if you load the optimized site and view its source, you should see that this small script has now been inlined into the HTML document, saving a network request:

```html
<script type="text/javascript">/* I am a small dummy JS script. I don't do very much */
var dummy = 'hello';
console.log(dummy);
</script>
```

The default behavior is to inline resources if they are smaller than 2KB. However, scripts with `defer` or `aysnc` attributes will not be inlined by default.

In your HTML code you can force or prevent inlining of script or style files. This overrides the automatic inlining by size. To force or prevent inlining of specific files, use the `ai-inline` attribute. Possible values are: 

* `true`: always force inlining
* `false`: prevent inlining
* `auto`: automatically decide inlining based on the file size (this is the default option)

> Note that if a script or style content contains `</`, it will never be inlined, because it could lead to parsing problems in the client. 


## CSS Inlining

The approach is similar for CSS: to enable style inlining, add `<style-inlining />` to your `config.xml`:  

```xml
<config>
  <acceleration>
    <style-inlining [ force-explicit="true" ] />
  </acceleration>
</config>
```
 
In the example site, there is a small CSS file `small.css` linked in the `<head>`:

```html
<link rel="stylesheet" href="assets/css/small.css" />
```

If you view the source of the optimized site, you should see that this file has now been inlined:

```html
<style>/* small.css */

html, body, h1, h2, h3, h4, h5, h6, p, ol, ul, li, dl,
dt, dd, blockquote, address{
    margin: 0;
    padding: 0;
}</style>
```

As with JavaScript resources, inlining can be applied or disabled for individual CSS resources using the `ai-inline` attribute, with the same values as before.

See the [JavaScript and CSS Inlining documentation](https://developer.sevenval.com/docs/current/web-accelerator/JsCssInlining.html) for more details and examples.

## Script Manager
The FIT Script Manager can dynamically aggregate and load JavaScript resources into a single bundled request, and store the scripts in the browser local storage (if available). On subsequent page requests the scripts are loaded from local storage so that no further requests are needed. 

The Script Manager is enabled in the `config.xml` file with the `<script-manager />` element:

```xml
<config>
  <acceleration>
    <script-manager />
  </acceleration>
</config>
```

If local storage is not supported, or if it is disabled, then FIT will still aggregate the scripts so that subsequent loads will require just a single request. 

As with Image Inlining, only scripts that are in the FIT cache will be loaded via the Script Manager. Scripts with `async` or `defer` attributes *won't* be loaded by the Script Manager. Individual scripts can be excluded from the Script Manager in the HTML document by adding the `ai-use-script-manager="false"` attribute to the appropriate `script` element.

In our example site, we can see the Script Manager in action. After adding `<script-manager />` to the config file, you should be able to see that some new scripts have been inserted into the HTML document, as shown in the dev tools inspector image below:

![FIT Script Manager using local storage to load scripts](https://raw.githubusercontent.com/ruborg/sevenval-tutorials/master/web-accelerator/images/script-manager-dev-tools.png "FIT Script Manager using local storage to load scripts")

In the image, we can see the following scripts:

* A script with `id` `AI_SCRIPT__loadJS_request`: This script loads any other scripts processed by the Script Manager.
* A script with `id` `AI_SCRIPT__loadJS_0`: This script, and scripts with similar `id`s, will insert loaded scripts at the correct place in the document.

If local storage is available, you can now also take a look at its contents in the developer tools:

![Script Manager local storage data](https://raw.githubusercontent.com/ruborg/sevenval-tutorials/master/web-accelerator/images/script-manager-localstorage.png "FIT Script Manager local storage data")

And if you examine this data, you will see it contains the contents of our scripts!

```javascript
{"version":"14.4.0","fit://filebroker/assets/js/ap/snippet":{"tag":"6b638cac","content":"window.ai=window.ai||{};ai.getViewportDim=function(){var e=window,t=document,n=-1,i=-1;if(\"BackCompat\"!==document.compatMode&&void 0!==t.documentElement&&void 0!==t.documentElement.clientWidth&&0!==t.documentElement.clientWidth){n=t.documentElement.clientWidth;i=t.documentElement.clientHeight}else if(void 0!==window.innerWidth){n=e.innerWidth;i=e.innerHeight}else{n=t.getElementsByTagName(\"body\")[0].clientWidth;i=t.getElementsByTagName(\"body\")[0].clientHeight}return{width:n,height:i}};function AI_ap_allowMirror(){return!1}\nfunction AI_ap_useValuesDetectedForLandscape(){return!0}"},"http://example.developer.sevenval.com/assets/js/dummy.js":{"content":"var dummy='hello';console.log(dummy);","expires":1466516751000,"tag":"e2971a19","oldTags":{}}}
```

See the [Script Manager documentation](https://developer.sevenval.com/docs/current/web-accelerator/ScriptManager.html) for more details on how the Script Manager loads scripts.

## Request reduction summary
Now that we've seen how FIT can reduce the number of requests for a web page, let's measure how well it's done.

At the beginning of this section, after applying image and content optimizations, our page started out with XX requests, and was YYY KB in size.

|            | Requests | Page size KB    | PageSpeed Insights (mobile) | PageSpeed Insights (desktop)
|     ---    | ---:      |     ---:| ---:                         | ---:
|**Before**  |    26    |  242      | 61                         |  84 
|**After**   |    26    |  242       | 61                          |  84  

The new configuration parameters we used in our `config.xml` file are listed below:

```xml
<config>
  <acceleration>
    ...
    <image-inlining />
    <script-inlining />
    <style-inlining />
    <script-manager />
  </acceleration>
</config>
```

