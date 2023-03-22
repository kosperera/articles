---
title: Non-opinionated code review guidelines
subtitle: A set of ground rules for agile development teams, contributors, and maintainers
slug: non-opinionated-code-reviews
# previously published url.
canonical: https://kosalanuwan.github.io/journal/
# ref: https://github.com/Hashnode/support/blob/main/misc/tags.json
tags: programming, developer, general-advice, software-development, clean-code, developers, code-review, design review, guide
cover: https://user-images.githubusercontent.com/958227/226826379-56df08b3-3166-4a08-aa58-cfab2d56941a.png?auto=compress
domain: keepontruckin.hashnode.dev
---

I'm not going to lie, <mark>opinions are like Arseholes, everyone has one</mark>. It would have been acceptable but the objectives of code reviews are simple. In general, not only is to get the best possible pull request, but also improving skills of the fellow contributors. Easy enuf, except <mark>we need a set of ground rules</mark>.



### Etiquette

The first thing we're going to need is a contributor etiquette. I like the idea of [Jante Law][wiki-jante-law-definition]. It gives everyone a very _Egalitarian_ feel since _you are not to think you're anyone special or you're better than everyone else_, as they teach in Scandinavian schools.

[wiki-jante-law-definition]: https://en.wikipedia.org/wiki/Law_of_Jante#Definition



### Workflow

The next is the release engineering strategy. All we need is a version control tool and [git][docs-git-scm] is the mainstream thingy for that. But [some of the branching strategies are pretty horrendous][docs-gitflow] so you have to <mark>keep the branch integrations to minimum</mark>.

<img class="img-responsive img-comic" alt="Using Git - Dilbert by Scott Adams" src="https://assets.amuniversal.com/ddf1fa20315201378d0e005056a9545d" width="100%">

[Trunk-based development][site-tbd] seems extraordinarily uncomplicated but there are more to consider.

- Branch policies for protecting the default branch.
- Naming conventions for branches.
- Pipelines for release engineering.

[docs-git-scm]: https://git-scm.io/docs/
[docs-gitflow]: https://atlassian.com/docs/git/gitflow/
[site-tbd]: https://trunkbaseddevelopment.com/



### Changesets

Some organizations often call this a _Pull request_ or a _Changelist_. And it has to be <mark>small-enuf to one self-contained change</mark>. Unfortunately, no playbook has a recipe for this so I'm going to have to let individual contributor decide the size of the pull request.

Once the pull request is ready to review, they have to describe it, specifically _what_ has been done and _why_ it was made as-is. This can be tricky, so a [conventional-based description][site-git-cc] seems like a great way of adding structure and meaning to it.

[site-git-cc]: https://www.conventionalcommits.org/en/v1.0.0/#summary



### Design and Health

<img class="img-responsive img-comic" alt="Its the holy grail of technology - Dilbert by Scott Adams" src="https://assets.amuniversal.com/268f57809f72012f2fe600163e41dd5b" width="100%">

This covers a lot of ground and can be challenging enuf. Unfortunately, you can only boil so many oceans at the same time, so what does each reviewer should look for in a pull request? In general, _readability and maintainability_ would hint a few mistakes here and there, but there are more to review.

- Intended functionality and the design.
- Over-engineering and complexity.
- Clear naming and conformance to style guide.
- Parallel programming and thread safety.
- Sufficient exception handling and logging et al.



### Style Guide

Yes, a homegrown standards and conventions isn't such a great idea. The major issue is that it would demand an intense concentration to keep up with each tech stack without giving away the *[egalitarian culture](#heading-etiquette)* completely since you would want to *Dictate-It-All*. And you do want contributors to maintain consistancy of the code. This may be difficult. I need a <mark>community-adept style guide</mark> to start with.



### Faster Reviews

A pull request won't get reviewed itself. And a contributor must intervene at some point. I can review shortly after it comes in. But that seems a little unproductive since it takes time to get back to _The Zone_ after been interrupted. So how fast should code reviews be? _One business day or 24 hours_ sounds reasonable but there is more to this.

- Professional courtesy when pointing out problems.
- Code analytics for obvious smelly codes, anti-patterns, conformance to standards, et al.



### Summary

<img class="img-responsive img-comic" alt="Too Dumb To Understand - Dilbert by Scott Adams" src="https://assets.amuniversal.com/dc424c1016c20134707c005056a9545d" width="100%">

All we need is non-opinionated guidelines for all kinds of contributors, so in doing a code review, you should value for:

- <mark>Technical facts and data</mark> over opinions and personal preferences.
- <mark>Style guides</mark> over personal preferences.
- <mark>Software design principles</mark> over opinionated-design.
- <mark>Consistency</mark> of the code over personal preferences.
- <mark>Team velocity</mark> over personal velocity.



/KP



> **PS:** You can find more about the topic in my curated list of standards, practices, guides, and discussions on [my awesome list #33][more-info].

[more-info]: https://github.com/kosalanuwan/keep-on-truckin/discussions/#readme
