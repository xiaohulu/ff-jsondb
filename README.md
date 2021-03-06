## ff-jsondb

ff-jsondb is a simple and (currently) unoptimized flat file JSON database. It
was created to allow reading/writing to the database without use of this API,
and to be minimal and straight forward. The entire API is synchronous, and I
don't feel bad about that one bit.

**NOTE:** Files are read/written using `Buffer`'s `'latin1'`/`'binary'`
encoding to prevent accidental data loss that can occur when converting a
string to `'utf8'`. Please keep this in mind if intending to open/edit the
files manually.


### API

#### `jsondb(db_path)`

* `db_path` {String}
* Returns new `JSONDB()` instance

`require('ff-jsondb')` returns a function that creates a new `JSONDB()`
instance at the given `db_path`. `db_path` is the path to the folder to the
database. A relative `db_path` is resolved against the script's `process.cwd()`
(i.e. the path where the script was executed).


#### `db.get(key[, regex_name])`

* `key` {String}
* `regex_name` {RegExp}
* Returns {Object}

Retrieve the JSON file(s) at given `key`. The `key` is actually a folder path
and file name of the JSON file. For example:

```js
db.get('/foo/bar');
```

Where from `db_path` in the folder `foo/` it will retrieve the file `bar`
(technically `bar.json`, but for simplicity that's omitted). So it is possible
to store data in the same path as the file. For example:

```js
db.set('/foo/bar', { foo: 'bar' });
db.set('/foo/bar/baz, { bar: 'baz });
```

If `regex_name` is passed then a directory lookup is done for all files
matching the passed `RegExp`. The return value is an `Object` whose keys are
the names of the matching entries.


#### `db.getRaw(key[, regex_name])`

* `key` {String}
* `regex_name` {RegExp}
* Returns {Buffer} or {Object}

Same as `db.get()` except instead of automatically running `JSON.parse()` on
the data it returns a `Buffer` or an `Object` of `Buffer`s.


#### `db.set(key, value)`

* `key` {String}
* `value` {Buffer}, {String} or {Object}

Replace contents at `key` with `value`. Sorry, it's all or nothing here. Either
a `Buffer`, `String` or `Object` can be passed. `Object`s will be passed to
`JSON.stringify()`.

All strings will be written to disk using the `'latin1'`/`'binary'` encoding
to prevent any data loss. Here's an example of the difference between writing
`'latin1'` and `'utf8'`:

```js
const str = 'h\u00e8ll\u00f3 w\u00f4rl\u00de';
// str === 'hèlló wôrlÞ'
fs.writeFileSync(path, str);  // written as 'utf8'
fs.readFileSync(path);
// byte sequence of file:
//   68 c3 a8 6c 6c c3 b3 20 77 c3 b4 72 6c c3 9e

fs.writeFileSync(path, str, { encoding: 'binary' });
fs.readFileSync(path);
// byte sequence of file:
//   68 e8 6c 6c f3 20 77 f4 72 6c de
```

So if you are going to edit these files manually remember to use the correct
character encoding. Otherwise `ff-jsondb` won't be able to read the file in
correctly.


#### `db.del(key)`

Delete entry at location `key`. `key` is always a file, not a directory. So no
deleting many records at once. For now at least.
