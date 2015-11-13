# gulp-inject-stringified-html

> Gulp Plugin to inject HTML templates into Javascript files.

## Usage

First, install `gulp-inject-stringified-html` as a development dependency:

```
npm install --save-dev gulp-inject-stringified-html
```

Then, add it to your gulpfile.js:

### Why?

When building single page web applications (SPAs), the View/ViewController/Controller/Presenter/ViewModel (the star in MV* which we'll refer to as ViewController) needs to know the "template" being rendered and bound to.  For example, when using [AngularJS Directives](https://docs.angularjs.org/guide/directive), either a `template` or `templateUrl` property can be given so the directive knows what html to render when compiled.  This has two unfortunate consequences:

1. `template` forces the developer to manually escape the html string value
2. `templateUrl` forces the developer to have to know the URL path that the template will be located at when deployed to a server or local environment

In a perfect world, ViewController logic should be completely orthogonal to how resources are retreived from URLs.  Using `template` directive attribute is tedious for the developer and makes the html code unreadable.  Using `templateUrl` forces a irregular dependency between the ViewController and an URL for its required template causing further complications.

`gulp-inject-stringified-html` solves these problems. 

### How?

Basically, by putting `{ gulp_inject: './path/to/your/file.html' }` inside your javascript file and adding `gulp-inject-stringified-html` to your gulp tasks that build your javascript source code, the html template referenced in the javascript code will be stringified (escaped for embedding html in a javascript string) and injected inline. Below is the basic usage of how to use `gulp-inject-stringified-html` in a project using gulp as a build process and its expected results.

```javascript
// gulpfile.js

var injectHtml = require('gulp-inject-stringified-html');

gulp.task('js', function () {
  return gulp.src(['src/views/hello.js'])
    .pipe(injectHtml())
    .pipe(gulp.dest('public/hello.js'));
});
```

```html
// src/views/hello.html

<p>Hello, World!</p>
```

```javascript
// src/views/hello.js

function sayHello() {
  return {
    gulp_inject: './hello.html'
  };
}
```

Result:
```javascript
// public/hello.js

function sayHello() {
  return "<h2>Hello, World!</h2>"
}
```

### JS Gulp Task Example

Below is a more robust example of how `gulp-inject-stringified-html` plugin 
fits into more complex Javascript build gulp tasks.

```javascript
// gulpfile.js

var path = require('path');
var gulp = require('gulp');
var newer = require('gulp-newer');
var concat = require('gulp-concat');
var uglify = require('gulp-uglify');
var sourcemaps = require('gulp-sourcemaps');
var injectHtml = require('gulp-inject-stringified-html');

gulp.task('js', function () {
  var dest = './public';
  var filename = 'app.js'
  
  return gulp.src(['src/**/*.js')
    .pipe(newer(path.join(dest, filename)))
    .pipe(sourcemaps.init())
      .pipe(injectHtml())  // <--- gulp-inject-stringified-html here.
      .pipe(concat(filename))
      .pipe(uglify())
    .pipe(sourcemaps.write())
    .pipe(gulp.dest(dest));
});
```

### Customize!  

You provide custom `pre` and `post` escape tags to change what the plugin looks for. 

```javascript
// gulpfile.js

var injectHtml = require('gulp-inject-stringified-html');

gulp.task('js', function () {
  return gulp.src(['src/views/hello.js'])
    .pipe(injectHtml({
      pre: '!!![[[',
      post: ']]]'
    }))
    .pipe(gulp.dest('public/hello.js'));
```

```javascript
// src/views/hello.js

function sayHello() {
  return !!![[[ 'hello.html' ]]];
}
```

### File Paths

##### Relative Paths

Often Single-Page-Application (SPA) projects put templates and javascript views 
components in the same directory structure - and thus `gulp-inject-stringified-html` 
allows "Relative Paths".  The examples above both use relative paths to inject 
the corresponding template.

```
// Common Directory Structure

project/
   gulpfile.js
   src/
      views/
         hello/
            hello.js   <-- JS View
            hello.html <-- HTML Template
         login/
            login.js
            login.html
         goodbye/
            goodbye.js 
      models/...
      styles/...
      vendor/...
   public/...
```

Let's just say we want the `goodbye.js` view to utilize the `hello.html` template.
We can actually put in a `..` within the path to get into the `views/` directory.  
See the `goodbye.js` example code below.

```javascript
// src/views/goodbye.js

function sayGoodbye() {
  return { gulp_inject: '../views/hello/hello.html' };
  //  or  './../views/hello/hello.html' --^
}
```

##### Absolute Paths

But sometimes projects have one directory holding all the templates - and thus 
`gulp-inject-stringified-html` uses absolute path syntax which is the path 
starting at where ever the `gulpfile.js` file exists (at `project/` directory).

```
// Alternative Common Directory Structure

project/
   gulpfile.js
   src/
      js/
         views/
            hello.js
            login.js
            goodbye.js
         models/...
         vendor/...
      templates/
         hello.html
         login.html
      styles/...
```

Below is an example using absolute path syntax.

```javascript
// src/js/views/hello.js

function sayHello() {
  return { gulp_inject: '/src/templates/hello.html' };
}
```

---

## License

The MIT License (MIT)

Copyright (c) 2015 Derek Severson

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.