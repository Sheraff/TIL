---
layout: post
title:  Logical operators coerce internally
tags:   ['JavaScript', 'Primitives']
---

**TL;DR** Logical operators `||` and `&&` can return non boolean values after internal boolean operations have resolved. Type coercion is only *internal*.
``` javascript
const foo = true && 'foo' // 'foo'
const bar = false || 'bar' // 'bar'
```

<hr>

Thanks to javascript's infamous **type coercion**, logical operators `||` and `&&` do boolean conversions internally, but actually return the value of original operands, *even if they are not boolean*.

```javascript
const foo = 'bar' && 42 // 42
```
For example, here a string is coerced to `true` as long as it's not the empty string `''`. So the second expression is returned.

| operator | coercion logic |
| --- | --- |
| `foo && bar` | if `foo` can be coerced to true, return `bar`<br>else return `foo` |
| `foo || bar` | if `foo` can be coerced to true, return `foo`<br>else return `bar` |

The list of *falsy* values, values that can be coerced to `false`, is short:
- `null`
- `NaN`
- `0`
- `''`
- `undefined`

We can also use it for optional parameters with fallback default values
```javascript
// basic
let foo = 'bar'
if (option) {
    foo = option
}

// ternary
const foo = option ? option : 'bar'

// logical operation
const foo = option || 'bar'
```

This also allows for conditional value setting without using an `if` block:
```javascript
let foo
if (flag) {
    foo = 'bar'
}
```
now becomes
```javascript
const foo = flag && 'bar'
```

*Careful*: in this case, both implementations aren't exactly equivalent in the case where `flag` is *falsy*, as the value of `foo` will be `undefined` in the first case and equal to `flag` in the second.