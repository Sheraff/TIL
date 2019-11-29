---
layout: post
title:  Proxy & Reflect
tags:   ['JavaScript', 'Methods']
---
You can customize all the ways objects are manipulated with `Proxy`, and still use the built in object prototype methods with `Reflect`.
**TL;DR** 
``` javascript
const proxy = new Proxy(object, Reflect)
```

<hr>

**What is `Proxy`?**

`Proxy` is a javascript class that allows you to intercept any type of call made to an object and respond in with your own methods instead of having the object respond with its native prototype methods.

For example, you can add *proxied keys* that return values not stored in the object but computed from its values:
```javascript
const handler = {
    get: (obj, key) => {
        if(key !== 'fullName')
            return obj[key]
        if(obj.firstName && obj.lastName)
            return `${obj.firstName} ${obj.lastName}`
        else 
            throw new ReferenceError(`Cannot compute ${key} from existing keys`)
    }
}
```
This example would be equivalent to what could be done with `Object.defineProperty`:
```javascript
Object.defineProperty(obj, 'fullName', {
    get() { return `${obj.firstName} ${obj.lastName}` }
})
```

However, proxies are much more powerful than `Object.defineProperty` because they allow you to define how *all* `get` calls are handled and not just how it is handled for one *specific* key.

For example, we can use it to define a fallback dictionnary:
```javascript
const defaultMessages = {
    warn: 'pretty sure you should not be doing this',
    error: 'you definitely made a mistake there pal'
}
const politeMessages = {
    warn: 'we recommend you do things differently'
}
const proxiedMessages = new Proxy(politeMessages, {
    get: (obj, key) => {
        if(obj[key])
            return obj[key]
        else
            return defaultMessages[key]
    }
})
console.log(proxiedMessages.warn) // we recommend you do things differently
console.log(proxiedMessages.error) // you definitely made a mistake there pal
```

From there, it's just up to your imagination! How about "private" keys:
```javascript
const handler = {
    get: (obj, key) => {
        if(key.startsWith('_'))
            throw new ReferenceError(`Key ${key} is private`)
        return obj[key]
    },
    set: (obj, key, val) => {
        if(key.startsWith('_'))
            throw new ReferenceError(`Key ${key} is private`)
        obj[key] = val
    }
}
const proxy = new Proxy(obj, handler)
```

And by the way, `Proxy` allows you to intercept **all** of the object's methods. And if you wanted to push your "private" keys further, you could add the `ownKeys`, `has` and `deleteProperty` methods to your handler:
```javascript
handler.ownKeys = (obj) => {
    return Object.keys(obj).filter(key => !key.startsWith('_'))
}
handler.has = (obj, key) => {
    if(key.startsWith('_'))
        return false
    return key in obj
}
handler.deleteProperty = (obj, key) => {
    if(key.startsWith('_'))
        throw new ReferenceError(`Key ${key} is private`)
    delete obj[key]
}
```
And now, even `Object.keys` or `Object.getOwnPropertyNames` could not reach your object's private keys!

There are many more methods that `Proxy` can intercept, take a look at the [MDN docs](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Proxy) to learn more.

**What does it have to do with `Reflect`?**

`Reflect` is a javascript object that allows you to use **all** of the native object prototype methods. 

If you look at all of our `Proxy` examples above, you can see that the fallback case is always to run the default Object method. Our `get` falls back to `return obj[key]`, our `has` defaults to `return key in obj`... This is where `Reflect` comes in.

Let's rewrite the `get` handler from our "private" keys example:
```javascript
const proxy = new Proxy(obj, {
    get: (obj, key) => {
        if(key.startsWith('_'))
            throw new ReferenceError(`Key ${key} is private`)
        return Reflect.get(obj,key)
    }
})
```

If we didn't have any conditions on our `get` method, it could further be simplified to
```javascript
handler.get = (obj, key) => Reflect.get(obj, key)
// or even further simplified
handler.get = Reflect.get
```

And a fully useless `Proxy` writes as follow:
``` javascript
const proxy = new Proxy(obj, Reflect)
```
In this case, the proxy does nothing but behave like the original `obj` object would. Of course, this serves no purpose but showing that `Reflect` contains **all** of the native object prototype methods.

Take a look at the [MDN doc](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Reflect) to see the full list of methods that `Reflect` offers.