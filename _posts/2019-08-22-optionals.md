---
layout: post
title: "Principles of using Java Optionals | John Gorski"
---

Java 8's Optional is a nuanced beast. Depending on code you read that uses it, you may be an acolyte or you might never want to touch Java 8's functional programming libraries again. Here's how you write code that gets readers into the first category rather than the second.

Optionals help us devote less of our code to avoiding NullPointerExceptions and give us elegant ways to consume function results which may be empty.

# Avoiding NPEs Before Optionals: The Null Object pattern

Our wonderful computer programming skills are meant for a greater purpose than null checks.

Has this ever happened to you?

```java
final Doer thingDoer = getThingDoer();

if (thingDoer != null) {
    thingDoer.doNow();
}
```

Makes enough sense. After all, we don't want an embarrassing NullPointerException ruining our oncall shift. But let's say that we're in charge of the method getThingDoer():

```java
private Doer getThingDoer() {
    // ...maybe we have something...
    if (/* ...we have a thing-doer to pass back... */) {
        return /* that thing */;
    } else {
        return null;
    }
}
```

Because we sometimes return null, callers who don't null check will wind up with NPEs in their code. Writing `return null` is what dooms them to this fate.

But let's say we honestly don't have anything for the thing-doer to do in some situations. Don't we have to return null here? This is what the null object pattern helps us avoid:

```java
public class Donter implements Doer {
    private static final INSTANCE = new Donter();

    public static Doer instance() {
        return INSTANCE;
    }

    private Donter() { }

    @Override
    public void doNow() { }

    @Override
    public void doLater() { }
}
```

`Donter` ("don't-er," get it?) instances are "null objects" which can be dereferenced without NPEs, but have appropriate "empty" behavior for our domain. Instead of returning null, our getDoer() method can now return Donter.instance().

Since getDoer() never returns null, the null-check burden of our client disappears and our client code snippet at the start collapses to

```java
getThingDoer().doNow();
```

Pretty neat! The fraction of our client code devoted to what we want to happen in our problem domain is now 100%, with 0% devoted to null-checking. This is a big improvement from our first snippet, which was 50/50 (20/80, if you count vertical space).

Our first example also required us to come up with an effective name for our variable to hold the returned reference from our getThingDoer() call. Since we don't need that in our second example, we don't need to argue what would be an effective and appropriate name during code review. Woo hoo!

# Optionals making us happy: This method will not return a result in all cases

So that was fun! We got rid of the null check burden for our client calls, and we haven't even talked about Optionals yet. In the last case, we had control over the behavior of the method getDoer(), and it was reasonable for us to change the contract. These conditions aren't always true.

If we have a method with a nullable return type, we can use Optional.ofNullable() instead of traditional null checks.

```java
Optional.ofNullable(getThingDoer())
    .ifPresent(thingDoer -> thingDoer.doNow());
```

While this doesn't have the vertical space of our {} from above, it's not clear that this is significantly better than our original code. We do get to be cool kids by using Java 8's lambda notation, but is that enough? Changes to something as pervasive as null checks in pre-Java 8 code should be able to get us a few more tangible benefits.

We can use Java 8's method reference feature instead of a lambda save ourselves the heartache of arguing about the merits of "thingDoer" as a variable name:

```java
Optional.ofNullable(getThingDoer())
    .ifPresent(Doer::doNow);
```

Saving that source of friction is reason enough to adopt the Optional version of the code when we can't stop getDoer() from having a nullable return type. It turns out that Optionals let us do even more than that.

# No longer Optional: Coping with emptiness

Invoking a procedure is only one thing we might want to do with an object which may or may not be present. Often the possibly-absent value is part of a longer calculation. At some point, we need our `Optional<T>` to become a `T` (Java's type-checker will stop us otherwise). If we look at the nullable value case, we have a few different choices for when the value is missing:

* Fall back to a default value.
* Try to get the value from somewhere else.
* Panic--throw an exception. This is a more appealing case for nullable values than for Optionals, as we'll discuss below.

Let's play, using the problem of finding an organizer for monthly demo days.

Default value example: getDemoDayVolunteer()'s return type is a nullable Employee

```java
final Employee volunteer = getDemoDayVolunteer();
final Employee demoDayOrganizer = volunteer != null
    ? volunteer
    : matt;
```

Default value example: getDemoDayVolunteer()'s return type is an `Optional<Employee>`

```java
final Employee demoDayOrganizer = getDemoDayVolunteer().orElse(matt);
```

Note that in this second listing we're once again saving ourselves an intermediate variable.

Let's say we didn't have a ready backup for our demo day organizer (Matt was going to be out of town, for example). In that case, we would need to consult several sources to make sure we have a completely fair way of assigning the task: seniority, astrology, decay of radioactive isotopes, etc. There's a problem using our backup procedure (return type of Employee, not `Optional<Employee>` in this example) with .orElse():

```java
// Problem!
final Employee demoDayOrganizer = getDemoDayVolunteer()
    .orElse(this.runGuaranateedOrganizerSelectionProcedure());
```

Do you see the problem?

Because method arguments in Java are evaluated before the method is invoked (eager evaluation, not lazy evaluation), we're running our backup procedure always, even when we already have a volunteer! This is good for astrologers' income, but otherwise anti-frugal. Instead, we can pass a `Supplier<Employee>` to .orElseGet():

```java
final Employee demoDayOrganizer = getDemoDayVolunteer()
    .orElseGet(this::runGuaranateedOrganizerSelectionProcedure);
```

Our Supplier is invoked only when we don't already have a value. That's better.

## Emptiness as a Problem

When our program can't continue when an Optional value isn't available, we can use orElseThrow() to give up.

```java
// On Demo Day....
getDemoDayVolunteer()
    .orElseThrow(() -> new DemoDayCanceled("No volunteers? Fine! No beer. No cheer of any kind."))
    .interrupt();
```

If the exception you're throwing is a NullPointerException, you can use .get() itself--but you probably have an exception type more actionable than a NullPointerException. If you find yourself calling .get(), step back and ask yourself why orElse(), orElseGet, or orElseThrow() don't fit your needs.

Notice how having an Optional return type doesn't force a client into a particular way of handling an empty result. Throwing a DemoDayCanceled exception from getDemoDayVolunteer() isn't always what we want.

# Let's get some lunch

Going out to lunch is delicious, but choosing where is hard. I'll usually go with the group when there is a decision, but I'll skip Veggie Grill. Other than that, I don't have a strong preference. Let's say Seattle passes a law that if I don't have a decision, I need to eat at Hurry Curry.

```java
final String groupLunch = getGroupDecision();
final String johnsPreference = groupLunch != null && !"Veggie Grill".equals(groupLunch)
    ? groupLunch
    : null;
final String johnsLunch = johnsPreference != null
    ? johnsPreference
    : "Hurry Curry";
```

Optional's filter() method takes a predicate that lets you test the value the Optional might be holding and returns itself when the predicate passes and an empty otherwise.

Let's change getGroupDecision()'s return type from String to `Optional<String>` and see how this changes things:

```java
// I'm not a fan of Ba Bar either, but I won't make a big deal about it.
final String johnsLunch = getGroupDecision()
    .filter(groupLunch -> !"Veggie Grill".equals(groupLunch))
    .orElse("Hurry Curry");
```

Hooray!

# Just passing through...

Has this ever happened to you?

```java
public Costume nextHalloweenCostume() {
    final Person nemesis = getNemesis();
    if (nemesis != null) {
        final Fear fear = nemesis.deepestFear();
        if (fear != null) {
            final Trigger trigger = fear.trigger();
            if (trigger != null) {
                return costumeBasedOn(trigger);
            }
        }
    }
    return null; // better luck next year
}
```

Yuck, we've all been there. We have all of these null checks because

* we might not have a nemesis
* our nemesis, if we have one, might not have a deepest fear
* our possible nemesis's possible fear might not have a trigger.

It's no surprise that with Optionals, we can do even better than this. Optional's map() method takes a `Function<T, U>` and returns an `Optional<U>` as the result of applying the function to the (maybe) value referred to by the Optional. If the mapping function itself returns null, map()'s result will be an empty Optional. Behold!

```java
public Costume nextHalloweenCostume() {
    return Optional.ofNullable(getNemesis())
        .map(Person::deepestFear)
        .map(Fear::trigger)
        .map(this::costumeBasedOn)
        .orElse(null);
}
```

Our new method's body takes 5 vertical lines rather than 11 while avoiding nesting. It also has one return statement, rather than two. Since map() returns an empty Optional if the result of the mapping function is null, it's still null-safe to chain even pre-Optional methods with nullable return types. We don't need to touch Person, Fear, or costumeBasedOn() in order to do this.

We're still paying forward the burden of further null checks by returning null when nothing is available. We can end the madness by returning `Optional<Costume>` instead of Costume removing the orElse() call at the end of the train.

## What about when my mapping function's return type is also Optional?

In the above example, the methods map() was using still had nullable return types. What if these methods were written with Optional return types instead, to show they might not return a value? We run into trouble right away:

* If getNemesis() now returns `Optional<Person>`, we don't have to wrap it in `Optional.ofNullable()` any more. So far, so good.
* If Person's deepestFear() now returns an `Optional<Fear>` rather than a nullable Fear, getNemesis().map(Person::deepestFear)'s type would now be `Optional<Optional<Fear>>`.
* Now the input type of our second map() call's function doesn't match the type the Optional is holding. Our code doesn't compile any more, which is good, because we're confused.

Even if we "fix" this confusion by changing the input types of our deepestFear(), trigger(), and costumeBasedOn() methods, at some point we're taking an `Optional<Optional<Optional<...>>>` as method's input type! Rather than reaching for migraine medication, we can recognize that this is our use case for Optional's flatMap() method.

```java
// Return types for deepestFear, trigger, etc., have been
// modified to have Optional return types instead of
// nullable return types.
public Optional<Costume> nextHalloweenCostume() {
    return getNemesis()
        .flatMap(Person::deepestFear)
        .flatMap(Fear::trigger)
        .flatMap(this::costumeBasedOn);
}
```

Functions passed into Optional's flatMap() method must have an Optional return type, and this lets us avoid building up layer after layer of Optionals.

flatMap() situations tend to bend the mind a little bit more. It's hard to get a feel for when they're needed until you've seen them in the wild a few times. For today, just remember that flatMap() exists if you run into situations where types start to look nested: `Optional<Optional<Optional<...>>>`. To flatten a situation you reached from using map(), think flatMap().

## Alternatives to the others

java.util.Optional has other methods, but you shouldn't need them. If you think you do, often you'll find that you're missing one of the features which Optional already affords us.

### get() / isPresent()

get() is basically the same as orElseThrow(NullPointerException::new). The uninitiated will sometimes use isPresent() to avoid this NPE. If orElse()/orElseGet()/ifPresent() don't seem to be fitting your needs and you still find yourself reaching for get() and isPresent(), it might be a good time to get a second pair of eyes on the issue.

### Optional.of() / Optional.empty()

Optional.of(null) throws a NullPointerException. In most cases, what you'll want instead is Optional.ofNullable() to transition from computations with null checks to computations with Optionals instead.

While there may be cases where you'll find yourself explicitly referencing Optional.empty(), know that in most cases you shouldn't have to.

```java
public Optional<Bagel> myOrder() {
    final Bagel cheapest = cheapestBagel();
    return !cheapest.ingredients().contains("Raisins")
        ? Optional.of(cheapest)
        : Optional.empty();
}

public Optional<Bagel> myOrder() {
    return Optional.of(cheapestBagel())
        .filter(cheapest -> !cheapest.ingredients().contains("Raisins"));
}
```

# Code structure alternatives to putting Optionals elsewhere than as a method's return type

> ⚠️  WARNING: Outside of method return types, Optionals tend to increase code complexity rather than decrease it.

> 👉 Note: The lessons about how these other uses make code more confusing apply to nullable values, too, so keep them in mind when reviewing code with nullable types.

You should never need to name an instance of an Optional. If you try to come up with a good name, you'll notice how hard it is. This isn't your fault for being bad at naming; it's Optional's fault for sometimes representing something and sometimes representing nothing.

It's worth considering if the difficulty coming up with a name for a thing is itself a warning against needing to name the thing. That's certainly the case with Optionals.

Let's discuss.

## Local variable

Optionals should be handled once each, at the earliest convenience. Needing to hold an Optional in a local variable, to be referred to at multiple times or handled in different ways, is a sign your method is probably doing too much (more than "one thing").

## Method parameter

```java
// Don't do this.
public static final Result foo(
    final Bar bar,
    final Optional<Baz> baz
) {
    // ...
}
```

This is an Optional we really don't need. Giving clients ambiguity about whether a method parameter is nullable/optional or not isn't as helpful as providing overrides instead:

```java
public static final Result foo(
    final Bar bar,
    final Baz baz
) {
    // ...
}

public static final Result foo(final Bar bar) {
    // ...
}
```

In practice, you'll find the code of each method is more than twice as clear as the code with the Optional parameter type.

But what about clients that already have an `Optional<Baz>`? What are they to do?

```java
final Result r = lookForABaz()
    .map(baz -> foo(bar, baz))
    .orElseGet(() -> foo(bar));
```

That's what. It's not foo()'s fault that her callers don't know if they have a Baz or not.

## Class field

```java
// Also bad
@Value
public class MyClass {
    private final A a;
    private final Optional<B> b;

    // numerous instance methods...
}
```

A few things could be happening here:

* a and b are rarely referenced in the same instance methods. MyClass would then be said to have poor cohesion, and it's a sign that MyClass should be multiple classes instead.
* Alternatively, a and b are frequently referenced in all of the instance methods, and whether b is present becomes a big feature in the code base. In this case, the remedy would be once again to have separate classes implementing the same interface. This is the class-level version of our method override example above.
* Finally, MyClass might be a pure data structure class without behaviors. Even if the intent is to only access b via getB(), we still shouldn't use an Optional field (though getB()'s return type could still be `Optional<B>`). A lot of libraries for working with Java data classes (e.g. serialization, equals/toString/hashcode builders, etc.) are set up to work with nullable fields rather than Optional fields, so even if a class field isn't required for all instances, you're better off leaving it nullable.

One last illustration of why this is misguided:

```java
// Last nail in the coffin
@Value
@Builder
public class MyClass {
    private final A a;
    private final Optional<B> b;
}

// later...
final MyClass myInstance = MyClass.builder().build();
myInstance.getB()
    .map(// ...
```

...it actually doesn't matter what follows next: `.getB()` returns `null`, so we get a NullPointerException. Using an Optional field with Lombok's @Builder means we need to null-check the getter's result!

We could maybe do this...

```java
// If you want everyone to learn to be a Lombok expert...
@Value
@Builder
public MyClass {
    private final A a;
    @NonNull // Better add this one, too, in case we end up passing null in the builder chain!
    @Builder.Default
    private final Optional<B> b = Optional.empty();
}
```

...but if you're going down this route, and you're not resolving the cohesion issues, writing your own getter is probably best.

```java
@Value
@Builder
public MyClass {
    private final A a;
    private final b;

    public Optional<B> getB() {
        return Optional.ofNullable(b);
    }
}
```

## Constants

```java
// Our talk on not having `Constants` classes will need to happen another day.
public class Constants {
    public static final Optional<String> OPINION_ON_WHITESPACE = ...;
}
```

If your constant needs to be referenced from so many different places in your code, those references will also have to handle whether it's null. That's a lot to ask. It's time to ask yourself if the idea you're trying to express in the code really needs to be a constant.

# Summing up

If you can use the Null Object pattern, you can avoid both null checks and Optionals!

Otherwise....

|Optional method|Use/rethink|Why|
|---|---|---|
|ofNullable|✓|Transition from nullables to optionals|
|ifPresent|✓|Run a procedure against an object if it's there|
|orElse|✓|Fall back to a default value|
|orElseGet|✓|Compute a default value if needed|
|orElseThrow|✓|Signal the inability to continue when the value is gone|
|filter|✓|Narrow the useful values an Optional may take|
|map|✓|Compute against the value, if present|
|flatMap|✓|Avoid nesting Optional types when mapping function's return type is also Optional|
|get|🚫|You probably have an exception type more aligned with your application. NPEs aren't specific to your problem.|
|isPresent|🚫|Doesn't add value over other methods|
|of|🚫|If you know you have a value, you're not Optional. If you need a value, you should throw a domain-specific exception rather than an NPE.|
|empty|🚫|Usually not needed--look extra-close for alternatives if upstream branching logic returns Optional.empty()|

|Optional declaration|Use/don't use|Why|
|---|---|---|
|Method return type|✓|Signal that the method does not return a value in all cases and prepare clients to handle the result|
|Local variable|🚫|Referencing an Optional value multiple times means your method is probably doing too much. Split it up.|
|Method parameter|🚫|Use method overrides instead|
|Class field|🚫|Signal you need separate classes|
|Constant|🚫|Next-level trolling|

👉 You should never need to name an Optional instance.

# References

* [java.util.Optional documentation](https://docs.oracle.com/javase/8/docs/api/java/util/Optional.html)
* [*Effective Java*, 3rd edition](https://www.oreilly.com/library/view/effective-java-3rd/9780134686097/)
* [*Clean Code*](https://www.oreilly.com/library/view/clean-code/9780136083238/)
* ["Maybe Not"](https://youtu.be/YR5WdGrpoug), talk by Rich Hickey, Clojure Conj 2018

