# Twelvety

*Note: This is the original Twelvety README from Greg Ives' [11ty starter](https://github.com/gregives/Twelvety).*
Twelvety is a pre-configured Eleventy starter project built to be fast. It includes:

- Component architecture
- CSS pipeline using Sass, PostCSS and CleanCSS
- JS pipeline using Browserify, Babel and Uglify
- Page-specific CSS and JS
- Inline critical CSS and defer non-critical CSS
- Minified HTML, CSS and JS
- Responsive picture shortcode **with AVIF and WebP support**
- Content-hash of assets

Write components like this:

```html
<main class="home">
  <h1 class="home__title">Twelvety</h1>
</main>

{% stylesheet 'scss' %}
  @import "mixins";

  .home {
    @include container;

    &__title {
      color: red;
    }
  }
{% endstylesheet %}

{% javascript %}
  console.log("Super fast 💨");
{% endjavascript %}
```
## Run Locally

Click the <kbd>Use this template</kbd> button at the top of this repository to make your own Twelvety repository in your GitHub account. Clone or download your new Twelvety repository onto your computer.

You'll need [Node.js](https://nodejs.org) and npm (included with Node.js). To install the required packages, run

```sh
npm install
```

### Commands

- Run `npm run serve` to run a development server and live-reload
- Run `npm run build` to build for production
- Run `npm run clean` to clean the output folder and Twelvety cache

The brains of Twelvety live in the `utils` folder: if you just want to make a website, then you don't need to touch anything inside `utils`. However, if you want to change any of the shortcodes, have a look around!

## Features

Twelvety sets up transforms, shortcodes and some sensible Eleventy options. Click the features below to learn how they work.

<details>
<summary><strong><code>stylesheet</code> paired shortcode</strong></summary>
<br>

Use the `stylesheet` paired shortcode to include your Sass. You can import Sass files from your `styles` directory (defined in `.twelvety.js`) and from `node_modules`. The Sass will be rendered using [dart-sass](https://github.com/sass/dart-sass#javascript-api), passed into [PostCSS](https://github.com/postcss/postcss) (with [PostCSS Preset Env](https://github.com/csstools/postcss-preset-env) and [Autoprefixer](https://github.com/postcss/autoprefixer) for compatibility) and either minified using [clean-css](https://github.com/jakubpawlowicz/clean-css) or beautified by [JS Beautifier](https://github.com/beautify-web/js-beautify) (in production and development respectively).

```html
{% stylesheet 'scss' %}
  @import "normalize.css/normalize";
  @import "mixins";

  .home {
    @include container;

    color: $color--red;
  }
{% endstylesheet %}
```

The second parameter of the `stylesheet` paired shortcode is the language; currently, this does nothing and is included solely to align with Shopify's definition of the shortcode. If you want to use Sass **indented syntax**, you can change the `indentedSass` Twelvety option, found in `.twelvety.js`.

The `stylesheet` paired shortcode also has a third parameter, which by default is set to `page.url`, the URL of the current page being rendered. This means that only the required CSS is included in each page. You can make your own 'chunk' of CSS using this parameter, for example, a CSS file common to all pages of your website.

___

</details>

<details>
<summary><strong><code>styles</code> shortcode</strong></summary>
<br>

The `styles` shortcode collects together all Sass written in `stylesheet` paired shortcodes for the given chunk and outputs the rendered CSS. The 'chunk' defaults to `page.url`, the URL of the current page being rendered.

```html
<!-- Inline all styles on current page -->
<style>
  {% styles page.url %}
</style>

<!-- Capture styles on current page -->
{% capture css %}
  {% styles page.url %}
{% endcapture %}
<!-- And output asset using `asset` shortcode -->
<link rel="stylesheet" href="{% asset css, 'css' %}" />
```

Note that the `styles` shortcode must be placed below any `stylesheet` paired shortcodes in the template; see the `append` paired shortcode and transform for more information.

___

</details>

<details>
<summary><strong><code>javascript</code> paired shortcode</strong></summary>
<br>

Include your JavaScript using the `javascript` paired shortcode. Twelvety uses [Browserify](http://browserify.org) so that you can `require('modules')` and [Babel](https://babeljs.io) so you can use the latest JavaScript. Your JavaScript will then be minified using [Uglify](https://github.com/mishoo/UglifyJS) in production or beautified by [JS Beautifier](https://github.com/beautify-web/js-beautify) in development.

```html
{% javascript %}
  const axios = require("axios");

  axios.get("/api/endpoint")
    .then((response) => {
      console.log("Yay, it worked!");
    })
    .catch((error) => {
      console.log("Uh oh, something went wrong");
    });
{% endjavascript %}
```

The `javascript` paired shortcode has a second parameter, which by default is set to `page.url`, the URL of the current page being rendered. This means that only the required JavaScript is included in each page. You can make your own 'chunk' of JavaScript using this parameter, for example, a JavaScript file for all vendor code.

The output of each `javascript` paired shortcode will be wrapped in an [IIFE](https://developer.mozilla.org/en-US/docs/Glossary/IIFE) so that your variables do not pollute global scope. If you want to define something on `window`, use `window.something =`.

___

</details>

<details>
<summary><strong><code>script</code> shortcode</strong></summary>
<br>

The `script` shortcode collects together all the JavaScript for the given chunk and outputs the JavaScript (after transpilation and minification). The 'chunk' defaults to `page.url`, the URL of the current page being rendered.

```html
<!-- Inline all JavaScript on current page -->
<script>
  {% script page.url %}
</script>

<!-- Capture JavaScript on current page -->
{% capture js -%}
  {% script page.url %}
{%- endcapture -%}
<!-- And output asset using `asset` shortcode -->
<script src="{% asset js, 'js' %}" defer></script>
```

Note that the `script` shortcode must be placed below any `javascript` paired shortcodes in the template; usually this is not a problem as JavaScript is often included immediately preceding `</body>`. If you want the JavaScript somewhere else, see the `append` paired shortcode and transform.

___

</details>

<details>
<summary><strong><code>asset</code> shortcode</strong></summary>
<br>

The `asset` shortcode outputs a content-hashed asset with the given content and extension. The content may be either a `String` or `Buffer`. Assets will be saved to the `assets` directory inside the `output` directory (both defined within `.twelvety.js`).

```html
<!-- Capture some content -->
{% capture css %}
h1 {
  color: red;
}
{% endcapture %}

<!-- Save content to content-hashed file with .css extension -->
<link rel="stylesheet" href="{% asset css, 'css' %}" />

<!-- Output of shortcode -->
<link rel="stylesheet" href="/_assets/58f4b924.css" />
```

You can import the `asset` shortcode function in JavaScript: this is how the `picture` shortcode saves your responsive images into the `assets` directory.

___

</details>

<details>
<summary><strong><code>picture</code> shortcode</strong></summary>
<br>

The `picture` shortcode takes `src` and `alt` parameters and outputs a responsive picture element with AVIF* and WebP support. Your images must be stored within the `images` directory, defined within `.twelvety.js`. Twelvety will save the outputted images to the `assets` directory inside the `output` directory (both defined within `.twelvety.js`). The `picture` shortcode also takes two other parameters: `sizes` which defaults to `90vw, (min-width: 1280px) 1152px`, based upon the breakpoint sizes; and `loading` which defaults to `lazy`, can also be `eager`.

*AVIF is disabled by default due to long build times. You can enable it in `.twelvety.js`.

```html
<!-- Picture shortcode with src, alt, sizes and loading -->
{% picture 'car.jpg', 'Panning photo of grey coupe on road', '90vw', 'eager' %}

<!-- Absolute paths also work -->
{% picture '/src/_assets/images/car.jpg', 'Panning photo of grey coupe on road', '90vw', 'eager' %}

<!-- Output of shortcode -->
<picture style="background-color:rgb(38%,28%,26%);padding-bottom:50%">
  <source srcset="/_assets/2263c1d0.avif 320w,/_assets/519fcdec.avif 640w,/_assets/b59349f7.avif 960w,/_assets/e8dae22f.avif 1280w,/_assets/4ba755ff.avif 1600w,/_assets/87c06dd1.avif 1920w" sizes="90vw" type="image/avif">
  <source srcset="/_assets/0e7cdd2f.webp 320w,/_assets/ba4e43dd.webp 640w,/_assets/bc541ea5.webp 960w,/_assets/6d620165.webp 1280w,/_assets/756857ea.webp 1600w,/_assets/483e9c95.webp 1920w" sizes="90vw" type="image/webp">
  <source srcset="/_assets/6a3b0321.jpeg 320w,/_assets/2bf90b83.jpeg 640w,/_assets/4a810813.jpeg 960w,/_assets/601b629c.jpeg 1280w,/_assets/c39ac58c.jpeg 1600w,/_assets/25a2b530.jpeg 1920w" sizes="90vw" type="image/jpeg">
  <img src="/_assets/25a2b530.jpeg" alt="Panning photo of grey coupe on road" width="2400" height="1200" loading="lazy">
</picture>
```

The `picture` shortcode uses native lazy-loading but it would be easy to add support for `lazysizes` or a similar library if you wished. The `picture` shortcode calculates the average colour of the image to show while the image loads, using `padding-bottom` to avoid layout shift.

The `picture` shortcode is automatically used for every image in Markdown. To disable this, you'll need to edit the instance of markdown-it (see Markdown feature).

```md
<!-- Automatically uses picture shortcode -->
![Panning photo of grey coupe on road](car.jpg)
```

**The images outputted by the `picture` shortcode are cached.** If you want to clear the cache, delete `.twelvety.cache` (just a JSON file) or run `npm run clean` to delete the cache and the output directory. If you delete the output directory but `.twelvety.cache`, things will break.

___

</details>

<details>
<summary><strong><code>append</code> paired shortcode and transform</strong></summary>
<br>

Okay folks, here it is: the one _gotcha_ with Twelvety. In order for the `styles` shortcode to work, it must come after all `stylesheet` paired shortcodes, which would usually be in the `body`. However, we want our CSS to be linked or inlined in the `head`. This is where the `append` paired shortcode and transform come in, to move the output of the `styles` shortcode back into the `head` where we want it.

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <!-- Everything in append paired shortcode will be moved here -->
  </head>
  <body>
    <!-- Stylesheet paired shortcodes can go here -->
    ...
    <!-- Append paired shortcode with styles inside -->
    {% append 'head' %}
      <style>
        {% styles page.url %}
      </style>
    {% endappend %}
  </body>
</html>
```

The `append` paired shortcode will actually be replaced with a `template`. The `append` transform then uses [jsdom](https://github.com/jsdom/jsdom) to append the contents of the `template` to the given selector (in this case, `head`).

The same problem exists for the `script` shortcode, however, this is not such a problem because it's very common to include JavaScript from the bottom of `body` anyway.

___

</details>

<details>
<summary><strong><code>markdown</code> paired shortcode and configuration</strong></summary>
<br>

Twelvety sets its own instance of markdown-it. The configuration options are:

```js
{
  html: true,
  breaks: true,
  typographer: true
}
```

Twelvety also modifies the `image` rule of the renderer: instead of outputting an `img` element, Twelvety uses the responsive `picture` shortcode to render each image. If you want to disable this, remove the following lines in `utils/markdown.js`.

```js
md.renderer.rules.image = function (tokens, index) {
  const token = tokens[index];
  const src = token.attrs[token.attrIndex("src")][1];
  const alt = token.content;
  return pictureShortcode(src, alt);
};
```

Twelvety also adds a `markdown` paired shortcode which uses the markdown-it configuration.

```html
{% markdown %}
# `markdown` paired shortcode

Lets you use **Markdown** like _this_.
{% endmarkdown %}
```

This is also really useful for including Markdown files into a template.

```html
{% markdown %}
  {%- include 'content.md' -%}
{% endmarkdown %}
```

Be careful of the [common pitfall of indented code blocks](https://www.11ty.dev/docs/languages/markdown/#there-are-extra-and-in-my-output) when using the `markdown` paired shortcode. If indented code blocks are becoming a nuisance, you can disable them in `utils/markdown.js` whilst retaining fenced code blocks.

```diff
   // Uncomment the following line to disable indented code blocks
-  // .disable("code")
+  .disable("code")
```

___

</details>

<details>
<summary><strong><code>critical</code> transform</strong></summary>
<br>

Instead of using a transform, Twelvety now uses [eleventy-critical-css](https://github.com/gregives/eleventy-critical-css) to extract and inline critical-path CSS on every page.

___

</details>

<details>
<summary><strong><code>format</code> transform</strong></summary>
<br>

The `format` transform beautifies HTML in development using [JS Beautifier](https://github.com/beautify-web/js-beautify) and minifies HTML in production using [HTMLMinifier](https://github.com/kangax/html-minifier). Any inline CSS and JavaScript will also be beautified or minified.

___

</details>
