# @prantlf/gulp-jsonlint
[![NPM version](https://badge.fury.io/js/%40prantlf%2Fgulp-jsonlint.svg)](https://badge.fury.io/js/%40prantlf%2Fgulp-jsonlint)
[![Build Status](https://travis-ci.com/prantlf/gulp-jsonlint.svg?branch=master)](https://travis-ci.com/prantlf/gulp-jsonlint)
[![Coverage Status](https://coveralls.io/repos/github/prantlf/gulp-jsonlint/badge.svg?branch=master)](https://coveralls.io/github/prantlf/gulp-jsonlint?branch=master)
[![Dependency Status](https://david-dm.org/prantlf/gulp-jsonlint.svg)](https://david-dm.org/prantlf/gulp-jsonlint)
[![devDependency Status](https://david-dm.org/prantlf/gulp-jsonlint/dev-status.svg)](https://david-dm.org/prantlf/gulp-jsonlint#info=devDependencies)

> [JSON]/[JSON5] file syntax validation plugin for [`gulp`] using [`jsonlint`]

This is a fork of the original package with the following enhancements:

* Validates with both [JSON] and [JSON5] standards.
* Supports [JSON Schema] drafts 04, 06 and 07.
* Optionally recognizes JavaScript-style comments and single quoted strings.
* Optionally ignores trailing commas and reports duplicate object keys as an error.
* Prefers using the 8x faster native JSON parser, if possible.
* Optionally reformats the output JSON and sorts object keys alphabetically.
* Depends on up-to-date npm modules with no installation warnings.

## Usage

First, install `@prantlf/gulp-jsonlint` as a development dependency:

```shell
npm i -D @prantlf/gulp-jsonlint
```

Then, add it to your `gulpfile.js`:

```javascript
var jsonlint = require("gulp-jsonlint");

gulp.src("./src/*.json")
    .pipe(jsonlint())
    .pipe(jsonlint.reporter());
```

Using a custom reporter:

```javascript
var jsonlint = require('gulp-jsonlint');
var log = require('fancy-log');

var myCustomReporter = function (file) {
    log('File ' + file.path + ' is not valid JSON.');
};

gulp.src('./src/*.json')
    .pipe(jsonlint())
    .pipe(jsonlint.reporter(myCustomReporter));
```

## API

### jsonlint(options)

Options can be passed as keys in an object to the `jsonlint` function. The following are their defaults:

    jsonlint({
        // parsing
        mode: 'json',
        ignoreComments: false,
        ignoreTrailingCommas: false,
        allowSingleQuotedStrings: false,
        allowDuplicateObjectKeys: true,
        // formatting
        format: false,
        indent: 2,
        sortKeys: false
    })

* `mode`, when set to "cjson" or "json5", enables some other flags automatically
* `ignoreComments`, when `true` JavaScript-style single-line and multiple-line comments will be recognised and ignored
* `ignoreTrailingCommas`, when `true` trailing commas in objects and arrays will be ignored
* `allowSingleQuotedStrings`, when `true` single quotes will be accepted as alternative delimiters for strings
* `allowDuplicateObjectKeys`, when `false` duplicate keys in objects will be reported as an error

* `format`, when `true` `JSON.stringify` will be used to format the JavaScript (if it is valid)
* `indent`, the value passed to `JSON.stringify`, it can be the number of spaces, or string like "\t"
* `sortKeys`, when `true` keys of objects in the output JSON will be sorted alphabetically (`format` has to be set to `true` too)

#### Schema Validation

You can validate JSON files using JSON Schema drafts 04, 06 or 07, if you specify the schema in addition to other options:

    jsonlint({
      schema: {
        src: 'some/manifest-schema.json',
        environment: 'json-schema-draft-04'
      }
    })

* `schema`, when set the source file will be validated using ae JSON Schema in addition to the syntax checks
* `src`, when filled with a file path, the file will be used as a source of the JSON Schema
* `environment`, can specify the version of the JSON Schema draft to use for validation: "json-schema-draft-04", "json-schema-draft-06" or "json-schema-draft-07" (if not set, the schema draft version will be inferred automatically)

### jsonlint.reporter(customReporter)

#### customReporter(file)

Type: `function`

You can pass a custom reporter function. If ommited then the default reporter will be used.

The `customReporter` function will be called with the argument `file`.

##### file

Type: `object`

This argument has the attribute `jsonlint` wich is an object that contains a `success` boolean attribute. If it's false you also have a `message` attribute containing the jsonlint error message.

### jsonlint.failOnError()

Stop a task/stream if an jsonlint error has been reported for any file.

```javascript
// Cause the stream to stop(/fail) before copying an invalid JS file to the output directory
gulp.src('**/*.js')
	.pipe(jsonlint())
	.pipe(jsonlint.failOnError())
	.pipe(gulp.dest('../output'));
```

### jsonlint.failAfterError()

Stop a task/stream if an jsonlint error has been reported for any file, but wait for all of them to be processed first.

## License

Copyright (C) 2013-2019 Rogério Vicente, Ferdinand Prantl

Licensed under the [MIT License].

[MIT License]: http://en.wikipedia.org/wiki/MIT_License
[`gulp`]: http://gulpjs.com/
[`jsonlint`]: https://prantlf.github.io/jsonlint/
[JSON]: https://tools.ietf.org/html/rfc8259
[JSON5]: https://spec.json5.org
[JSON Schema]: https://json-schema.org
