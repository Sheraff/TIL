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
            if (hint == 'number')
                return 10
            if (hint == 'string')
                return 'hello'
            return true
        }
    }
    console.log(+obj2);     // 10        -- hint is "number"
    console.log(`${obj2}`); // "hello"   -- hint is "string"
    console.log(obj2 + ''); // "true"    -- hint is "default"
    ```
    `Symbol.iterator` also exists to define iterating behavior, `Symbol.species` to define the prototype

- [ ] Private fields using `Symbol` or `WeakMap`
    ```javascript
    // in module.js
    const privateKey = Symbol('privateField')
    export class SymbolPrivate {
        constructor() {
            this[privateKey] = 42
            console.log(`internally: ${this[privateKey]}`)
        }
    }

    // in main.js
    const a = new SymbolPrivate() // internally: 42
    console.log(`externally: ${a.privateKey}`) // externally: undefined
    console.log(a) // SymbolPrivate {Symbol(privateField): 42}
    ```
    w/ `Symbol`, field is readable (easy debug) but not hidden (though not secure, through `getOwnPropertySymbols`)
    ```javascript
    // in module.js
    const privateInstances = new WeakMap()
    export class WeakMapPrivate {
        constructor() {
            privateInstances.set(this, { privateKey: 42 })
            console.log(`internally: ${privateInstances.get(this).privateKey}`)
        }
    }

    // in main.js
    const a = new WeakMapPrivate() // internally: 42
    console.log(`externally: ${a.privateKey}`) // externally: undefined
    console.log(a) // WeakMapPrivate {}
    ```
    w/ `WeakMap`, field is "class private" (only accessible by "siblings" of the same class)

- [ ] Parent classes access child classes through `new.target`
    ```javascript
    class Unextendable {
        constructor() {
            if(new.target !== Unextendable)
                throw new Error()
        }
    }
    ```
    ```javascript
    class A {
        constructor() {
            new.target.classMethod()
        }
        static classMethod() {
            console.log('hello from A')
        }
    }
    class B extends A {
        static classMethod() {
            console.log('hello from B')
        }
    }
    const a = new A() // hello from A
    const b = new B() // hello from B
    ```
    Could be used to allow overriding how the `constructor` builds some initial state
    ```javascript
    class A {
        constructor() {
            this.state = new.target.makeState()
        }
        static makeState() {
            return { }
        }
    }
    class B extends A {
        static makeState() {
            return [ ]
        }
    }
    ```
- [ ] Uses for `void` in modern JS. It allows us to evaluate an expression and still return undefined. "In defense of void"
    We're sometimes a bit quick in how we use arrow functions
    ```javascript
    const fn = () => doSomething(arg) // result of doSomething will leak
    const fn = () => { doSomething(arg) } // result won't leak but intent isn't clear
    const fn = () => void doSomething(arg) // intent is clear: call function for side-effects, drop returned value
    ```
    `void` is also good with IIFEs
    Here's an example of how the classic IIFE syntax can cause issues that the void operator prevents.
    ```javascript
    function a() { console.log('yo') }
    const b = a
    void function (){ console.log('IIFE') }()
    // instead of
    function a() { console.log('yo') }
    const b = a
    (function (){ console.log('IIFE') })()
    ```
    Iterate over a constant array used only once, a pretty common case
    ```javascript
    const s = "hello"
    [1, 2, 3].forEach(console.log)
    // would be fixed by
    void [1, 2, 3].forEach(console.log)
    ```
    ```javascript
    const a = '1' + '1'
    +a === 11 ? console.log('yes') : console.log('no')
    ```
    ```javascript
    const s = "here is a string"
    /[a-z]/g.exec(s) // this practically never happens though :)
    ```
    An easy way to think about this is: every time you are writing an expression that would usually be held within a variable, but aren't defining a variable, you should "assign it to `void`"
    ```javascript
    (function(){})()
    // is the equivalent of
    function iife(){}
    iife()
    // so without assigning to `iife`
    void function(){}()
    ```
    ```javascript
    [1, 2, 3].forEach(console.log)
    // is the equivalent of 
    const arr = [1, 2, 3]
    arr.forEach(console.log)
    // so without assigning to `arr`
    void [1, 2, 3].forEach(console.log)
    ```
    ```javascript
    const fn = () => doSomething(arg)
    // is the equivalent of 
    const fn = () => {
        const result = doSomething(arg)
        return result
    }
    // so without assigning to `result`
    const fn = () => void doSomething(arg)
    ```
    But be careful, `void` is only applied to the the next statement
    ```javascript
    b = '1' + '1'
    void +b === 11 ? console.log('yes') : console.log('no')     // no => because it reads `undefined === 11`
    void (+b === 11) ? console.log('yes') : console.log('no')   // no => because it reads `undefined ? ... : ...`
    void (+b === 11 ? console.log('yes') : console.log('no'))   // yes
    ```
- [ ] toggle between `0` and `1`
    ```javascript
    // using boolean type coercion `!` and number type coercion `+`
    a = 0
    a = +!a // 1
    a = +!a // 0
    // alternates between 0 and 1
    ```
    ```javascript
    // using bitwise XOR operator `^` (exclusive disjuction)
    a = 0
    a ^= 1 // 1
    a ^= 1 // 0
    // alternates between N and N+1 (where N is an even integer)
    ```
    ```javascript
    // using the bitwise NOT operator `~` (turns bit 0 into 1)
    a = 0
    a = -~-a // 1
    a = -~-a // -0
    a = -~-a // 1
    // alternates between -N and N+1 (where N is the initial value of a)
    ```
 
- [ ] `Math.pow()` is old, we now have exponential operator `**`
- [ ] labeled statements
    you can better manage nested loops that need to `break` or `continue`
    ```javascript
    level1: while(true) { 
        level2: while(true) { 
            break level1
        }
    }

    while(true) { 
        while(true) { 
            break
        }
        break
    }
    ```
    you can even `break` out of blocks that aren't loops
    ```javascript
    named: if(true) {
        console.log('yo')
        break named
        console.log('hey')
    }
    ```
- [ ] css `mask-image` for opacity gradient (overflowing text) !
- [ ] Template literals have a grammar to call a function for custom template processing
    ```javascript
    console.log`a ${1} b ${2}` // [ 'a ', ' b ', '' ] 1 2
    ```
    ```javascript
    function fn(parts, ...joints) {
        console.log(parts)
        console.log(joints)
    }
    fn`a ${1} b ${2}`
    // [ 'a ', ' b ', '' ]
    // [ 1, 2 ]
    ```
- [ ] css `all: initial` resets all styles 
    ```css
    button {
        all: initial; /* won't have any style, including vendor defaults */
    }
    ```