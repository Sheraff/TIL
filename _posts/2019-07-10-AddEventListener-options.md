---
layout: post
title:  AddEventListener third argument
tags:   ['Javascript', 'Methods', 'Good practices', 'Performance']
---

**TL;DR** We don't often see addEventListener() be used with a third argument, but it allows for great preformance improvements and cleaner code.

``` javascript
window.addEventListener('scroll', () => {}, {passive: true})
$el.addEventListener('click', () => {}, {once: true})
```

<hr>

With sites requiring more and more reactivity, complex interactions and responsivity, we tend to multiply event listeners in ways that can end up having a big impact on performance.

Thankfully, there are simple options to remedy the most common problems: `addEventListener` accepts an options object as its third parameter. 

**Freeing the main thread**: under normal circumstances, when responding to an event, the browser has to block its main thread — sometimes thought of as the *render thread* — until it knows how the listener will impact the event. The main example of this is for a `scroll` event (or `touchstart` and `touchmove`): it is impossible for the browser to know in advance whether the event will be cancelled through `.preventDefault()` and thus it halts scrolling until the listener has resolved. This results in a *slow and jumpy scroll*. 

Using the option `{passive: true}` you can deactivate `.preventDefault()` from having any effect in the associated listener, insuring the browser can still resolve the scroll position as soon as it needs to make the frame rate deadline. This results in a *much smoother scroll feel*.

```javascript
window.addEventListener('scroll', event => {
    event.preventDefault() // will not have any effect
}, {passive: true})
```

**Auto-removing listeners**: most listeners attached to the DOM with `addEventListener` should have a corresponding `removeEventListener` to remove them when they're not needed anymore. Accumulating event listeners will result in a slower [event loop](https://developer.mozilla.org/en-US/docs/Web/JavaScript/EventLoop). But there are many cases where the event will be called only once and it seems like a lot of code, de-anonymizing the function and calling `removeEventListener`, for a simple use-case.

Using the option `{once: true}` you tell the browser that the listener will be invoked at most once. It will then automatically remove it for you after it is first called.

```javascript
$element.addEventListener('click', event => acceptCookies(), {once: true})
```

Take a look at the [Mozilla doc](https://developer.mozilla.org/en-US/docs/Web/API/EventTarget/addEventListener) to take a look at the other properties you can set in the `options` argument.