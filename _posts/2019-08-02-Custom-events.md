---
layout: post
title:  Synthetic events
tags:   ['JavaScript', 'Web API', 'Design pattern']
---

**TL;DR** We can create our own event types, listen to and emmit them from anywhere, and implement event driven behaviors.
``` javascript
$el.addEventListener('foo', () => { console.log('received foo event') })
$el.dispatchEvent(new CustomEvent('foo'))
```

<hr>

The `Event` Web API allows us to create custom event types — also called *synthetic events* — with the constructor `CustomEvent()`.
```javascript
const event = new CustomEvent('foo')
```
These events can be dispatched by any `EventTarget`, usually DOM nodes, with `.dispatchEvent()`.
```javascript
$el.dispatchEvent(event)
```
This allows for more modularity, asynchronicity, and atomisation of the code structure. Let's take a look at some examples.

**Event bubbling**: A custom even emmitted by a DOM node can bubble up like any normal event if the option is set in `CustomEvent()`. We can then use a *parent* to catch the events of its children and maybe even centralize the handling logic of the listeners.
```html
<div id='parent'>
    <div id='child'></div>
</div>
```
``` javascript
parent.addEventListener('foo', event => {
    console.log(event.type)
})

child.dispatchEvent(new CustomEvent('foo', { 
    bubbles: true 
}))

// 'foo'
```
This strategy can also be used to emmit "global" events: `window` can both emmit events and be the target of listeners, and is accessible from (almost-)anywhere. Similarly, `document` is a parent of (almost-)every DOM node and can have event listeners attached to it as well.

**Passing data with the event**: When sending an event, we might want to pass along some data.
```javascript
$el.addEventListener('foo', event => {
    console.log(event.detail)
})

$el.dispatchEvent(new CustomEvent('foo', {
    detail: 'bar' 
}))

// 'bar'
```
We can even send objects and they will not be serialized. Meaning that if we pass an object `foo` as `{detail: foo}`, strict equality is preserved (`foo===event.detail`) and changes to one will affect the other.

You can take a look at the [Mozilla doc](https://developer.mozilla.org/en-US/docs/Web/API/Event/Event) for the `EventInit` dictionary to see the list of available options when creating an event with `CustomEvent()`.

**Custom event targets**: we can even create our own event targets with the constructor `EventTarget()` which will both be able to emmit events and be the target of listeners.
``` javascript
const foo = new EventTarget()
foo.addEventListener('bar', () => { console.log('received bar event') })
foo.dispatchEvent(new CustomEvent('bar'))
// 'received bar event'
```