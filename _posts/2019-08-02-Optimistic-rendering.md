---
layout: post
title:  "Optimistic rendering"
---

UI principle consisting of rendering placeholders with available client-side data until fresher data is fetched from the server for hydration. 

Optimistic rendering is not appropriate for all cases. For actions like posting a comment, or liking content; it is absolutely appropriate to use optimistic rendering, since failure to complete the action does not result in a high cost. But for certain tasks such as, a bank transfers or a submission of a tax form, it is more appropriate to use pessimistic ui when rendering a response.


stale-while-revalidate chrome 75