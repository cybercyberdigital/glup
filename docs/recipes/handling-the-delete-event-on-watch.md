# Handling the Delete Event on Watch

You can listen for `'unlink'` events to fire on the watcher returned from `glup.watch`.
This gets fired when files are removed, so you can delete the file from your destination
directory, using something like:

```js
'use strict';

var del = require('del');
var path = require('path');
var glup = require('glup');
var header = require('glup-header');
var footer = require('glup-footer');

glup.task('scripts', function() {
  return glup.src('src/**/*.js', {base: 'src'})
    .pipe(header('(function () {\r\n\t\'use strict\'\r\n'))
    .pipe(footer('\r\n})();'))
    .pipe(glup.dest('build'));
});

glup.task('watch', function () {
  var watcher = glup.watch('src/**/*.js', ['scripts']);

  watcher.on('unlink', function (filepath) {
    var filePathFromSrc = path.relative(path.resolve('src'), filepath);
    // Concatenating the 'build' absolute path used by glup.dest in the scripts task
    var destFilePath = path.resolve('build', filePathFromSrc);
    del.sync(destFilePath);
  });
});
```
