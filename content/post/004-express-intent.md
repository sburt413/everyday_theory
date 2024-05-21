---
title: "Express Intent"
date: 2024-05-11T12:00:00-04:00
description: "Function signatures matter. A lot."
featured: false
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
  - Coding
tags:
  - Intent
  - Naming
---

Sometimes, it's what you don't say that hurts you the most.  Let's go.

# Side Effects

A warning before we begin, I will use the words "side effects" heavily in this post.  I come at this from the functional school of thought.  Quite honestly, in said school of thought, "side" effects can be the very necessary things your system does.  Writing to a database is technically a side effect.  If something happens in the system outside the expected return value, then I will refer to it as a "side effect".

Let's say we have an `addWidget` function, but in addition to writing the `Widget` to the database, it also updates a global count of `Widgets`.  The function name isn't `addWidgetAndUpdateGlobalCount`, so updating the global count is a hidden side effect of `addWidget`.

Rather than attempting to formally define the concept of side effect, or attempt to quantify "effect" versus "side effect" versus "hidden side effect", I should use the word "side effect" in any of those three cases.  Any mutation of input parameters will also be considered a "side effect".

# Signatures Matter

A function signature should be sufficient to tell entirely [what](/post/002-what-from-how/) a function does.  As a [consumer of code](/post/001-driver-and-service/), how a function does something should be immaterial to you.  If you must read the body of a function you are attempting to consume, that's serious a code smell.  Let's take [this](https://docs.oracle.com/javase/8/docs/api/java/util/Collections.html#reverse-java.util.List-) function signature from base Java:

``` java
public static void reverse(List<?> list)
```

We see that it take a `List` rather than `Collection`, which makes sense, order matters.  Now, does this mutate an input parameter?  Usually this is a no-no.  The name `reverse` doesn't tell us.  Is that a problem?  Nope.  That said, a _signature_ isn't just the name.  

Look at the return type: `void`.  If you ever see `void` for a return type, immediately think "side effects".  The obvious side effect in this situation is mutation of input parameters.  So we can pull that information from just those words.  

(A brief aside, `reverse` originates from the olden days of Java where, _at the time_, speed was a concern, thus mutation of input parameters was an acceptable tradeoff)

## Naming

Good naming requires extreme application of Murphy's Law.  Assume any possible misinterpretation of your function signature will happen, then address each of them.  Make [Amelia Bedelia](https://ameliabedeliabooks.com/) your spirit animal.  Keep mental lists of bugs you've encountered in the wild because bad naming habits.  When you write or touch function signatures, go through this list to make sure those bugs don't happen again.

Consider the following signatures:

``` java
  Widget createWidget(Widget widget)
```
Is the `Widget` I get back the same `Widget` I passed in?  That is ambiguous.

---------------

``` java
  Widget createWidget(String id, String name, double price)
```
Does `create` imply that the object is to be persisted, or just instantiated?  `save` is much less ambiguous.

---------------

``` java
  void saveWidget(String id, String name, double price)
```

Do we need `void` to signal the side effect?  It makes the intent more clear.  That said, the caller cannot retrieve the newly created object.  The call may actually need to do IO to retrieve the object.  In this case, we would want to return the new object.

---------------

``` java
  Widget saveWidget(WidgetData data)
```

For this, we'll make `WidgetData` as a [parameter object](https://refactoring.com/catalog/introduceParameterObject.html).  It is clearly the data we need to make a `Widget`, not the `Widget` itself.  That leaves it so that the return value is clearly the new resultant object.  The word `save` clearly indicates a persistence side effect.

# Ask For Everything You Need

A function should ask for everything it needs to fulfill its purpose.  Let's consider a `buildInvoice` function.  

Do you need `Widget`s to build one?  Are we declaring this function in a service or controller?

The class itself can ask for what it needs.  This can fulfill the needs of multiple functions in the class.  [Dependency Injection](https://en.wikipedia.org/wiki/Dependency_injection) is your friend.  It would make sense that a `WidgetService` would need a `WidgetRepository` or some other access to a persistence layer.  It would make sense that an `OrderController` might need the `WidgetService`.  Make sure that you pick things that logically make sense, and always pick from the same or deeper levels in the system.  Controllers should only pick from services.  Services should only pick from other services and repositories.

## Example

Consider this case in service code:
``` java
  enum StateCode { ... }
  record Widget(StateCode stateCode, String name) {}

  class InvoiceService {
    public void saveInvoice(Invoice invoice) { ... }
    public void saveInvoices(Collection<Invoice> invoice) { ... }

    public Invoice buildInvoice(Collection<Widget> widgets) {
      Map<Widget, Double> widgetToPrice = pricingService.getPrices(widgets);
      double subtotal = widgets.stream()
                               .mapToDouble(widgetToPrice::get)
                               .sum();

      double taxRate = billingService.getTaxRate(widgets.get(0).stateCode());
      double tax = subtotal * taxRate;

      double total = subtotal + tax;

      return new Invoice(widgets, total);
    }
  }
```

What's the issue with this code?  It doesn't ask for a critical piece of information: what state do we use for the sales tax?  It implicitly assumes that the first widget has the state that we should use for the tax (sales tax is collected in the state of the purchaser, not the original location of the merchandise).  In short: tell in the signature, don't ask in the body.

A good way to think about this is that is your function '_a function of_' the given data.  If you change _x_ does _y_ change?  If you change a parameter, does it change the output of the function?  If it does change the result then the function is _a function of_ that input.  Unfortunately, as illustrated in the example in this section, what should and should not be factored out in to the function signature can often be obscure.  If faced with ambiguity, err on the side of declaring the need in the function signature.

It is much easier to see the source of the issue in:
``` java
  widgetService.buildInvoice(widgets, widgets.get(0).stateCode())
``` 
than in:
``` java
  widgetService.buildInvoice(widgets)
```

# Say Everything That You Are Going To Do

At some point you will hit a situation where it is reasonable to assume domain knowledge.  Not every function signature and every piece of code is going to be readily understandable to an outside coder.  That said, don't make it such that an outside developer will take six months to get up to speed on your product.  Say everything that a function does in its signature.

Consider:
``` java
  class OrderController {
    private OrderService orderService;
    private WidgetService widgetService;
    private InvoiceService invoiceService;

    public void addToOrder(Order order, Widget widget) {
      Invoice invoice = invoiceService.getOrCreateInvoice(order);
      addLineItem(order, widget);
      invoiceService.addWidget(invoice, widget);
    }

    private void addLineItem(Order order, Widget widget) { ... }
    ...
  }
```

Let's say we had a new requirement that we need to reserve the widget in inventory before we can add it to a line item for the order:
``` java
  public boolean addToOrder(Order order, Widget widget) {
    boolean successfullyReserved = reserveWidget(order, widget);
    if (!successfullyReserved) {
      return false;
    }

    Invoice invoice = invoiceService.getOrCreateInvoice(order);
    addLineItem(order, widget);
    invoiceService.addWidget(invoice, widget);
  }
```

Let's look at the signature.  What does the return value `true` mean for `addToOrder`.  It means it was successful, right...  What does a `false` mean?  It means [RTFM](https://en.wikipedia.org/wiki/RTFM), and remember comments lie.

There is a cute way this code could unexpectedly fail.  Let's unveil more code in the private implementation details of the other functions:

``` java
  class OrderController {
    ...

    private void addLineItem(Order order, Widget widget) {
      if (Objects.isNull(widget.getId())) {
        widget = widgetService.createWidget(widget);
      }
      ...
    }

    private boolean reserveWidget(Widget widget) {
      if (Objects.isNull(widget.getId())) {
        // Cannot reserve non-existent widget
        return false;
      }
      ...
    }
  }
```

So under what condition would `addToOrder` stop working where it was 'working' before?  It would fail in cases where the `Widget` was not persisted prior to the `addToOrder` call.  The reservation call would return `false`, which would then cause `addToOrder` to return false.  The name `addToOrder` doesn't _say_ that it will persist the `Widget`.  Creating the `Widget` if it doesn't exist is a side effect.  Any code that depends on this side effect becomes very hard to debug.

Often you will find these "conveniences" (persisting the object for you) buried in service code.  Sadly, these sorts of "conveniences" are often depended upon in controller code.  It is often a hard and error prone process to untangle when there is controller level code buried in side effects of the service level.

## Function Naming

My rules for function naming are as such:
1) Express everything the function does in the name
1) Use the most concise wording

These rules are ordered.  Do not make the wording more concise if you do not express everything the function does.  This means, it is better to be verbose than concise.  

# Concision

There is a trick you can use to make signatures more concise: don't repeat yourself.  No, I do *not* mean the policy that is better stated as [Single Source of Truth](https://en.wikipedia.org/wiki/Single_source_of_truth).

You can shorten `getWidgetByWidgetId(Id widgetId)` to `getWidget(Id widgetId)` since 9 times out of 10 you will see it in code as `Widget widget = getWidget(widgetId)`.  Seeing repetition in the same line is usually a clue you can apply this.

# `get` Methods

Don't ever add side effects in a `get` method.  Factor out side effects from existing `get` methods.  If naming a function with side effects and `get` comes up, open a thesaurus if necessary, there is usually a better verb you can use.

# Conclusion

Don't suck at naming.  Naming is about breaking down a problem and expressing the solution to another person (perhaps a future self).  If you don't make it easy to understand the solution, you are not a good problem solver.  Remember, at the end of the day, you aren't a coder or developer, you are a problem solver.
