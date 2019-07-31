---
layout: post
title:  Idle until urgent
tags:   ['JavaScript', 'Design pattern', 'Web API', 'Performance']
---

Defer calculation of costly values to idle periods, but compute immediately if it is needed before that.
```javascript
let foo = {
    get value() {
        if (this._value)
            return this._value
        cancelIdleCallback(idleHandle)
        return costly()
    }
}
const idleHandle = requestIdleCallback(() => foo._value = costly())
```

<hr>

Some values might be computationally costly to calculate, and might not be needed before some time. These would be a perfect case for this design principle. 

Let's create an artificially long function:
```javascript
function costlyComputation() {
    const now = new Date().getTime()
    do { } while (new Date().getTime() < now + 1000)
    return 42
}

const foo = costlyComputation() // assigning a value to `foo` takes a long time
```

In this example, if `foo` isn't needed right away — maybe it's only used to respond to an event sometime later — then it could be blocking the main thread during critical moments, like at page load and reducing time-to-interactive... To prevent stalling, we could calculate its value only when needed:

```javascript
document.addEventListener('someEvent', () => {
    const foo = costlyComputation()
    ... // responding to the event takes a long time
})
```

But with this solution, responding to the event will be delayed by however much time it take to calculate `foo`. Another solution would be to pre-calculate it whenever the event loop has some free time:

```javascript
let foo
requestIdleCallback(() => {
    foo = costlyComputation()
    document.addEventListener('someEvent', () => {
        ... // starting to listen to the event takes a long time
    })
})
```

But again, this solution poses a problem. We won't even be listening to the event as long as `foo` hasn't been computed. Enters **idle until urgent**.

This concept was described by [Philip Walton](https://philipwalton.com/articles/idle-until-urgent/) in 2018 and follows the release of the [`requestIdleCallback` web API](https://developer.mozilla.org/en-US/docs/Web/API/Window/requestIdleCallback). Basically what you do is give an object a [lazy getter](http://til.florianpellet.com/2019/07/08/Lazy-getter/) for one of its properties, but add in a requestIdleCallback to *pre-compute* the value if the browser ever finds some available time.

```javascript
let foo = {
    idleHandle: requestIdleCallback(() => foo._value = costlyComputation()),
    get value() {
        if (this._value)
            return this._value
        cancelIdleCallback(this.idleHandle)
        return costlyComputation()
    }
}
document.addEventListener('someEvent', () => {
    ... // listening to the event from the get go
})
```

Using this method, we can act as if `foo.value` is available right at the begining. If some time in the event loop was found before `foo` was needed, then the value will have been pre-computed without blocking the thread. Otherwise, it will be calculated on the fly. 

We can even push things a little further and make this into a class:

```javascript
class IdleValue {
  constructor(init: function) {
    this._init = init;
    this._value;
    this._idleHandle = requestIdleCallback(() => {
      this._value = this._init();
    });
  }

  get value() {
    if (this._value === undefined) {
      cancelIdleCallback(this._idleHandle);
      this._value = this._init();
    }
    return this._value;
  }
}

// initialized value but deferred computation
const foo = new IdleValue(costlyComputation)

// obtaining value whether through previous idle callback or direct computation
console.log(foo.value)
```

In more complex implementations, it would be recommended to check the available time for the callback to run by calling `timeRemaining()` on the `IdleDeadline` parameter — and if necessary, chain multiple requests. A full class implementation by [GoogleChromeLabs](https://github.com/GoogleChromeLabs/idlize) is available on Github.