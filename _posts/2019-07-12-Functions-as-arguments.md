---
layout: post
title:  Functions as arguments
tags:   ['JavaScript', 'Anonymous functions', 'Good practices']
---

**TL;DR** You don't need to declare an *anonymous function* to use as an argument.
``` javascript
// Don't
functionCallback(value => console.log(value))
// Do
functionCallback(console.log)
```

<hr>

In many cases, I see developers declaring anonymous functions where they don't need to, especially when used as arguments for other functions. 

For example, `console.log` already *is* a function, and it can take as many argument as you like. The two following functions will behave similarly in most cases:
```javascript
// encapsulating in an anonymous function
const func1 = (value) => console.log(value)

// just using `.log` itself
const func2 = console.log
```

Unless you need the *encapsulation*, you can simplify your code by just passing the function itself:

```javascript
// Don't
somePromise.then(result => myFunction(result))
setTimeout(() => myFunction(), 1000)
window.addEventListener('load', event => init(event))

// Do
somePromise.then(myFunction)
setTimeout(myFunction, 1000)
window.addEventListener('load', init)
```

You just need to be careful with the number of arguments that will be passed:

```javascript
function willCall(callback) {
    callback(42, 'bar')
}

willCall(value => console.log(value)) // 42
willCall(console.log) // 42 'bar'
```