## topics
- [ ] chrome dev tools
  - [ ] Live Expression in Console tab allows you to write some JS and see its output live as you use the page
- [ ] SVG anim
- [ ] react
  - [ ] ref as store across renders
  - [ ] lifecycle functions have a callback as second argument
- [ ] Web APIs 
  - [ ] IntersectionObserver
  - [ ] IndexDB
  - [ ] Service Worler
- [ ] CSS attribute `contain: strict;` restricts layout / paint calculations (https://developers.google.com/web/updates/2016/06/css-containment)
- [ ] PNGs to GIF client-side using WebAssembly ImageMagick rewrite (https://github.com/KnicKnic/WASM-ImageMagick)
- [ ] preload
- [ ] css grid 
  - [ ] fluid width
  - [ ] square cells hack
- [ ] V8 TurboFan bailout reasons: https://github.com/petkaantonov/bluebird/wiki/Optimization-killers
- [ ] layout thrashing: https://gist.github.com/paulirish/5d52fb081b3570c81e3a
- [ ] closures
- [ ] Object.defineProperty
- [ ] Page lifecycle API: https://developers.google.com/web/updates/2018/07/page-lifecycle-api
- [ ] navigator.connection.saveData && navigator.connection.effectiveType
- [ ] networkIdleCallback ~= boredom-loading (in contrast w/ lazy-loading) (https://github.com/pastelsky/network-idle-callback): basically places a ServiceWorker to intercept all `.fetch()` and add a debounce cooldown period to trigger idleCallback (another way of doing this involves also debouncing on other user events like mouse movements, scrolling... to prevent loading smth when the user might trigger the loading of smth else)
- [ ] Block Formatting Contexts (https://www.smashingmagazine.com//2017/12/understanding-css-layout-block-formatting-context/)
- [ ] Static vs Live NodeList (https://www.stefanjudis.com/blog/accessing-the-dom-is-not-equal-accessing-the-dom/)
- [ ] CSS `overscroll-behavior: contain;` to prevent a scrollable element to scroll the parent when reaching a boundary
- [ ] CSS `backdrop-filter: blur(10px);` (https://codepen.io/chriscoyier/pen/GRKqQBo)
- [ ] Memoization (https://nick.scialli.me/an-introduction-to-memoization-in-javascript/) (=== lazy getter?)
- [ ] Proxies https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Proxy
```javascript
// objects w/ default property values
var handler = {
    get: function(obj, prop) {
        return prop in obj ?
            obj[prop] :
            37;
    }
};

var p = new Proxy({}, handler);
p.a = 1;
p.b = undefined;
```
- [ ] Reflect (http://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Reflect) allows access to Object definition methods (`Reflect.set(object, property, value)`). Especially useful from within Proxies or Object.defineProperty()
    > We can use the Reflect module to call the original function that would have ran if we haven't proxied the object. We could use obj[prop] too, like in the previous example, but this is cleaner, and it uses the same implementation a non-proxied object would use. This maybe doesn't sound important in a simple trap like get, when we can easily replicate the original behavior, but some other traps like ownKeys would be more difficult and error prone to replicate, so it's best to get into the habit of using Reflect in my opinion.
    —— http://dealwithjs.io/es6-features-10-use-cases-for-proxy/