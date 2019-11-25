https://aerotwist.com/blog/flip-your-animations/

FLIP stands for First, Last, Invert, Play.

Let’s break it down:

First: the initial state of the element(s) involved in the transition.
Last: the final state of the element(s).
Invert: here’s the fun bit. You figure out from the first and last how the element has changed, so – say – its width, height, opacity. Next you apply transforms and opacity changes to reverse, or invert, them. If the element has moved 90px down between First and Last, you would apply a transform of -90px in Y. This makes the elements appear as though they’re still in the First position but, crucially, they’re not.
Play: switch on transitions for any of the properties you changed, and then remove the inversion changes. Because the element or elements are in their final position removing the transforms and opacities will ease them from their faux First position, out to the Last position.

```javascript
// Get the first position.
var first = el.getBoundingClientRect();

// Now set the element to the last position.
el.classList.add('totes-at-the-end');

// Read again. This forces a sync
// layout, so be careful.
var last = el.getBoundingClientRect();

// You can do this for other computed
// styles as well, if needed. Just be
// sure to stick to compositor-only
// props like transform and opacity
// where possible.
var invert = first.top - last.top;

// Invert.
el.style.transform =
    `translateY(${invert}px)`;

// Wait for the next frame so we
// know all the style changes have
// taken hold.
requestAnimationFrame(function() {

  // Switch on animations.
  el.classList.add('animate-on-transforms');

  // GO GO GOOOOOO!
  el.style.transform = '';
});

// Capture the end with transitionend
el.addEventListener('transitionend',
    tidyUpAnimations);
```


```javascript
// Get the first position.
var first = el.getBoundingClientRect();

// Move it to the end.
el.classList.add('totes-at-the-end');

// Get the last position.
var last = el.getBoundingClientRect();

// Invert.
var invert = first.top - last.top;

// Go from the inverted position to last.
var player = el.animate([
  { transform: `translateY(${invert}px)` },
  { transform: 'translateY(0)' }
], {
  duration: 300,
  easing: 'cubic-bezier(0,0,0.32,1)',
});

// Do any tidy up at the end
// of the animation.
player.addEventListener('finish',
    tidyUpAnimations);
```