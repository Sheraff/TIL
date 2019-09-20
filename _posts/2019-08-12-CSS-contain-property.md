---
layout: post
title:  "[WIP] CSS “contain” property"
tags:   ['CSS', 'Performance']
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
<p class="codepen" data-height="265" data-theme-id="light" data-default-tab="result" data-user="sheraff" data-slug-hash="RwbVgxd" style="height: 265px; box-sizing: border-box; display: flex; align-items: center; justify-content: center; border: 2px solid; margin: 1em 0; padding: 1em;" data-pen-title="CSS containment - 1 - structure"></p>
<script async src="https://static.codepen.io/assets/embed/ei.js"></script>

**Limiting repaints**: 

```css
.containment {
    contain: paint;
}
.parent:hover .child {
    left: 100px;
}
```
<p class="codepen" data-height="265" data-theme-id="light" data-default-tab="result" data-user="sheraff" data-slug-hash="xxKdLgB" style="height: 265px; box-sizing: border-box; display: flex; align-items: center; justify-content: center; border: 2px solid; margin: 1em 0; padding: 1em;" data-pen-title="CSS containment - 1 - structure"></p>
<script async src="https://static.codepen.io/assets/embed/ei.js"></script>

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
<p class="codepen" data-height="265" data-theme-id="light" data-default-tab="result" data-user="sheraff" data-slug-hash="MWgmvpG" style="height: 265px; box-sizing: border-box; display: flex; align-items: center; justify-content: center; border: 2px solid; margin: 1em 0; padding: 1em;" data-pen-title="CSS containment - 1 - structure"></p>
<script async src="https://static.codepen.io/assets/embed/ei.js"></script>

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
<p class="codepen" data-height="265" data-theme-id="light" data-default-tab="result" data-user="sheraff" data-slug-hash="RwbVZVW" style="height: 265px; box-sizing: border-box; display: flex; align-items: center; justify-content: center; border: 2px solid; margin: 1em 0; padding: 1em;" data-pen-title="CSS containment - 1 - structure"></p>
<script async src="https://static.codepen.io/assets/embed/ei.js"></script>

**Limiting reflows**: [block formatting context](https://developer.mozilla.org/en-US/docs/Web/Guide/CSS/Block_formatting_context) A BFC Is A Mini Layout In Your Layout, and it prevents margins collapsing

```css
.containment {
    contain: layout;
}
.child {
    margin-top: 20px;
}
```
<p class="codepen" data-height="265" data-theme-id="light" data-default-tab="result" data-user="sheraff" data-slug-hash="oNvWeev" style="height: 265px; box-sizing: border-box; display: flex; align-items: center; justify-content: center; border: 2px solid; margin: 1em 0; padding: 1em;" data-pen-title="CSS containment - 1 - structure"></p>
<script async src="https://static.codepen.io/assets/embed/ei.js"></script>

[triggers](https://csstriggers.com/)