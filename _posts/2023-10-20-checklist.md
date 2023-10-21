---
layout: post
title: "Code review checklist"
---

A suggested checklist before publishing code reviews, with rationale for the checklist items and a discussion of why you might consider using such a checklist in the first place.

### The checklist

Before publishing a pull request, attest to the following (with ❌ or ✅ symbols):

Commit messages
- My commit messages describe the software’s new behavior.
- My commit descriptions explain the reason for the change.

Tests
- I have protected new system assumptions with tests, each named for the assumption it protects.
- I have seen each new test fail when the assumption it protects is violated.
- My test code and production code follow the same code cleanliness standards.
- I wrote my tests at the cheapest layer which can falsify their protected assumptions.

Compounding value
- I have not introduced unnecessary mutable state.

Brevity
- My comments refer to the code’s current state, not its anticipated evolution.
- I have removed tests which protect obsolete assumptions.
- I have removed code not covered by the test suite.
- I have not allowed a characteristic of the system to be encoded more than once.
- I have defined variables at the smallest scope from which they are referenced.

Deployment discipline
- I will monitor the propagation of this change to all deployment destinations.
- If there is a problem during deployment related to this change, I will push a revert commit first and then diagnose the error locally.

### Why these checklist items?

These items have come up hundreds of time over 15+ years of writing and reviewing code professionally.

Writing checklist items in first-person, active voice emphasizes the submitter's responsibility to meet these items as team expectations, even under pressure.

#### Commit messages

Code archaeology matters: how long has that code been there? Did the last dev to touch that line do it to implement a feature, fix a bug, or as part of a standalone refactoring commit?

> My commit messages describe the software’s new behavior.

When you need to search for which commits enabled which changes, good commit messages are the difference between an effective search and a frustrating one.

> My commit descriptions explain the reason for the change.

Why was the change happening? Feature? Bug? dependency upgrade? Refactoring? Are there related tickets or work items? Your reasons don't have to be "official," or even good, but the reasons should exist, and you should describe them. Otherwise, why bother?

#### Tests

If you can't demonstrate what's new and different about your code with tests, how do you know it's even remotely close to what you need?

> I have protected new system assumptions with tests, each named for the assumption it protects.

Tests protect assumptions about your system's behavior by failing when those assumptions are violated. They vigilantly protect these assumptions on every change without any additional human effort. As Agent Smith says, "Never send a human to do a machine's job."

> I have seen each new test fail when the assumption it protects is violated.

Tests make you feel good when they pass, but they only save you trouble if they fail when they need to. The cheapest time to see a test fail is when adding the code which makes it succeed.

> My test code and production code follow the same code cleanliness standards

No duplication, descriptive variable names, etc. Test code is code. Rules for readability and maintainability don't change based on whether code resides in a directory for test code instead of production code.

> I wrote my tests at the cheapest layer which can falsify their protected assumptions

Test effort is finite. Using an integration test to protect an assumption which could be protected by a build-time unit test is monumentally wasteful.

#### Compounding value

Not only should your system be able to do more over time, it should get easier to add capabilities to the system over time. Software is a compounding asset.

> I have not introduced unnecessary mutable state.

Any code touching data relying on mutable state could be affected by further code that modifies that state. New code reading mutable state has to make sure the state can't be written somewhere inconvenient.

Early on, adding state feels so natural, like telling a story. The problem is that scanning through code referencing stateful components every time you write code referencing that state slows the team down *fast* as the code base evolves.

#### Brevity

> My comments refer to the code’s current state, not its anticipated evolution.

Don't clutter code describing what *does* happen with speculative explanations of what *might* happen. Let your backlog capture what's coming next. Who knows? You might have to pivot.

> I have removed tests which protect obsolete assumptions.

Why would you keep them?

> I have removed code not covered by the test suite

If not, provide the reason to retain that code.

Remember there are two strategies for increasing line and branch coverage: adding tests and removing untested code. If the untested code is obsolete, removing it is the much better choice.

> I have not allowed a characteristic of the system to be encoded more than once.

Don't Repeat Yourself (DRY). I will sidestep the irony of restating the finer points of DRY here.

> I have defined variables at the smallest scope from which they are referenced.

E.g. a value used in a single method shouldn’t be defined in a global constants file.

When variable scope is too large, it needs a longer name to keep it free of collisions with other variables in that scope. This often takes the form of describing the scope where the variable is actually used. Minimizing the scope of your variables makes it easier to use intuitive names and avoids cluttering broader scopes with needless symbols.

#### Deployment discipline

This is an up-front commitment for how the author will behave after pushing the PR's commits.

> I will monitor the propagation of this change to all deployment destinations.

The whole team is responsible for keeping its deployment pipelines clear. Pushing new changes is an ideal time to ensure the pipeline is healthy.

> If there is a problem during deployment related to this change, I will push a revert commit first and then diagnose the error locally.

This is especially important for busier pipelines. Agreeing up-front to revert problem changes helps keep the deployment pipeline clear for other features which are ready to go.

### Why bother with a code review checklist?

Checklists can save time and frustration on both sides of code reviews by clarifying baseline team values. Submitters will have fewer surprises during the review itself. Customers will benefit from systems where code reviews can focus on clarity, maintainability, and deeper concerns. The team's code base will be easier to understand and more edifying to emulate.

Because more general feedback is handled during self-review, the team has more time, effort, and adrenaline available for meatier disputes in the substance of the code change.

Checklists teach team members team values and coding practices automatically.

Finally, when an item is on the checklist, it proves that enforcing that item isn't personal. The checklist applies to all team members and code changes equally. Pull requests don't get special treatment based on who wrote it or who reviewed it. Brand-new team members can point out checklist violations of even seasoned team members.

### Can I keep adding to this? More is always better, right?

Make sure your team is on board with a checklist and any amendments thereto, lest they rightfully become resentful.

Take care to avoid unprincipled additions to the checklist. It's tempting, maybe even justified, to bypass items which are poorly-motivated. Unprincipled items on a checklist remove the list's ability to teach. Using a good checklist should be a welcome exercise that reduces work for the team over all.

Finally, keep the list short enough to use.

