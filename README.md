# HipJSON

A high performance implementation of JSON parser with std.json syntax. Used by Redub and Hipreme Engine.

## Usage

```d
///Parsing
void main()
{

    import hip.data.json;
    enum jsonSource = q{
    {
        "unicode": "こんいちは",
        "こんにちは": "using unicode key",
        "hello": "oii",
        "test": "teste",
        "com,ma": "val,ue",
        "integer": -5345,
        "floating": -54.23,
        "array": [1,2],
        "strArr":  ["hello", "friend"],
        "mixedArr":  ["hello", 523, -53.23],
        "arrInArr": ["hello", [1, -2, -52.23], "again"],
        "emptyObj": {

        },
        "simpleObj": {
            "path": "sound.wav",
            "data": [1, 2, 3, 4, 5, 6]
        },
        "testObj": {
            "simpleObj": {
                "path": "sound.wav",
                "data": [1, 2, 3, 4, 5, 6]
            },
            "anotherObj": {
                "key": "balanced"
            }
        }
    }};

    JSONValue v = parseJSON(jsonSource);
}

///Mutating and creating the DOM

void main()
{
    JSONValue m = JSONValue.emptyObject;
    m["someKey] = JSONValue(500);
    m["here"] = 500;
    import std.stdio;
    writeln = m.toString;
}
```

## Streaming API

HipJSON also supports parsing JSON in stream. A following example:
```d
import hip.data.json;
import std.exception;
JSONValue myJson;
JSONParseState state = JSONParseState.initialize(0);
enforce(JSONValue.parseStream(myJson, state, `{`) == JSONValue.IncompleteStream);
enforce(JSONValue.parseStream(myJson, state, `"`) == JSONValue.IncompleteStream);
enforce(JSONValue.parseStream(myJson, state, `h`) == JSONValue.IncompleteStream);
enforce(JSONValue.parseStream(myJson, state, `e`) == JSONValue.IncompleteStream);
enforce(JSONValue.parseStream(myJson, state, `l`) == JSONValue.IncompleteStream);
enforce(JSONValue.parseStream(myJson, state, `l`) == JSONValue.IncompleteStream);
enforce(JSONValue.parseStream(myJson, state, `o`) == JSONValue.IncompleteStream);
enforce(JSONValue.parseStream(myJson, state, `"`) == JSONValue.IncompleteStream);
enforce(JSONValue.parseStream(myJson, state, `:`) == JSONValue.IncompleteStream);
enforce(JSONValue.parseStream(myJson, state, ` `) == JSONValue.IncompleteStream);
enforce(JSONValue.parseStream(myJson, state, `"`) == JSONValue.IncompleteStream);
enforce(JSONValue.parseStream(myJson, state, `w`) == JSONValue.IncompleteStream);
enforce(JSONValue.parseStream(myJson, state, `o`) == JSONValue.IncompleteStream);
enforce(JSONValue.parseStream(myJson, state, `r`) == JSONValue.IncompleteStream);
enforce(JSONValue.parseStream(myJson, state, `l`) == JSONValue.IncompleteStream);
enforce(JSONValue.parseStream(myJson, state, `d`) == JSONValue.IncompleteStream);
enforce(JSONValue.parseStream(myJson, state, `"`) == JSONValue.IncompleteStream);
enforce(JSONValue.parseStream(myJson, state, `}`) != JSONValue.IncompleteStream);
import std.stdio;
writeln(myJson);
```

Of course this is only an example of how it works. It is best suited to be used together with a fetch API.
Also, setting a good initial size for the internal string pool makes the JSON parser way faster. It's default is to use 75% of the mentioned data size to use it. So if you can query the size before parsing the stream, it is a big win as the data won't be fragmented and intermediary allocations won't happen:

```d
JSONValue myJson;
JSONParseState state = JSONParseState.initialize(querySize("someapi.com/a/json")); // or std.file.getSize
char[4096] buffer;
while(fetch("someapi.com/a/json", buffer)) //or file.byChunk
{
    if(JSONValue.parseStream(myJson, state, buffer) != JSONValue.IncompleteStream)
        break;
}
//Your json has finished parsing
```


### Testing

With `dub -c test -b release-debug --compiler=ldc2`:
```
STD JSON: 330 ms, 592 μs, and 1 hnsec (50000 Tests)
JSONPIPE: 209 ms, 604 μs, and 3 hnsecs (50000 Tests)
MIR JSON: 259 ms, 756 μs, and 1 hnsec (50000 Tests)
HipJSON: 78 ms, 604 μs, and 5 hnsecs (50000 Tests)
```

HipJSON is currently optimized with d-segmented-hashmap, which makes it get a much faster parsing speed as it never rehashes its dictionaries.
It also has a string buffer performance optimization which makes it even faster when you're dealing with mostly strings.


## JSONs with large strings objects and strings (dub registry dump)

When it is mostly strings, HipJSON is able to reach in my PC up to 735 MB per second.
```
Parsed: 1528 MB
Took: 1779ms
MB per Second: 859.013
Allocated: 2969.01 MB
Free: 740.691 MB
Used: 1218.01 MB
Collection Count: 9
Collection Time: 287 ms, 386 ╬╝s, and 6 hnsecs
```


### Using the Javascript large object generation

Call `node genLargeObject.js` first to generate testJson.json
- JS performance of the parseJSON:
Parsed: 50.00 MB in 0.7036 s
Speed: 71.06 MB/s

- HipJSON parsing that same file
`Call with dub test -b release-debug --compiler=ldc2`

```
Took: 606ms
MB per Second: 86.5162
Allocated: 739.969 MB
Free: 68.7608 MB
Used: 739.962 MB
Collection Count: 7
Collection Time: 273 ms, 757 μs, and 5 hnsecs
```