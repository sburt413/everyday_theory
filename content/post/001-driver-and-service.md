---
title: "Driver and Service" # Title of the blog post.
date: 2024-04-30T00:00:00-04:00 # Date of post creation.
description: "The code that uses a service has differing concerns than the code it calls." # Description used for search engine.
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
  - Driver/Service Code
  - Antipatterns
  - Garbage In; Garbage Out
  - Multiple Hats
---

Code has one of two roles: 
- Add value to your business
- Drive the prior bullet point

The first bullet point is usually called _service_ code.  It's the service your company is being paid to do.  The second bullet point can be one of many things, it could be controllers/webapps/etc.  It can be cron jobs that perform various functions.  It's the code for how people interact with the system.  Their functionality is usually described in the 'Use Case' field of the Jira ticket.  I'll favor of the word 'driver' for this article when talking about any production code that exercises the service.

Why do these distinctions matter?  Let's dig in.

# Separation of Concerns

I'm also sure many of you have heard the advice a function should only do one thing.  Let's take a look at a low level service function (`addWidget`):

``` java
  interface WidgetService {
    public List<Widget> getWidgets(String widgetIds);
  }

  class InvoiceService {
    private WidgetService widgetService;

    public Invoice getInvoice(String invoiceId) { ... }

    private void addLineItems(Invoice invoice, List<Widget> widgets) { ... }

    public void addWidgets(String invoiceId, Collection<String> widgetIds) {
      Invoice invoice;
      try {
        invoice = getInvoice(invoiceId);
      } catch (InvoiceNotFoundException infe) {
        throw new InvalidInvoiceException(infe);
      }

      List<Widget> widgets;
      try {
        widgets = widgetService.getWidgets(widgetsIds);
      } catch (WidgetNotFoundException wnfe) {
        throw new InvalidInvoiceException(wnfe);
      }

      if (!invoice.isActive()) {
        throw new InvalidInvoiceException("Invoice is not active");
      }

      addLineItems(invoice, widgets);
    }
  }
```

I will contend that `addWidget` doesn't convey everything that the function does, and that it does *multiple* things.  It has critical concerns that are not expressed in the function name.  What if I used this function to rectify an invoice that had a problem?  I, as a programmer, could guess from the signature `addWidget(String, Collection<String>)` that it's likely to load data, but I'm not going to guess that it only works on *active* invoices.  So the name would have to be changed to `addWidgetToActiveInvoice`.  Otherwise the person reviewing the code would have to dig through the implementation details to grok the function.  

If we were being more pedantic, I could argue the name should be `validateAndAddWidgetToActiveInvoice`.  The word `And` being an obvious [code smell](https://en.wikipedia.org/wiki/Code_smell).

Let's jump back to the use case where we are rectifying an inactive invoice.  How do I resolve that situation?  Let's consider how we could do this:
- Don't touch `addWidget` so that we aren't touching "working code", create an `addWidgetToInactiveInvoice` function and factor out common code between the two
- Rename `addWidget` to `addWidgetToActiveInvoice`, create a `addWidget` method that does exactly what it says it does and factor out common code between the two

The first option has the benefit of a smaller source control diff.  That said, you are likely to have other developers hit the same snag you do when they read the existing code.  The second option creates a much larger diff and touches points in the code that are unrelated to your current ticket.

This problem grows in magnitude by the number of callsites you have for the function.  It further grows if those callsites are not easily groked.  This shows the point that you must take great care to not bury controller concerns in service code.

# Validation is a Driver Level Concern

Let's consider this approach.  Let's make `addWidget` do what it says, nothing more.  That means that if a driver cares about whether the invoice is active or not, the driver will do the check explicitly.

``` java
  interface WidgetService {
    public List<Widget> getWidgets(String widgetIds);
  }

  class InvoiceService {
    private WidgetService widgetService;

    public Invoice getInvoice(String invoiceId) { ... }

    private void addLineItems(Invoice invoice, List<Widget> widgets) { ... }

    public void addWidgets(Invoice invoice, Collection<Widget> widgets) {
      addLineItems(invoice, new ArrayList<>(widgets));
    }
  }
```

Now this doesn't solve the issue of having to go through existing code paths that drive `addWidget` and updating them.  However, this does go to show that _validation is a driver level concern_.  Things that are invalid in one case are not necessarily invalid in another use case.  A batch process is not necessarily going to have an active `User` to validate against Access Control Lists.  Admin tools are likely going to need to get around certain business requirements to be maximally useful.

Let's drive this home with one more common example: web controllers.  Let's imagine in this case we have our `addWidget` that (implicitly) does validation.  Let's (poorly) write some web controller code:

``` java
  class InvoiceWebController {
    private InvoiceService invoiceService;
    ...
    public HttpResponse<Invoice> updateOrder(String invoiceId, Collection<String> newWidgetIds) {
      Invoice invoice;

      try {
        invoice = invoiceService.addWidgets(invoiceId, newWidgetIds);
      } catch (InvalidInvoiceException iie) {
        return HttpResponse.badRequest(iie.getMessage());
      }

      return HttpResponse.ok(invoice);
    }
  }
```

I will argue that a user giving a bad id is completely expectable.  This also suspciously looks like using exceptions to control code flow.  Both these practices are antipatterns.

It _does_ makes sense for a controller to validate inputs.  The controller know how to properly respond to its use case.  The proper way this web controller should look is:

``` java
  class InvoiceWebController {
    ...
    public HttpResponse<Invoice> updateOrder(String invoiceId, Collection<String> newWidgetIds) {
      // Refactored invoice and widget services to return empty values for non-existent ids
      Optional<Invoice> possibleInvoice = invoiceService.getInvoice(invoiceId);
      if (possibleInvoice.isEmpty()) {
        return HttpResponse.notFound(INVOICE_NOT_FOUND);
      }
      Invoice invoice = possibleInvoice.get();

      List<Widget> widgets = widgetService.getWidgets(newWidgetIds);
      
      List<String> missingWidgetIds = findMissingWidgetIds(newWidgetIds, widgets);
      if (!missingWidgetIds.isEmpty()) {
        return HttpResponse.notFound(WIDGETS_NOT_FOUND + ": " + missingWidgetIds);
      }

      invoiceService.addWidget(invoice, widgets);

      return HttpResponse.ok(invoice);
    }
  }
```

Web controllers take a HTTP request and translate the date data into an ingestable format.  Then they drive service code.  That's their purpose.

# Data Loading is a Driver level concern

Here's a tricky case.  What's wrong with this code:

``` java
  interface InvoiceService {
    void addWidget(String invoiceId, String widgetId);
  }
```

Validation is a driver level concern!  Ok, ok, we know _that_ is an issue.  What else?

Did you make the presumption that `Invoice`s and `Widget`s reside in the same database?  What if they are in different microservices?  This can lead to interesting problems because you are presuming another thing about the caller.  You're presuming that the loading a *single* invoice and widget is the only IO the caller needs to perform.  Ok, let's upgrade this service:

``` java
  interface InvoiceService {
    void addWidget(String invoiceId, Collection<String> widgetIds);
  }
```

We've solved it, right?  Nope.  It's not that easy.  Let's consider the case that we actually have a batch controller that actually takes a batch of several invoices with their respective widgets.  That will still produce _n_ queries per batch.  The solution is to do this:

``` java
  interface InvoiceService {
    default Optional<Invoice> getInvoice(String invoiceId) {
      List<Invoice> foundInvoices = getInvoices(Collections.singletonList(invoiceId));
      if (foundInvoices.isEmpty()) {
        return Optional.empty();
      } else {
        return Optional.of(foundInvoices.get(0));
      }
    }

    List<Invoice> getInvoices(Collection<String> invoiceIds);

    void addWidget(Invoice invoice, Collection<Widget> widgets);
  }
```

This way, a controller can determine how to optimally perform IO.  That means the code driving this service would look like.

``` java
  interface WidgetService {
    default Optional<Widget> getWidget(String widgetId) { ... }
    List<Widget> getWidgets(Collection<String> widgetIds);
  }

  interface InvoiceService {
    default Optional<Invoice> getInvoice(String invoiceId) { ... }
    List<Widget> getInvoices(Collection<String> invoiceIds);
  }

  class InvoiceWebController {
    private WidgetService widgetService;
    private InvoiceService invoiceService;
    ...
    public HttpResponse<Collection<Invoice>> updateOrders(Map<String, Collection<String>> invoiceIdToNewWidgetIds) {
      List<Invoice> allInvoices = invoiceService.getInvoices(invoiceService.keySet());
      
      List<String> missingInvoiceIds = findMissingInvoiceIds(invoiceService.keySet(), allInvoices);
      if (!missingInvoiceIds.isEmpty()) {
        return HttpResponse.notFound(INVOICES_NOT_FOUND + ": " + missingInvoiceIds);
      }

      List<Widget> allWidgets = widgetService.getWidgets(flatten(invoiceIdToNewWidgetIds.values()));
      
      List<String> missingWidgetIds = findMissingWidgetIds(newWidgetIds, widgets);
      if (!missingWidgetIds.isEmpty()) {
        return HttpResponse.notFound(WIDGETS_NOT_FOUND + ": " + missingWidgetIds);
      }

      // Transform the map of id->ids to map of Invoice->Widgets
      Map<Invoice, List<Widget>> invoiceToNewWidgets = mapValues(invoiceIdToNewWidgetIds, allInvoices, allWidgets);

      invoiceToNewWidgets.forEach(invoiceService::addWidgets);

      return HttpResponse.ok(invoiceToNewWidgets.keySet());
    }
    ...
  }
```

# Garbage In; Garbage Out

You could ask for the case of `InvoiceService`: what if I as the service get an invalid `Widget`.  The answer is simply that you have a programming error in your driver code.  The statement ["Garbage In; Garbage Out"](https://en.wikipedia.org/wiki/Garbage_in,_garbage_out) exists for a reason.  If you want to guard against someone creating a rogue `Widget` instance, qualify the constructors with `private` or `protected`.  Beyond that, you can use static analysis to find all the callsites that call the constructor (IntelliJ's "Call Hierarchy" is your friend).  Make it so that the only code paths that instantiate the object are _inside_ the service code.  If you truly believe there is some unavoidable risk to delgating this responsibility to the driving code, I humbly suggest updating test suites and revisit the issue.

Failing early in the driver code prevents classes of bugs that are very difficult to debug.  Burying IO as a side effect makes it difficult to reason about the performance of your code.  Surfacing the IO make it makes it much easier to reason about and optimize your code.  Explicit is always better than implicit.

# Conclusion

There is a hidden danger in burying driver level concerns in service code.  Those hidden presumptions will eventually yield actual costs.  Finding these hidden presumptions can be difficult because usually the same person writing the code being driven is writing the code doing the driving.  Get used to wearing different hats simulatenously.  Be your own devil's advocate.  Instead of just asking "what could go wrong with this code block", also ask "what assumption about this function could a caller overlook".

For now, if you are writing a service, don't presume:
- What is an error in the driving code (if it is even an error)
- How callers wish to structure their IO

We'll explore more examples in future posts.