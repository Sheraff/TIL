---
layout: post
title:  Proxy use cases
tags:   ['JavaScript', 'Methods']
---
`Proxy` allows you to intercept all object prototype methods.
**TL;DR** 
``` javascript
new Proxy(object, {
    get: (obj, key) => { return 42 }
}) // the answer to everything is 42
```

<hr>

**What is `Proxy`?**

`Proxy` is a javascript class that allows you to intercept any type of call made to an object and respond in with your own methods instead of having the object respond with its native prototype methods.

The simplest example to understand is to use a proxy to provide a default value:
```javascript
const object = {a: 1, b: 2}
const proxy = new Proxy(object, {
    get: (obj, key) => key in obj ? obj[key] : 42
})
console.log(proxy.a) // 1
console.log(proxy.c) // 42
```
There are many more methods than just `get` that `Proxy` can intercept, take a look at the [MDN docs](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Proxy) to learn more.

Another classic is to add *proxied keys* that return values not stored in the object but computed from its values:
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
Though `Object.defineProperty` would be a better tool for this job because it targets a specific property instead of the prototype method itself:
```javascript
Object.defineProperty(obj, 'fullName', {
    get() { return `${obj.firstName} ${obj.lastName}` }
})
```

However, proxies are much more powerful than `Object.defineProperty` because they allow you to define how *all* `get` calls are handled and not just how it is handled for one *specific* key.

<hr>

**Real world example #1: fallback dictionnary**

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
        if(key in obj)
            return obj[key]
        else
            return defaultMessages[key]
    }
})
console.log(proxiedMessages.warn) // we recommend you do things differently
console.log(proxiedMessages.error) // you definitely made a mistake there pal
```

<hr>

**Real world example #2: private keys**

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

<hr>

**Real world example #3: temporary state**

Imagine you are programming a game in which a player can be affected temporarily with different conditions. We can make a proxied object that will automatically return an up to date state when accessed!
```javascript
const playerCondition = new Proxy({}, {
    set: (obj, key, value) => {
        obj['_'+key] = Date.now() + 5000
        obj[key] = value
    },
    get: (obj, key) => {
        if(obj['_'+key] > Date.now())
            return obj[key]
        return undefined
    }
})

playerCondition.paralysed = true
console.log(playerCondition.paralysed) // true
// wait for 5 seconds
console.log(playerCondition.paralysed) // undefined
```

<hr>

**Real world example #4: memoization of API calls**

If you want to make all of your calls to some data the same way, whether the data has never been fetched before or its already loaded, you can create a proxy to this data object to dispatch the calls for you.
```javascript
const userDataStore = {
    1: {name: 'Maria'},
    2: {name: 'Morty'},
}
const handler = {
    get: (obj, key) => (
        key in obj
        ? Promise.resolve(obj[key])
        : fetch('site.com/api/users/'+key)
            .then(async response => {
                const data = await response.json()
                obj[key] = data
                return data
            })
    )
}
const userApiCalls = new Proxy(userDataStore, handler)

userApiCalls[2].then(console.log) // {name: 'Morty'}
userApiCalls[9].then(console.log) // {name: 'Chuck'}
```

<hr>

**Real world example #5: expiration time on memoization**

Combining the two previous examples, you can have the proxy fetch and memoize data for you, and set an expiry date on the data!
```javascript
const apiCalls = new Proxy({}, {
    get: (obj, key) => {
        if(obj['_'+key] > Date.now())
            return Promise.resolve(obj[key])
        return fetch('site.com/api/'+key)
            .then(async response => {
                const data = await response.json()
                obj['_'+key] = Date.now() + 60000 // in 1 minute
                obj[key] = data
                return data
            })
    }
})
```

<hr>

**Real world example #6: event based state**

If you want to monitor the changes made to an object, you can use a `Proxy` to dispatch an event every time such a change occurs. This is useful in an event based architecture if you want to update your DOM based on a JSON object for example.
```javascript
const target = new EventTarget()
const state = new Proxy(target, {
    set: (obj, key, value) => {
        Reflect.set(obj, key, value)
        obj.dispatchEvent(new CustomEvent('change', {detail: {key, value}}))
    },
    deleteProperty: (obj, key) => {
        Reflect.deleteProperty(obj, key)
        obj.dispatchEvent(new CustomEvent('change', {detail: {key, value: undefined}}))
    }
})
target.addEventListener('change', e => console.log(e.detail))
state.foo = 'bar'
// {key: "foo", value: "bar"}
delete state.foo
// {key: "foo", value: undefined}
```

PS: if the use of `Reflect` is confusing to you, I wrote [an article](http://til.florianpellet.com/2019/12/02/Proxy-and-Reflect/) about how it relates to `Proxy`!