---
layout: post
title: "Code review checklist"
---
Before you publish a pull request, confirm the following:

Tests:
- ❌✅: I have protected new system assumptions with tests, each named for the assumption it protects.
- ❌✅: I have seen each new test fail when the assumption it protects is violated.
- ❌✅: My test code and production code follow the same code cleanliness standards (e.g. no duplication, descriptive variable names, etc.)
- ❌✅: I wrote my tests at the cheapest layer which can falsify their protected assumptions (e.g. I’m not using a pipeline integration test to protect an assumption which could be protected by a build-time unit test).

Compounding value:
- ❌✅: I have not introduced unnecessary mutable state.

Brevity:
- ❌✅: My comments refer to the code’s current state, not its anticipated evolution.
- ❌✅: I have removed tests which protect obsolete assumptions.
- ❌✅: I have removed code not covered by the test suite (if not, provide the reason to retain that code).
- ❌✅: I have not allowed a characteristic of the system to be encoded more than once.
- ❌✅: I have defined variables at the smallest scope from which they are referenced (e.g. a value used in a single method shouldn’t be defined in a global constants file).

Commit messages:
- ❌✅: My commit messages describe the software’s new behavior
- ❌✅: My commit descriptions explain the reason for the change

Deployment discipline:
- ❌✅: I will monitor the propagation of this change to all deployment destinations.
- ❌✅: If there is a problem during deployment related to this change, I will push a revert commit first and then diagnose the error locally.


