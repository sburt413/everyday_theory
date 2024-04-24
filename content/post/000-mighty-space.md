---
title: "The Mighty Whitespace" # Title of the blog post.
date: 2024-04-24T12:00:00-04:00 # Date of post creation.
description: "Consider a function as a collection of thoughts." # Description used for search engine.
featured: true # Sets if post is a featured post, making it appear on the sidebar. A featured post won't be listed on the sidebar if it's the current page
draft: false # Sets whether to render this page. Draft of true will not be rendered.
toc: false # Controls if a table of contents should be generated for first-level links automatically.
usePageBundles: false # Set to true to group assets like images in the same folder as this post.
# featureImage: "/images/path/file.jpg" # Sets featured image on blog post.
# featureImageAlt: 'Description of image' # Alternative text for featured image.
# featureImageCap: 'This is the featured image.' # Caption (optional).
# thumbnail: "/images/path/thumbnail.png" # Sets thumbnail image appearing inside card on homepage.
# shareImage: "/images/path/share.png" # Designate a separate image for social media sharing.
codeMaxLines: 10 # Override global value for how many lines within a code block before auto-collapsing.
codeLineNumbers: true # Override global value for showing of line numbers within code block.
figurePositionShow: true # Override global value for showing the figure label.
showRelatedInArticle: false # Override global value for showing related posts in this series at the end of the content.
categories:
  - Coding
tags:
  - Uncle Bob
  - Thoughts
  - Whitespace
---

I've seen multiple criticisms of Uncle Bob's _Clean Code_ involving the (short) length of functions.  I've also seen multiple great responses to those rules of thumb.  I want to derive another way to reason about the length of a function.  Let's begin.

# Thoughts

I want the number and density/complexity of *thoughts* to be the primary measure people think about a function.  Now, there's still [cyclomatic complexity](https://en.wikipedia.org/wiki/Cyclomatic_complexity), which is a valid metric.  I still think it is a valid point of view that there are complex function even without branching code paths.  Let's work through this with some code. Let's assume these shapes for the example code we’ll be working through:
``` java
  enum WidgetType {
      THINGY,
      DOOHICKY,
      WHATCHAMACALLIT
  }

  record Widget(WidgetType type, double unitPrice) {}

  record Discount(WidgetType widgetType, Double discountPercentage) {}

  class State {
      private String name;
      ... 
  }

  record Invoice(double total, List<Widget> items) {}
```

We want to build a function that builds an Invoice of some given widgets.  Let's sketch this out at a high level:
``` java
  public Invoice buildInvoice(List<Widget> items, State state, Map<WidgetType, Double> percentageDiscountByWidgetType) {
    // Look up discounts
    // Find the subtotal
      // Start with the unit price of each item
      // Apply applicable discounts
    // Apply sales tax
      // Look up Sales tax
      // Add sales tax to subtotal
    // Build the result
    // Persist Invoice
  }
```

# Thoughts to Code

As you can see in the prior block, we have five high level thoughts.  Let's put some code in there.  For the sake of making things messy, let's ignore that Stream exists in Java.
``` java
  public Invoice buildInvoice(List<Widget> items, State state) {
    // Look up discounts
    Set<WidgetType> widgetTypes = new HashSet<>();
    for (Widget item : items) {
      widgetTypes.add(item.type());
    }
    List<Discount> discounts = orm.query("SELECT widget_type, discount_percent FROM discounts WHERE widget_type IN :types")
                                  .withParameter("types", widgetTypes)
                                  .run();
    Map<WidgetType, Double> discountPercentByWidgetType = new HashMap<>();
    for (Discount discount : discounts) {
      discountPercentByWidgetType.put(discount.widgetType(), discount.discountPercentage());
    }
    // Find the subtotal
    double subtotal = 0.0;
    for (Widget item : items) {
      // Start with the unit price of each item
      // Apply applicable discounts
      Double discountPercentage = discountPercentByWidgetType.getOrDefault(item.type(), 0.0);
      double unitPrice = item.unitPrice - item.unitPrice*discountPercentage;
      subtotal += unitPrice;
    }
    // Look up Sales tax
    List<Double> salesTaxRates = orm.query("SELECT sales FROM tax_rates WHERE state = :stateName")
                                    .withParameter("stateName", state.getName())
                                    .run();
    double salesTaxPercentage;
    if (salesTaxRates.isEmpty()) {
      salesTaxPercentage = 0.0;
    } else {
      salesTaxPercentage = salesTaxRates.get(0);
    }
    // Add sales tax to subtotal
    double total = subtotal + subtotal*salesTaxPercentage;
    // Build the result
    Invoice invoice = new Invoice(total, items);
    // Persist Invoice
    return orm.insert(invoice).run();
  }
```

Now let's remove the comments, because comments lie:
``` java
  public Invoice buildInvoice(List<Widget> items, State state) {
    Set<WidgetType> widgetTypes = new HashSet<>();
    for (Widget item : items) {
        widgetTypes.add(item.type());
    }
    List<Discount> discounts = orm.query("SELECT widget_type, discount_percent FROM discounts WHERE widget_type IN :types")
                                  .withParameter("types", widgetTypes)
                                  .run();
    Map<WidgetType, Double> discountPercentByWidgetType = new HashMap<>();
    for (Discount discount : discounts) {
        discountPercentByWidgetType.put(discount.widgetType(), discount.discountPercentage());
    }
    double subtotal = 0.0;
    for (Widget item : items) {
        Double discountPercentage = discountPercentByWidgetType.getOrDefault(item.type(), 0.0);
        double unitPrice = item.unitPrice - item.unitPrice*discountPercentage;
        subtotal += unitPrice;
    }
    List<Double> salesTaxRates = orm.query("SELECT sales FROM tax_rates WHERE state = :stateName")
                                    .withParameter("stateName", state.getName())
                                    .run();
    double salesTaxPercentage;
    if (salesTaxRates.isEmpty()) {
        salesTaxPercentage = 0.0;
    } else {
        salesTaxPercentage = salesTaxRates.get(0);
    }
    double total = subtotal + subtotal*salesTaxPercentage;
    Invoice invoice = new Invoice(total, items);
    return orm.insert(invoice).run();
  }
```

Now let's move to Pascal styled declarations just to add more chaos (I've seen it in live Java code):
``` java
  public Invoice buildInvoice(List<Widget> items, State state) {
    Invoice invoice;
    double subtotal;
    double total;
    double salesTaxPercentage;
    Set<WidgetType> widgetTypes = new HashSet<>();
    for (Widget item : items) {
        widgetTypes.add(item.type());
    }
    List<Discount> discounts = orm.query("SELECT widget_type, discount_percent FROM discounts WHERE widget_type IN :types")
                                  .withParameter("types", widgetTypes)
                                  .run();
    Map<WidgetType, Double> discountPercentByWidgetType = new HashMap<>();
    for (Discount discount : discounts) {
        discountPercentByWidgetType.put(discount.widgetType(), discount.discountPercentage());
    }
    subtotal = 0.0;
    for (Widget item : items) {
        Double discountPercentage = discountPercentByWidgetType.getOrDefault(item.type(), 0.0);
        double unitPrice = item.unitPrice - item.unitPrice*discountPercentage;
        subtotal += unitPrice;
    }
    List<Double> salesTaxRates = orm.query("SELECT sales FROM tax_rates WHERE state = :stateName")
                                    .withParameter("stateName", state.getName())
                                    .run();
    if (salesTaxRates.isEmpty()) {
        salesTaxPercentage = 0.0;
    } else {
        salesTaxPercentage = salesTaxRates.get(0);
    }
    total = subtotal + subtotal*salesTaxPercentage;
    invoice = new Invoice(total, items);
    return orm.insert(invoice).run();
  }
```
# Maybe it’s easy because I know the answer…

Let's imagine this is your first time reading this code (or you are returning to it for the first time in six months).  You'd parse it line by line and mentally recreate the comments we started off with.  We have all had examples of when we've had to do this.

Now you could apply the _Clean Code_ standards and create several short subfunctions to break up this mess.  That in itself has problems.  This method would be called by a controller and quite possibly from a service layer.  It is already going to be two or three levels deep in the call stack at this point.  More levels of depth means more jumping around the file to determine what's going on.  Worse, this method is poorly written in that it has some persistence side effects buried in it.  Further burying these side effects in subfunctions would make these problems less obvious.

Let's do a simple exercise.  Let's replace the comments with just blank lines:
``` java
  public Invoice buildInvoice(List<Widget> items, State state) {
      Set<WidgetType> widgetTypes = new HashSet<>();
      for (Widget item : items) {
        widgetTypes.add(item.type());
      }
      List<Discount> discounts = orm.query("SELECT widget_type, discount_percent FROM discounts WHERE widget_type IN :types")
                                    .withParameter("types", widgetTypes)
                                    .run();
      Map<WidgetType, Double> discountPercentByWidgetType = new HashMap<>();
      for (Discount discount : discounts) {
        discountPercentByWidgetType.put(discount.widgetType(), discount.discountPercentage());
      }

      double subtotal = 0.0;
      for (Widget item : items) {
        Double discountPercentage = discountPercentByWidgetType.getOrDefault(item.type(), 0.0);
        double unitPrice = item.unitPrice - item.unitPrice*discountPercentage;
        subtotal += unitPrice;
      }

      List<Double> salesTaxRates = orm.query("SELECT sales FROM tax_rates WHERE state = :stateName")
                                      .withParameter("stateName", state.getName())
                                      .run();
      double salesTaxPercentage;
      if (salesTaxRates.isEmpty()) {
        salesTaxPercentage = 0.0;
      } else {
        salesTaxPercentage = salesTaxRates.get(0);
      }

      double total = subtotal + subtotal*salesTaxPercentage;

      Invoice invoice = new Invoice(total, items);

      return orm.insert(invoice).run();
  }
```

# Another Approach

Let’s assume each whitespace delimited block is a coherent chunk.  Let's examine the logical chunks and see if it is cohesive:
``` java
  Set<WidgetType> widgetTypes = new HashSet<>();
  for (Widget item : items) {
    widgetTypes.add(item.type());
  }
  List<Discount> discounts = orm.query("SELECT widget_type, discount_percent FROM discounts WHERE widget_type IN :types")
                                .withParameter("types", widgetTypes)
                                .run();
  Map<WidgetType, Double> discountPercentByWidgetType = new HashMap<>();
  for (Discount discount : discounts) {
    discountPercentByWidgetType.put(discount.widgetType(), discount.discountPercentage());
  }
```

The last line is a for loop that builds `discountPercentByWidgetType`, that seems reasonable.  At almost a dozen lines, this does seem like a candidate to factor out to a subfunction.

---------------

``` java
  double subtotal = 0.0;
  for (Widget item : items) {
    Double discountPercentage = discountPercentByWidgetType.getOrDefault(item.type(), 0.0);
    double unitPrice = item.unitPrice - item.unitPrice*discountPercentage;
    subtotal += unitPrice;
  }
```

The first line defines `subtotal` and each lines goes explicitly towards that goal.

---------------

``` java
  List<Double> salesTaxRates = orm.query("SELECT sales FROM tax_rates WHERE state = :stateName")
                                  .withParameter("stateName", state.getName())
                                  .run();
  double salesTaxPercentage;
  if (salesTaxRates.isEmpty()) {
    salesTaxPercentage = 0.0;
  } else {
    salesTaxPercentage = salesTaxRates.get(0);
  }
```

The last line defines `salesTaxPercentage` as either the found value or a default of `0.0`.  Pretty simple.

---------------

``` java
  double total = subtotal + subtotal*salesTaxPercentage;

  Invoice invoice = new Invoice(total, items);

  return orm.insert(invoice).run();
```

These are all simple one liners.

---------------

One of the most important pieces of advice a profressor gave me was to always put a new line before and after each control block (loops, conditionals, etc).  Articulating a reason why didn't come to me until well into my career.  _If you have a control block, you have a complex thought that should be reasoned about independently._

If we extend that to more than just control blocks, we can generalize the advice to: Add new lines between all independent thoughts in a function.

# Conclusion

Obviously, this is just advice, take it or leave it.  Each company/team is going to accept differing levels of cognitive load in a single function.  The thing that this gives you is that you can pragmatically factor out subfunctions from larger functions with ease.

Try this the next time you are having to work with a legacy block of code: add new lines to seperate the independent thoughts in an existing long or complex function.  If you need to shuffle some things around, maybe you found some underlying complexity in the function that needs other refactoring.  Aside from that, I've never seen just a couple of blank lines make a `git diff` incomprehensible.
