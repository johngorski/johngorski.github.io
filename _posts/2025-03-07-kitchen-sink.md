---
layout: post
title: "Kitchen Sink antipattern"
---

> Hey, we seem to be duplicating a lot of code. Let's create a common package so all of our helper utility methods can be easily shared across our services.

Antipattern! In brief:

- Sharing = Good
- "Kitchen sink" packages = Bad

> What's a "kitchen sink" package?

It comes from the phrase "everything but the kitchen sink," i.e. just about any package whose name has Shared/Utils/Common/Helpers/etc. somewhere.

Here's the problem with kitchen-sink packages:

When should you add to it? What should you look for in there? If the answer to these questions is "any code that you might want to share," then you get a package that becomes a bottleneck in the maintenance of your system. You significally increase the frequency of:
- tricky dependency version conflicts
  - The kitchen-sink package will accrete dependencies, and these will conflict with each other and with the dependency versions needed by the kitchen-sink package's consumers.
- difficult merges
  - The kitchen-sink package becomes a magnet for all "shared" logic across a growing team, increasing the concurrent changes to the package.
- security escalations for kitchen-sink consuming services
  - If the kitchen-sink package takes a dependency with a vulnerability, you'll get security alerts for each consumer, even if the vulnerability is in code paths only exercised by a small fraction of kitchen-sink consumers.
- breaking changes for consumers
  - Unless you rebuild all consuming modules with every change, it's easy for an "improvement" made on behalf of a single consumer to be a breaking change. While we should all take care to avoid breaking changes anyway, working teams have mixed awareness of breaking change prevention practices.

Most teams stumble into this at some point. These packages all end up becoming problems. Kitchen Sink = Bad.

> But such packages make it easier to share logic, and therefore easier to avoid duplication and adhere to DRY ("Don't Repeat Yourself"). Are you against DRY?

DRY doesn't say you need all shared logic for a team in the same package: you adhere to DRY expressing each characteristic of your system exactly once. 

> But Sharing = Good, right? How should I share instead?

Push back against words like shared/common/utils/helpers in package names.

Insist that your team names packages according to their responsibilities/reasons for change. This will make it clear who should take a dependency on which package, and when the evolution of a given package is welcome and desired rather than fraught and resented.

