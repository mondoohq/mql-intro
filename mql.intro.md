# MQL

An intro

---

## What is MQL?

```json [1-6|1|2-5|1-6]
ports.listening {
  port
  protocol == "tcp"
  process.executable
}
```

```json
ports.listening: [
  0: {
    port: 6463
    protocol == "tcp": true
    process.executable: "Discord"
  }
  1: {
    port: 8000
    protocol == "tcp": true
    process.executable: "gulp serve"
  }
  ...
]
```
<!-- .element: class="fragment" -->


## What is MQL?

<!-- .slide: data-transition="fade" -->

A way to extract data and make assertions.

MQL is supported by Motor + Lumi.
<!-- .element: class="fragment" -->

Note:
- acknowledge taht there are supporting players:
  - Motor to connect to systems and ingest data
  - Lumi to structure resources and make sense of the world


## Why MQL?

1. Easy data query <small>(vs SQL, GraphQL, raw code)</small>
2. Scripting <small>(vs SQL, GraphQL)</small>
3. Simplicity <small> (vs Rego)</small>

Note:
- apart from the reasons listed we also have:
  - concurrent execution
  - event-based updates
  - uniquely identifiable code (dedupe)
  - statically typed (find errors, autocomplete)

---

## MQL 101


## Resources

- structure all the things
- give access to data

```json []
> resource.field
```


#### Resource Example (1/2)

```json [1|3]
> mondoo.version

mondoo.version: "6.3.0+7218"
```


#### Resource Example (2/2)

Sometimes we access related resources

```json [1|3]
> sshd.config.file

sshd.config.file: file id = /etc/ssh/sshd_config
```
<!-- .element: class="fragment" -->

```json [1|3]
> sshd.config.file.path

sshd.config.file.path: "/etc/ssh/sshd_config"
```
<!-- .element: class="fragment" -->


#### Blocks

A way to express context.

```json []
> resource {
  ..
}
```

<small>Borrowed from GraphQL.</small>


#### Block example

```json [1-3|5-7]
> mondoo {
  version
}

mondoo: {
  version: "6.3.0+7218"
}
```


#### Lists

A.k.a. the thing giving blocks superpowers 🦸🏾 

```json [1|2|3]
> [1, 2, ...]
> [0, "one", 2.3]
> packages.list
```


#### List and block example

```json [1|3-99]
> users.list { name uid }

users.list: [
  0: {
    uid: 0
    name: "root"
  }
  1: {
    uid: 1
    name: "bin"
  }
  ...
```


#### Inshe🐑eption

```json [1|2|3-6]
> ports.listening {
  port
  process { 
    executable
    pid
  }
}
```


#### 🐑

```json [|2,8,9,15|3,10|4-7,11-14]
ports.listening: [
  0: {
    port: 6463
    process: {
      executable: "Discord"
      pid: 427839
    }
  }
  1: {
    port: 8000
    process: {
      executable: "gulp serve"
      pid: 535774
    }
  }
]
```


#### What fields are available?

Sometimes we just forget

```json [1|3-99]
> mondoo { * }

mondoo: {
  asset: mondoo.asset id = mondoo.asset
  version: "6.5.0+7301"
  capabilities: [
    0: "run-command"
    1: "file"
  ]
  ...
```


#### Flatten a field

You can use `map` to access and flatten fields

```json [1|3-99]
> users.map( name )

users.map: [
  0: "root"
  1: "bin"
  2: "daemon"
  3: "mail"
  ...
```


#### Assertions

A way to say what you want

```json []
> A == B

> A != B

> A < B

...
```


#### Assertion example (1/2)

```json [1|2]
> sshd.config.params["Protocol"] == "2"
[ok] value: "2"
```


#### Assertion example (2/2)

```json [1|2-99]
> sshd.config.params["Protocol"] == "2"
[failed] sshd.config.params[Protocol] == "1"
  expected: == "2"
  actual:   "1"
```


#### Soft assertion (1/2)

<small>Types are often annoying in simple assertions.</small>

<small>Be expressive and simple:</small>

```json [1|2]
> sshd.config.params["Protocol"] == 2
[ok] value: "2"
```


#### Soft assertion (2/2)

```json [1|3|5|7]
  2 == 2

  2 == "2"

  2.0 == "2"

  [2] == 2
```


#### Regex assertion (1/3)

<small>There are a ton of useful regular expressions for comparisons</small>

```json
regex {
  ipv4() regex
  ipv6() regex
  url() regex
  email() regex
  mac() regex
  uuid() regex
  emoji() regex
  semver() regex
  creditCard() regex
}
```
<!-- .element: class="fragment" -->


#### Regex assertion (2/3)

<small>Some of these are complex</small>

```json [1|2-99]
> regex.semver
regex.semver: /(0|[1-9]\d*)\.(0|[1-9]\d*)\.
  (0|[1-9]\d*)(?:-((?:0|[1-9]\d*|\d*[a-zA-Z-]
  [0-9a-zA-Z-]*)(?:\.(?:0|[1-9]\d*|\d*[a-zA-Z-]
  [0-9a-zA-Z-]*))*))?(?:\+([0-9a-zA-Z-]+
  (?:\.[0-9a-zA-Z-]+)*))?/
```
<!-- .element: class="fragment" -->


#### Regex assertion (3/3)

<small>They are very easy to use</small>

```json [1|2]
> mondoo.version == regex.semver
[ok] value: "6.3.0+7218"
```
<!-- .element: class="fragment" -->


#### List assertions (1/3)

Some assertions don't use comparison operators

```json [1|2-99]
> users.all( uid != 0 )
[failed] users.all()
  actual:   [
    0: user id = user/0/root
  ]
```
<!-- .element: class="fragment" -->


#### List assertions (2/3)

The opposite of `all` is `none`

```json [1|2-99]
> users.none( uid -= 0 )
[failed] users.none()
  actual:   [
    0: user id = user/0/root
  ]
```


#### List assertions (3/3)

There is also `one`

```json [1|2-99]
> users.one( uid >= 1000)
[failed] users.one()
  actual:   [
    0: user id = user/65534/nobody
    1: user id = user/1000/zero
  ]
```

... and `any`
<!-- .element: class="fragment" -->

```json [1|2-99]
> users.any( uid >= 1000)
[ok] value: true
```
<!-- .element: class="fragment" -->



#### Why list assertions?

```json [1|1-99]
> users.list { uid != 0 }
users.list: [
  0: {
    uid != 0: false
  }
  1: {
    uid != 0: true
  }
  2: {
    uid != 0: true
  }
```

vs
<!-- .element: class="fragment" -->

```json []
> users.all( uid != 0 )
[failed] users.all()
  actual:   [
    0: user id = user/0/root
  ]
```
<!-- .element: class="fragment" -->



---

## That's it!

<iframe src="https://giphy.com/embed/xUPGcEghH2dZdXvZSw" width="480" height="270" frameBorder="0" class="giphy-embed" allowFullScreen></iframe><p><a style="display:none" href="https://giphy.com/gifs/schittscreek-schittscreek-eugenelevy-funny-tv-comedy-pop-xUPGcEghH2dZdXvZSw">via GIPHY</a></p>

---

## Upcoming stuff

- [asset relationships](https://www.notion.so/mondoo/Asset-Clustering-Relationships-308ef433bb7b45a2960679132ed5d301)
- [useful data defaults](https://www.notion.so/mondoo/Useful-data-defaults-for-MQL-resources-2ac2684203534aa89d39e8a4c7640481)
- [query child resources](https://www.notion.so/mondoo/Query-all-child-resources-c4a156fe08da4631b408b37ca863f6ed)


## Moar Resources 
<!-- .element style="color:#3e2121; background: #fdf4e0;" -->
<!-- .slide: data-background-image="pokemon.png" style="color:black" -->
<!-- image is by: https://dribbble.com/shots/15925733-Snorlax/attachments/7757057?mode=media -->
