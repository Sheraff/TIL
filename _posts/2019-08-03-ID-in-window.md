---
layout: post
title:  '#id is in window'
tags:   ['JavaScript', 'HTML', 'Good practice']
---

**TL;DR** DOM node IDs are directly available in the `window` object.
```html
<div id='foo'></div>
```
``` javascript
console.log(window.foo) // HTMLElement
```
Though, you shouldn't access the DOM that way.

<hr>

Unless it conflicts with other attributes of the `window` object, all DOM nodes with a specified `id` are directly accessible as properties of the `window` object.

```html
<div id='foo'></div>
<div id='Date'></div>
```
``` javascript
console.log(window.foo)  // HTMLElement
console.log(foo)         // HTMLElement
console.log(window.Date) // ƒ Date()
```

*Careful*: it isn't recommended to access the DOM this way as it will easily lead to hard to find errors and could conflict with native APIs. You should try and use `getElementById()` instead. But still, knowing this is useful for rapid prototyping and dirty code.

```javascript
console.log(document.getElementById('Date')) // HTMLElement
```