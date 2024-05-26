---
title: "Propriety"
date: 2024-05-25T12:00:00-04:00
description: "Ownership is 9/10s of the law... and 9/10s of software architecture."
featured: true
draft: false
toc: false
usePageBundles: false
# featureImage: "/images/path/file.jpg" # Sets featured image on blog post.
# featureImageAlt: 'Description of image' # Alternative text for featured image.
# featureImageCap: 'This is the featured image.' # Caption (optional).
# thumbnail: "/images/path/thumbnail.png" # Sets thumbnail image appearing inside card on homepage.
# shareImage: "/images/path/share.png" # Designate a separate image for social media sharing.
codeMaxLines: 10
codeLineNumbers: true
figurePositionShow: true
showRelatedInArticle: false
categories:
  - Architecture
tags:
  - Ownership
  - Abstractions
  - Multiple Hats
---

To save the author some work dear reader, imagine the 3 Spiderman meme.  Fill in "Who owns client data validation?" as the title and label the Spidermans (Spidermen?) Controller/Service/Persistence.  One place has to own that responsibility.  Let's say we had a bug where we accidentally allowed updating `Widget`s to have an empty `price` field.  How do you fix that?  _Where_ do you fix that?

Knowing _who_ owns a responsibility in your application is essential.

For the sake of this post, I will abstractly use the word "component".  "Components" may also be subdivided into smaller components.  As such, "component" can be any logical grouping in code, be it a project/subproject, module/submodule, package/namespace and/or a class.  The semantics and implications of code cohesion within modules is a topic for a whole other post.

# Proprietary Information

Let's take a different example.  How much should an `OrderController` know about the details of `Widget` inventory?  If you have the `OrderController` know the `Widget` database schema, it can do the exact queries that it needs to fulfill its business cases.  This is often much more efficient than writing _n_ different queries in the service layer.

The problem with this line of thinking is now you have _n_ different places that know how something (persistence of `Widget`s) is done.  Any change now has _n_ different consumers that need to change.  Beyond that, how each place consumes the schema might differ in their underlying assumptions about the datastore.  Let's say that you move the `Widget` service to a different database type.  Let's say we have to move from a server instance of the database to a managed cloud instance (like [RDS](https://aws.amazon.com/rds/)).  What if our consumers rely on features that aren't yet supported in the cloud version?  That means your migration has quickly become either infeasible or a costly rebuild.  Worse so, it's often not obvious when this promiscuous access happens.

# Leaky Abstraction

A singular component must own and have the responsibility for [_how_](/post/002-what-from-how/) something is done.  External consumers _must_ rely on _what_ said component does.  Otherwise you have a condition called a [leaky abstraction](https://en.wikipedia.org/wiki/Leaky_abstraction).  To state it another way, _leaky abstractions are when propriety information of how something is done is used by components who don't own that information_.

# Minimizing Ownership

It is ideal if we extract ownership of differing responsibilities to different components, so long as we maintain cohesion.  This means we should extract how to parse HTTP requests from a service component (that is, we want a `WidgetWebController` to handle that responsibility, not `WidgetService`).  This clarifies the earlier question of "where do you fix that"?  Smaller components are easier to grok.  Beyond that, "and" is usually a code smell.  If someone were to say to me, "the widget project handles web requests, business logic and persistence", the first statement I would respond with is: "I hope it is broken in to subprojects".

If someone were to say that "that component handles inventory management and sales orders", I would ask what that entails.  Sometimes a responsibility is so small that it may not justify having its own component.  Having an unmanageable constellation of component/services has its own costs.  That said, god objects and unwieldy monoliths tend to be the more likely failure cases of software architecture.

# Persistence Versus Service

Often, it's most tempting to just put persistence in with the business logic.  Those two things are often tightly entwined and persistence details don't often change.  I contend that separating persistence is still important.  It is much easier to unit tests and assert details about your business logic without standing up a persistence layer.  While things like [SQLite](https://www.sqlite.org/) and [Localstack](https://www.localstack.cloud/) exist, it's still easier to use these tools when the persistence responsibilities are split off.

If you look at a persistence service, you should be able to tell intuitively what indices you would need to create for a relational database.  You should also be able to sanity check if any of the signatures are nonsensical.  You also get access to Find Usages/Call Hierarchy, which are invaluable for debugging issues and evaluating refactors.  These are non-trivial benefits when maintaining the application.

In short, have persistence responsibilities owned by their own component.

# Orchestration

This all begs a question: "what does a controller component own?".  It owns how to transform input and output to requests from a particular medium.  It owns putting several collaborating components together to complete a use case.  It should not take responsibilities from the lower level services.

Consider the workflow needed to handle a use case.  We want to be able to update the `Widget`s that are in an `Order`.  We have an existing `OrderWebController` that we can update.
```
  PATCH /orders/1234
  {
    "widgets": [456, 798]
  }
```

Here is a brief overview of the responsibilities for the given components (from the lowest level going up):

### Widget Repository
- Query `WidgetEntity` table with given ids

---------------

### Widget Service _(Collaborators: Widget Repository)_
- Retrieve `Widget`s by id
  - Use Widget Repository to retrieve `WidgetEntity`s by id
  - Convert from `WidgetEntity` to `Widget` (service object)

---------------

### Order Repository
- Query `OrderEntity` table with given id
- Update `OrderEntity` / `WidgetEntity` join table

---------------

### Order Service _(Collaborators: Order Repository)_
- Retrieve `Order` by id
  - Use Widget Repository to retrieve `OrderEntity`s by id
  - Convert from `OrderEntity` to `Order`
- Add given `Widget`s to a given `Order`
  - Use Widget Repository to create link entities between `Order` and `Widget`s

---------------

### Order Web Controller _(Collaborators: Order Service, Widget Service)_
- Update an `Order` from a web request
  - Validate the user is authorized and has access to `PATCH /orders/{orderId}` endpoint
  - Parse path parameters and JSON body
  - Use Order Service to load and ensure `{orderId}` is a valid `Order` in the system
  - Use Widget Service to load and ensure that the given widget ids are valid `Widget`s in the system
  - Ensure user has access to said `Order` and `Widget`s
  - Use Order Service to add the loaded `Widget`s to the loaded `Order`

---------------

Please note these are responsibilities, not instructions.  Services can violate proprietary knowledge of the controller.  Assuming details about the callers (like the shape of the JSON request) violates proprietary information about the controller.  Just because the use case above is a web request, Widget Service should not assume that is always the case.

# Conclusion

Having only own owner of propriety information is critical to design.  That gives you a [Single Source Of Truth](https://en.wikipedia.org/wiki/Single_source_of_truth).  Having a single source of truth means you only need to fix one place if there's a bug.  It means having a single place that exists to redesign a component.  (the principle is bigger than just "don't copy/paste code blocks")

When you are writing an implementation, ask yourself "does this component own this information I'm encoding it to it".  Again, this can be difficult because the person coding the client and implementation are often the same person.  Answering this question requires wearing multiple hats.  The discipline to think about propriety when coding a solution is essential to avoiding "new engineer" mistakes.
