---
layout: post
title:  Object assign
tags:   ['Javascript', 'Methods']
---

**TL;DR** `Object.assign()` is a method that allows you to easily merge, override, clone, default object properties.
``` javascript
const foo = {foo: 'foo'}
Object.assign(foo, {bar: 'bar'})
console.log(foo.bar) // 'bar'
```

<hr>

This cool little function has so many use cases.

**Merging**: the first argument of `Object.assign()` will receive all the keys from the objects passed as the following arguments. Only the first argument is mutated.
```javascript
const foo = {foo: 'foo'}
const bar = {bar: 'bar'}
Object.assign(foo, bar)
console.log(foo) // { foo: 'foo', bar: 'bar' }
```

**Cloning**: assigning to an empty object `{}` will make `Object.assign()` return a deep clone.
```javascript
const foo = {foo: 'foo'}
const clone = Object.assign({}, foo)
console.log(foo)           // { foo: 'foo' }
console.log(clone)         // { foo: 'foo' }
console.log(foo === clone) // false
```

**Overriding**: using a constant `important`, we can use `Object.assign()` to make sure the passed object has at all the keys from `important` set to `important`'s values.
```javascript
const important = {foo: 'foo'}
const foo = {foo: 'bar'}
Object.assign(foo, important)
console.log(foo) // { foo: 'foo' }
```

**Defaults**: using a constant `default`, we can use `Object.assign()` to make sure an object has at least all the keys from `default` (and if it doesn't, they'll be set to `default`'s values).
```javascript
const defaults = {foo: 'foo'}
const foo = {bar: 'bar'}
Object.assign(foo, defaults, foo)
console.log(foo) // { foo: 'foo', bar: 'bar' }
```

Take a look at the [Mozilla doc](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/assign) for more on this.