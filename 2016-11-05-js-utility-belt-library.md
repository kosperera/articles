---
title: Writing a Lodash-like JS utility belt library
# subtitle: optional catchy phrase for the post
slug: js-utility-belt-library
# previously published url.
canonical: https://kosalanuwan.github.io/journal/
# ref: https://github.com/Hashnode/support/blob/main/misc/tags.json
tags: frontend, js, vanilla-js-1, javascript-library, lodash
# ref: https://github.com/kosalanuwan/hn-blogs-notes-to-self/issues
cover: https://user-images.githubusercontent.com/958227/184608671-42fa288a-e462-4e64-a1b7-a17a78db85a4.png?auto=compress
domain: notestoself.hashnode.dev
---

I prefer having [my own JavaScript utility belt libraries][gh-vanilla-js-snippets] as they do on [Lodash][url-lodash] and [Underscore.js][url-underscorejs], say, <mark>a stripped-down version</mark> of it, then bundled together into the `global` namespace as `_`.

```js
// Utility '_' in use.
_.format('Hello {0}!', username);
```

[gh-vanilla-js-snippets]: https://github.com/kosalanuwan/vanilla-js-snippets/tree/main/helper-lodash-nano
[url-lodash]: https://lodash.com/
[url-underscorejs]: https://underscorejs.org/



### Stealing from the best

The first thing we're going to need is a few vanilla JS utility libraries. Fortunately, GitHub has [plenty of them][gh-search-topic-utilities]. I'm not gonna lie,  [Lodash][gh-lodash], [Underscore.js][gh-underscorejs], [Angular.js][gh-angularjs], and [Babel][gh-babel] code are a little intense. All I need to do is break out those code snippets I just found in individual repos to get started. Excellent.

[gh-search-topic-utilities]: https://github.com/topics/utilities?l=javascript&o=desc&s=stars
[gh-lodash]: https://github.com/lodash/lodash
[gh-underscorejs]: https://github.com/jashkenas/underscore
[gh-angularjs]: https://github.com/angular/angular.js/
[gh-babel]: https://github.com/babel/babel



### Creating a js library

The other thing we're going to need is a micro-library. I'm not exactly sure how Lodash and the like are bundled to its namespaces, say, `_` , `ng`, or  `angular`, so I'm going to go ahead and say that they use a combination of [namespace][glossary-ns] and a little [polyfill][glossary-polyfill] magic to encapsulate those utility methods. Indeed.

```js
// Create a public namespace.
var _ = window._ || (window._ = {});
// Use IIFE to load the library.
; (function (undefined) {
    function _format(template) {
      // TODO: String format logic must go here.
    }
		// Define helper method, if not.
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

Next is extention methods. And that means [mixin][glossary-mixin]. Not only this require a [closure pattern][glossary-closure], but you have to [emulate methods][glossary-emulate-methods]. It's taking a few tries to compile the script. This may be difficult. I'll come back later.

```js
// Before:
// The utlity belt approach.
if (!_.isblank(username)) {
	_.format('Hello {0}!', username);
  _.toKebabCase(username);
}
```

```javascript
// After: 
// Using C#-like extension methods.
if (!username.isblank()) {
	'Hello {0}!'.format(username);
	username.toKebabCase();
}
```

You'd be amazed how intense it is to enable both namespace and prototype options as they do in <mark>C# extension methods</mark>. The [Underscore.String][gh-underscore-str] library has [mixin for `String` types][gist-mixin-string], so I'm gonna go ahead and break out that same script and use on my utility script. Excellent.

[glossary-mixin]: https://developer.mozilla.org/en-US/docs/Glossary/Mixin
[glossary-closure]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Closures
[glossary-emulate-methods]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Closures#Emulating_private_methods_with_closures
[gh-underscore-str]: https://github.com/esamattis/underscore.string/blob/master/index.js#L105-L140
[gh-underscore-str]: https://github.com/esamattis/underscore.string
[gist-mixin-string]: https://github.com/esamattis/underscore.string/blob/master/index.js#L105-L140



/KP



> **PS:** You can find more about the topics in my curated list of online resources on [my awesome list #25][more-info]. For the complex work, see [my vanilla-js-snippets repo on GitHub][gh-repo].

[more-info]: https://github.com/kosalanuwan/journal/discussions/25
[gh-repo]: https://github.com/kosalanuwan/vanilla-js-snippets/#readme