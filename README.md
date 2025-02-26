express-csp
===========

[![npm Version][npm-badge]][npm]
[![Build Status][travis-badge]][travis]

Usage
-----

This is an Express extension which allows you to set the [`content-security-policy`](https://w3c.github.io/webappsec-csp/) for your Express Application.

API
---

### extend
```js
var csp = require('express-csp');

var app = express();

csp.extend(app, {
    policy: {
        directives: {
            'default-src': ['self', 'https://*.foo.com'],
            'script-src': ['*.apis.bar.com']
        }
    },
    reportPolicy: {
        useScriptNonce: true,
        useStyleNonce: true,
        directives: {
            'default-src': ['self', 'https://*.foo.com'],
            'script-src': ['*.apis.bar.com'],
            'plugin-types': ['application/pdf']
        }
    }
});
```

The `extend` method takes two arguments. A reference to the express application, `app`, and
a config object containing the following properties:


#### policy
An object containing necessary information to generate policy directives to be added to the [`content-security-policy`](https://w3c.github.io/webappsec-csp/#csp-header) header. The `policy` object can contain the following possible properties:

##### useScriptNonce

When set to true, a [`nonce`](https://w3c.github.io/webappsec-csp/#directive-script-src) will be generated for the `'script-src'` directive of each response and made available as the `res.locals.cspToken` value. This value can then be used in your templates to allow for specified inline script blocks. If [`useStyleNonce`](#useStyleNonce) is also true, the same token will be added to the `'style-src'` directive and the same token will be available for inline style blocks.

##### useStyleNonce

When set to true, a [`nonce`](https://w3c.github.io/webappsec-csp/#directive-style-src) will be generated for the `'style-src'` directive of each response and made available as the `res.locals.cspToken` value. This value can then be used in your templates to allow for specified inline script and style blocks. If [`useScriptNonce`](#useScriptNonce) is also true, the same token will be added to the `'script-src'` directive and the same token will be available for inline script blocks.

```html
<script nonce="{{res.locals.cspToken}}">
foo();
</script>
```

##### directives 
An object of key/value pairs representing [CSP Policy Directives](https://w3c.github.io/webappsec-csp/#directives) in which the keys refer to the directive
name and the value is an array of rules to apply to that value. 

- [`base-uri`](https://w3c.github.io/webappsec-csp/#directive-base-uri)
- [`block-all-mixed-content`](https://w3c.github.io/webappsec-csp/#directives-elsewhere)
- [`child-src`](https://w3c.github.io/webappsec-csp/#directive-child-src)
- [`connect-src`](https://w3c.github.io/webappsec-csp/#directive-connect-src)
- [`default-src`](https://w3c.github.io/webappsec-csp/#directive-default-src)
- [`font-src`](https://w3c.github.io/webappsec-csp/#directive-font-src)
- [`form-action`](https://w3c.github.io/webappsec-csp/#directive-form-action)
- [`frame-ancestors`](https://w3c.github.io/webappsec-csp/#directive-frame-ancestors)
- [`frame-src`](https://w3c.github.io/webappsec-csp/#frame-src)
- [`img-src`](https://w3c.github.io/webappsec-csp/#img-src)
- [`media-src`](https://w3c.github.io/webappsec-csp/#media-src)
- [`object-src`](https://w3c.github.io/webappsec-csp/#directive-object-src)
- [`plugin-types`](https://w3c.github.io/webappsec-csp/#directive-plugin-types)
- [`prefetch-src`](https://w3c.github.io/webappsec-csp/#directive-prefetch-src)
- [`report-uri`](https://w3c.github.io/webappsec-csp/#directive-report-uri)
- [`reflected-xss`](https://w3c.github.io/webappsec-csp/#directive-report-uri)
- [`require-sri-for`](https://w3c.github.io/webappsec-csp/#directive-report-uri)
- [`script-src`](https://w3c.github.io/webappsec-csp/#directive-script-src)
- [`style-src`](https://w3c.github.io/webappsec-csp/#directive-style-src)
- [`upgrade-insecure-requests`](https://w3c.github.io/webappsec-csp/#directive-report-uri)
- [`worker-src`](https://w3c.github.io/webappsec-csp/#directive-worker-src)
- [`manifest-src`](https://w3c.github.io/webappsec-csp/#directive-manifest-src)



#### reportPolicy
An object containing necessary information to generate policy directives to be added to the [`content-security-policy-report-only`](https://w3c.github.io/webappsec-csp/#cspro-header) header. The `reportPolicy` object can contain the same properties specified for the [`policy`](#policy) object.


### signScript

Generates and adds a [valid hash](https://w3c.github.io/webappsec-csp/#directive-script-src) to the `script-src` directive. 

At the app level
```js
app.signScript('foo();');
```

Enables `foo();` throughout the app
```html
<script>foo();</script>
```
At the response level
```js
app.route('/').get(function (req, res) {
    res.signScript('bar();');
});
```
Enables `bar();` for the route only.
```html
<script>bar();</script>
```

These will not work with the above examples.
```html
<script>
foo();
</script>

<script>
bar();
</script>
```

### signStyle

Generates and adds a [valid hash](https://w3c.github.io/webappsec-csp/#directive-style-src) to the `style-src` directive. 

```js
app.signStyle('body{background-color:#eee}');
```

```js
app.route('/').get(function (req, res) {
    res.signStyle('body{background-color:#eee}');
});
```

### res.setPolicy
Allows policy to be set per request. The app level policy set in `extend` will be ignored when `res.setPolicy` is used. This method takes the same config object as the `extend` method.

```js
app.get('/', function(req, res, next) {
    res.setPolicy({
        policy: {
            directives: {
                'script-src' : ['unsafe-inline', '*.foo.com']
            }
        },
        reportPolicy: {
            useNonce: true,
            directives: {
                'script-src' : ['*.foo.com']
            }
        }
    });
});
```
### License

Code licensed under the BSD license. See [LICENSE file][] file for terms.

[LICENSE file]: https://github.com/yahoo/express-csp/blob/master/LICENSE
[travis]: https://travis-ci.org/yahoo/express-csp
[travis-badge]: http://img.shields.io/travis/yahoo/express-csp.svg?style=flat-square
[npm]: https://www.npmjs.org/package/express-csp
[npm-badge]: https://img.shields.io/npm/v/express-csp.svg?style=flat-square
