## topics
- [ ] chrome dev tools
- [ ] SVG anim
- [ ] react
  - [ ] ref as store across renders
  - [ ] lifecycle functions have a callback as second argument
- [ ] Web APIs 
  - [ ] IntersectionObserver
  - [ ] IndexDB
  - [ ] Service Worler
- [ ] look into using `contain: strict;` for <card> and possibly other elements to restrict layout / paint calculations (https://developers.google.com/web/updates/2016/06/css-containment)
- [ ] PNGs to GIF client-side using WebAssembly ImageMagick rewrite (https://github.com/KnicKnic/WASM-ImageMagick)
- preload
- css grid 
  - fluid width
  - square cells hack
- V8 TurboFan bailout reasons: https://github.com/petkaantonov/bluebird/wiki/Optimization-killers
- layout thrashing : https://gist.github.com/paulirish/5d52fb081b3570c81e3a
- closures
- Object.defineProperty

## improvements


## note keeping

### listing site-wide posts tags
{% assign tags = "" | split: ',' %}

{% for post in site.posts %}
  {% for tag in post.tags %} 
    {% unless tags contains tag %}
      {% assign tags = tags | push: tag %}
    {% endunless %}
  {% endfor %}
{% endfor %}