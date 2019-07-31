---
layout: post
title:  Idle until urgent
tags:   ['JavaScript', 'Design pattern', 'Web API']
---

Strategy for computationally intensive values consisting of deferring computation to idle periods but then run immediately as soon as it’s needed.

This concept was described by [Philip Walton](https://philipwalton.com/articles/idle-until-urgent/) in 2018 and follows the release of the [`requestIdleCallback` web API](https://developer.mozilla.org/en-US/docs/Web/API/Window/requestIdleCallback).

Basically what you do is give an object a [lazy getter](http://til.florianpellet.com/2019/07/08/Lazy-getter/) for one of its properties, but add in a requestIdleCallback to *pre-compute* the value if the browser ever finds some vailable time.

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
const foo = new IdleValue(() => { return 5 })

// obtaining value whether through previous idle callback or direct computation
console.log(foo.value)
```

In more complex implementations, it would be recommended to check the available time for the callback to run by calling `timeRemaining()` on the `IdleDeadline` parameter — and if necessary, chain multiple requests.