---
layout: post
title:  Thenables and trigger promises
tags:   ['JavaScript', 'Design pattern', 'Promises']
---

**TL;DR** Any object containing a `then()` property — also called a thenable — can be turned into a [promise](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise).
``` javascript
const foo = {
    then: fn => fn(42)
}
Promise.resolve(foo).then(console.log) // 42
```

<hr>

Promises are a powerful construct in JavaScript: they allow us to write asynchronous code in a concise and readable format. 

Imagine you want to calculate some DOM sizes but you need to wait for your components' hydration before you compute DOM sizes because you use proper [Optimistic Rendering](http://til.florianpellet.com/2019/08/02/Optimistic-rendering/):

```javascript
const hydrationPromise = new Promise(resolve => {
    document.addEventListener('hydration', resolve)
})
hydrationPromise.then(() => {
    ... // components are hydrated
})
```

But you might also need to wait for your image to load:

```javascript
const imagePromise = new Promise(resolve => {
    image.addEventListener('load', resolve)
})
Promise.all([
    hydrationPromise,
    imagePromise
]).then(() => {
    ... // components are hydrated AND image was loaded
})
```

As things get more and more complicated, it can start to become tricky when you have multiple objects responding to multiple events at different levels of your code...

```javascript
let hydrationResolve
const hydrationPromise = new Promise(resolve => {
    hydrationResolve = resolve
})

let imageResolve
const imagePromise = new Promise(resolve => {
    imageResolve = resolve
})

Promise.all([
    hydrationPromise,
    imagePromise
]).then(() => {
    ... // components are hydrated AND image was loaded
})

document.addEventListener('hydration', hydrationResolve)
image.addEventListener('load', imageResolve)
```

And that's when you start passing `resolve` functions around. Doesn't that remind you somewhat of the callback hell from back when we didn't have promises? We need to be able to manipulate Promises more cleanly... 

Enter **thenables**: objects with a `then` property whose value is a function will become Promises when used in Promise chain (for example in `Promise.resolve(thenable)`, `Promise.all([thenable])`...) — or with `await`. 

```javascript
const thenable = {
    then: resolve => {
        ...
        resolve(42)
    }
}
Promise.resolve(thenable)
    .then(console.log) // 42
```

This allows us to construct some interesting objects on top of promises, like **trigger promises**. 

``` javascript
class TriggerPromise {
    constructor() {
        this._promise = new Promise(resolve => this.resolve = resolve)
    }

    then(callback) {
        return this._promise.then(callback)
    }
}
```

This class allows us to *encapsulate* in a single object the promise and its `resolve` function. Now we can rewrite our previous example:

```javascript
hydrationTrigger = new TriggerPromise()
imageTrigger = new TriggerPromise()

Promise.all([
    hydrationTrigger,
    imageTrigger
]).then(() => {
    ... // components are hydrated AND image was loaded
})

document.addEventListener('hydration', hydrationTrigger.resolve)
image.addEventListener('load', hydrationTrigger.resolve)
```

