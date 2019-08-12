---
layout: post
title:  CSS “contain” property
tags:   ['CSS', 'performance']
---

**TL;DR** The `contain` CSS property allows you to define an element as a style boundary in order to optimize the browser's *paints*, *layouts*, and *style contexts* calculations.
``` CSS
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
```CSS
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
<html>
    <style>
        .parent {
            background-color: lightslategray;
        }
        .child {
            box-shadow: inset 0 0 0 1px black;
            margin: 10px;
            height: 20px;
            width: 80px;
        }
    </style>
    <div class='parent'>
        <div class='child'></div>
    </div>
    <div class='parent containment'>
        <div class='child'></div>
    </div>
<html>

**Limiting repaints**: 
To observe the repaints in Chrome, in the *developer tools*, click <kbd>⋮ > More tools > Rendering</kbd> and check Paint flashing.
```CSS
.containment {
    contain: paint;
}
.parent:hover .child {
    left: 100px;
}
```

```CSS
.containment {
    contain: size;
}
.parent:hover .child {
    width: 200px;
}
```

```CSS
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

```CSS
.containment {
    contain: layout;
}
.child {
    margin: 10px;
}
```
