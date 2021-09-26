---
layout: post
title: "On Shrinking Large Methods"
---

# How big is too big?

Short-term memory capacity in humans has long been cited at 7±2 "things," with later research being more pessimistic about our ability to juggle. For software developers, this becomes a constraint worth applying to our code, since what matters for the maintainability of the code is a human's ability to understand it. Computers, of course, don't care, hence polyfills, Babel, minifiers, compilers, and interpreters.

Smaller is better with the "things" we're juggling, especially since we'll want margin left over for developers to have mental slots to consider details from elsewhere in our program, too. Short-term memory capacity, even for very sharp developers, is finite.

# How do we break up a large method?

* Conceptual level: Problems often come in layers as we break these down. The topology of your methods should mirror the layers in your problem such that no method should need to deal with the problem at multiple conceptual layers.
* Let header comments guide you: These suggest themselves as helper methods, often naming themselves as well.
* Limit the depth of nesting: Nesting in the code bodies is often a concrete hint that you're intermixing conceptual levels.

We often think of modular decomposition (breaking our problem down into smaller and smaller subproblems until the code is obvious) as something we do during the creative process of solving novel problems--and it is. But what we often miss is when there are subproblems already broken down, staring us in the face. A web controller method body which parses a request, dispatches appropriate behavior, implements those various behaviors, and interfaces with the storage layer and external dependencies would clearly be doing too much.

