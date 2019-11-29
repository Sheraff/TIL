---
layout: post
title:  Proxy & Reflect
tags:   ['JavaScript', 'Methods']
---

**TL;DR** You can customize all the ways objects are manipulated with `Proxy`, and still use the built in object prototype methods with `Reflect`.
```javascript
const proxy = new Proxy(object, Reflect)
```

<hr>

**What is `Proxy`?**

I already wrote [an article](http://til.florianpellet.com/2019/12/01/Proxy-use-cases/) about what `Proxy` is and what it can do. But here's the most basic example anyway:
```javascript
const object = {a: 1, b: 2}
const handler = {
    get: (obj, key) => key in obj ? obj[key] : 42
}
const proxy = new Proxy(object, handler)
console.log(proxy.a) // 1
console.log(proxy.c) // 42
```
`Proxy` allows you to intercept all of the calls to object prototype methods.

**What does it have to do with `Reflect`?**

`Reflect` is a javascript object that allows you to **call** all of the native object prototype methods that `Proxy` can intercept. *It is the safe way to return from your proxy's traps.*

If you look at our `Proxy` example above, or any of the other example from my [Proxy use cases article](http://til.florianpellet.com/2019/12/01/Proxy-use-cases/), you can see that the fallback behavior is always to run the default Object method. Our `get` falls back to `return obj[key]`, our `has` defaults to `return key in obj`... This is where `Reflect` comes in.

Let's rewrite the `get` handler from the example above, this time using `Reflect`:
```javascript
const handler = {
    get: (obj, key) => {
        if(Reflect.has(obj, key))
            return Reflect.get(obj,key)
        return 42
    }
}
```

If we don't have any conditions on our `get` method, it could further be simplified to
```javascript
handler.get = (obj, key) => Reflect.get(obj, key)
// or even further simplified
handler.get = Reflect.get
```

And a fully useless `Proxy` writes as follows:
``` javascript
const proxy = new Proxy(obj, Reflect)
```
In this case, the proxy does nothing but behaving like the original `obj` object would. Of course, this serves no purpose but showing that `Reflect` contains **all** of the native object prototype methods.

**What's the point exactly?**

Does a `setter` need to return a value? Does `delete` return something ? What does `Object.keys()` return exactly ? Using `Reflect` insures that you properly handle all of the edge cases, and you don't need to worry about what to return!

Take a look at the [MDN doc](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Reflect) to see the full list of methods that `Reflect` offers.