# PurifyCSS Extended

[![Travis](https://img.shields.io/travis/HapLifeMan/purifycss-extended/master.svg)]()
[![Downloads](https://img.shields.io/npm/dt/purifycss-extended.svg)](https://www.npmjs.com/package/purifycss-extended)
[![Realease](https://img.shields.io/npm/v/purifycss-extended.svg)](https://github.com/HapLifeMan/purifycss-extended/releases)
[![License](https://img.shields.io/npm/l/tailwindcss.svg)](https://github.com/HapLifeMan/purifycss-extended/blob/master/LICENSE)

**This is a fork from the original [purifycss/purifycss](https://github.com/purifycss/purifycss).
Since it's not maintained for months and pull requests not merged, I decided to create a new NPM package called `purifycss-extended` based on it with some fixes.**

**Everything runs as the original `purifycss` package, commands haven't changed.**

A function that takes content (HTML/JS/PHP/etc) and CSS, and returns only the **used CSS**.  
PurifyCSS Extended does not modify the original CSS files. You can write to a new file, like minification.  
If your application is using a CSS framework, this is especially useful as many selectors are often unused.

### Potential reduction

* [Bootstrap](https://github.com/twbs/bootstrap) file: ~140k
* App using ~40% of selectors.
* Minified: ~117k
* Purified + Minified: **~35k**


## Usage

### Standalone

Installation  

```bash
npm i -D purifycss-extended
```

```javascript
import purifycss from "purifycss-extended"
const purifycss = require("purifycss-extended")

let content = ""
let css = ""
let options = {
    output: "filepath/output.css"
}
purify(content, css, options)
```

### Build Time

- [Grunt](https://github.com/purifycss/grunt-purifycss) (old purifycss, open an issue if you are interested in)
- [Gulp](https://github.com/purifycss/gulp-purifycss) (old purifycss, open an issue if you are interested in)
- [Webpack](https://github.com/HapLifeMan/purifycss-extended-webpack) **extended special**

### CLI Usage

```
$ npm install -g purifycss-extended
```

```
$ purifycss -h

purifycss <css> <content> [option]

Options:
  -m, --min        Minify CSS                         [boolean] [default: false]
  -o, --out        Filepath to write purified css to                    [string]
  -i, --info       Logs info on how much css was removed
                                                      [boolean] [default: false]
  -r, --rejected   Logs the CSS rules that were removed
                                                      [boolean] [default: false]
  -w, --whitelist  List of classes that should not be removed
                                                           [array] [default: []]
  -h, --help       Show help                                           [boolean]
  -v, --version    Show version number                                 [boolean]
```


## How it works

### Used selector detection

Statically analyzes your code to pick up which selectors are used.  
But will it catch all of the cases?  

#### Let's start off simple.
#### Detecting the use of: `button-active`

``` html
  <!-- html -->
  <!-- class directly on element -->
  <div class="button-active">click</div>
```

``` javascript
  // javascript
  // Anytime your class name is together in your files, it will find it.
  $(button).addClass('button-active');
```

#### Now let's get crazy.
#### Detecting the use of: `button-active`

``` javascript
  // Can detect if class is split.
  var half = 'button-';
  $(button).addClass(half + 'active');

  // Can detect if class is joined.
  var dynamicClass = ['button', 'active'].join('-');
  $(button).addClass(dynamicClass);

  // Can detect various more ways, including all Javascript frameworks.
  // A React example.
  var classes = classNames({
    'button-active': this.state.buttonActive
  });

  return (
    <button className={classes}>Submit</button>;
  );
```

### Examples


##### Example with source strings

```js
var content = '<button class="button-active"> Login </button>';
var css = '.button-active { color: green; }   .unused-class { display: block; }';

console.log(purify(content, css));
```

logs out:

```
.button-active { color: green; }
```


##### Example with [glob](https://github.com/isaacs/node-glob) file patterns + writing to a file

```js
var content = ['**/src/js/*.js', '**/src/html/*.html'];
var css = ['**/src/css/*.css'];

var options = {
  // Will write purified CSS to this file.
  output: './dist/purified.css'
};

purify(content, css, options);
```


##### Example with both [glob](https://github.com/isaacs/node-glob) file patterns and source strings + minify + logging rejected selectors

```js
var content = ['**/src/js/*.js', '**/src/html/*.html'];
var css = '.button-active { color: green; } .unused-class { display: block; }';

var options = {
  output: './dist/purified.css',

  // Will minify CSS code in addition to purify.
  minify: true,

  // Logs out removed selectors.
  rejected: true
};

purify(content, css, options);
```
logs out:

```
.unused-class
```


##### Example with callback

```js
var content = ['**/src/js/*.js', '**/src/html/*.html'];
var css = ['**/src/css/*.css'];

purify(content, css, function (purifiedResult) {
  console.log(purifiedResult);
});
```


##### Example with callback + options

```js
var content = ['**/src/js/*.js', '**/src/html/*.html'];
var css = ['**/src/css/*.css'];

var options = {
  minify: true
};

purify(content, css, options, function (purifiedAndMinifiedResult) {
  console.log(purifiedAndMinifiedResult);
});
```

### API in depth

```javascript
// Four possible arguments.
purify(content, css, options, callback);
```

#####  The `content` argument
##### Type: `Array` or `String`

**`Array`** of [glob](https://github.com/isaacs/node-glob) file patterns to the files to search through for used classes (HTML, JS, PHP, ERB, Templates, anything that uses CSS selectors).

**`String`** of content to look at for used classes.

<br />

##### The `css` argument
##### Type: `Array` or `String`

**`Array`** of [glob](https://github.com/isaacs/node-glob) file patterns to the CSS files you want to filter.

**`String`** of CSS to purify.

<br />

##### The (optional) `options` argument
##### Type: `Object`

##### Properties of options object:

* **`minify:`** Set to `true` to minify. Default: `false`.

* **`output:`** Filepath to write purified CSS to. Returns raw string if `false`. Default: `false`.

* **`info:`** Logs info on how much CSS was removed if `true`. Default: `false`.

* **`rejected:`** Logs the CSS rules that were removed if `true`. Default: `false`.

* **`whitelist`** Array of selectors to always leave in. Ex. `['button-active', '*modal*']` this will leave any selector that includes `modal` in it and selectors that match `button-active`. (wrapping the string with *'s, leaves all selectors that include it)



##### The (optional) ```callback``` argument
##### Type: `Function`

A function that will receive the purified CSS as it's argument.

##### Example of callback use
``` javascript
purify(content, css, options, function(purifiedCSS){
  console.log(purifiedCSS, ' is the result of purify');
});
```

##### Example of callback without options
``` javascript
purify(content, css, function(purifiedCSS){
  console.log('callback without options and received', purifiedCSS);
});
```

##### Example CLI Usage

```
$ purifycss src/css/main.css src/css/bootstrap.css src/js/main.js --min --info --out src/dist/index.css
```
This will concat both `main.css` and `bootstrap.css` and purify it by looking at what CSS selectors were used inside of `main.js`. It will then write the result to `dist/index.css`

The `--min` flag minifies the result.

The `--info` flag will print this to stdout:
```
    ________________________________________________
    |
    |   PurifyCSS has reduced the file size by ~ 33.8%
    |
    ________________________________________________

```
The CLI currently does not support file patterns.
