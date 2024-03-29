<!-- front-matter
id: watch
title: watch()
hide_title: true
sidebar_label: watch()
-->

# watch()

Allows watching globs and running a task when a change occurs. Tasks are handled uniformly with the rest of the task system.

## Usage

```js
const { watch } = require('glup');

watch(['input/*.js', '!input/something.js'], function(cb) {
  // body omitted
  cb();
});
```

## Signature

```js
watch(globs, [options], [task])
```

### Parameters

| parameter | type | note |
|:--------------:|:-----:|--------|
| globs<br>**(required)** | string<br>array | [Globs][globs-concepts] to watch on the file system. |
| options | object | Detailed in [Options][options-section] below. |
| task | function<br>string | A [task function][tasks-concepts] or composed task - generated by `series()` and `parallel()`. |

### Returns

An instance of [chokidar][chokidar-instance-section] for fine-grained control over your watch setup.

### Errors

When a non-string or array with any non-strings is passed as `globs`, throws an error with the message, "Non-string provided as watch path".

When a string or array is passed as `task`, throws an error with the message, "watch task has to be a function (optionally generated by using glup.parallel or glup.series)".

### Options

| name | type | default | note |
|:-------:|:------:|-----------|--------|
| ignoreInitial | boolean | true | If false, the task is called during instantiation as file paths are discovered. Use to trigger the task during startup.<br>**Note:** This option is passed to [chokidar][chokidar-external] but is defaulted to `true` instead of `false`. |
| delay | number | 200 | The millisecond delay between a file change and task execution. Allows for waiting on many changes before executing a task, e.g. find-and-replace on many files. |
| queue | boolean | true | When true and the task is already running, any file changes will queue a single task execution. Keeps long running tasks from overlapping. |
| events | string<br>array | [ 'add',<br>'change',<br>'unlink' ] |  The events being watched to trigger task execution. Can be `'add'`, `'addDir'`, `'change'`, `'unlink'`, `'unlinkDir'`, `'ready'`, and/or `'error'`. Additionally `'all'` is available, which represents all events other than `'ready'` and `'error'`.<br>_This option is passed directly to [chokidar][chokidar-external]._ |
| persistent | boolean | true | If false, the watcher will not keep the Node process running. Disabling this option is not recommended.<br>_This option is passed directly to [chokidar][chokidar-external]._ |
| ignored | array<br>string<br>RegExp<br>function |  | Defines globs to be ignored. If a function is provided, it will be called twice per path - once with just the path, then with the path and the `fs.Stats` object of that file.<br>_This option is passed directly to [chokidar][chokidar-external]._ |
| followSymlinks | boolean | true | When true, changes to both symbolic links and the linked files trigger events. If false, only changes to the symbolic links trigger events.<br>_This option is passed directly to [chokidar][chokidar-external]._ |
| cwd | string |  | The directory that will be combined with any relative path to form an absolute path. Is ignored for absolute paths. Use to avoid combining `globs` with `path.join()`.<br>_This option is passed directly to [chokidar][chokidar-external]._ |
| disableGlobbing | boolean | false | If true, all `globs` are treated as literal path names, even if they have special characters.<br>_This option is passed directly to [chokidar][chokidar-external]._ |
| usePolling | boolean | false | When false, the watcher will use `fs.watch()` (or [fsevents][fsevents-external] on Mac) for watching. If true, use `fs.watchFile()` polling instead - needed for successfully watching files over a network or other non-standard situations. Overrides the `useFsEvents` default.<br>_This option is passed directly to [chokidar][chokidar-external]._ |
| interval | number | 100 | Combine with `usePolling: true`. Interval of file system polling.<br>_This option is passed directly to [chokidar][chokidar-external]._ |
| binaryInterval | number | 300 | Combine with `usePolling: true`. Interval of file system polling for binary files.<br>_This option is passed directly to [chokidar][chokidar-external]._ |
| useFsEvents | boolean | true | When true, uses fsevents for watching if available. If explicitly set to true, supersedes the `usePolling` option. If set to false, automatically sets `usePolling` to true.<br>_This option is passed directly to [chokidar][chokidar-external]._ |
| alwaysStat | boolean | false | If true, always calls `fs.stat()` on changed files - will slow down file watcher. The `fs.Stat` object is only available if you are using the chokidar instance directly.<br>_This option is passed directly to [chokidar][chokidar-external]._ |
| depth | number |  | Indicates how many nested levels of directories will be watched.<br>_This option is passed directly to [chokidar][chokidar-external]._ |
| awaitWriteFinish | boolean | false | Do not use this option, use `delay` instead.<br>_This option is passed directly to [chokidar][chokidar-external]._ |
| ignorePermissionErrors | boolean | false | Set to true to watch files that don't have read permissions. Then, if watching fails due to EPERM or EACCES errors, they will be skipped silently.<br>_This option is passed directly to [chokidar][chokidar-external]._ |
| atomic | number | 100 | Only active if `useFsEvents` and `usePolling` are false. Automatically filters out artifacts that occur from "atomic writes" by some editors. If a file is re-added within the specified milliseconds of being deleted, a change event - instead of unlink then add - will be emitted.<br>_This option is passed directly to [chokidar][chokidar-external]._ |

## Chokidar instance

The `watch()` method returns the underlying instance of [chokidar][chokidar-external], providing fine-grained control over your watch setup. Most commonly used to register individual event handlers that provide the `path` or `stats` of the changed files.

**When using the chokidar instance directly, you will not have access to the task system integrations, including async completion, queueing, and delay.**

```js
const { watch } = require('glup');

const watcher = watch(['input/*.js']);

watcher.on('change', function(path, stats) {
  console.log(`File ${path} was changed`);
});

watcher.on('add', function(path, stats) {
  console.log(`File ${path} was added`);
});

watcher.on('unlink', function(path, stats) {
  console.log(`File ${path} was removed`);
});

watcher.close();
```


`watcher.on(eventName, eventHandler)`

Registers `eventHandler` functions to be called when the specified event occurs.

| parameter | type | note |
|:--------------:|:-----:|--------|
| eventName | string | The events that may be watched are `'add'`, `'addDir'`, `'change'`, `'unlink'`, `'unlinkDir'`, `'ready'`, `'error'`, or `'all'`. |
| eventHandler | function | Function to be called when the specified event occurs. Arguments detailed in the table below. |

| argument | type | note |
|:-------------:|:-----:|--------|
| path | string | The path of the file that changed. If the `cwd` option was set, the path will be made relative by removing the `cwd`. |
| stats | object | An [fs.Stat][fs-stats-concepts] object, but could be `undefined`. If the `alwaysStat` option was set to `true`, `stats` will always be provided. |

`watcher.close()`

Shuts down the file watcher. Once shut down, no more events will be emitted.

`watcher.add(globs)`

Adds additional globs to an already-running watcher instance.

| parameter | type | note |
|:-------------:|:-----:|--------|
| globs | string<br>array | The additional globs to be watched. |

`watcher.unwatch(globs)`

Removes globs that are being watched, while the watcher continues with the remaining paths.

| parameter | type | note |
|:-------------:|:-----:|--------|
| globs | string<br>array | The globs to be removed. |

[chokidar-instance-section]: #chokidar-instance
[options-section]: #options
[tasks-concepts]: ../api/concepts.md#tasks
[globs-concepts]: ../api/concepts.md#globs
[fs-stats-concepts]: ../api/concepts.md#file-system-stats
[chokidar-external]: https://github.com/paulmillr/chokidar
[fsevents-external]: https://github.com/strongloop/fsevents
