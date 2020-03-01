---
layout: post
title:  Generators for idle-until-urgent
tags:   ['JavaScript', 'Design Pattern', 'Performance']
---

**TL;DR** Using a generator function's `yield` we can segment a long running process into small chunks that fit the *Idle Until Urgent* pattern.
``` javascript
const promise = new IdlePromise(function* (resolve, reject) {
    chunkA()
    yield
    chunkB()
    yield
    chunkC()
    resolve()
})
```

<hr>

The [*Idle Until Urgent*](http://til.florianpellet.com/2019/07/15/Idle-until-urgent/) pattern proposes that we run tasks in a [`requestIdleCallback`](https://developer.mozilla.org/en-US/docs/Web/API/Window/requestIdleCallback) ahead of time, and if we end up needing the result of these tasks before they've had the opportunity to run, we `cancelIdleCallback` and run them immediately.

This works well for small tasks, but it breaks when you need to run some computationnaly intensive processes (i.e. too much for a single `requestIdleCallback`) that can't be handed over to [Web Workers](https://developer.mozilla.org/en-US/docs/Web/API/Web_Workers_API) (eg. tasks that need access to some DOM methods). 

To solve this problem, we can combine [`function*`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/function*) and [promises](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise) to create an object that will behave like a `Promise` whose callback is run in small chunks, thanks to the generators' `yield`, through a series of `requestIdleCallback`. I'm calling this bundle of joy an `IdlePromise`.

Let's look at an how to use a generator function to segment a big task into smaller ones:

```javascript
function* bigTask() {
    console.log('run chunkA')
    yield chunkA()
    console.log('run chunkB')
    yield chunkB()
    console.log('run chunkC')
    return chunkC()
}
const iterator = bigTask()
const { value: valueFromA } = iterator.next() // run chunkA
const { value: valueFromB } = iterator.next() // run chunkB
const { value: valueFromC } = iterator.next() // run chunkC
```

Having a series of `yield` points at which the function can be paused and later resumed is a perfect scenario for `requestIdleCallback`: we can run our task bit by bit without ever committing too much time and blocking the main thread.

We can now loop over `iterator` until either the idle callback `IdleDeadline` is over, or `iterator.next()` returns `{done: true}`. This will execute *as much as possible* of our `bigTask` without blocking the main thread.

```javascript
const iterator = bigTask()
requestIdleCallback(deadline => {
    while(deadline.timeRemaining() > 0) {
        const { done } = iterator.next()
        if(done) break
    }
})
```

Now we can wrap our `requestIdleCallback` and call it recursively until we get a `{done: true}` from `iterator.next()`:

```javascript
function run() {
    requestIdleCallback(deadline => {
        while(true) {
            const { done } = iterator.next()
            if(done) break
            if(deadline.timeRemaining() < 0) {
                run()
                break
            }
        }
    })
}
```

One last thing about generators that will be useful to us here is that they can send data at each `yield` and receive some back before resuming execution. For our purpose, we don't need to meddle with what `bigTask` is doing but what we might want to know is "how much time will you need for your next chunk of code?".

```javascript
function* bigTask() {
    // ...
    yield 10 // between this and the next `yield` statement, we need 10ms
    // ..
}
const iterator = bigTask()
function run() {
    requestIdleCallback(deadline => {
        while(true) {
            const { done, value } = iterator.next()
            if(done) break
            if(deadline.timeRemaining() < value) {
                run()
                break
            }
        }
    })
}
```

All that remains now is to wrap this concept into a nice `class` as a ["thenable"](http://til.florianpellet.com/2019/07/31/Thenables-and-trigger-promises/) so that it behaves like a `Promise`. And to add a function to bypass all of the `requestIdleCallback` shenanigans in case this becomes urgent (see [*Idle Until Urgent*](http://til.florianpellet.com/2019/07/15/Idle-until-urgent/) article).

```javascript
class IdlePromise {
    static duration = Symbol('Next yield duration') // semi-private key because messing with this would break stuff
    static padding = 1 // if `yield` doesn't give any information about timing, assume 1ms

    // appropriate all methods from a promise, but store `resolve` and `reject` to be used elsewhere
    promise = new Promise((resolve, reject) => {
        this.resolve = resolve
        this.reject = reject
    })
    then = this.promise.then.bind(this.promise)
    catch = this.promise.catch.bind(this.promise)
    finally = this.promise.finally.bind(this.promise)

	// this construtor can be used exactly like `new Promise()`,
	// the generator will receive `resolve` and `reject`
    constructor(generator) {
        this[IdlePromise.duration] = 0
        this.iterator = generator(this.resolve, this.reject)
        this.run()
    }

    // executing one chunk
    async step() {
        const { value, done } = await this.iterator.next()
        this.done = done
        if (!done) this[IdlePromise.duration] = value || IdlePromise.padding
    }

    // loop asynchronously, with `requestIdleCallback`
    run() {
        this.idleCallbackId = requestIdleCallback(async idleDeadline => {
            while (!this.done && this[IdlePromise.duration] < idleDeadline.timeRemaining())
                await this.step()
            if (!this.done) this.run()
        })
    }

    // cancel current `requestIdleCallback` and run immediately
    async finish() {
        if (this.idleCallbackId) cancelIdleCallback(this.idleCallbackId)
        while (!this.done) await this.step()
        return this.promise
    }
}
```

And it's as easy to use as a regular `Promise`:

```javascript
const idlePromise = new IdlePromise(function* (resolve, reject) {
    // initialization (before first `yield`) should be minimal
    // ...
    yield 10 // next chunk will require less than 10ms
    // ...
    yield 20
    // ...
    yield 5
    // ...
    resolve('all done!')
})
idlePromise.then(console.log) // all done!

// If the result ever becomes urgent, we can revert to synchronous execution
idlePromise.finish()
```