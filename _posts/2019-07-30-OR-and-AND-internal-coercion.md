---
layout: post
title:  "Logical operators internal coercion"
tags:   ['JS', 'Primitives']
---

**TL;DR**Logical operators `||` and `&&` can return non boolean values after internal boolean operations have resolved.
``` javascript
const foo = true && 'foo' // 'foo'
const bar = false || 'bar' // 'bar'
```

<hr>

Thanks to javascript's infamous **type coercion**, logical operators `||` and `&&` do boolean conversions internally, but actually return the value of original operands, *even if they are not boolean*.

```javascript
const foo = 'bar' && 42 // 42
```
Here a string is always coerced to `true` as long as it's not the empty string `''`. So the second expression is returned.

| operator | coercion logic |
| --- | --- |
| `foo && bar` | If `foo` can be coerced to true, returns `bar`; else, returns `foo`. |
| `foo || bar` | If `foo` can be coerced to true, returns `foo`; else, returns `bar`. |


This allows for conditional value setting without using an `if` block
```javascript
const foo = flag && 'bar' // false or 'bar'
```
Or optional setting for default values
```javascript
const foo = option || 'bar' // option or 'bar'
```