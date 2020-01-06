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
- [X] CSS attribute `contain: strict;` restricts layout / paint calculations (https://developers.google.com/web/updates/2016/06/css-containment)
- [ ] PNGs to GIF client-side using WebAssembly ImageMagick rewrite (https://github.com/KnicKnic/WASM-ImageMagick)
- [ ] preload
- [ ] css grid 
  - [ ] fluid width
  - [ ] square cells hack
- [ ] V8 TurboFan bailout reasons: https://github.com/petkaantonov/bluebird/wiki/Optimization-killers
- [ ] layout thrashing: https://gist.github.com/paulirish/5d52fb081b3570c81e3a
- [ ] Object.defineProperty
- [ ] Page lifecycle API: https://developers.google.com/web/updates/2018/07/page-lifecycle-api
- [ ] navigator.connection.saveData && navigator.connection.effectiveType
- [ ] networkIdleCallback ~= boredom-loading (in contrast w/ lazy-loading) (https://github.com/pastelsky/network-idle-callback): basically places a ServiceWorker to intercept all `.fetch()` and add a debounce cooldown period to trigger idleCallback (another way of doing this involves also debouncing on other user events like mouse movements, scrolling... to prevent loading smth when the user might trigger the loading of smth else)
- [ ] Block Formatting Contexts (https://www.smashingmagazine.com//2017/12/understanding-css-layout-block-formatting-context/)
- [ ] Static vs Live NodeList (https://www.stefanjudis.com/blog/accessing-the-dom-is-not-equal-accessing-the-dom/)
- [ ] CSS `overscroll-behavior: contain;` to prevent a scrollable element to scroll the parent when reaching a boundary
- [ ] CSS `backdrop-filter: blur(10px);` (https://codepen.io/chriscoyier/pen/GRKqQBo)
- [ ] Memoization (https://nick.scialli.me/an-introduction-to-memoization-in-javascript/) (=== lazy getter?)
- [X] Proxies https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Proxy (cool and pertinent use cases: http://dealwithjs.io/es6-features-10-use-cases-for-proxy/) (to find use cases, google Facade pattern)
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
    Proxies can also be useful for adding properties to Arrays in a clean way
- [X] Reflect (http://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Reflect) allows access to Object definition methods (`Reflect.set(object, property, value)`). Especially useful from within Proxies or Object.defineProperty()
    > We can use the Reflect module to call the original function that would have ran if we haven't proxied the object. We could use `obj[prop]` too, like in the previous example, but this is cleaner, and it uses the same implementation a non-proxied object would use. This maybe doesn't sound important in a simple trap like `get`, when we can easily replicate the original behavior, but some other traps like `ownKeys` would be more difficult and error prone to replicate, so it's best to get into the habit of using Reflect in my opinion.
    —— http://dealwithjs.io/es6-features-10-use-cases-for-proxy/

- [ ] Event driven architecture *needs a bit more googling* (https://github.com/gergob/jsProxy/blob/master/04-onchange-object.js)
- [ ] `Symbol.toPrimitive` allows you do dictate how an object coerces based on the type of value the script is asking of it (https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Symbol/toPrimitive) (https://www.keithcirkel.co.uk/metaprogramming-in-es6-symbols/)
    ```javascript
    // An object with Symbol.toPrimitive property.
    var obj2 = {
    [Symbol.toPrimitive](hint) {
        if (hint == 'number') {
        return 10;
        }
        if (hint == 'string') {
        return 'hello';
        }
        return true;
    }
    };
    console.log(+obj2);     // 10        -- hint is "number"
    console.log(`${obj2}`); // "hello"   -- hint is "string"
    console.log(obj2 + ''); // "true"    -- hint is "default"
    ```

- [ ] Private fields using `Symbol` or `WeakMap`
    ```javascript
    // in module.js
    const privateKey = Symbol('privateField')
    export class SymbolPrivate {
        set publicKey (value) {
            return this[privateKey] = value
        }

        get publicKey () {
            return this[privateKey]
        }
    }

    // in main.js
    const a = new SymbolPrivate()
    a.publicKey = "coucou"
    console.log(a) // SymbolPrivate {Symbol(privateField): "coucou"}
    ```
    w/ `Symbol`, field is readable (easy debug) but not accessible
    ```javascript
    // in module.js
    const privateInstances = new WeakMap()
    export class WeakMapPrivate {
        constructor() {
            privateInstances.set(this, {})
        }

        set publicKey (value) {
            return Object.assign(privateInstances.get(this), { privateKey: value })
        }

        get publicKey () {
            return privateInstances.get(this).privateKey
        }
    }

    // in main.js
    const a = new WeakMapPrivate()
    a.publicKey = "coucou"
    console.log(a) // WeakMapPrivate {}
    ```
    w/ `WeakMap`, field is really provate (except accessible by "siblings" of the same class)