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
  - [ ] Service Worker
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
    const object = {
        [Symbol.toPrimitive](hint) {
            if (hint === 'number')
                return 10
            if (hint === 'string')
                return 'hello'
            return true
        }
    }
    console.log(+object)     // 10        -- hint is "number"
    console.log(`${object}`) // "hello"   -- hint is "string"
    console.log(object + '') // "true"    -- hint is "default"
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
    w/ `Symbol`, field is readable (easy debug) but not hidden (and not secure, through `getOwnPropertySymbols`)
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
    // so without assigning to `result` (and without returning)
    const fn = () => void doSomething(arg)
    ```
    But be careful, `void` is only applied to the the next statement
    ```javascript
    b = '1' + '1'
    void +b === 11 ? console.log('yes') : console.log('no')     // no => because it reads `undefined === 11`
    void (+b === 11) ? console.log('yes') : console.log('no')   // no => because it reads `undefined ? ... : ...`
    void (+b === 11 ? console.log('yes') : console.log('no'))   // yes
    ```
- [ ] actually useful bitwise operations for day to day programming
    * toggle between `0` and `1`
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
        // equivalent to a += 1-2*a
        ```
    * is my `indexOf` returning `-1`? 
        ```javascript
        // bitwise NOT of 0 is -1 (and vice versa), and 0 coerces to false
        !~-1 === true
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
    you can even `break` out of blocks that aren't loops or conditionals
    ```javascript
    named: {
        console.log('yo')
        break named
        console.log('hey')
    }
    ```
    **Use case** When speed is crucial (for example, when responding to fetch events within a Service Worker)
    ```javascript
    let response
    switchBlock: {
        const file = matchDataFileURL(event.request.url) // non trivial operation
        if(file) {
            response = fetchDataFile(CACHE_NAME, file)
            break switchBlock
        }
        const staticImg = matchStaticImageURL(event.request.url) // non trivial operation
        if(staticImg) {
            response = fetchStaticImage(CACHE_NAME, staticImg)
            break switchBlock
        }
        response = fetch(event.request)
    }
    // do something with response
    return response
    ```
    to write the same script without labeled block, you'd need a series of nested `if / else` blocks
    ```javascript
    let response
    const file = matchDataFileURL(event.request.url) // non trivial operation
    if(file) {
        response = fetchDataFile(CACHE_NAME, file)
    } else {
        const staticImg = matchStaticImageURL(event.request.url) // non trivial operation
        if(staticImg) {
            response = fetchStaticImage(CACHE_NAME, staticImg)
        } else {
            response = fetch(event.request)
        }
    }
    // do something with response
    return response
    ```
- [ ] css `mask-image` for opacity gradient (usually needs `-webkit-` prefix)
    - example 1: fade to background (overflowing text!)
        ```css
        p {
            mask-image: linear-gradient(to bottom, transparent 25%, black 75%);
        }
        ```
    - example 2: clipped content ([codepen](https://codepen.io/sheraff/pen/poJNXXq))
    ![Mask-image example: mountain photo with stripped clipping](/public/images/mask-image-example.png)
        ```css
        img {
            mask-image: repeating-linear-gradient(-45deg,
                transparent 0 20px,
                black 20px 40px);
        }
        ```
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
- [ ] css `display: contents` to prevent tag from creating a box. Layout wise, all the children will behave as though they were siblings of the `display: contents` node and "parent-to-children" layout properties can *pass through* (like a parent `position: relative` and a child `position: absolute`, or a parent `display: flex`and a child `flex: 1`...). It still creates a node in the DOM tree (for CSS selectors and for JS `document`). See MDN [Display box model](https://developer.mozilla.org/en-US/docs/Web/CSS/display-box).
    ```html
    <grid>
        <card>
            <cell></cell>
        </card>
    </grid>
    ```
    ```css
    grid { display: grid }
    card { display: contents }
    cell { grid-row-start: 1 }
    ```
- [ ] JS: sparse arrays are a thing. An array can have some of its indexes without assigned value (never assigned, deleted). 
    - It has low overhead (compared to assigning `undefined` for example), 
    - It logs out explicitely like this `[ <2 empty items>, 'foo', <1 empty item> ]`
    - Iterators apply their callbacks *only to the values that aren't empty* 
        ```javascript
        const array = [,1,,]
        array.map(x => x) // [ <1 empty item>, 1, <1 empty item> ]
        ```
    - `.length` still counts the empty items (meaning that a `for` loop will iterate over empty items whereas a `forEach` won't)
- [ ] css selector `:only-child`, equivalent to `:first-child:last-child`
- [ ] JS to `querySelector` with "preprocessor"-like `&` you can use `:scope`
    ```js
    const context = document.getElementById('context')
    const selected = context.querySelectorAll(':scope > div')
    ```
- [ ] Reconciliate *inheritance* and *composition*: create a class for common traits (entities in a game, components in a web page, ...), but instead of making a long prototype chain, compose traits from a set of possible traits.
    ```javascript
    class Robot {
        constructor(id) {
            this.id = id
        }
    }

    const featureSet = {
        assemble() {
            console.log(`#${this.id} assembled an object`)
        },
        explore() {
            console.log(`#${this.id} explored the planet`)
        },
        pickup() {
            console.log(`#${this.id} picked up materials`)
        },
        sample() {
            console.log(`#${this.id} sampled the ground`)
        },
    }

    function composeRobot(...traits) {
        class ComposedRobot extends Robot {}
        traits.forEach(trait => ComposedRobot.prototype[trait] = featureSet[trait])
        return ComposedRobot
    }

    const ScienceRobot = composeRobot('sample', 'explore')
    const FactoryRobot = composeRobot('pickup', 'assemble')
    const Replicator = composeRobot('explore', 'assemble')
    const Transformer = composeRobot('assemble', 'explore', 'pickup', 'sample')

    const robot1 = new ScienceRobot(3986)
    const robot2 = new FactoryRobot(9478)

    robot1.sample()   // #3986 sampled the ground
    robot2.pickup()   // #9478 picked up materials
    robot1.assemble() // TypeError: robot1.assemble is not a function
    robot2.explore()  // TypeError: robot2.explore is not a function
    ```
- [ ] there is a base 8 integer grammar in JS (prefix number with `0`)
    ```js
    const a = 012 // 10 (8 + 2)
    const b = 0120 // 80 (64 + 16)
     ```

- [ ] `AbortController` allows cancelling an ongoing XHR request
    ```js
    let controller
    function request(url) {
        if(controller)
            controller.abort()
        controller = new AbortController()
        return fetch(url, {signal: controller.signal})
        .then(() => console.log(`done: ${url}`))
        .catch(() => console.log(`failed: ${url}`))
    }
    request('/endpoint?q=a')
    request('/endpoint?q=ab')
    request('/endpoint?q=abc')
    // failed: /endpoint?q=a
    // failed: /endpoint?q=ab
    // done: /endpoint?q=abc
    ```
- [ ] CSS line clamping with elipsis
    ```css
    {
        display: -webkit-box;
        -webkit-line-clamp: 3;
        -webkit-box-orient: vertical;  
        overflow: hidden;
    }
    ```
- [ ] order of script loading & execution: from [v8 dev](https://v8.dev/features/modules#defer)
    ![](/public/images/script-async-defer.svg)
- [ ] CSS: invert black to white, don't change colors:
    ```css
    filter: invert(.862745) hue-rotate(180deg);
    ```
- [ ] TO TEST: fix iOS 100vh issue?
    ```css 
    .my-element {
        height: 100vh; /* Fallback for browsers that do not support Custom Properties */
        height: calc(var(--vh, 1vh) * 100);
    }
    ```
    ```javascript
    // First we get the viewport height and we multiple it by 1% to get a value for a vh unit
    let vh = window.innerHeight * 0.01;
    // Then we set the value in the --vh custom property to the root of the document
    document.documentElement.style.setProperty('--vh', `${vh}px`);
    ```
- [ ] CSS `scroll-padding` (for scroll snap) also applies to anchor offsets (e.g. JS `scrollIntoView` with CSS `scroll-padding-top: 70px`)
- [ ] JS sequence expression
    ```javascript
    if (val = foo(), val < 10) {}
    while (val = foo(), val < 10);
    ```

- [ ] JS structure type & primitive wrapper objects
    ```javascript
    const a = {}
    a.constructor === Object // true
    typeof a === 'object'    // true
    a instanceof Object      // true

    const b = []
    b.constructor === Array  // true
    typeof b === 'object'    // true
    b instanceof Array       // true

    const c = 1              // primitive
    c.constructor === Number // true
    typeof c === 'number'    // true
    c instanceof Number      // false

    const d = new Number(1)  // primitive wrapper object
    d.constructor === Number // true
    typeof d === 'number'    // false
    d instanceof Number      // true
    d instanceof Object      // true

    const e = Number('1')    // explicit primitive coercion
    e.constructor === Number // true
    typeof e === 'number'    // true
    e instanceof Number      // false
    e instanceof Object      // false

    d == 1  // true
    e == 1  // true
    e == d  // true
    e === 1 // true
    d === 1 // false

    class Yo{}
    const g = new Yo()
    g.constructor === Yo     // true
    typeof g === 'object'    // true
    g instanceof Yo          // true
    g instanceof Object      // true
    ```
- [ ] `-webkit-tap-highlight-color` CSS property allows for setting the color of the blue highlight that flashes when tapping on mobile devices
- [ ] `env(safe-area-inset-bottom)` CSS value is the amount of space (bottom in this case) that may be encroached on by the UI (usually 0 for standard desktops, more for iOS w/ the bottom bar, or androids w/ rounded corners)
- [ ] JS check for specific font loaded state
    ```javascript
    document.fonts.check('400 1em EB Garamond')
    ```
- [ ] `navigator.sendBeacon` queues up a request to be sent even if the user closes the page
- [ ] `'visibilitychange'` event on `document` is the last event reliably fired before a user leaves the page (but also fires when the user just switches to another tab/window without actually closing)

- [ ] use cases for comma operator
    ```js
    // double for loop in one
    const matrix = [
        [1, 2, 3],
        [4, 5, 6]
    ]
    const m = matrix.length
    const n = matrix[0].length
    for (
        let i = 0, j = 0;
        i < m;
        j++, i+=!(j%n), j%=n
    ) {
        console.log(matrix[i][j])
    }
    // 1, 2, 3, 4, 5, 6
    ```
    ```js
    // avoid duplicate code to initialize a while loop
    const set = new Set([1, 2, 3])
    let a
    while (a = iterator.next(), !a.done) {
        console.log(a.value)
    }
    // 1, 2, 3
    ```
    ```js
    // simplify small reduce functions
    [].reduce(
        (acc, cur) => (acc.push(cur), acc),
        []
    )
    ```
    ```js
    // add a console log in implicit returns
    [].reduce(
        (acc, cur) => (console.log(cur), acc + cur),
        0
    )
    ```

- [ ] ways to switch
    ```js
    switch(a) {
        case 1:
            console.log(1)
            break
        case 2:
            console.log(2)
            break
    }
    ```
    ```js
    switch(true) {
        case a === 1:
            console.log(1)
            break
        case a === 2:
            console.log(2)
            break
    }
    ```
    ```js
    label: {
        if (a === 1) {
            console.log(1)
            break label
        }
        if (a === 2) {
            console.log(2)
            break label
        }
    }
    ```

- [ ] useful bitwise operations II
    ```js
    const sign = (x) => x >> 31 || 1
    // -1 | 1
    ```
    ```js
    const floor = (x) => x | 0
    // 12.6 => 12
    ```
    ```js
    const odd = (x) => x & 1
    // 1 | 0
    ```
    ```js
    const even = (x) => ~x & 1
    // 1 | 0
    ```
    ```js
    const toggle = (x) => x ^ 1
    // 1 | 0
    ```