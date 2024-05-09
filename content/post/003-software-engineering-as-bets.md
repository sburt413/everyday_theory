---
title: "Software Engineering as Bets" # Title of the blog post.
date: 2024-05-09T12:00:00-04:00 # Date of post creation.
description: "Engineering decisions should be viewed as future bets." # Description used for search engine.
featured: false # Sets if post is a featured post, making it appear on the sidebar. A featured post won't be listed on the sidebar if it's the current page
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
  - Design
  - Chess
  - Bets
  - YAGNI
---

Chess has a long history of structured openings.  You have a set of steps that put you in a certain position, with known strengths and weaknesses.  You basically start the actual game several moves in.  From the opening, you have several options to branch out from.  

There is also a common nomenclature.  If I say, ["Queen's Gambit"](https://en.wikipedia.org/wiki/Queen%27s_Gambit)... honestly most people will think of the Netflix series.  In reality, the term does refer to an actual opening that a chess expert should be able to visualize from just the name alone.  Presumably, an expect would be able to think of several counters to the opening.  As a senior software engineer, you should have several design patterns to solve common problems in your head... and know the potential downfalls for each one.  

Let's go further in to this topic.

# Cost and Value

The answer to every question in Software Engineering is "it depends".  There is an obvious follow up question: "What does it depend on?".  To answer that you need to know what  your resources are and your position.  In this case, it's always good to refer to [RFC 1925 Section 2, Point 6a](https://datatracker.ietf.org/doc/html/rfc1925#section-2): Good/Fast/Cheap, Pick 2 (I am aware of the fact this advise is repeated in several places).  What of those three matter most?  The answer depends on the context.  Consider these two features:
- Feature A: this is a table stakes feature in your industry and your emergent product doesn't have it
- Feature B: there is a growing market need for a new feature to take advantage of a new opportunity

Each situation evaluates Good/Fast/Cheap in different ways?

The cost and value of doing something can also be weighted against how long it will take to do it.  Don't underestimate the value of time.  On a chessboard if you can set up checkmate in 5 moves, it doesn't matter if your opponent can checkmate you in 3.

Say you can build the equivalent of a [category killer](https://en.wikipedia.org/wiki/Category_killer) in a software market.  If it takes you 5 years (e.g. forever) to get something to market, your competition can likely beat you to the punch.  Beyond that, five years is also plenty time for the market to move in a different direction.

Another point: you don't buy a $100 on a safe to store $50.  Engineering time is finite and costly.  You only have so much space on the schedule.  Always be mindful of the value you are getting.  Don't spend that time adding perfect code coverage to a tangential module if it is unlikely to get changed.  Don't rewrite working code without getting a tangible benefit.

# Predicting the Future

You can't predict the future.  Your product, even if best in class, could still get beaten out by competitors.  No plan that you make is guaranteed to yield the predicted outcome.  To address this, Agile in it's purest form is meant to reduce iteration cycles so that plans can adjust based on the current situation.  That leave us with two questions: why do we plan and what is a good plan?

## Bets

We have finite resources that can be spent on tasks that have potentially ill defined value.  Let's go back to chess for a second.  You are moving your pieces so that you control and threaten more of the board.  Gaining that position takes time (moves) and material (pieces).  On your move, you have to decide whether moving one piece _now_ gives you an advantage, without opening yourself up for problems later in the game.

If you dedicate 100% of your calendar to new feature development, you leave yourself open for bugs to slip though and code quality issues.  You also risk doing feature work that will yield zero value.  You also leave no spare capacity if critical issues or opportunities comes up.  

If you don't build features, competitors can leave you in the dust.

So if you are doing all this without perfect knowledge, what are you doing?  The answer making bets.

The key to being a good software engineer is giving yourself options in all situations and minimizing the cost of "failure".  Google has had a culture where they literally celebrated failure.  Don't let anyone convince you failure isn't an option, because it's certainly always a possibility.  No amounts of scrum or planning or Jira tickets or spreadsheets removes that possibility.

# Estimates

What is a good bet in our profession?

Let's consider a project that has an estimated time of three month.  That seems like a reasonable statement: "three months".  Statistically, though, this doesn't tell you nearly enough.  Say I went to a sports book and was given an option to bet on a game between Newton or Middletown.  The thing is the odds are missing.  I'm missing critical information of cost and value.  It is illogical to bet on the underdog without being offered a better payout.

Back to our estimate, does "three months" mean: "this will take no more than three months"?  Does it mean: "50/50 shot this gets done in three month"?  Is that just a [SWAG](https://en.wikipedia.org/wiki/Scientific_wild-ass_guess) that should never, ever be shown to management?  The differences in odds changes a plan from feasible to impractical.

If I was not in engineering, I would imagine that I would want to both know what I'm getting in terms of features and how likely I'm getting it on schedule.  Does slippage in the schedule matter?  Does slippage in the functionality matter?  Defining where are the timelines and stretch goals are important.

As an engineer, estimates are a way to tell people the odds.  If someone is forcing you to make a bad bet, make sure that you convey all the necessary information.

# Conclusions

There are several levels of expertise in our profession:
1) Saying it depends
1) Knowing what it actually depends on
1) Knowing what it depends on and being able to enumerate the options going forward with their costs and risks

The best engineers aren't psychic.  Even if they could read minds, they can't predict the future.  The best engineers enumerate, communicate and reduce the known risks and have contingency plans.  They understand and communicate all the costs and values.  They fail gracefully and early when there are no better options.  They make the smart bets and put in all the necessary hedges.  At the end of the day, all professional engineers are gamblers.
