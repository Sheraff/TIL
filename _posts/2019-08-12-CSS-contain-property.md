---
layout: post
title:  "[TEST] CSS “contain” property"
tags:   ['CSS', 'performance']
---

**TL;DR** The `contain` CSS property allows you to define an element as a style boundary in order to optimize the browser's *paints*, *layouts*, and *style contexts* calculations.
```css
.el {
    contain: strict;
}
```

<hr>

The `contain` CSS property is somewhat obscure but still supported well by Chrome and Firefox and has some powerful performance potential behind it.

When you make any changes to your DOM dynamically, either through CSS events (`:hover`, `animation`, ...) or through JavaScript, the browser has to recalculate all or parts of the page to determine how the layout has to reflow, what layers it has to repaint, and how CSS-generated content might be affected. This can often become a very costly operation and is a good place to look at for fluidity optimizations.

To demonstrate the different possibilities, let's set up a basic DOM and stylesheet:
```html
<div class='parent'>
    <div class='child'></div>
</div>
<div class='parent containment'>
    <div class='child'></div>
</div>
```
```css
.parent {
    background-color: lightslategray;
}
.child {
    box-shadow: inset 0 0 0 1px black;
    margin: 10px;
    height: 20px;
    width: 80px;
}
```
<pre class='demo setup'>
    <style>
        .demo .parent {
            background-color: lightslategray;
            transition: all 1s;
        }
        .demo .child {
            box-shadow: inset 0 0 0 1px black;
            margin: 10px;
            height: 20px;
            width: 80px;
            transition: all 1s;
        }
        pre.demo {
            display: flex;
            flex-direction: column;
            align-items: left;
            /* justify-items: baseline; */
        }

        .parent:first-of-type {
            margin-bottom: 1rem;
        }
    </style>
    <div class='parent'>
        <div class='child'></div>
    </div>
    <div class='parent containment'>
        <div class='child'></div>
    </div>
</pre>

**Limiting repaints**: 

```css
.containment {
    contain: paint;
}
.parent:hover .child {
    left: 100px;
}
```
<pre class='demo paint'>
    <style>
        .demo.paint .containment {
            contain: paint;
        }
        .demo.paint .child {
            position: relative;
            left: 0;
        }
        .demo.paint .parent:hover .child {
            left: 100px;
        }
    </style>
    <div class='parent'>
        <div class='child'></div>
    </div>
    <div class='parent containment'>
        <div class='child'></div>
    </div>
</pre>

To observe the repaints in Chrome, in the *developer tools*, click <kbd>⋮ > More tools > Rendering</kbd> and check Paint flashing.

**Simplifying layout calculations**: 

```css
.containment {
    contain: size;
}
.parent:hover .child {
    width: 200px;
}
```
<pre class='demo size'>
    <style>
        .demo.size .containment {
            contain: size;
        }
        .demo.size .parent {
            height: 3rem;
        }
        .demo.size .parent:hover .child {
            width: 200px;
        }
    </style>
    <div class='parent'>
        <div class='child'></div>
    </div>
    <div class='parent containment'>
        <div class='child'></div>
    </div>
</pre>

**Simplifying CSS scopes side-effects**: 

```css
.containment {
    contain: style;
}
.parent {
    counter-reset: i;
}
.parent::after, .child::after {
    content: counter(i);
    counter-increment: i;
}
```
<pre class='demo style'>
    <style>
        .demo.style .containment {
            contain: style;
        }
        .demo.style .parent {
            counter-reset: i;
        }
        .demo.style .parent::after, .demo.style .child::after {
            content: counter(i);
            counter-increment: i;
        }
    </style>
    <div class='parent'>
        <div class='child'></div>
    </div>
    <div class='parent containment'>
        <div class='child'></div>
    </div>
</pre>

**Limiting reflows**: 

```css
.containment {
    contain: layout;
}
.child {
    margin: 10px;
}
```
<pre class='demo layout'>
    <style>
        .demo.layout .containment {
            contain: layout;
        }
        .demo.layout .child {
            margin: 10px;
        }
    </style>
    <div class='parent'>
        <div class='child'></div>
    </div>
    <div class='parent containment'>
        <div class='child'></div>
    </div>
</pre>

[triggers](https://csstriggers.com/)
