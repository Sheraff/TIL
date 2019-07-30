---
layout: post
title:  "|| and && internal coercion"
---

# || and && internal coercion

Logical operators such as `||` and `&&` do boolean conversions internally, but actually return the value of original operands, even if they are not boolean.

```javascript
// returns number 123, instead of returning true
// 'hello' and 123 are still coerced to boolean internally to calculate the expression
let x = 'hello' && 123;   // x === 123
```

This allows for conditional value setting:
```javascript
const foo = flag && 'bar' // false or 'bar'
```
