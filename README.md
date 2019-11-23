[![Build Status](https://travis-ci.org/Borewit/strtok3.svg?branch=master)](https://travis-ci.org/Borewit/strtok3)
[![NPM version](https://badge.fury.io/js/strtok3.svg)](https://npmjs.org/package/strtok3)
[![npm downloads](http://img.shields.io/npm/dm/strtok3.svg)](https://npmcharts.com/compare/strtok3,strtok2?start=600&interval=30)
[![Dependabot Status](https://api.dependabot.com/badges/status?host=github&repo=Borewit/music-metadata)](https://dependabot.com)[![Coverage status](https://coveralls.io/repos/github/Borewit/strtok3/badge.svg?branch=master)](https://coveralls.io/github/Borewit/strtok3?branch=master)
[![Known Vulnerabilities](https://snyk.io/test/github/Borewit/strtok3/badge.svg?targetFile=package.json)](https://snyk.io/test/github/Borewit/strtok3?targetFile=package.json)
[![Total alerts](https://img.shields.io/lgtm/alerts/g/Borewit/strtok3.svg?logo=lgtm&logoWidth=18)](https://lgtm.com/projects/g/Borewit/strtok3/alerts/)
[![Codacy Badge](https://api.codacy.com/project/badge/Grade/59dd6795e61949fb97066ca52e6097ef)](https://www.codacy.com/app/Borewit/strtok3?utm_source=github.com&amp;utm_medium=referral&amp;utm_content=Borewit/strtok3&amp;utm_campaign=Badge_Grade)
[![Language grade: JavaScript](https://img.shields.io/lgtm/grade/javascript/g/Borewit/strtok3.svg?logo=lgtm&logoWidth=18)](https://lgtm.com/projects/g/Borewit/strtok3/context:javascript)
# strtok3

A promise based streaming [*tokenizer*](#tokenizer) for [Node.js](http://nodejs.org) and browsers.
This node module is a successor of [strtok2](https://github.com/Borewit/strtok2).

The `strtok3` contains a few methods to turn different input into a [*tokenizer*](#tokenizer). Designed to
*   Support a streaming environment
*   Decoding of binary data, strings and numbers in mind
*   Read custom tokens.
*   Optimized [*tokenizers*](#tokenizer) for reading from [file](#method-strtok3fromfile), [stream](#method-strtok3fromstream) or [buffer](#method-strtok3frombuffer).

It can read from:
*   A file (taking a file path as an input)
*   A Node.js [stream](https://nodejs.org/api/stream.html).
*   A [Buffer](https://nodejs.org/api/buffer.html)
*   HTTP chunked transfer provided by [streaming-http-token-reader](https://github.com/Borewit/streaming-http-token-reader).

## Usage

Use one of the methods to instantiate an [*abstract tokenizer*](#tokenizer):
*   [strtok3.fromFile](#method-strtok3fromfile)
*   [strtok3.fromStream](#method-strtok3fromstream)
*   [strtok3.fromBuffer](#method-strtok3frombuffer)

### strtok3 methods

All of the strtok3 methods return a [*tokenizer*](#tokenizer), either directly or via a promise.

#### Method `strtok3.fromFile()`
Returns, via a promise, a [*tokenizer*](#tokenizer) which can be used to parse a file.

```js
import strtok3 from 'strtok3';
import Token from 'token-types';
    
strtok3.fromFile("somefile.bin").then(tokenizer => {
  return tokenizer.readToken<number>(Token.UINT8).then(myUint8Number => {
    console.log("My number: %s", myUint8Number);
  });
})
```

#### Method `strtok3.fromStream()`

Create [*tokenizer*](#tokenizer) from a node.js [readable stream](https://nodejs.org/api/stream.html#stream_class_stream_readable).

| Parameter  | Type                                                                        | Description                     |
|------------|-----------------------------------------------------------------------------|---------------------------------|
| stream     | [Readable](https://nodejs.org/api/stream.html#stream_class_stream_readable) | Stream to read from             |

```js
import strtok3 from 'strtok3';
import Token from 'token-types';

strtok3.fromStream(stream).then(tokenizer => {
  return tokenizer.readToken<number>(Token.UINT8).then(myUint8Number => {
    console.log("My number: %s", myUint8Number);
  });
});
```
Returns a [*tokenizer*](#tokenizer), via a Promise, which can be used to parse a buffer.

#### Method `strtok3.fromBuffer()`

Returns a [*tokenizer*](#tokenizer) which can be used to parse a buffer.
```js
import strtok3 from 'strtok3';
    
const tokenizer = strtok3.fromBuffer(buffer);

tokenizer.readToken<number>(Token.UINT8).then(myUint8Number => {
  console.log("My number: %s", myUint8Number);
});
```
## Tokenizer
The tokenizer allows us to *read* or *peek* from the *tokenizer-stream*. The *tokenizer-stream* is an abstraction of a [stream](https://nodejs.org/api/stream.html), file or [Buffer](https://nodejs.org/api/buffer.html).
It can also be translated in chunked reads, as done in [streaming-http-token-reader](https://github.com/Borewit/streaming-http-token-reader);

What is the difference with Nodejs.js stream?
*   The *tokenizer-stream* supports jumping / seeking in a the *tokenizer-stream* using [`tokenizer.ignore()`](#method-tokenizerignore)
*   In addition to *read* methods, it has *peek* methods, to read a ahead and check what is coming.

The [tokenizer.position](#attribute-tokenizerposition) keeps tracks of

### Tokenizer attributes

#### Attribute `tokenizer.fileSize`
Optional attribute of the total file or stream length in bytes

#### Attribute `tokenizer.position`
Pointer to the current position in the [*tokenizer*](#tokenizer) stream.
If a *position* is provided to a *read* or *peek* method, is should be, at least, equal or greater than this value.

### Tokenizer methods 

There are to groups of methods

*   *read* methods: used to read a *token* of [Buffer](https://nodejs.org/api/buffer.html) from the [*tokenizer*](#tokenizer). The position of the *tokenizer-stream* will advance with the size of the token.
*   *peek* methods: same as the read, but it will *not* advance the pointer. It allows to read (peek) ahead.

#### Method `tokenizer.readBuffer()`

Read buffer from stream.
`readBuffer(buffer, offset?, length?, position?, maybeless?)`

| Parameter  | Type                                                           | Description                                                                                                                           |
|------------|----------------------------------------------------------------|---------------------------------------------------------------------------------------------------------------------------------------|
| buffer     | [Buffer](https://nodejs.org/api/buffer.html) &#124; Uint8Array | Target buffer to write the data read to                                                                                               |
| offset?    | number                                                         | The offset in the buffer to start writing at; if not provided, start at 0                                                             |
| length?    | number                                                         | An integer specifying the number of bytes to read                                                                                     |
| position?  | number                                                         | An integer specifying where to begin reading from in the file. If position is null, data will be read from the current file position. |
| maybeless? | boolean                                                        | If set, will not throw an EOF error if the less then the requested length could be read                                               |

Return value `Promise<number>` Promise with number of bytes read

#### Method `tokenizer.peekBuffer()`

Peek (read ahead) buffer from [*tokenizer*](#tokenizer)
`peekBuffer(buffer, offset?, length?, position?, maybeless?)`

| Parameter  | Type                     | Description                                                                                                           |
|------------|--------------------------|-----------------------------------------------------------------------------------------------------------------------|
| buffer     | Buffer &#124; Uint8Array | Target buffer to write the data read (peeked) to                                                                      |
| offset?    | number                   | The offset in the buffer to start writing at; if not provided, start at 0                                             |
| length?    | number                   | The number of bytes to read.                                                                                          |
| position?  | number                   | Offset where to begin reading within the file. If position is null, data will be read from the current file position. |
| maybeless? | boolean                  | If set, will not throw an EOF error if the less then the requested length could be read                               |

Return value `Promise<number>` Promise with number of bytes read

#### Method `tokenizer.readToken()`

Read a *token* from the tokenizer-stream.
`readToken(token, position?, maybeless?)`

| Parameter  | Type                    | Description                                                                                                                           |
|------------|-------------------------|---------------------------------------------------------------------------------------------------------------------------------------|
| token      | [IGetToken](#IGetToken) | Token to read from the tokenizer-stream.                                                                                              |
| position?  | number                  | Offset where to begin reading within the file. If position is null, data will be read from the current file position.                 |
| maybeless? | boolean                 | If set, will not throw an EOF error if the less then the requested length could be read.                                              |

Return value `Promise<T>`. Promise with token value read from the *tokenizer-stream*.

#### Method `tokenizer.peekToken()`

Peek a *token* from the tokenizer-stream.
`peekToken(token, position?, maybeless?)`

| Parameter  | Type                       | Description                                                                                                             |
|------------|----------------------------|-------------------------------------------------------------------------------------------------------------------------|
| token      | [IGetToken<T>](#IGetToken) | Token to read from the tokenizer-stream.                                                                                |
| position?  | number                     | Offset where to begin reading within the file. If position is null, data will be read from the current file position.   |
| maybeless? | boolean                    | If set, will not throw an EOF error if the less then the requested length could be read                                 |

Return value `Promise<T>` Promise with token value peeked from the *tokenizer-stream*.

#### Method `tokenizer.readNumber()`

Peek a numeric [*token*](#token) from the *tokenizer-stream*.
`readNumber(token)`

| Parameter  | Type                            | Description                                        |
|------------|---------------------------------|----------------------------------------------------|
| token      | [IGetToken<number>](#IGetToken) | Numeric token to read from the tokenizer-stream.   |

Return value `Promise<number>` Promise with number peeked from the *tokenizer-stream*.

#### Method `tokenizer.ignore()`

Peek a numeric [*token*](#token) from the tokenizer-stream.
`ignore(length)`

| Parameter  | Type   | Description                                                          |
|------------|--------|----------------------------------------------------------------------|
| ignore     | number | Numeric of bytes to ignore. Will advance the `tokenizer.position`    |

Return value `Promise<number>` Promise with number peeked from the *tokenizer-stream*.

#### Method `tokenizer.close()`
Clean up resources, such as closing a file pointer if applicable.

## Token

The *token* is basically a description what to read form the [*tokenizer-stream*](#tokenizer). 
A basic set of *token types* can be found here: [*token-types*](https://github.com/Borewit/token-types).

Below is an example of parsing the the first byte from a readable stream as an unsigned-integer:

```js
import strtok3 from 'strtok3';
import Token from 'token-types';
    
let readableStream; // stream.Readable;

strtok3.fromStream(readableStream).then(tokenizer => {
  return tokenizer.readToken<number>(Token.UINT8).then(myUint8Number => {
    console.log("My number: %s", myUint8Number);
  });
})
```

## Browser
To exclude fs based dependencies, you can use a submodule-import from 'strtok3/lib/core'.

| function              | 'strtok3'           | 'strtok3/lib/core'  |
| ----------------------| --------------------|---------------------|
| `parseBuffer`         | ✓                   | ✓                   |
| `parseStream`         | ✓                   | ✓                   |
| `fromFile`            | ✓                   |                     |

Example submodule-import:
```js
import strtok3core from 'strtok3/lib/core';

const tokenizer = strtok3core.fromStream(stream);
```

If you plan to use `fromStream` you need to polyfill: 
1.   buffer: [buffer](https://www.npmjs.com/package/buffer)
2.   stream: [web-streams-polyfill](https://www.npmjs.com/package/web-streams-polyfill)
