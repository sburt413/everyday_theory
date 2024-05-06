---
title: "Separate What From How" # Title of the blog post.
date: 2024-05-06T12:00:00-04:00 # Date of post creation.
description: "What you are doing should be separate from how you are doing it." # Description used for search engine.
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
  - Architecture
tags:
  - Antipatterns
  - Design
  - Naming
---

The primary point of good Software Design is to separate out _what_ your code does from _how_ your code does it.  Being able to make this distinction in real time when coding is critical.  Let’s explore.

# Core versus Shell

Look hard enough in the right places and you'll find the saying: ”Functional Core; Imperative Shell”.  It’s an interesting and useful mantra.  You could also equally say “Object Oriented Core; Imperative Shell”.  Here's another piece of truth embedded in said statement: _how_ you do something is separate from _what_ your code does.

Consider the case of a web server.  _What_ your web server does is respond to imperative commands coming in the form of HTTP verbs.  _How_ it does it is running a Spring/http4s/Zio/etc application.  If I as a consumer have just documentation of your endpoints, it doesn’t matter to me how you fulfill those requests.  Let's say I have a UI.  _What_ my UI does is drive your server and present data.  Inside your server code, you could have web controller code that drives service code.  Inside that server, you have web controllers driving service code.

# Separating Layers

These principles hold true whether the client is using HTTP/JMS/etc.  You have _what_ and all the piece of _how_ you do it.  To restate the critical point in a different way: _How_ a controller does something is composed of _what_ services does.  The "_how_" that the service layer does that can be made up of "_what_" the lower level services do.

| What                  | How                   |
| --------------------- | --------------------- |
| Widget Web Page       | Widget Web Controller |
| Widget Web Controller | Widget Service        |
| Widget Service        | Widget Repository     |
| Widget Repository     | Spring JPA            |

It's onions all the way down.

Getting both _What_ and _How_ correct are critically important.  A web page isn't broken if the controller feeding it the wrong data.  A web controller isn't broken if the service layer is doing a faulty calculation.  These layers of abstraction around _what_ and _how_ better declare who owns a particular responsibility.  This also better delineates the scope of what should be tested where.  Even better is that if you build up features from the service layer through the controller, you have much less needs for feature flags.  Other benefits will be explored in future posts.

(As a side note, _why_ you are doing something is usually expressed in comments or in Jira tickets.  At minimum one of those two things [lie](https://www.reddit.com/r/ProgrammerHumor/comments/82zg9t/code_never_lies/))

# You can not always derive What from How

Given a particular quantity of ingredients, do the following:

``` java
1) For each ingredient, place in food processor and blend together
  a) Tomatoes
  b) Apples
  c) Garlic
2) Heat oil in large skillet
...
```

_What_ are you doing in this case?  These directions are perfectly clear in _how_ to perform the tasks.  Now you could say one should be able to just read code and grok, but how many people caught the bug in the script?  Most people I hope, but I'm certain someone would just skip over the fact we put `Apples` in the [marinara recipe](https://www.allrecipes.com/recipe/11966/best-marinara-sauce-yet/).  _What_ we are doing (the title of the recipe) gives us critical clues as to intent.  (If we really wanted Apples in our Marinara recipe, we'd better express that intent in the name: "Apple Marinara Sauce")

What your code does should be evident entirely in the public function signatures.  One thing I will not stand for is when a software engineer claims they are bad at naming things.  That tells me that you cannot express _what_ you are doing.  Unless you are in a particular set of subdisciplines our field, most modern software engineers writing code need to be able to express _what_ they are doing.

_How_ you do something is the body of the function and any supporting private functions.  These can be subject to change, whether by extension, improvement or other refactorings.  Those are usually comparatively easy things to fix or clean.  They only need be indirectly tested as they are subject to modification at any point during a refactoring task.

## Unclear Intent as a Code Smell

[LISP](https://en.wikipedia.org/wiki/Lisp_(programming_language)) is very cool to look at if you are looking at something simple like a factorial function.  LISP is much less fun to look at when you are three levels deep looking at `cons`, `car` and `cdr` expressions.  The nature of the language implicitly encourages programmers to inline expressions and has powerful meta-programming functionality.  While that can be fun to work with, it requires a very high amount of discipline to use cleanly.  Please take some time to look up examples of LISP if you are unfamiliar with the language.  Look for non-trivial examples and take note of the following elements:
1) Whether the code can be quickly groked
1) Whether the code requires being very keen on domain experience in the language
1) Whether comments are used (or required) to explain the code

I, the humble author, am trying to emphasis a code smell.  Code that does not express _what_ it does is problematic.  LISP is a very extreme, if not impractical, language to prove this point but I think this provides necessary contrast to show my point.  If LISP requires intimate domain knowledge to understand the base intent of what a piece of code is doing, that is likely a problem.  If I say: "I'm cooking marinara sauce from scratch", then I would imagine most readers would have some mental imagery of what that task could look like.  This would hold true even if the reader has little to no practical experience cooking.  If you see a function call: `cookMarinaraSauce()`, you should be able to understand the intent.

# Further Topics

Splitting what from how allows us to derive which code has propriety knowledge of concepts.  This allows us to properly determine places to apply abstraction.  For example, does the widget controller own the idea of how a widget is persisted.  Abstraction and it's subtopics, such as unit testing and mocking, are left for future posts.

# Conclusions

REST endpoints are _what_ your web server does.  A `PUT` to `users/{userId}/settings` with a JSON body should be pretty obvious _what_ it intends to do.

REST endpoints are _how_ your web page functions.  You could refactor the application to use SOAP, if for some need be, and not change an element on the page.  You also should be able to successfully run all tests on the page.

On a practical level when dealing with code changes: _You only change WHAT an application does if you have a business requirement to do so.  You should not alter existing use cases when changing HOW an application does something._  If you do not split _what_ and _how_ correctly, you make your job (code maintenance and improvement) more difficult.

It can be difficult to split _what_ from _how_, especially if you are the one defining both sides of that equation.  It can be hard conceptualizing a problem in both terms and factoring out _what_ steps are as opposed to _how_ the steps are performed.  Honestly, this is a golden age that if you _can_ do said factoring, there are tools out there that perform 90%+ of the details of how to do things.  ORMs handle how to persist data.  Think of how ubiquitous JSON marshallers are.  The limits do more tend to how much you can imagine.

While there are interesting criticisms of relying too much on other people to do the details makes you [less of an engineer](https://medium.com/@ajouamymiloud/the-difference-between-an-engineer-and-a-frameworker-992474ea46d1), I think there value to equaling the playing field to allow for more people to produce useful software.  Yes, someone has to know the difference between [quicksort and mergesort](https://stackoverflow.com/questions/680541/quick-sort-vs-merge-sort) and [Big O notation](https://en.wikipedia.org/wiki/Big_O_notation), but how many use cases need that deep of engineering knowledge.  That said, not all engineers who possess those deep skills are good at naming things.

I will say this: I have handled more bugs and pain in my professional life caused by code that poorly expressing _what_ code does than code that had some tricky performance bug.
