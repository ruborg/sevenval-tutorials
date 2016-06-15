# Request Reduction

Now that the assets have been minified, let's take a look at how FIT can reduce the number of requests. Since there is a performance overhead for each HTTP request, having too many requests is bad for web performance. FIT can help reduce the number of requests by inling small external resources such as [images](https://developer.sevenval.com/docs/current/web-accelerator/Image_Inlining.html), [scripts, and stylesheets](https://developer.sevenval.com/docs/current/web-accelerator/JsCssInlining.html).

> Note that these optimizations only really apply to HTTP/1.x. With HTTP/2, network connections are multiplexed and therefore request reduction does not apply. For HTTP/2, FIT will automatically disable request reduction, unless you override. See below for more details.

## Image Inlining
Image files can be automatically inlined into the main HTML document by FIT if they are smaller than a certain threshold KB size. FIT will automatically inline up to 50KB of images per page. However, inling may be forced or prevented on a per element basis directly in your HTML documents.

Image inlining is enabled by adding the `<image-inlining />` element to your `config.xml`.

```xml
<config>
  <acceleration>
    <image-inlining [ force-explicit="true" ] />
  </acceleration>
</config>
```

In our example site we can see that the smaller images have been converted into data URIs and are now inlined. This has saved us five network requests.

IMAGE - DEV TOOLS BEFORE /AFTER

To force or prevent inlining on a per image basis, use the `ai-inline` attribute on an individual `img` element:

```html
    <img ai-inline="true" src="big-photo.jpg" width="2000" height="2000">
    <img ai-inline="false" src="tiny-icon.png" width="10" height="10">
```

`ai-inline="true"` will force inlining, and `ai-inline="false"` will disable inlining on an individual image, regardless of its size.

See the [Image Inlining documentation](https://developer.sevenval.com/docs/current/web-accelerator/Image_Inlining.html) for further details.


## JavaScript and CSS inlining

Small JavaScript an CSS resources can also be inlined by FIT. The approach is similar for both JavaScript and CSS, and can be configured separately. For JavaScript add `<script-inlining />`, and for CSS add `<style-inlining />` to your `config.xml` file. So to enabled for both, your `config.xml` file would look like:

```xml
<config>
  <acceleration>
    <script-inlining [ force-explicit="true" ] />
    <style-inlining [ force-explicit="true" ] />
  </acceleration>
</config>
```

As with image inlining, script and style inlining can be configured separately for each included resource in the HTML document, using the `ai-inline="true"` attribute.


EXAMPLE

See the [JavaScript and CSS Inlining documentation](https://developer.sevenval.com/docs/current/web-accelerator/JsCssInlining.html) for further details.

## Script Manager
The FIT script manager can dynamically aggregate and load JavScript resources into a single bundled request, and store the scripts in the browser local storage (if available). On subsequent page requests the scripts are loaded from local storage so that no further requests are needed. 

The script manager is enabled in the `config.xml` file with the `<script-manager />` element:

```xml
<config>
  <acceleration>
    <script-manager />
  </acceleration>
</config>
```

If local storage is not supported, or if it is disabled, then FIT will still aggregate the scripts so that subsequent loads will require just a single request.

The execution order of scripts will be maintained.

Minification and text-filtering will be applied by the script manager, even if they are not explicitly activated in the config file.

A tag is generated for each script. The tag contains a hash of the script, and the client capabilities, so that a cached version will be updated when either the script or the client capabilties change.

Scripts with async or defer attribute won't be loaded by the script manager.

Individual scripts can be excluded from the script manager in the HTML document using the `ai-use-script-manager="false"`

EXAMPLE

See the [Script Manager documentation](https://developer.sevenval.com/docs/current/web-accelerator/ScriptManager.html) for more details on how the script manager loads scripts.

## Note on HTTP1 vs HTTP2 multiplexing
The optimization by inlining of small resources is mostly beneficial only for HTTP/1.x connections. HTTP/2 introduces the concept of connection multiplexing, which means that the same connection can be used for mutiple resources. This obviates the need to implement the request reduction optimizations described here; in fact, these measures can be counter productive for HTTP2, so FIT will automatically deactivate Style and script inlining for the HTTP/2 protocol, that is, when the Delivery Context Property request/http2 is set.

However, you can still force inlining of resources with `ai-inline="true"` even in an HTTP/2 context by adding the `force-explicit="true"` attribute to the respective element.