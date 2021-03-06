# html loader for webpack

Exports HTML as string. HTML is minimized when the compiler demands.

By default every local `<img src="image.png">` is required (`require("./image.png")`). You may need to specify loaders for images in your configuration (recommended `file-loader` or `url-loader`).

Also every `<img srcset="..."`> is converted to `require` statements. For example
``` html
<img src="image.jpg" srcset="image.jpg 1x, image@2x.jpg 2x">
```
is converted to
``` javascript
"<img src=\"" + require("./image.jpg") + "\" srcset=\"" + require("./image.jpg") + " 1x," + require("./image@2x.jpg") + " 2x \">"
```
You can specify which tag-attribute combination should be processed by this loader via the query parameter `attrs`. Pass an array or a space-separated list of `<tag>:<attribute>` combinations. (Default: `attrs=[img:src, img:srcset]`)

To completely disable tag-attribute processing (for instance, if you're handling image loading on the client side) you can pass in `attrs=false`.

## Usage

[Documentation: Using loaders](http://webpack.github.io/docs/using-loaders.html)

## Examples

With this configuration:

``` javascript
{
	module: { loaders: [
		{ test: /\.jpg$/, loader: "file-loader" },
		{ test: /\.png$/, loader: "url-loader?mimetype=image/png" }
	]},
	output: {
		publicPath: "http://cdn.example.com/[hash]/"
	}
}
```

``` html
<!-- fileA.html -->
<img  src="image.jpg"  data-src="image2x.png" >
```

``` javascript
require("html!./fileA.html");
// => '<img  src="http://cdn.example.com/49e...ba9f/a9f...92ca.jpg"  data-src="image2x.png" >'

require("html?attrs=img:data-src!./file.html");
// => '<img  src="image.png"  data-src="data:image/png;base64,..." >'

require("html?attrs=img:src img:data-src!./file.html");
require("html?attrs[]=img:src&attrs[]=img:data-src!./file.html");
// => '<img  src="http://cdn.example.com/49e...ba9f/a9f...92ca.jpg"  data-src="data:image/png;base64,..." >'

require("html?-attrs!./file.html");
// => '<img  src="image.jpg"  data-src="image2x.png" >'

/// minimized by running `webpack --optimize-minimize`
// => '<img src=http://cdn.example.com/49e...ba9f/a9f...92ca.jpg data-src=data:image/png;base64,...>'

```

## 'Root-relative' urls

For urls that start with a `/`, the default behavior is to not translate them.
If a `root` query parameter is set, however, it will be prepended to the url
and then translated.

With the same configuration above:
``` html
<!-- fileB.html -->
<img src="/image.jpg">
```

``` javascript

require("html!./fileB.html");
// => '<img  src="/image.jpg">'

require("html?root=.!./fileB.html");
// => '<img  src="http://cdn.example.com/49e...ba9f/a9f...92ca.jpg">'

```

## Interpolation

You can use `interpolate` flag to enable interpolation syntax for ES6 template strings, like so:

```
require("html?interpolate!./file.html");
```

```
<img src="${require(`./images/gallery.png`)}" />
<div>${require('./partials/gallery.html')}</div>
```

## Advanced options

If you need to pass [more advanced options](https://github.com/webpack/html-loader/pull/46), especially those which cannot be stringified, you can also define an `htmlLoader`-property on your `webpack.config.js`:

``` javascript
module.exports = {
  ...
  module: {
    loaders: [
      {
        test: /\.html$/,
        loader: "html"
      }
    ]
  }
  htmlLoader: {
  	ignoreCustomFragments: [/\{\{.*?}}/]
  }
};
```

If you need to define two different loader configs, you can also change the config's property name via `html?config=otherHtmlLoaderConfig`:

```javascript
module.exports = {
  ...
  module: {
    loaders: [
      {
        test: /\.html$/,
        loader: "html?config=otherHtmlLoaderConfig"
      }
    ]
  }
  otherHtmlLoaderConfig: {
    ...
  }
};
```

## License

MIT (http://www.opensource.org/licenses/mit-license.php)
