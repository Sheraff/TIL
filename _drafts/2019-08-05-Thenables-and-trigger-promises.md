---
layout: post
title:  "Thenables and trigger promises"
---

Any object containing a `then` property — also called a thenable — can be handled like a Promise.
``` javascript
const foo = {
    then: fn => fn(42)
}
Promise.all([foo]).then(console.log) // [ 42 ]
```

<hr>

This allows us to construct some interesting objects like **trigger promises**. Imagine you want to create images from DOM content that needs both to wait for your preferred font to have loaded and for content hydration. 

YOU COULD DO THAT THE OLD WAY
``` javascript
```

OR YOU COULD DO THAT WITH TRIGGER PROMISES

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