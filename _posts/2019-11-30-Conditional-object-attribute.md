---
layout: post
title:  Conditional object attribute
tags:   ['JavaScript']
---

**TL;DR** Just a simple way to conditionnaly add a key-value pair *at object declaration*.
``` javascript
const obj = {
    ...(flag && {key: value})
}
```

<hr>

What I tend to do if some keys of an object should only conditionnaly be declared is to set them after the object has been instanciated:
```javascript
const meal = {
    carb: 'pasta'
}
if(withSauce)
    meal.sauce = 'pesto'
```

But there is a more concise way of doing this that doesn't require multiple steps:
```javascript
const meal = {
    carb: 'pasta',
    ...(withSauce && {sauce: 'pesto'})
}
```

PS: if you are confused by the use of the boolean operator in this last example, I wrote a post about how [logical operators coerce internally](http://til.florianpellet.com/2019/07/30/OR-and-AND-internal-coercion/).