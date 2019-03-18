# Magic Attribute Protocol
### A system for mapping arbitrary hyper media to a global identifier.

Prefix: *1PuQa7K62MiKCtssSLKy1kh56WWU7MtUR5*

## Usage

```markdown
<OP_RETURN | <input>>
MAP
<SET | DELETE>
<key>
<value>
```

#### Use cases
- map a comment to a url
- map an action to a txhash (like, repost, or flag a comment)
- map a photo to a geolocation
- map a 'type' to some data (this is a 'post' or a 'reply')
- map ______ to a _______

#### PROTOCOL CHAINING

MAP is designed to be chained together with other OP_RETURN micro-protocols. The input stream flows from the left and can be piped like unix commands. We chain content from B protocol, and map it to some global identifier (txhash, url etc), and finally sign that using the Author Identity protocol:

    B | MAP | AUTHOR_IDENTITY

More about B protocol
- https://github.com/unwriter/B

More on protocol piping:
- https://github.com/unwriter/Bitcom/issues/2

More on Autor Identity Protocol:
- https://github.com/BitcoinFiles/AUTHOR_IDENTITY_PROTOCOL

# Examples
## SET
In this example we will use `SET` to comment on a URL with an identity (not using the sender address as the identity's public key). Here is a simple piped OP_RETURN sequence for mapping B data to a global ID, `url` = `http://map.sv`.

```
OP_RETURN B | MAP SET 'url' 'https://map.sv' | AUTHOR_IDENTITY_PROTOL
```

A more detailed view of the same transaction. Each line here is a new pushdata:

```markdown
OP_RETURN
19HxigV4QyBv3tHpQVcUEQyq1pzZVdoAut (B)
"## Hello small world"
text/markdown
utf8
|
1PuQa7K62MiKCtssSLKy1kh56WWU7MtUR5 (MAP)
'SET'
'url'
'https://map.sv'
|
15PciHG22SNLQJXMoSUaWVi7WSqc7hCfva (AUTHOR_IDENTITY)
1
ecdsa
<pubkey>
<signature>
 ```

Constructing a BitQuery for MAP data for a given url still assumes a 'fixed protocol' for now, but soon we will release some tools for searching 'relative to protocol'. There are some projects in the works to make querying / working with this protocol much nicer in the future.

A response from BITDB for a comment on a URL would look something like this:
```json
{
  "c": [{
    "i": 0,
    "b0": { "op": 106 },
    "s1": "19HxigV4QyBv3tHpQVcUEQyq1pzZVdoAut",
    "s2": "## Hello small world",
    "s3": "text/markdown",
    "s4": "utf8",
    "s5": "|",
    "s6": "1PuQa7K62MiKCtssSLKy1kh56WWU7MtUR5",
    "s7": "SET",
    "s8": "url",
    "s9": "https://twitter.com/",
    "s10": "|",
    "s11": "15PciHG22SNLQJXMoSUaWVi7WSqc7hCfva",
    "s12": "ecdsa",
    "s13": "1HQ8momxTp9MYkzDLy9bFMUQgnba189qZE",
    "s14": "<signature>"
  }],
  "u": []
}
```

Transforming the Output

By defining a protocol schema, we can transform a bitquery response and make the output really nice also.
_Later this can be done automatically by a planaria node or js library_

```javascript
  let protocolSchema = {
    "19HxigV4QyBv3tHpQVcUEQyq1pzZVdoAut": "B",
    "1PuQa7K62MiKCtssSLKy1kh56WWU7MtUR5": "MAP",
    "15PciHG22SNLQJXMoSUaWVi7WSqc7hCfva": "AUTHOR_IDENTITY"
  }

  let querySchema = {
    "B": [
      {"content": "string"},
      {"content-type": "string"},
      {"encoding": "string"}
    ],
    "MAP": [
      {"cmd": "string"},
      [
        {"key": "string"},
        {"val": "string"}
      ]
    ],
    "AUTHOR": [
      {"algo": "string"},
      {"pubkey": "string"},
      {"sig":"string"}
    ]
  }
```
That should look like a nice transformed response:
```json
{
  "B": [
    {"content": "# Hello small world"},
    {"content-type": "text/markdown"},
    {"encoding": "utf8"}
  ],
  "MAP": [
    {"cmd": "SET"},
    {"key": "url"},
    {"val": "https://twitter.com/"}
  ],
  "AUTHOR": [
    {"algo": "ecdsa"}, 
    {"pubkey": "1HQ8momxTp9MYkzDLy9bFMUQgnba189qZE"}
  ]
}
```

## SET Multiple Keys at once
Keys and values can be repeated to set multiple attributes at once:
```
MAP
SET
<key>
<val>
<key>
<val>
```
more detailed example:
```
1PuQa7K62MiKCtssSLKy1kh56WWU7MtUR5 (MAP)
SET
'app'
'my cool app'
'profile.link'
'http://mywebsite.com'
'profile.name'
'username123'
```

## DELETE: Remove Profile Data
To delete one of the keys->value mappings from the example above.
```
1PuQa7K62MiKCtssSLKy1kh56WWU7MtUR5 (MAP)
'DELETE'
'profile.name'
```

# Concepts
## Keys are Namespaces

Since the keyspace is shared, you can either prefix your keys with a unique identifier, or operate in the global space, sharing that dataset and inheriting the emergent schema. Sharing the global naimspace can be useful when it is intended to be shared among many apps.

  Potential Namespaces for Global Identifiers

    url
    tx
    ethtx
    btctx
    topic
    upc
    infohash
    ifps
    isbn
    md5

Some global identifiers have more than one value...

  *Coordinates*

    coordinates.lat = 
    coordinates.lng =
    coordinates.alt =

  *Phone*

    phone.country_code = 1
    phone.url = 9549549544

  *Profile*

    profile.pubkey = "1HQ..."
    profile.name = "Satchmo"
    profile.text = "Hello small world!"
    profile.image = "b://98bcef1cc43ae..."
    profile.banner = "https://www..."
    

## Keys are Actions
If the above example is namespace as a noun, in this example we show a namespace can be used as a verb too. Actions usually dont need input data. Instead you act upon something that already exists. These begin new op_return chains instead of taking input from a previous protocol. This is useful if all you need is a single key to begin the chain, such as 'like'.

#### Comparison to Memo Protocol for Actions
Please note: These are not intended to limit your imagination with how to organize and manage the keys you use by your app, but to give examples of one way you could do it. There are many viable methods for each of these cases:

Like something by txid:
```
Memo
0x6d04	txhash(32)

MAP
MAP SET 'like' 'true' tx <txhash>
```

Set your profile
```
Memo (multiple txs)
1. Set name
  0x6d01 <name>(217)

2. Set profile text
  0x6d05 <message>(217)

3. Set profile picture
  0x6d0a <url>(217)

MAP
MAP SET 'profile.name' 'Satchmo' 'profile.text' 'Some cool text' 'profile.picture' 'b://986...'
```

Follow / Unfollow users

```
Memo
Follow user	0x6d06	address(35)		
Unfollow user	0x6d07	address(35)		

MAP
MAP SET 'follow.user' <address>
MAP SET 'unfollow.user' <address>
```

Follow / unfollow topic
```
Memo
Topic follow	0x6d0d	<topic_name>(variable)		
Topic unfollow	0x6d0e	<topic_name>(variable)	

MAP
MAP SET 'follow.topic' <topic_name>
MAP SET 'unfollow.topic' <topic_name>

```
## Attach Content
#### Memo Commands with Content

Post
```
  Memo
  0x6d02  <message>(217)	

  MAP
  B <message> <content-type> <encoding> | MAP SET 'type' 'post'
```

Reply to Tx
```
  Memo
  0x6d03  <txhash>(32)  message(184)	

  MAP
  B <message> <content-type> <encoding> | MAP SET 'type' 'reply' 'tx' <txhash> | AUTHOR_IDENTITY
```

Repost
```
  Memo
  0x6d0b  <txhash>  <message>

  MAP
  B <message> <content-type> <encoding> | MAP SET 'type' 'repost' 'tx'  <txhash> 
```

Topic Post
```
  Memo
  0x6d0c  topic_name(variable)  message(214 - topic length)	

  MAP
  B <message> <content-type> <encoding> | 'type' 'post' 'topic' <topic_name>
```

## MAP is Powerful - More Use Cases

Comment on a URL "anonymously"
```
B <message> <content-type> <encoding> | MAP SET 'type' 'comment' 'url' http://google.com
```

Comment with an identity
```
B <message> <content-type> <encoding> | MAP SET 'type' 'comment' 'url' http://google.com | AUTHOR_IDENTITY
```

Attach a picture to a geolocation
```
B <image> <content-type> <encoding> | MAP SET 'coordinates.lat' <latitude> 'coordinates.lng' <latitude> 'coordinates.alt' <altitude>
```

Comment on a phone number
```
B <message> <content-type> <encoding> | MAP SET 'phone.country_code' <country_code> 'phone.number' <phone_number>
```

Comment on a UPC code
```
B <message> <content-type> <encoding> | MAP SET 'type' 'comment' 'upc' <upc_code>
```

# SCRIPT SCHEMA
https://github.com/unwriter/Bitcom/issues/3

We can define the protocol name as well as the field names. v1 of script scema does not support variadic arguments... For now let's assume we have a hypothetical v2 schema that supports variadic arguments like this:

```
{
  "v": 2,
  "s": {
    "out.s1": "{{map}}
    "$repeat": {
      "out.si": "{{key}}",
      "out.si": "{{value}}"
    }
  }
}
```

Then register your schema on chain...

    `OP_RETURN MAP SET 'appname.schema' 'b://txid' | AUTHOR_IDENTITY`

## Querying NameSpaces (coming soon)
Thanks to Script Schema + A custom BitDB Planaria, we can create an api that allows you to query by protocol and field names. It can also pre-process identity signatures to make sure you only index transactions with valid signature identity, so your frontend doesnt need to do the validation work.

`protocol: { field: xxx }`

Query for all url posts from a particular user:
```
{
  v: 1,
  q: {
    find: {
      map: {                          // <- protocol name is auto-mapped by MAP planaria
        key: 'url'
      },
      author: {                       // <- protocol name
        pubkey: <pubkey>
      }
    }
  }
}
```

Find all likes/dislikes mapped to a geolocation
```
{
  q: {
    find: {
      map: {
         $and: [{
          'coordinates.lat': '40.68433',
          'coordinates.lng': '-74.39967'
         },{
          'like': {'$or': ['true', 'false']}
         }]
      }
    }
  }
}
```
