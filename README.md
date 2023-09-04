## 𝌠 μDSV

A [faster](#performance) CSV parser in [5KB (min)](https://github.com/leeoniya/uDSV/tree/main/dist/uDSV.iife.min.js) _(MIT Licensed)_

---
### Introduction

μDSV is a fast JS library for parsing well-formed CSV strings, either from memory or incrementally from disk or network.
It is mostly [RFC 4180](https://datatracker.ietf.org/doc/html/rfc4180) compliant, with support for quoted values containing commas, escaped quotes, and line breaks¹.
The aim of this project is to handle the 99.5% use-case without adding complexity and performance trade-offs to support the remaining 0.5%.

¹ Line breaks (`\n`,`\r`,`\r\n`) within quoted values must match the row separator.

---
### Features

What does μDSV pack into 5KB?

- [RFC 4180](https://datatracker.ietf.org/doc/html/rfc4180) compliant
- Incremental or full parsing, with optional accumulation
- Auto-detection and customization of delimiters (rows, columns, quotes, escapes)
- Multi-row header skipping and column renaming
- Whitespace trimming of values, skipping empty lines
- Schema inference and value typing: `string`, `number`, `boolean`, `date`, `json`
- Defined handling of `''`, `'null'`, `'NaN'`
- Multiple outputs: arrays (tuples), objects, nested objects, columnar arrays

Of course, _most_ of these are table stakes for CSV parsers :)

---
### Performance

Is it Lightning Fast™ or Blazing Fast™?

No, those are too slow! μDSV has [Ludicrous Speed™](https://www.youtube.com/watch?v=ygE01sOhzz0);
it's faster than the parsers you recognize and faster than those you've never heard of.

On a Ryzen 7 ThinkPad, Linux v6.4.11, and NodeJS v20.5.1, a diverse set of benchmarks show a 1x-5x performance boost relative to [Papa Parse](https://www.papaparse.com/).
Papa Parse is used as a reference not because it's the fastest, but due to its [outsized popularity](https://github.com/search?q=csv+parser&type=repositories&s=stars&o=desc), battle-testedness, and [some external validation](https://leanylabs.com/blog/js-csv-parsers-benchmarks/) of its performance claims.

For _way too many_ synthetic and real-world benchmarks, head over to [/bench](/bench)...and don't forget your coffee!

```
┌───────────────────────────────────────────────────────────────────────────────────────────────┐
│ uszips.csv (6 MB, 18 cols x 34K rows)                                                         │
├────────────────────────┬────────┬─────────────────────────────────────────────────────────────┤
│ Name                   │ Rows/s │ Throughput (MiB/s)                                          │
├────────────────────────┼────────┼─────────────────────────────────────────────────────────────┤
│ uDSV                   │ 754K   │ ░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░ 135 │
│ achilles-csv-parser    │ 474K   │ ░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░ 84.7                    │
│ d3-dsv                 │ 433K   │ ░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░ 77.3                       │
│ csv-rex                │ 361K   │ ░░░░░░░░░░░░░░░░░░░░░░░░░░░ 64.4                            │
│ PapaParse              │ 310K   │ ░░░░░░░░░░░░░░░░░░░░░░░ 55.5                                │
│ csv-js                 │ 296K   │ ░░░░░░░░░░░░░░░░░░░░░░ 52.8                                 │
│ csv42                  │ 285K   │ ░░░░░░░░░░░░░░░░░░░░░ 50.9                                  │
│ comma-separated-values │ 258K   │ ░░░░░░░░░░░░░░░░░░░ 46.1                                    │
│ CSVtoJSON              │ 247K   │ ░░░░░░░░░░░░░░░░░░░ 44.2                                    │
│ csv-simple-parser      │ 245K   │ ░░░░░░░░░░░░░░░░░░ 43.8                                     │
│ dekkai                 │ 244K   │ ░░░░░░░░░░░░░░░░░░ 43.6                                     │
│ csv-parser (neat-csv)  │ 229K   │ ░░░░░░░░░░░░░░░░░ 40.9                                      │
│ ACsv                   │ 223K   │ ░░░░░░░░░░░░░░░░░ 39.8                                      │
│ SheetJS                │ 207K   │ ░░░░░░░░░░░░░░░░ 36.9                                       │
│ @vanillaes/csv         │ 199K   │ ░░░░░░░░░░░░░░░ 35.5                                        │
│ node-csvtojson         │ 170K   │ ░░░░░░░░░░░░░ 30.4                                          │
│ csv-parse/sync         │ 123K   │ ░░░░░░░░░ 22                                                │
│ @fast-csv/parse        │ 80K    │ ░░░░░░ 14.3                                                 │
│ jquery-csv             │ 55.1K  │ ░░░░░ 9.84                                                  │
│ but-csv                │ ---    │ Wrong row count! Expected: 33790, Actual: 1                 │
│ @gregoranders/csv      │ ---    │ Invalid CSV at 1:109                                        │
│ utils-dsv-base-parse   │ ---    │ unexpected error. Encountered an invalid record. Field 17 o │
│ json-2-csv             │ ---    │ Wrong row count! Expected: 33790, Actual: 0                 │
└────────────────────────┴────────┴─────────────────────────────────────────────────────────────┘
```

---
### Installation

---
### Basic Usage

```js
import { inferSchema, initParser } from 'udsv';

let csvStr = 'a,b,c\n1,2,3\n4,5,6';

let schema = inferSchema(csvStr);
let parser = initParser(schema);

// fastest/native format
let stringArrs = parser.stringArrs(csvStr); // [ ['1','2','3'], ['4','5','6'] ]

// typed formats (internally converted from native)
let typedArrs  = parser.typedArrs(csvStr);  // [ [1, 2, 3], [4, 5, 6] ]
let typedObjs  = parser.typedObjs(csvStr);  // [ {a: 1, b: 2, c: 3}, {a: 4, b: 5, c: 6} ]
let typedCols  = parser.typedCols(csvStr);  // [ [1, 4], [2, 5], [3, 6] ]
```

Nested/deep objects can be re-constructed from column naming via `.typedDeep()`:

```js
// deep/nested objects (from column naming)
let csvStr2 = `
_type,name,description,location.city,location.street,location.geo[0],location.geo[1],speed,heading,size[0],size[1],size[2]
item,Item 0,Item 0 description in text,Rotterdam,Main street,51.9280712,4.4207888,5.4,128.3,3.4,5.1,0.9
`.trim();

let schema2 = inferSchema(csvStr2);
let parser2 = initParser(schema2);

let typedDeep = parser2.typedDeep(csvStr2);

/*
[
  {
    _type: 'item',
    name: 'Item 0',
    description: 'Item 0 description in text',
    location: {
      city: 'Rotterdam',
      street: 'Main street',
      geo: [ 51.9280712, 4.4207888 ]
    },
    speed: 5.4,
    heading: 128.3,
    size: [ 3.4, 5.1, 0.9 ],
  }
]
*/
```

---
### Incremental / Streaming

μDSV has no inherent knowledge of streams.
Instead, it exposes a generic incremental parsing API to which you can pass sequential chunks.
These chunks can come from various sources, such as a [Web Stream](https://css-tricks.com/web-streams-everywhere-and-fetch-for-node-js/) or [Node stream](https://nodejs.org/api/stream.html) via `fetch()` or `fs`, a [WebSocket](https://developer.mozilla.org/en-US/docs/Web/API/WebSockets_API), etc.

Here's what it looks like with Node's [fs.createReadStream()](https://nodejs.org/api/fs.html#fscreatereadstreampath-options):

```js
let stream = fs.createReadStream(filePath);

let parser = null;
let result = null;

stream.on('data', (chunk) => {
  // convert from Buffer
  let strChunk = chunk.toString();
  // on first chunk, infer schema and init parser
  parser ??= initParser(inferSchema(strChunk));
  // incremental parse to string arrays
  parser.chunk(strChunk, parser.stringArrs);
});

stream.on('end', () => {
  result = p.end();
});
```

...and Web streams [in Node](https://nodejs.org/api/webstreams.html), or [Fetch's Response.body](https://developer.mozilla.org/en-US/docs/Web/API/Response/body):

```js
let stream = fs.createReadStream(filePath);

let webStream = Stream.Readable.toWeb(stream);
let textStream = webStream.pipeThrough(new TextDecoderStream());

let parser = null;

for await (const strChunk of textStream) {
  parser ??= initParser(inferSchema(strChunk));
  parser.chunk(strChunk, parser.stringArrs);
}

let result = parser.end();
```

The above examples show accumulating parsers -- they will buffer the full `result` into memory.
This may not be something you want (or need), for example with huge datasets where you're looking to get the sum of a single column, or want to filter only a small subset of rows.
To bypass this auto-accumulation behavior, simply pass your own handler as the third argument to `parser.chunk()`:

```js
// ...same as above

let sum = 0;

let reducer = (rows) => {
  for (let i = 0; i < rows.length; i++) {
    sum += rows[i][3]; // sum fourth column
  }
};

for await (const strChunk of textStream) {
  parser ??= initParser(inferSchema(strChunk));
  parser.chunk(strChunk, parser.typedArrs, reducer); // typedArrs + reducer
}

parser.end();
```

---
### TODO?

- handle #comment rows
- emit empty-row and #comment events?