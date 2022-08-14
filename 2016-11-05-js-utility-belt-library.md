---
title: Writing a Lodash-like JS utility belt library
# subtitle: optional catchy phrase for the post
slug: try-javascript-utility-belt-library
# ref: https://github.com/Hashnode/support/blob/main/misc/tags.json
tags: frontend, js, vanilla-js-1, javascript-library, lodash
# ref: https://github.com/kosalanuwan/hn-blogs-notes-to-self/issues
cover: https://user-images.githubusercontent.com/958227/184541928-6bbc34d8-e4de-46da-9858-9e2506a987bb.png?auto=compress
domain: notestoself.hashnode.dev
---

I prefer having [my own JavaScript utility belt libraries][gh-vanilla-js-snippets] as they do on [Lodash][url-lodash] and [Underscore.js][url-underscorejs], say, a stripped-down version of it, then bundled together into the `global` namespace as `_`.

```js
// Utility `_` functions in use.
_.format('Hello {0}!', username);
```

[gh-vanilla-js-snippets]: https://github.com/kosalanuwan/vanilla-js-snippets/tree/main/helper-lodash-nano
[url-lodash]: https://lodash.com/
[url-underscorejs]: https://underscorejs.org/



### Stealing from the best

The first thing we're going to need is a few utility libraries. Fortunately, GitHub has plenty of [JS utility libraries][gh-search-topic-utilities]. I'm not gonna lie,  [Lodash][gh-lodash], [Underscore.js][gh-underscorejs], [Angular.js][gh-angularjs] code may be difficult to understand but the [Babel][gh-babel] code is wierdly insane. All I need to do is break out those code snippets I just found in individual repos to get started.

[gh-search-topic-utilities]: https://github.com/topics/utilities?l=javascript&o=desc&s=stars
[gh-lodash]: https://github.com/lodash/lodash
[gh-underscorejs]: https://github.com/jashkenas/underscore
[gh-angularjs]: https://github.com/angular/angular.js/
[gh-babel]: https://github.com/babel/babel



### Creating a js library

The other thing we're going to need is a micro-library. I'm not exactly sure how the Lodash and the like are bundled to `_`, so I'm going to go ahead and say that they use a combination of [namespace][glossary-ns] and a little [polyfill][glossary-polyfill] magic to encapsulate the helper methods. Excellent.

```js
var _ = window._ || (window._ = {});
; (function (undefined) {
    function _format(template) {
      // TODO: String format logic must go here.
    }

    if (!_.format) (function () {
        _.format = function format() {
            return _format.apply(this, arguments);
        };
    })();
})();
```

[glossary-ns]: https://www.oreilly.com/library/view/learning-javascript-design/9781449334840/ch13s15.html
[glossary-polyfill]: https://developer.mozilla.org/en-US/docs/Glossary/Polyfill



### C#-like extension methods

Next is [mixin][glossary-mixin]. Not only this require a [closure pattern][glossary-closure], but you also have to [emulate methods][glossary-emulate-methods]. It's taking a few tries to compile the script. This may be difficult. I'll come back later.

```js
// Before:
// The utlity belt approach.
if (!_.isblank(username)) {
	_.format('Hello {0}!', username);
  _.toKebabCase(username);
}

// After: 
// Using C#-like extension methods.
if (!username.isblank()) {
	'Hello {0}!'.format(username);
	username.toKebabCase();
}
```

You'd be amazed how intense it is to enable both namespace and prototype options as they do in <mark>C# Extension Methods</mark>. [@esamattis/underscore.string][gh-underscore-str] has mixin for `String` types. So I'm gonna go ahead and break out that same script and modify my utility script. Excellent.

[glossary-mixin]: https://developer.mozilla.org/en-US/docs/Glossary/Mixin
[glossary-closure]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Closures
[glossary-emulate-methods]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Closures#Emulating_private_methods_with_closures
[gh-underscore-str]: https://github.com/esamattis/underscore.string/blob/master/index.js#L105-L140

---

### q.v.

- See [Awesome #25][more-info] for the curated list of online resources and discussions.
- See [vanilla-js-snippets][source-code] repo on GitHub for the complete work.

[more-info]: https://github.com/kosalanuwan/journal/discussions/25
[source-code]: https://github.com/kosalanuwan/vanilla-js-snippets/#readme
