# node-stream-concat
Simple and efficient node stream concatenation.

`node-stream-concat` concatenates several streams into one single readable stream. The input streams can either be existing streams or can be determined on the fly by a user specified function. `node-stream-concat` has been tested from Node versions v0.12 through v9.2.0.

    npm install stream-concat

# Usage

```js
  var StreamConcat = require('stream-concat');
  var combinedStream = new StreamConcat(streams, [options]);
```

## streams
The simplest way to use StreamConcat is to supply an array of readable streams.

```js
var fs = require('fs');

var stream1 = fs.createReadStream('file1.csv');
var stream2 = fs.createReadStream('file2.csv');
var stream3 = fs.createReadStream('file3.csv');

var output = fs.createWriteStream('combined.csv');

var combinedStream = new StreamConcat([stream1, stream2, stream3]);
combinedStream.pipe(output);
```

However, when working with large amounts of data, this can lead to high memory usage and relatively poor performance (versus the original stream). This is because all streams' read queues are buffered and waiting to be read.

A better way is to defer opening a new stream until the moment it's needed. You can do this by passing a function into the constructor that returns the next available stream, or `null` if there are no more streams.

If we're reading from several large files, we can do the following.

```js
var fs = require('fs');

var fileNames = ['file1.csv', 'file2.csv', 'file3.csv'];
var fileIndex = 0;
var nextStream = function() {
  if (fileIndex === fileNames.length) {
    return null;
  }

  return fs.createReadStream(fileNames[fileIndex++]);
};

var combinedStream = new StreamConcat(nextStream);
```

Once StreamConcat is done with a stream it'll call `nextStream` and start using the returned stream (if not null);

## options
These are standard `Stream` [options](http://nodejs.org/api/stream.html#stream_new_stream_transform_options) passed to the underlying `Transform` stream.

* `highWaterMark` Number The maximum number of bytes to store in the internal buffer before ceasing to read from the underlying resource. Default=16kb
* `encoding` String If specified, then buffers will be decoded to strings using the specified encoding. Default=null
* `objectMode` Boolean Whether this stream should behave as a stream of objects. Meaning that stream.read(n) returns a single value instead of a Buffer of size n. Default=false

Additional options:
* `advanceOnClose` Boolean Controls if the concatenation should move onto the next stream when the underlying streams emit close event, useful when operating on `Transform` streams and calling destroy on them to skip the remaining data (supported on node >=8). Default=false

## StreamConcat.addStream(newStream)
If you've created the StreamConcat object from an array of streams, you can use `addStream()` as long as the last stream hasn't finishing being read (StreamConcat hasn't emitted the `end` event).

To add streams to a StreamConcat object created from a function, you should modify the underlying data that the function is accessing.

# Tests

    npm run test
