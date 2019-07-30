---
layout: post
title:  "Lazy getter"
excerpt: >
    Compute a value only when needed, but then only once.
---

<h2>TL;DR</h2>
```
get bar() {
    const bar = getBar()
    Object.defineProperty(this, 'bar', { value: bar })
    return bar
}
```
{{ post.excerpt }}

We don't need to compute each and every value at object instanciation. In some cases it's better to deffer some of the CPU work for later. A good way to do this is by using **lazy getters**: to compute a value only when needed, but then only once.

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

console.log('first', foo.bar) // 42
console.log('second', foo.bar) // 42
```

Which would output
``` powershell
Computing value...
42
42
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