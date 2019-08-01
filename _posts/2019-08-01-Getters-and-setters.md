---
layout: post
title:  Getters & setters
tags:   ['JavaScript', 'Design pattern']
---

**TL;DR** JavaScript allows you to intercept when an object property is accessed or assigned to.
``` javascript
const foo = {
    bar: 42,
    get bar() { return this.bar + 1 }
}
console.log(foo.bar) // 43
```

<hr>

If you want to add more functionnalities to your object, provide an extra layer of abstraction, or add some protection to "*private*" properties, it is possible in javascript to intercept when an object's property is accessed — with a *getter* — or assigned to — with a *setter*.

**Validating property assignments**: you can parse values, format, validate — or whatever else — right at property assignment. 
```javascript
const foo = {
    set name(value) { this.name = value.toLowerCase() }
}

foo.name = 'ArtHur'
console.log(foo.name) // arthur
```

**Adding a new property**: You can create *readers* that allow you to access properties that are resolved from other properties within the object, and that won't occupy memory storage.
```javascript
const foo = {
    first: 'Arthur',
    last: 'Dent',
    get name() { return `${this.first} ${this.last}` }
}
console.log(foo.name) // 'Arthur Dent'
```

**Simulating a *private* property**: As long as JavaScript doesn't implement actual [private fields](https://github.com/tc39/proposal-class-fields#private-fields), the convention has been to prefix *conceptually* private properties with a `_`. You can enforce this with a *setter*.
```javascript
const foo = {
    set bar(value) { this._bar = value }
}

foo.bar = 42
console.log(foo.bar) // undefined
```
Of course, `foo._bar` is still conventionally accessible, but it *should* only be accessed internally with `this._bar`.

**React to property changes**: In the context of a `class` or complex objects, it becomes sensible to add side effects affecting the DOM.
```javascript
const foo = {
    $el: document.querySelector('.switch'),
    set state(bool) {
        this._state = bool
        this.$el.classList[bool ? 'add' : 'remove']('switched-on')
    }
}
```
You could even imagine emmitting a custom event through `.dispatchEvent()` on property changes for a nice touch of event driven application.

*Careful*: it is easy to create an infinite recursion when intercepting access or assignment to the same property. *Getters and setters are also used internally!*
```javascript
const foo = {
    get bar() { return this.bar },
    set bar(value) { this.bar = value }
}
foo.bar = 42         // RangeError: Maximum call stack size exceeded
console.log(foo.bar) // RangeError: Maximum call stack size exceeded
```

