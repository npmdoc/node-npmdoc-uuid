# api documentation for  [uuid (v3.0.1)](https://github.com/kelektiv/node-uuid#readme)  [![travis-ci.org build-status](https://api.travis-ci.org/npmdoc/node-npmdoc-uuid.svg)](https://travis-ci.org/npmdoc/node-npmdoc-uuid)
#### RFC4122 (v1 and v4) generator

[![NPM](https://nodei.co/npm/uuid.png?downloads=true)](https://www.npmjs.com/package/uuid)

[![apidoc](https://npmdoc.github.io/node-npmdoc-uuid/build/screen-capture.buildNpmdoc.browser._2Fhome_2Ftravis_2Fbuild_2Fnpmdoc_2Fnode-npmdoc-uuid_2Ftmp_2Fbuild_2Fapidoc.html.png)](https://npmdoc.github.io/node-npmdoc-uuid/build..beta..travis-ci.org/apidoc.html)

![package-listing](https://npmdoc.github.io/node-npmdoc-uuid/build/screen-capture.npmPackageListing.svg)



# package.json

```json

{
    "bin": {
        "uuid": "./bin/uuid"
    },
    "browser": {
        "./lib/rng.js": "./lib/rng-browser.js"
    },
    "bugs": {
        "url": "https://github.com/kelektiv/node-uuid/issues"
    },
    "contributors": [
        {
            "name": "Robert Kieffer",
            "email": "robert@broofa.com"
        },
        {
            "name": "Christoph Tavan",
            "email": "dev@tavan.de"
        },
        {
            "name": "AJ ONeal",
            "email": "coolaj86@gmail.com"
        },
        {
            "name": "Vincent Voyer",
            "email": "vincent@zeroload.net"
        },
        {
            "name": "Roman Shtylman",
            "email": "shtylman@gmail.com"
        }
    ],
    "dependencies": {},
    "description": "RFC4122 (v1 and v4) generator",
    "devDependencies": {
        "mocha": "3.1.2"
    },
    "directories": {},
    "dist": {
        "shasum": "6544bba2dfda8c1cf17e629a3a305e2bb1fee6c1",
        "tarball": "https://registry.npmjs.org/uuid/-/uuid-3.0.1.tgz"
    },
    "gitHead": "374de826de71d8997f71b4641f65552e48956ced",
    "homepage": "https://github.com/kelektiv/node-uuid#readme",
    "keywords": [
        "uuid",
        "guid",
        "rfc4122"
    ],
    "license": "MIT",
    "maintainers": [
        {
            "name": "broofa",
            "email": "robert@broofa.com"
        },
        {
            "name": "defunctzombie",
            "email": "shtylman@gmail.com"
        },
        {
            "name": "vvo",
            "email": "vincent.voyer@gmail.com"
        }
    ],
    "name": "uuid",
    "optionalDependencies": {},
    "readme": "ERROR: No README data found!",
    "repository": {
        "type": "git",
        "url": "git+https://github.com/kelektiv/node-uuid.git"
    },
    "scripts": {
        "test": "mocha test/test.js"
    },
    "version": "3.0.1"
}
```



# <a name="apidoc.tableOfContents"></a>[table of contents](#apidoc.tableOfContents)

#### [module uuid](#apidoc.module.uuid)
1.  [function <span class="apidocSignatureSpan">uuid.</span>v1 (options, buf, offset)](#apidoc.element.uuid.v1)
1.  [function <span class="apidocSignatureSpan">uuid.</span>v4 (options, buf, offset)](#apidoc.element.uuid.v4)



# <a name="apidoc.module.uuid"></a>[module uuid](#apidoc.module.uuid)

#### <a name="apidoc.element.uuid.v1"></a>[function <span class="apidocSignatureSpan">uuid.</span>v1 (options, buf, offset)](#apidoc.element.uuid.v1)
- description and source-code
```javascript
function v1(options, buf, offset) {
  var i = buf && offset || 0;
  var b = buf || [];

  options = options || {};

  var clockseq = options.clockseq !== undefined ? options.clockseq : _clockseq;

  // UUID timestamps are 100 nano-second units since the Gregorian epoch,
  // (1582-10-15 00:00).  JSNumbers aren't precise enough for this, so
  // time is handled internally as 'msecs' (integer milliseconds) and 'nsecs'
  // (100-nanoseconds offset from msecs) since unix epoch, 1970-01-01 00:00.
  var msecs = options.msecs !== undefined ? options.msecs : new Date().getTime();

  // Per 4.2.1.2, use count of uuid's generated during the current clock
  // cycle to simulate higher resolution clock
  var nsecs = options.nsecs !== undefined ? options.nsecs : _lastNSecs + 1;

  // Time since last uuid creation (in msecs)
  var dt = (msecs - _lastMSecs) + (nsecs - _lastNSecs)/10000;

  // Per 4.2.1.2, Bump clockseq on clock regression
  if (dt < 0 && options.clockseq === undefined) {
    clockseq = clockseq + 1 & 0x3fff;
  }

  // Reset nsecs if clock regresses (new clockseq) or we've moved onto a new
  // time interval
  if ((dt < 0 || msecs > _lastMSecs) && options.nsecs === undefined) {
    nsecs = 0;
  }

  // Per 4.2.1.2 Throw error if too many uuids are requested
  if (nsecs >= 10000) {
    throw new Error('uuid.v1(): Can\'t create more than 10M uuids/sec');
  }

  _lastMSecs = msecs;
  _lastNSecs = nsecs;
  _clockseq = clockseq;

  // Per 4.1.4 - Convert from unix epoch to Gregorian epoch
  msecs += 12219292800000;

  // 'time_low'
  var tl = ((msecs & 0xfffffff) * 10000 + nsecs) % 0x100000000;
  b[i++] = tl >>> 24 & 0xff;
  b[i++] = tl >>> 16 & 0xff;
  b[i++] = tl >>> 8 & 0xff;
  b[i++] = tl & 0xff;

  // 'time_mid'
  var tmh = (msecs / 0x100000000 * 10000) & 0xfffffff;
  b[i++] = tmh >>> 8 & 0xff;
  b[i++] = tmh & 0xff;

  // 'time_high_and_version'
  b[i++] = tmh >>> 24 & 0xf | 0x10; // include version
  b[i++] = tmh >>> 16 & 0xff;

  // 'clock_seq_hi_and_reserved' (Per 4.2.2 - include variant)
  b[i++] = clockseq >>> 8 | 0x80;

  // 'clock_seq_low'
  b[i++] = clockseq & 0xff;

  // 'node'
  var node = options.node || _nodeId;
  for (var n = 0; n < 6; ++n) {
    b[i + n] = node[n];
  }

  return buf ? buf : bytesToUuid(b);
}
```
- example usage
```shell
...

Browser-ready versions of this module are available via [wzrd.in](https://github.com/jfhbrook/wzrd.in).

'''html
<script src="http://wzrd.in/standalone/uuid@latest"></script>

<script>
uuid.v1(); // -> v1 UUID
uuid.v4(); // -> v4 UUID
</script>
'''

(Note: Do not do this in production.  Just don't.  wzrd.in is a great service, but if you're deploying a "real" service you should
 be using a packaging tool like browserify or webpack.  If you do go this route you would be well advised to link to a specific
version instead of 'uuid@latest' to avoid having your code break when we roll out breaking changes.)
...
```

#### <a name="apidoc.element.uuid.v4"></a>[function <span class="apidocSignatureSpan">uuid.</span>v4 (options, buf, offset)](#apidoc.element.uuid.v4)
- description and source-code
```javascript
function v4(options, buf, offset) {
  var i = buf && offset || 0;

  if (typeof(options) == 'string') {
    buf = options == 'binary' ? new Array(16) : null;
    options = null;
  }
  options = options || {};

  var rnds = options.random || (options.rng || rng)();

  // Per 4.4, set bits for version and 'clock_seq_hi_and_reserved'
  rnds[6] = (rnds[6] & 0x0f) | 0x40;
  rnds[8] = (rnds[8] & 0x3f) | 0x80;

  // Copy bytes to buffer, if provided
  if (buf) {
    for (var ii = 0; ii < 16; ++ii) {
      buf[i + ii] = rnds[ii];
    }
  }

  return buf || bytesToUuid(rnds);
}
```
- example usage
```shell
...
Browser-ready versions of this module are available via [wzrd.in](https://github.com/jfhbrook/wzrd.in).

'''html
<script src="http://wzrd.in/standalone/uuid@latest"></script>

<script>
uuid.v1(); // -> v1 UUID
uuid.v4(); // -> v4 UUID
</script>
'''

(Note: Do not do this in production.  Just don't.  wzrd.in is a great service, but if you're deploying a "real" service you should
 be using a packaging tool like browserify or webpack.  If you do go this route you would be well advised to link to a specific
version instead of 'uuid@latest' to avoid having your code break when we roll out breaking changes.)


## API
...
```



# misc
- this document was created with [utility2](https://github.com/kaizhu256/node-utility2)
