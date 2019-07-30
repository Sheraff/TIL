---
layout: post
title:  "Lazy getter"
excerpt: >
    Compute a value only when needed, but then only once.
snippet: >
    ```
    get bar() {
        const bar = getBar()
        Object.defineProperty(this, 'bar', { value: bar })
        return bar
    }
    ```
---

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
            value: bar
        })

        return bar
    }
} 

console.log(foo.bar)
console.log(foo.bar)
```