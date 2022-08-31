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

I'm not going to write yet another jQuery library but I like the idea of having [my own collection of helper functions][nano-dom-js], say, a <mark>stripped-down version</mark> of big jQuery-ish libraries. Think of it as a mix and match of [native browser API][native-browser-api] calls for DOM manipulation.

[nano-dom-js]: https://github.com/kosalanuwan/vanilla-js-snippets/tree/main/helper-jquery-nano
[native-browser-api]: https://developer.mozilla.org/en-US/docs/Web/API/Document_Object_Model



The first thing we're going to need is a [global][what-is-global] [identifier][what-is-namespace] for the nano-library, say, `jqNano`, and an alias like `$`. The other thing we're going to need is the support for [JS module bundlers and loaders][what-is-umd]. Unfortunately, JavaScript coding styles are [very opinionated][style-guides], so I am going to have to pick up the bits and pieces from individual repos that are in.

> **Note**
>
> I'm not exactly sure why or how jQuery came up with the idea of [the `$` in JavaScript][what-is-dollar-sign-in-js].

[what-is-global]: https://developer.mozilla.org/en-US/docs/Glossary/Global_object
[what-is-namespace]: https://www.oreilly.com/library/view/learning-javascript-design/9781449334840/ch13s15.html
[what-is-umd]: https://dev.to/iggredible/what-the-heck-are-cjs-amd-umd-and-esm-ikm
[style-guides]: https://javascript.info/coding-style#style-guides
[what-is-dollar-sign-in-js]: https://www.quora.com/What-does-mean-in-JavaScript-2/answer/Andrew-Smith-1766



### Structuring the JS library

First up is structure for each JS library. [jQuery's Core Style Guide][jquery-style-guide] recommend maintaining JS libraries with [full-file closures][jquery-style-guide-modules], so a little [anonymous closure][what-is-anonymous-closure] magic is all that is necessary to get started. Kids' stuff. 

[jquery-style-guide]: https://contribute.jquery.org/style-guide/js/
[jquery-style-guide-modules]: https://contribute.jquery.org/style-guide/js/#full-file-closures
[what-is-anonymous-closure]: https://web.archive.org/web/20171124074544/http://www.adequatelygood.com/JavaScript-Module-Pattern-In-Depth.html



### Defining the modules and helpers

Next on the list is composing the helpers as modules. And that means module augmentation. I can import the module, add props and members, then export it. But there's more to do. I need to manage private state.

[Idiomatic Style Guide][idiomatic-style-guide] recommend encapsulating [internals and private states][idiomatic-style-guide-modules] of the module when implementing JS libraries. And that means [revealing module pattern][what-is-revealing-module-pattern]. All I need to do is declare `jqNano` as the global identifier `$` as its alias. But there are problems. They may have already been declared by another module alongside.

[idiomatic-style-guide]: https://github.com/rwaldron/idiomatic.js
[idiomatic-style-guide-modules]: https://github.com/rwaldron/idiomatic.js#practical
[what-is-revealing-module-pattern]: https://vanillajstoolkit.com/boilerplates/revealing-module-pattern/



### Dealing with identifier conflicts

[AngularJS][angular-js] is interesting. They [use jQuery if present][angular-js-use-jquery-if-found], otherwise default to [jqLite][angular-js-jqlite], a stripped-down version of their own. I'm not exactly sure how this [polyfill pattern][what-is-polyfill] fits into this scenario since it would require both libraries to follow the exact method signatures. That seems a little hard to manage and you do want to use `jqNano` alongside without giving away completely. Writing loose augmented modules to take care of that seems like the right pattern. But there's more to it.

[angular-js]: https://github.com/angular/angular.js
[angular-js-use-jquery-if-found]: https://github.com/angular/angular.js/blob/master/src/Angular.js#L1901-L1951
[angular-js-jqlite]: https://github.com/angular/angular.js/blob/master/src/jqLite.js#L318
[what-is-polyfill]: https://javascript.info/polyfills



[AirBnB JavaScript Style Guide][airbnb-style-guide] encourage [writing modules][airbnb-style-guide-modules] with a `noConflict` method to resolve module conflicts. jQuery too [advise][jquery-no-conflict] to resolve its alias `$` by calling the `.noConflict()`. So how is jQuery library handling `noClonflict`? [Time to break out jQuery and peek into that script.][jquery-v1]

> **Note**
>
> jQuery's module identifier is `jQuery`, and `$` is the short-hand for it, an alias. I can call `noConflict` and jQuery will restore the previous module for me, say, a variant of jQuery from the multiverse that is been used alongside.

[airbnb-style-guide]: https://github.com/airbnb/javascript/tree/es5-deprecated/es5
[airbnb-style-guide-modules]: https://github.com/airbnb/javascript/tree/es5-deprecated/es5#modules
[jquery-no-conflict]: https://api.jquery.com/jQuery.noConflict/
[jquery-v1]: https://ajax.googleapis.com/ajax/libs/jquery/1.12.4/jquery.js



### Supporting module bundlers and loaders

Next up is bundlers and loaders. That means [UMD][what-is-umd] and the [umd repo][umd-js-templates] has templates with variants that works everywhere. I can use [jqueryPlugin.js][umd-js-jquery-template] template for starters but the script literally depends on jQuery. So how is jQuery handling module bundlers and loaders?

> **Note**
>
> [jQuery library][jquery-v1] register as a named AMD module for global import and module export. Then they get the `jQuery` instance from the module for CommonJS-like environments that contains a proper `window` instance.

[umd-js-templates]: https://github.com/umdjs/umd
[umd-js-jquery-template]: https://github.com/umdjs/umd/blob/master/templates/jqueryPlugin.js



This may be confusing. I'll come back later.



### Mixing with native DOM

Another thing we're going to need is [chaining][what-is-chaining] functions with [native DOM elements][what-is-native-dom]. And that means [mixin][what-is-mixin]. It is basically same as [C#-like extension methods][what-is-extension-methods-in-csharp] for native DOM. I'm not going to lie, you'd be amazed how difficult it is to find the pieces and if you bind to the wrong DOM element, chaining breaks then nothing works. There's no way I'm going to go thru 100s of DOM elements to bind methods one element at a time. So it's definitely necessary to break out helper functions and modify that to bind in interface or abstract level.

[what-is-chaining]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Inheritance_and_the_prototype_chain
[what-is-mixin]: https://developer.mozilla.org/en-US/docs/Glossary/Mixin
[what-is-extension-methods-in-csharp]: https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/how-to-implement-and-call-a-custom-extension-method



[Underscore.String][underscore-string] use [prototype][what-is-prototype-inheritance] and [emulate function][what-is-emulate-methods] combo for [chaining utility functions to `String` types][underscore-string-chaining]. All I need to do is break out that script and we're set.

[underscore-string]: https://github.com/esamattis/underscore.string
[what-is-prototype-inheritance]: https://github.com/getify/You-Dont-Know-JS/blob/1st-ed/this%20%26%20object%20prototypes/ch5.md
[what-is-emulate-methods]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Closures#Emulating_private_methods_with_closures
[underscore-string-chaining]: https://github.com/esamattis/underscore.string/blob/6a65c389135c432f77df27f606f8457849f662f2/index.js#L98-L116



/KP



> **PS:** You can find more about the topic in my curated list of online resources on [my awesome list #28][more-info]. For the complete work, see [my vanilla-js-snippets repo on GitHub][gh-repo].

[more-info]: https://github.com/kosalanuwan/keep-on-truckin/discussions/#readme
[gh-repo]: https://github.com/kosalanuwan/vanilla-js-snippets/#readme

