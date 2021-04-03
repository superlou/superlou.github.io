+++
title = "Converting a jQuery array to an Ember array"
date = 2013-04-17
[taxonomies]
tags = ["Ember.js", "jQuery"]
+++

> Warning: this is way out of date.

<!-- more -->

Ember.js is incorporating more UI abilities natively every day, but for now there’s a lot of functionality provided by jQueryUI that is nice to take advantage of.  In order to convert the result of a jQuery selector or other array-like jQuery object, use the following:

```js
properEmberArray = Ember.A($.makeArray(jQueryArrayLike));
```

Sorry if this is obvious, but it gave me some grief before I realized that I was calling map on a jQuery object and not a fully fledged Ember Array.
