---
layout: post
title: "Code is a Liability"
---
Software is an asset. Code is a liability.
 
Lines-of-Code corresponds [little](https://github.com/EnterpriseQualityCoding/FizzBuzzEnterpriseEdition/tree/uinverse/src/main/java/com/seriouscompany/business/java/fizzbuzz/packagenamingpackage) with how valuable a software system is while corresponding quite a bit with its cost to produce and maintain.

Each line of code is an opportunity for [complecting](https://www.youtube.com/watch?v=SxdOUGdseq4&t=1894s) your implementation with your problem domain, confusing fellow programmers, and quibbling during code reviews. Each symbol (variable, method, type, etc.) defined makes it harder to find concise names which do not conflict with one another. Each piece of state introduced is another character to track in the epic fantasy novel which is your team's code, except that if anyone introduces a plot hole your customers will experience it as a bug.

A larger code change takes more time to review, and when reviewed, is more likely to miss larger maintenance issues due to the size of the change itself. Larger code bases require more physical scrolling and reading to understand. It's less likely that a short snippet of code will be meaningful in isolation if you want to discuss it with a teammate who doesn't have a copy of the entire project locally.

With code, less is more.

### Writing less code

Establish conceptual clarity around your problem and solution in your own mind first. Consider the improbability of clear code arising from an unclear mental model.

Choose your names from vocabulary anyone interested in your problem will share.

Avoid method names which merely restate their implementation. The point of a name to be shorthand. Keep it short.

Restrict the scope of any symbol definition as much as you can. Notice how often the start of a constant's name is little more than an address of the (often lone) place in the code which refers to it. If the remainder of the constant's name is a restatement of the value, the named constant is strictly worse than inlining the single occurrence of the value.

Not everything is a class. If you can conceive of only a single possible implementation for an interface, or only a single relevant instance of a class, you may be pushing domain logic into your type system rather than using your type system to help maintain guardrails around your domain logic.

Some programming languages are more verbose than others for your intended style of programming. If you want to write Java in a functional style, you can expect to declare almost every symbol `final`, and your colleagues will probably notice and discuss this. If you want to write Clojure in an imperative style, you can expect to use a lot of atoms, and your colleagues will probably notice and discuss this.

Make it kind of a pain to write more code: require doc strings for all public symbols. Ban `@VisibleForTesting`-style annotations without relaxing your code coverage requirements. This is an opportunity for a simplified, easier-to-grok public API. However, if it's not understood as this opportunity and becomes purely friction, expect only pain when teammates start writing babble as docstrings instead of restricting visibility.

### Low-hanging fruit

* Use your tools. IntelliJ's getting better support for simplifying code all the time. See what it can do for you when that gutter turns yellow.
* Watch for “Pokemon” statements, where a Java-style static type has the same name as a field which has the same name as a value/constant, similar to how Pokemon keep on saying their own name: "`public static final AnimationSprite ANIMATION_SPRITE = AnimationSprite.ofType(ANIMATION_SPRITE_TYPE)`"
* Dead code: important enough to write, important enough to compile, important enough to code review, important enough to test, no path to it from the entry points: was it really that important? Remove it, if you want it back it's only a `git revert` away.
* Untested code: important enough to write, important enough to compile, important enough to code review, not so important as to implement an assumption your system requires?
* Duplicated code: A larger code base more strongly tempts programmers to copy-and-paste to get their work done, making the code base even larger yet and eventually elevating a temptation to duplicate to "standard boilerplate," falsely enshrining a bug-risking, maintenance-slowing, project-delaying antipattern as a "team standard" or so-called "best practice." Since boilerplate code represents a failure to encode each needed characteristic of the system exactly once ([DRY](http://wiki.c2.com/?DontRepeatYourself)), copying it arguably reinforces a "worst practice."
* "Enterprise"-y names, Managers, Adapters, Factories, Observers, Impls, etc. appearing in your names suggest the names may have been chosen before clarifying the problem in plain English.
 
