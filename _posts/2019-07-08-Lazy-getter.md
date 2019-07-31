---
layout: post
title:  Lazy getter
tags:   ['JavaScript', 'Design pattern', 'Performance']
---

**TL;DR** Compute a value only when needed, but then only once.
``` javascript
get bar() {
    const bar = getBar()
    Object.defineProperty(this, 'bar', { value: bar })
    return bar
}
```

<hr>

We don't need to compute each and every value at object instanciation. In some cases it's better to deffer some of the CPU work for later, especially if we're not sure the value is ever going to get used. A good way to do this is by using **lazy getters**: to compute a value only the moment it is needed, but then store it so it doesn't need to be calculated again.

``` javascript
function getBar() { 
    console.log('Computing value...')
    return 42 
}

const foo = {
    get bar() {
        const bar = getBar()

        Object.defineProperty(this, 'bar', {
            value: bar,
            writable: true
        })

        return bar
    }
} 

console.log(foo.bar)
// Computing value...
// 42
console.log(foo.bar)
// 42
```

However, if you want to make sure your lazy value behaves like a normal object property, don't forget to make it writable and define a setter too. Read the doc for more on this: [Object.defineProperty()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/defineProperty).
``` javascript
const foo = {
    get bar() {
        const bar = getBar()

        Object.defineProperty(this, 'bar', {
            value: bar,
            writable: true
        })

        return bar
    },
    set bar(value) {
        Object.defineProperty(this, 'bar', {
            value,
            writable: true
        })
    }
} 
```