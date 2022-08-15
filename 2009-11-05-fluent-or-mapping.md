---
title: Write pipes and filters for fluent OR-mapping
# subtitle: Use of pipes and filters
slug: fluent-or-mapping
# previously published url.
canonical: https://kosalanuwan.github.io/journal/
# ref: https://github.com/Hashnode/support/blob/main/misc/tags.json
tags: backend, apis, csharp, design-patterns, orm
# ref: https://github.com/kosalanuwan/hn-blogs-notes-to-self/issues
cover: https://user-images.githubusercontent.com/958227/184608269-7cff7014-fb2b-4c9f-8d8c-cefdbc5aac5b.png?auto=compress
domain: notestoself.hashnode.dev
---

I like the idea of [fluent interfaces][url-fluent-interface]. It gives the whole thing a very *Ubiquitous* feel since the code becomes more understandable, say, two months from now like [Uncle Bob says in the Clean Code][url-clean-code].

```csharp
// An example LINQ expression to filter all active products
// that belongs to a specified supplier.
db.Products
  .Where(p => !p.Discontinued && p.SupplierId == supplierId)
  .Take(10)
  .ToList();
```

[url-fluent-interface]: https://martinfowler.com/bliki/FluentInterface.html
[url-clean-code]: https://gist.github.com/wojteklu/73c6914cc446146b8b533c0988cf8d29



### Simplifying query expressions

The first thing we are going to need is a lot of [query expressions][url-query-exp]. I can [decompose or consolidate][url-refactor-simplify-cond] query expressions so a little [extract method][url-refactor-methods] magic is all that is necessary to get started. Kids' stuff.

```csharp
// Better readability.
db.Products.Where(p => Active(p) && WithSupplier(p, supplierId));
// Better resuability.
db.Suppliers.Where(s => s.Products.Any(p => Active(p));
```

That seems a little better since it allows reusing the query expressions. [LINQ][url-linq] may look like the best thing that happened for .NET, but it's getting all kinds of help from our friends at C#. It's the [extension methods][url-ext-methods].

[url-query-exp]: https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/operators/lambda-expressions#lambdas-with-the-standard-query-operators
[url-refactor-simplify-cond]: https://sourcemaking.com/refactoring/simplifying-conditional-expressions
[url-refactor-methods]: https://sourcemaking.com/refactoring/composing-methods
[url-linq]: https://docs.microsoft.com/en-us/dotnet/csharp/linq/linq-in-csharp
[url-ext-methods]: https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/extension-methods



### Chaining query expressions

The other thing we're going to need is chaining. And that means [extension methods][url-ext-methods]. It's slightly obnoxious that extension methods require <mark>non-nested non-generic static classes</mark>. So what does each predicate have that can be used for chaining that the fluent interface requires? `IQueryable`? 

```cs
// Use filters to create the pipes.
db.Products.Active().WithSupplier(supplierId)
  .Take(10)
  .ToList();
```

```csharp
// Extension methods for `IQueryable<Product>`.
static class IQueryableOfProduct
{
    static IQueryable<Product> Active(this IQueryable<Product> query)
        => query.Where(p => !p.Discontinued);
    static IQueryable<Product> WithSupplier(
        this IQueryable<Product> query, int supplierId)
        => query.Where(p => p.SupplierId == supplierId)
}
```

I can pass in the `IQueryable<>` domain entity type as the first parameter on which the method must operate. All I have to do is add the `this` modifier to the parameter and the C# compiler will do the rest for me. I'm not going to lie, you'd be amazed how tricky it is and if you return the wrong type, chaining breaks and nothing gets compiled.



### Describing pipes and filters

Naming things is hard. But there are more problems. Not only are the extension methods should be reusable, but they must be understandable. And you do want the code to scream what kind of business there is. Using a [domain-specific language][url-dsl] to describe methods seems like the right answer. Indead.

[url-dsl]: https://martinfowler.com/bliki/DomainSpecificLanguage.html



/KP



> **PS:** You can find more about the topics in my curated list of online resources on [my awesome list #31][more-info]. For the complete work, see [my skol repo on GitHub][gh-repo].

[more-info]: https://github.com/kosalanuwan/journal/discussions/31
[gh-repo]: https://github.com/kosalanuwan/skol-backend-monolithic-webapi/#readme
