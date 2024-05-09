

# 003-software-engineering-as-bets

Once in college a professor and student playing (on an actual board with a clock, not online) and they both played the exact same opening.  Neither the professor nor student were actually paying attention to what the other was doing.  They were almost blindly moving pieces regardless of what the opponent was doing.  Let's see how this relates to software engineering.

There are people who can competently gamble to the point it is their literal profession.  In our profession, it's a bit more obtuse.

## Architecture

Each abstraction you introduce creates a cost.  Indirections make code harder to follow.  It _does_ makes it cheaper to change your mind later.  

Let's say you introduce promiscuous raw SQL in to your code.  You needed to migrate a group of tables to a different database server.  Now you have to grep the entirety of your code to find all the references to the tables and manually pull things apart.  That is a lot more expensive and dangerous than if you had put in a layer for a persistence service.  Starting by creating the persistence service has an initial cost.  That's the money you put down on the bet.  If you never have to change the persistence, you lose the time you put in.  If your service is designed wrong, you lose the bet and more.  

Whether to make the bet or not depends on the cost of putting down the bet and the payoff for whether you are right or not.  Just a humble reminder, sometimes being wrong once in your career is enough to inform the rest of your career.  Sometimes you don't know the cost of the bet _until_ you lose it once in your career.

### Standard Bets

Another interpretation of what I'm saying is that [Model-View-Controller](https://en.wikipedia.org/wiki/Model%E2%80%93view%E2%80%93controller) is indeed a bet.  It probably has 100-1 odds in your favor, but it's still a bet.

Microservices are a bet.  You don't get orchestration of a kubernetes cluster for free.  It requires specialized knowledge and tools to manage.  What do you get for that cost: a mature tool stack and scalability.  That said, I can teach a new graduate how to deploy a war file into an old Tomcat instance.  If your application can be serviced by a single server over the course of the applications lifetime, what's the better choice?

Most industry standards are there for a reason.  Most of the reasons suit most of the people's use cases.  Just remember that not everything sticking out of a board in a woodworking shop is a nail.

## Tools of the Trade

So "it depends".  It depends on the value of the objective.  It depends on the opportunity cost in not pursuing other objectives.  It depends on when you can achieve it.  It depends on known and unknown unknowns.  It depends on the schedule.  A good software engineer will use the tools at their disposal to reduce the risk.

### Frameworks

A framework is a bet.  If you choose Hibernate for persistence, it's has costs, risks and future implications for your application.  That said, you have a strong community to support you if you have problems and a well of talent to draw from if you need more experts on your team.  

Libraries usually offer you more flexibility as your code drives the library code.  Libraries are [_how_](/post/002-what-from-how/) your application does something, and thus able to be refactored.  Swapping out [Guava](https://guava.dev/) for another library isn't going to require a complete rethink of everything else in your app.

This all comes down to: "What's the cost/value tradeoff for using other people's code".  Having others actively doing maintenance, or even being able to do so yourself, is a valuable option.  Having external idioms that don't need to be taught to new hires is a valuable option. 

Remember, though, if you use other people's services, you get what you pay for.  If you don't have an SLA and are dependent upon a third party service, your don't control all the on/off switches to your application.  For a community fan site, that probably is ok trade.  For a banking app, less so.

### YAGNI

[YAGNI](https://en.wikipedia.org/wiki/You_aren%27t_gonna_need_it) is a common bet that doing work now is not necessarily going to yield results in the future.  Interestingly enough, this can apply to system design.  Is your application going to outgrow a monolith?  Does every entity in your domain need its own bounded context?  Maybe your team has the expertise and existing infrastructure to make the cost of adding the services to the cloud negligible.

Be aware of both the cost, now and in the future, and the value.

### Iteration Cycles

There exist some blackjack tables that allow you to [surrender](https://www.casino.org/blog/when-should-you-surrender-in-blackjack/).  You lose the hand, but at only half the cost of your initial bet.  Some times after seeing your first two cards the best option is to walk away from the hand.

The most valuable tool, by far, that you can have in this profession is short iteration cycles.  Having short iteration cycles means having more feedback more often.  More feedback yields more informed decisions.  Informed decisions are safer bets than otherwise.

If a project is going to take three months, is it worth it to take a week to do a proof of concept?  That depends on the certainty of design.  It also depends on deadlines.  If you aren't up against a deadline, upping the cost to 3.25 months might be within your calendar budget.  This is especially true if you determine that first week that your plan was flawed from the start.  That is a lot cheaper than trying to integrate changes in at week 12 and finding said fatal flaw.

Even if you are certain of the design, starting with an [MVP](https://en.wikipedia.org/wiki/Minimum_viable_product) and iteratively building it up offers you flexibility in your plan.  If you start running out of runway before a release, it's better to have a working product.  It's easier to determine what you need to cut from the plan, rather than wonder what you can cram in.  Having an MVP and iterating usually builds more good will with stakeholders.  Good will is like playing with house money when you place your future engineering bets.

## Conclusions

Just make sure you aren't blindly moving all the pieces on the board.