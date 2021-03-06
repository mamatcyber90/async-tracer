# async-tracer

Trace all async operations, output as newline delimited JSON logs,
with minimal overhead.

[![Build Status](https://travis-ci.org/davidmarkclements/async-tracer.svg)](https://travis-ci.org/davidmarkclements/async-tracer)

## Supports

Node v4 to v6

## Usage

### Write to a stream

```js
require('async-tracer')(process.stderr)
```

### Write to a path

```js
require('async-tracer')('/var/async.' + process.pid + '.log')
```

### Write to a file handle

```js
require('async-tracer')(1) // 1 is STDOUT, default
require('async-tracer')() //same thing
```


## API

```js
require('async-tracer') => (WritableStream, opts) => {enable, disable}
```

```js
require('async-tracer') => (String: path, ops) => {enable, disable}
```

```js
require('async-tracer') => (Number: handle, opts) => {enable, disable}
```

### Interface

#### `enable`

Start tracing (will start automatically if `autostart` option is `true`)

#### `disable`

Stop all tracing

### Opts

#### `autostart` [default: `true`] `Boolean`

Begin tracing immediately

#### `append` [default: `false`] `Boolean`

Only applies to when a path is supplied, opens file with `a` flag
instead of `w` flag.

#### `prefix` [default: `undefined`] `Object`

Additional data to attach to the beginning of each log message.

#### `suffix` [default: `undefined`] `Object`

Additional data to attach to the end of each log message.

#### `stacks` [default: `false`] `Boolean` or `Number`

If `true` then include an array of call sites in
each `init` log, as the `stack` property. The stack array takes the following form:

```
["functionName:fileName:lineNum:colNum"]
```

If set to a number, (from 1 to Infinity) `stacks` will also
determine the maximum amount of frames to capture for the log
(defaults to `Infinity` if `true`). 

#### `contexts` [default: `false`] `Boolean`

Supply the operations context in the `pre` and `post` logs as the `ctx` property. 
The context is an exposed C object that holds state for the async op.


## Benchmarks

Overhead of using `async-tracer` is about 25%.

```sh
npm run benchmark
```

### With tracing
```
Running 10s test @ http://localhost:3000
10 connections with 10 pipelining factor

Stat         Avg      Stdev     Max
Latency (ms) 0.23     0.8       54
Req/Sec      33283.64 2291.73   35135
Bytes/Sec    3.7 MB   250.51 kB 3.93 MB
```

### Without tracing
```
Running 10s test @ http://localhost:3000
10 connections with 10 pipelining factor

Stat         Avg      Stdev     Max
Latency (ms) 0.13     0.44      33
Req/Sec      45426.91 1279.75   46303
Bytes/Sec    5.06 MB  150.72 kB 5.24 MB
```

Overhead of turning on tracing with `async_wrap`
is around 8%, so the net overhead is 17%, mostly
this is the cost of writing to a stream.

### Benchmarking Options

The cost of turning `stacks` and `contexts` options on can also be determined with: 

```sh
npm run benchmark-options
```

Overhead of enabling context is surprisingly low, 4580k reqs without contexts
4370k reqs with context - about 5% overhead (profiled on Node 6.1.0, Mac OS X 2013, 2.6ghz i7, 16gb).

However, YMMV based on real world usage. Another consideration of logging contexts is the log file
size (although compression is likely to be quite effective). 

Overhead of enabling `stacks` is roughly the same as for enabling `contexts`.

## Example

```js
var http = require('http')
require('async-tracer')()

http.createServer(function (req, res) {
  res.end('hello world')
}).listen(3000)
```

```sh
curl http://localhost:3000
```

## Test

```sh
npm test
```

## License

MIT

## Acknowledgements

Sponsored by [nearForm](http://nearform.com)