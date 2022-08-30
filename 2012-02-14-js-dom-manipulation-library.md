---
title: Writing a jQuery-like DOM manipulation library
# subtitle: optional catchy phrase for the post
slug: js-dom-manipulation-library
# previously published url.
canonical: https://kosalanuwan.github.io/journal/
# ref: https://github.com/Hashnode/support/blob/main/misc/tags.json
tags: frontend-frameworks, frontend-development, javascript, javascript-library
# ref: https://github.com/kosalanuwan/hn-blogs-notes-to-self/issues
cover: https://user-images.githubusercontent.com/958227/187492946-aa256667-d053-4ce2-92be-f76c62275ae6.png?auto=compress
domain: keepontruckin.hashnode.dev
---

I'm not going to write yet another jQuery library but I like the idea of having [my own collection of helper functions][nano-dom-js], say, a <mark>stripped-down version</mark> of big jQuery-ish libraries. Think of it as a <mark>mix and match of native browser API calls</mark> for DOM manipulation.

[nano-dom-js]: https://github.com/kosalanuwan/vanilla-js-snippets/tree/main/helper-jquery-nano

The first thing we're going to need is a global identifier like `$` or `jqNano`. The other thing we're going to need is support for [UMD][what-is-umd], say, JS module bunlders and loaders. Unfortunately, JavaScript coding style seems [very opinionated][style-guides], so I am going to have to collect the bits and prices from individual repos that are in.

> **Note**
>
> I'm not exactly sure [why or how jQuery came up with the idea of `$` in JavaScript][what-is-dollar-sign-in-js].

[what-is-dollar-sign-in-js]: https://www.quora.com/What-does-mean-in-JavaScript-2/answer/Andrew-Smith-1766
[what-is-umd]: https://dev.to/iggredible/what-the-heck-are-cjs-amd-umd-and-esm-ikm
[style-guides]: https://javascript.info/coding-style#style-guides

## Creating the nano-library

First up is [Idiomatic JS style guide][idiomatic-js]. They [recommend][idiomatic-js-modules] encapsulating the library using [anonymous closure][what-is-anonymous-closure]. And the DOM manipulation can be written using [namespace][what-is-namespace] or [revealing module pattern][what-is-revealing-module-patter]. Kids' stuff.

[idiomatic-js]: https://github.com/rwaldron/idiomatic.js
[idiomatic-js-modules]: https://github.com/rwaldron/idiomatic.js#practical
[what-is-anonymous-closure]: https://web.archive.org/web/20171201033208/http://benalman.com/news/2010/11/immediately-invoked-function-expression/#iife
[what-is-revealing-module-pattern]: https://vanillajstoolkit.com/boilerplates/revealing-module-pattern/

## Handling the identity crisis

[AngularJS][angular-js] is interesting. They keep a stripped-down version of jQuery called [jqLite][jqlite-js], so it [uses jQuery if it exists][angular-js-use-jquery-if-found], otherwise default to jqLite. It's slightly obnoxious that they use their own module loading and dependency injection, but I like their pragmatic usecase of the [Polyfill pattern][what-is-polyfill].

[angular-js]: https://github.com/angular/angular.js
[jqlite-js]: https://github.com/angular/angular.js/blob/master/src/jqLite.js#L318
[angular-js-use-jquery-if-found]: https://github.com/angular/angular.js/blob/master/src/Angular.js#L1901-L1951
[what-is-polyfill]: https://javascript.info/polyfills

[ES5 style guide][airbnb-style-guide-js] from AirBnB is a little better. They [recommend][airbnb-js-modules] writing modules similar to Idiomatic JS style :point_up: but there's more to it. The module must handle [namespace][what-is-namespace] conflicts, say, using another module or library alongside that has the same namespace or identifier like how [jQuery advise to resolve its alias `$` with `.noConflict()`][jquery-no-conflict].

[airbnb-style-guide-js]: https://github.com/airbnb/javascript/tree/es5-deprecated/es5
[airbnb-js-modules]: https://github.com/airbnb/javascript/tree/es5-deprecated/es5#modules
[what-is-namespace]: https://www.oreilly.com/library/view/learning-javascript-design/9781449334840/ch13s15.html
[jquery-no-conflict]: https://api.jquery.com/jQuery.noConflict/

## Supporting module bundlers and loaders

Next on the list is [UMD][umd-js]. They have code snippets with variations of module bundling and loading pattern in a [GitHub repo][umd-js-samples]. Neat!

[umd-js]: https://dev.to/iggredible/what-the-heck-are-cjs-amd-umd-and-esm-ikm
[umd-js-samples]: https://github.com/umdjs/umd

[jQuery core style guide][jquery-core] also has a couple of [recommended snippets with UMD][jquery-modules] but with no namespace conflicts. So how is jQuery handling module bundlers and loaders? [Time to break out jQuery and peek into that script.][jquery-v1]

> **Note**
>
> [jQuery][jquery-v1] also use the [Polyfill pattern][what-is-polyfill] with `noConflict` and register the module as a [named module][glossary-named-module] to [support UMD][howto-support-umd]. Excellent!

[jquery-core]: https://contribute.jquery.org/style-guide/js/
[jquery-modules]: https://contribute.jquery.org/style-guide/js/#full-file-closures
[jquery-v1]: https://ajax.googleapis.com/ajax/libs/jquery/1.12.4/jquery.js

## Mixing helpers with native DOM

Another thing we're going to need is [chaining][what-is-chaining] functions with [native DOM elements][what-is-native-dom]. And that means [mixin][what-is-mixin]. It is basically same as [C#-like extension methods][what-is-extension-methods-in-csharp] for native DOM.

[what-is-chaining]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Inheritance_and_the_prototype_chain
[what-is-native-dom]: https://developer.mozilla.org/en-US/docs/Web/API/Document_Object_Model
[what-is-mixin]: https://developer.mozilla.org/en-US/docs/Glossary/Mixin
[what-is-extension-methods-in-csharp]: https://

[Underscore.String][underscore-string] use [prototype][what-is-prototype-inheritance] and [emulate function][what-is-emulate-function] patterns combo for [chaining but they limit the *target* to `String` types][underscore-string-chaining].

[underscore-string]: https://github.com/esamattis/underscore.string
[what-is-prototype]: https://github.com/getify/You-Dont-Know-JS/blob/1st-ed/this%20%26%20object%20prototypes/ch5.md
[what-is-emulate-methods]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Closures#Emulating_private_methods_with_closures
[underscore-string-chaining]: https://github.com/esamattis/underscore.string/blob/6a65c389135c432f77df27f606f8457849f662f2/index.js#L98-L116

I'm not going to lie, you'd be amazed how difficult it is to find the pieces and if you bind to the wrong DOM element, chaining breaks then nothing works. There's no way I'm going to go thru 100s of DOM elements to bind methods one element at a time. So it's definitely necessary to break out helper functions and modify that to bind in interface or abstract level.



/KP



> **PS:** You can find more about the topic in my curated list of online resources on [my awesome list #28][more-info]. For the complete work, see [my vanilla-js-snippets repo on GitHub][gh-repo].

[more-info]: https://github.com/kosalanuwan/keep-on-truckin/discussions/#readme
[gh-repo]: https://github.com/kosalanuwan/vanilla-js-snippets/#readme

