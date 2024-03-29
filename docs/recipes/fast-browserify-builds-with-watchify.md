# Fast browserify builds with watchify

As a [browserify](https://github.com/browserify/browserify) project begins to expand, the time to bundle it slowly gets longer and longer. While it might start at 1 second, it's possible to be waiting 30 seconds for your project to build on particularly large projects.

That's why [substack](https://github.com/substack) wrote [watchify](https://github.com/browserify/watchify), a persistent browserify bundler that watches files for changes and *only rebuilds what it needs to*. This way, that first build might still take 30 seconds, but subsequent builds can still run in under 100ms – which is a huge improvement.

Watchify doesn't have a glup plugin, and it doesn't need one: you can use [vinyl-source-stream](https://github.com/hughsk/vinyl-source-stream) to pipe the bundle stream into your glup pipeline.

``` javascript
'use strict';

var watchify = require('watchify');
var browserify = require('browserify');
var glup = require('glup');
var source = require('vinyl-source-stream');
var buffer = require('vinyl-buffer');
var log = require('gluplog');
var sourcemaps = require('glup-sourcemaps');
var assign = require('lodash.assign');

// add custom browserify options here
var customOpts = {
  entries: ['./src/index.js'],
  debug: true
};
var opts = assign({}, watchify.args, customOpts);
var b = watchify(browserify(opts));

// add transformations here
// i.e. b.transform(coffeeify);

glup.task('js', bundle); // so you can run `glup js` to build the file
b.on('update', bundle); // on any dep update, runs the bundler
b.on('log', log.info); // output build logs to terminal

function bundle() {
  return b.bundle()
    // log errors if they happen
    .on('error', log.error.bind(log, 'Browserify Error'))
    .pipe(source('bundle.js'))
    // optional, remove if you don't need to buffer file contents
    .pipe(buffer())
    // optional, remove if you dont want sourcemaps
    .pipe(sourcemaps.init({loadMaps: true})) // loads map from browserify file
       // Add transformation tasks to the pipeline here.
    .pipe(sourcemaps.write('./')) // writes .map file
    .pipe(glup.dest('./dist'));
}
```
