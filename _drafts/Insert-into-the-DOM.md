---
layout: post
title:  Insert into DOM fragments
tags:   ['JavaScript', 'HTML', 'Web API', 'Performance', 'Good practice']
---

**TL;DR** The `DocumentFragment` Web API allows us to prepare DOM nodes without paying the price of repeated HTML parsing and page reflow.
``` javascript
// Don't
$el.innerHTML = ('text/html or text/xml')
// Do
const fragment = document.createRange().createContextualFragment('text/html or text/xml')

// Don't
$el.appendChild($newNode)
// Do
const fragment = document.createDocumentFragment()
fragment.appendChild($newNode)
```

<hr>

A `DocumentFragment` is an object meant to contain a DOM Tree but can't itself be part of the DOM. It is used to create or modify parts of the main DOM without triggering page reflows — the process of recalculating and redrawing parts of a webpage.

Creating a fragment is very simple:
```javascript
const fragment = document.createDocumentFragment()
```

A common use case for a `DocumentFragment` would be when we receive JSON data to hydrate our webpage and have to create a series of nodes accordingly:
```javascript
const json = ['foo', 'bar']
json.forEach(text => {
    const li = document.createElement('li')
    li.textContent = text
    fragment.appendChild(li)
})
document.body.appendChild(fragment)
```
Only the last line of this code snippet will trigger a page reflow. In addition, when passing a `DocumentFragment` as an argument to such functions as `.appendChild()`, it's the fragments children that are inserted instead of the fragment itself.

If instead of receiving JSON data, we get a serialized HTML or XML string, we can directly create a `DocumentFragment` *from* this string.
```javascript
const serialized = '<div>foo</div>'
const fragment = document.createRange().createContextualFragment(serialized)
document.body.appendChild(fragment)
```

And since the `DocumentFragment` isn't in the main DOM tree, you can manipulate it, change classes, remove / add nodes... And all that at very little cost.
