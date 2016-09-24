# ajv-pack

Produces a compact module exporting JSON-schema validation functions compiled by [Ajv](https://github.com/epoberezkin/ajv)

[![Build Status](https://travis-ci.org/epoberezkin/ajv-pack.svg?branch=master)](https://travis-ci.org/epoberezkin/ajv-pack)
[![npm version](https://badge.fury.io/js/ajv-pack.svg)](https://www.npmjs.com/package/ajv-pack)
[![Coverage Status](https://coveralls.io/repos/github/epoberezkin/ajv-pack/badge.svg?branch=master)](https://coveralls.io/github/epoberezkin/ajv-pack?branch=master)


## Purpose

This package allows to create standalone modules for validation functions that are pre-compiled and can be used without Ajv. It can be necessary for several reasons:

- to reduce the browser bundle size - Ajv is not included in the bundle (although for a large number of schemas the bundle size can become bigger instead - when the total size of generated validation code is bigger than Ajv code).
- to reduce the startup time - the validation and compilation of schemas will happen during build time.
- to avoid dynamic code evaluation with Function constructor (used for schema compilation) - it can be prohibited in case [Content Security Policy](http://www.html5rocks.com/en/tutorials/security/content-security-policy/) is used.


## Install

```
npm install ajv-pack
```


## Usage

```javascript
var Ajv = require('ajv'); // version >= 4.7.4
var ajv = new Ajv({sourceCode: true}); // this option is required
var pack = require('ajv-pack');

var schema = {
  type: 'object',
  properties: {
    foo: {
      type: 'string',
      pattern: '^[a-z]+$'
    }
  }
};

var validate = ajv.compile(schema);
var moduleCode = pack(ajv, validate);

// now you can
// 1. write module code to file
var fs = require('fs');
var path = require('path');
fs.writeFileSync(path.join(__dirname, '/validate.js'), moduleCode);

// 2. require module from string
var requireFromString = require('require-from-string');
var packedValidate = requireFromString(moduleCode);
```

Ajv should still be a run-time dependency, but generated modules will only depend on some parts of it, the whole Ajv will not be included in the bundle if you require these modules from your code.


## Limitations

At the moment ajv-pack does not support schemas with:

- custom keywords ('compiled' and 'validated'); custom inline and macro keywords are supported.
- custom formats (they will be ignored during validation).
- mutually recursive references (reference to the current schema `{ "$ref": "#" }` is supported).
- asynchronous schemas (they require custom keywords/formats).


## License

[MIT](https://github.com/epoberezkin/ajv-pack/blob/master/LICENSE)
