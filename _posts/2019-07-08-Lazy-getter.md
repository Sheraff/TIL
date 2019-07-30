---
layout: post
title:  "Lazy getter"
excerpt: >
    Compute a value only when needed, but then only once.
---

Compute a value only when needed, but then only once.

```
Object.defineProperty(MyObjWithLazyId.prototype, 'id', {
    get: function() {
        var id = getUniqueId();
    
        Object.defineProperty(this, 'id', {
            value: id
        });

        return id;
    }
}); 
```