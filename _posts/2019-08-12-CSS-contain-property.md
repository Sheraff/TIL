---
layout: post
title:  "[WIP] CSS “contain” property"
tags:   ['CSS', 'Performance']
---

**TL;DR** The `contain` CSS property allows you to define an element as a style boundary in order to optimize the browser's *paints*, *layouts*, *composite*, and *style contexts* calculations.
```css
.el {
    contain: strict;
}
```

<hr>

The `contain` CSS property is somewhat obscure but well supported by Chrome and Firefox and has some powerful performance potential behind it. The [W3C recommendation specs](https://www.w3.org/TR/css-contain-1/) just landed on November 21, 2019.

When you make any changes to your DOM dynamically, either through CSS events (`:hover`, `animation`, ...) or through JavaScript, or even by adding or removing DOM nodes, the browser has to recalculate all or parts of the page to determine how the layout has to *reflow*, what layers it has to *repaint*, how to *composite* the result, and how CSS-generated content might be affected. This can often become a very costly operation and is a good place to look at for fluidity optimizations. And [many CSS properties](https://csstriggers.com/) do cause this!

To demonstrate the different possibilities, let's set up a basic DOM and stylesheet, and have a basic animation:
```html
<div class='parent'>
    <div class='child'>regular</div>
</div>
<div class='parent containment'>
    <div class='child'>contained</div>
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
    position: relative;
    left: 0;
    animation: anim 1s linear infinite alternate;
}
@keyframes anim {
  from { left: 0; }
  to { left: 120px; }
}
```
<p class="codepen" data-height="265" data-theme-id="light" data-default-tab="result" data-user="sheraff" data-slug-hash="RwbVgxd" style="height: 265px; box-sizing: border-box; display: flex; align-items: center; justify-content: center; border: 2px solid; margin: 1em 0; padding: 1em;" data-pen-title="CSS containment - 1 - structure"></p>
<script async src="https://static.codepen.io/assets/embed/ei.js"></script>

**Limiting repaints**: 

The simplest of the values to understand is `paint`: nothing outside the `contain: paint` box can be painted. If the box itself if outside of the viewport, it won't be painted at all. This allows the browser to skip many *paint* steps!

```css
.containment {
    contain: paint;
}
```
<p class="codepen" data-height="265" data-theme-id="light" data-default-tab="result" data-user="sheraff" data-slug-hash="xxKdLgB" style="height: 265px; box-sizing: border-box; display: flex; align-items: center; justify-content: center; border: 2px solid; margin: 1em 0; padding: 1em;" data-pen-title="CSS containment - 1 - structure"></p>
<script async src="https://static.codepen.io/assets/embed/ei.js"></script>

To observe the repaints in Chrome, in the *developer tools*, click <kbd>⋮ > More tools > Rendering</kbd> and check Paint flashing.

**Simplifying layout calculations**: 

Next up is `size`: nothing inside the `contain: size` box can change its dimensions. This is a powerful improvement because it greatly simplifies what needs to be computed during the *layout* phase: you don't need to look at the children to know the parent's size.

```css
.containment {
    contain: size;
}
@keyframes anim {
  from { width: 80px; }
  to { width: 200px; }
}
```
<p class="codepen" data-height="265" data-theme-id="light" data-default-tab="result" data-user="sheraff" data-slug-hash="MWgmvpG" style="height: 265px; box-sizing: border-box; display: flex; align-items: center; justify-content: center; border: 2px solid; margin: 1em 0; padding: 1em;" data-pen-title="CSS containment - 1 - structure"></p>
<script async src="https://static.codepen.io/assets/embed/ei.js"></script>

**Simplifying CSS scopes side-effects**: 

The most obscure of the available values is `style`. It prevents style side-effects from reaching outside of the `contain: style` box. However, up bubbling effects are contrary to the *cascading* principle, and thus rarely used. This property was actually dropped from the latest [W3C recommendation](https://www.w3.org/TR/css-contain-1/).

Here's an example anyway:

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

**Limiting reflows**: 

This last property, `layout`, is probably the most powerful in terms of optimizing reflow calculations. It forces the `contain: layout` box to become an independent BFC ([block formatting context](https://developer.mozilla.org/en-US/docs/Web/Guide/CSS/Block_formatting_context)). A BFC is like a mini layout. Floated elements are contained within the BFC. Margins don't collapse with the outside of the BFC. `absolute` and `fixed` positionning are calculated in reference to the BFC. It creates its own [stacking context](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_Positioning/Understanding_z_index/The_stacking_context)...

```css
.containment {
    contain: layout;
}
@keyframes anim {
  from { margin-top: 0; }
  to { margin-top: 20px; }
}
```
<p class="codepen" data-height="265" data-theme-id="light" data-default-tab="result" data-user="sheraff" data-slug-hash="oNvWeev" style="height: 265px; box-sizing: border-box; display: flex; align-items: center; justify-content: center; border: 2px solid; margin: 1em 0; padding: 1em;" data-pen-title="CSS containment - 1 - structure"></p>
<script async src="https://static.codepen.io/assets/embed/ei.js"></script>