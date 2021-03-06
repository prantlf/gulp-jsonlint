# @prantlf/gulp-jsonlint
[![NPM version](https://badge.fury.io/js/%40prantlf%2Fgulp-jsonlint.svg)](https://badge.fury.io/js/%40prantlf%2Fgulp-jsonlint)
[![Build Status](https://travis-ci.com/prantlf/gulp-jsonlint.svg?branch=master)](https://travis-ci.com/prantlf/gulp-jsonlint)
[![codecov](https://codecov.io/gh/prantlf/gulp-jsonlint/branch/master/graph/badge.svg)](https://codecov.io/gh/prantlf/gulp-jsonlint)
[![Dependency Status](https://david-dm.org/prantlf/gulp-jsonlint.svg)](https://david-dm.org/prantlf/gulp-jsonlint)
[![devDependency Status](https://david-dm.org/prantlf/gulp-jsonlint/dev-status.svg)](https://david-dm.org/prantlf/gulp-jsonlint#info=devDependencies)

> [JSON]/[JSON5] file syntax validation plugin for [`Gulp`] using [`JSONLint`]

This is a fork of the original package with the following enhancements:

* Validates with both [JSON] and [JSON5] standards.
* Supports [JSON Schema] drafts 04, 06 and 07.
* Optionally recognizes JavaScript-style comments and single quoted strings.
* Optionally ignores trailing commas and reports duplicate object keys as an error.
* Can sort object keys alphabetically.
* Offers pretty-printing including comment-stripping and object keys without quotes (JSON5).
* Prefers using the 7x faster native JSON parser, if possible.
* Optionally reformats the output JSON and sorts object keys alphabetically.
* Offers alternative error location formatters and message reporters.
* Depends on up-to-date npm modules with no installation warnings.

## Usage

First, install `@prantlf/gulp-jsonlint` as a development dependency:

```shell
npm i -D @prantlf/gulp-jsonlint
```

Then, add it to your `gulpfile.js`:

```javascript
var jsonlint = require("@prantlf/gulp-jsonlint");

gulp.src("./src/*.json")
    .pipe(jsonlint())
    .pipe(jsonlint.reporter());
```

Using a custom reporter:

```javascript
var jsonlint = require('@prantlf/gulp-jsonlint');
var log = require('fancy-log');

var myCustomReporter = function (file) {
    log('File ' + file.path + ' is not valid JSON.');
};

gulp.src('./src/*.json')
    .pipe(jsonlint())
    .pipe(jsonlint.reporter(myCustomReporter));
```

Using an alternative error location *formatter* and error message *reporter*:

```javascript
var jsonlint = require('@prantlf/gulp-jsonlint');

gulp.src('./src/*.json')
    .pipe(jsonlint({
        formatter: 'msbuild', // prose or msbuild
        reporter: 'jshint'    // exception or jshint
    }))
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
        prettyPrint: false,
        indent: 2,
        sortKeys: false,
        pruneComments: false,
        stripObjectKeys: false,
        enforceDoubleQuotes: false,
        enforceSingleQuotes: false,
        trimTrailingCommas: false
    })

* `mode`, when set to "cjson" or "json5", enables some other flags automatically
* `ignoreComments`, when `true` JavaScript-style single-line and multiple-line comments will be recognised and ignored
* `ignoreTrailingCommas`, when `true` trailing commas in objects and arrays will be ignored
* `allowSingleQuotedStrings`, when `true` single quotes will be accepted as alternative delimiters for strings
* `allowDuplicateObjectKeys`, when `false` duplicate keys in objects will be reported as an error

* `format`, when `true` `JSON.stringify` will be used to format the JavaScript (if it is valid)
* `prettyPrint`, when `true` `JSON.stringify` will be used to format the JavaScript (if it is valid)
* `indent`, the value passed to `JSON.stringify`, it can be the number of spaces, or string like "\t"
* `sortKeys`, when `true` keys of objects in the output JSON will be sorted alphabetically (`format` has to be set to `true`)
* `pruneComments`, when `true` comments will be omitted from the prettified output (CJSON feature, `prettyPrint` has to be set to `true`)
* `stripObjectKeys`, when `true` quotes surrounding object keys will be stripped if the key is a JavaScript identifier name (JSON5 feature, `prettyPrint` has to be set to `true`)
* `enforceDoubleQuotes`, when `true` string literals will be consistently surrounded by double quotes (JSON5 feature, `prettyPrint` has to be set to `true`)
* `enforceSingleQuotes`, when `true` string literals will be consistently surrounded by single quotes (JSON5 feature, `prettyPrint` has to be set to `true`)
* `trimTrailingCommas`, when `true` trailing commas after all array items and object entries will be omitted (JSON5 feature, `prettyPrint` has to be set to `true`)

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

This argument has the attribute `jsonlint` which is an object that contains a `success` boolean attribute. If it's false you also have a `message` attribute containing the jsonlint error message.

### jsonlint.reporter(options)

##### options

Type: `object`: { `formatter`, `reporter` }

If you pass an object to `reporter`, the default reported will not be used. There will be two optional string properties recognized in the object - `formatter` and `reporter`. The report will be divided to two parts - the single-line error location (for the formatter) and the rest of the error message (for the reporter). The two parts will be handled by the chosen formatter and reporter.

| Formatter | Description |
| :-------- | :---------- |
| `prose`   | Writes a sentence with the file name and the line and column of the error occurrence (default) |
| `msbuild` | Mimics the format of the first line of the error report printed by MS Visual Studio |

| Reporter    | Description |
| :---------- | :---------- |
| `exception` | Writes the original message returned by `jsonlint` (default) |
| `jshint`    | Mimics the format of the error report printed by `jshint`    |

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

## Examples

The following examples can be tested and the console output observed by using the sample [gulpfile.js] in this repository. All of them can be run by `npm run examples`:

Successful JSON file validation:

    $ gulp valid
    [06:59:17] Using gulpfile .../gulp-jsonlint/gulpfile.js
    [06:59:17] Starting 'valid'...
    [06:59:17] Finished 'valid' after 15 ms

Validation stopped immediately after the first error occurred, plain error printed as returned by `jsonlint`, no colouring:

    $ gulp one
    [06:59:17] Using gulpfile .../gulp-jsonlint/gulpfile.js
    [06:59:17] Starting 'one'...
    [06:59:17] 'one' errored after 17 ms
    [06:59:17] JSONLintError in plugin "gulp-jsonlint"
    Message:
        Parse error on line 2, column 14:
    {    "key1": forgotquotes    "key...
    --------------^
    Unexpected token o
    Details:
        domain: [object Object]
        domainThrown: true

Validation stopped after printing all validation failures, the default (single) error reporter used, colouring enabled:

    $ gulp two
    [06:59:18] Using gulpfile .../gulp-jsonlint/gulpfile.js
    [06:59:18] Starting 'two'...
    [06:59:18] Error on file .../gulp-jsonlint/test/fixtures/comments.json
    [06:59:18] Parse error on line 1, column 1:
    /* Commented configu...
    ^
    Unexpected token /
    [06:59:18] 'two' errored after 25 ms
    [06:59:18] JSONLintError in plugin "gulp-jsonlint"
    Message:
        Failed with 1 error
    Details:
        domain: [object Object]
        domainThrown: true

Validation stopped after printing all validation failures, the default (separate) formatter and reporter used, colouring enabled:

    $ gulp three
    [06:59:18] Using gulpfile .../gulp-jsonlint/gulpfile.js
    [06:59:18] Starting 'three'...
    [06:59:18] File test/fixtures/json5.json failed JSON validation at line 2, column 5.
    {    // String parameter...
    -----^
    Unexpected token /
    [06:59:18] 'three' errored after 17 ms
    [06:59:18] JSONLintError in plugin "gulp-jsonlint"
    Message:
        Failed with 1 error
    Details:
        domain: [object Object]
        domainThrown: true

Validation stopped after printing all validation failures, the `msbuild` formatter and `jshint` reported used, colouring enabled:

    $ gulp all
    [06:59:19] Using gulpfile .../gulp-jsonlint/gulpfile.js
    [06:59:19] Starting 'all'...
    [06:59:19] test/fixtures/comments.json(1,1): error: failed JSON validation
         1 | /* Commented configu...
             ^ Unexpected token /
    [06:59:19] test/fixtures/invalid.json(2,14): error: failed JSON validation
         2 | ..."key1": forgotquo...
                         ^ Unexpected token o
    [06:59:19] test/fixtures/json5.json(2,5): error: failed JSON validation
         2 | {    // String param...
                  ^ Unexpected token /
    [06:59:19] test/fixtures/single-quotes.json(2,5): error: failed JSON validation
         2 | {    'key1': 'value'...
                  ^ Unexpected token '
    [06:59:19] test/fixtures/trailing-commas.json(3,1): error: failed JSON validation
         3 | ... "value",}
                         ^ Unexpected token }
    [06:59:19] 'all' errored after 32 ms
    [06:59:19] JSONLintError in plugin "gulp-jsonlint"
    Message:
        Failed with 5 errors
    Details:
        domain: [object Object]
        domainThrown: true

## License

Copyright (C) 2013-2019 Rogério Vicente, Ferdinand Prantl

Licensed under the [MIT License].

[MIT License]: http://en.wikipedia.org/wiki/MIT_License
[`Gulp`]: http://gulpjs.com/
[`JSONLint`]: https://prantlf.github.io/jsonlint/
[JSON]: https://tools.ietf.org/html/rfc8259
[JSON5]: https://spec.json5.org
[JSON Schema]: https://json-schema.org

[gulpfile.js]: ./gulpfile.js
