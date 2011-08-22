# BufferStream

painless stream buffering, cutting and piping.

## install

make sure you have `node-waf` installed (contained in `nodejs-dev` package).

    npm install bufferstream

## api

BufferStream is a full node.js [Stream](http://nodejs.org/docs/v0.4.7/api/streams.html) so it has apis of both [Writeable Stream](http://nodejs.org/docs/v0.4.7/api/streams.html#writable_Stream) and [Readable Stream](http://nodejs.org/docs/v0.4.7/api/streams.html#readable_Stream).

### BufferStream

```javascript
BufferStream = require('bufferstream')
stream = new BufferStream([{encoding:'utf8', size:'none'}]) // default
```
 * `encoding` default encoding for writing strings
 * `size` defines buffer level or sets buffer to given size (see ↓`setSize` for more)

### stream.setSize

```javascript
stream.setSize(size) // can be one of ['none', 'flexible', <number>]
```

different buffer behaviors can be triggered by size:

 * `none` when output drains, bufferstream drains too
 * `flexible` buffers everthing that it gets and not piping out
 * `<number>` `TODO` buffer has given size. buffers everthing until buffer is full. when buffer is full then  the stream will drain

### stream.enable

```javascript
stream.enable()
```

enables stream buffering __default__

### stream.disable

```javascript
stream.disable()
```

flushes buffer and disables stream buffering.
BufferStream now pipes all data as long as the output accepting data.
when the output is draining BufferStream will buffer all input temporary.

```javascript
stream.disable(token, ...)
stream.disable(tokens) // Array
```
 * `token[s]` buffer splitters (should be String or Buffer)

disables given tokens. wont flush until no splitter tokens are left.

### stream.split

```javascript
stream.split(token, ...)
stream.split(tokens) // Array
```
 * `token[s]` buffer splitters (should be String or Buffer)

each time BufferStream finds a splitter token in the input data it will emit a __split__ event.
this also works for binary data.

### Event: 'split'

```javascript
stream.on('split', function (chunk, token) {…})
stream.split(token, function (chunk, token) {…}) // only get called for given token
```

BufferStream slices its buffer to the first position of on of the splitter tokens and emits it.
this data will be lost when not handled.

__Warning:__ try to avoid calling `stream.emit('data', newchunk)` more than one time, because this will likely throw `Error: Offset is out of bounds`.

### stream.getBuffer

```javascript
stream.getBuffer()
// or just
stream.buffer
```

returns its [Buffer](http://nodejs.org/docs/v0.4.7/api/buffers.html).

### stream.toString

```javascript
stream.toString()
```

shortcut for `stream.buffer.toString()`

### stream.length

```javascript
stream.length
```

shortcut for `stream.buffer.length`

### PostBuffer

```javascript
PostBuffer = require('bufferstream/postbuffer')
post = new PostBuffer(req)
```
 * `req` http.ServerRequest

for if you want to get all the post data from a http server request and do some db reqeust before.

buffer http client

### post.onEnd

```javascript
post.onEnd(function (data) {…});
```

set a callback to get all post data from a http server request

### post.pipe

```javascript
post.pipe(stream, options);
```

pumps data into another stream to allow incoming streams
given options will be passed to Stream.pipe

## example

```javascript
BufferStream = require('bufferstream')
stream = new BufferStream({encoding:'utf8', size:'flexible'})
stream.split('//', ':')
stream.on('split', function (chunk, token) {
    console.log("got '%s' by '%s'", chunk.toString(), token.toString())
})
stream.write("buffer:stream//23")
console.log(stream.toString())
```

results in

    got 'buffer' by ':'
    got 'stream' by '//'
    23

* https://github.com/dodo/node-bufferstream/blob/master/example/split.js

