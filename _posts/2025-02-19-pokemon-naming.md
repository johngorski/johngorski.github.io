---
layout: post
title: "Pokemon Naming antipattern"
---

I don't think the phrase "Pokemon Naming" is original to me, but can't find the original reference. In the Pokemon Naming antipattern, variable names keep announcing parts of the context which are already clear from the current scope, similar to how Pokemon (barring exceptions such as Team Rocket's Meowth) just keep saying their own names.

```typescript
import {SQUIRTLE_NAME, SQUIRTLE_HEALTH} from './constants/squirtle';

const squirtle: Squirtle = new Squirtle(SQUIRTLE_NAME, SQUIRTLE_HEALTH); // we get it

// Also, in 90% of cases SQUIRTLE_NAME will be 'Squirtle'
```

Pokemon Naming is a subtle DRY violation. It reminds me a little bit of a chant or a mantra. Like a chant, it can soothe you to sleep.

Pokemon Naming lowers the information density of the code you're reading. As such, it's rarely what you want.


#### Prior art

The [Coding Horror](https://blog.codinghorror.com/new-programming-jargon) blog called attention to the "Smurf Naming Convention", original credit to StackOverflow user [sal](https://stackoverflow.com/users/13753/sal).

If I'm to draw a distinction between Smurf Naming and Pokemon Naming, I would use Smurf Naming to point to when a single variable name prefix appears all over a project, and Pokemon Naming to point to when prefixes tend to redundantly repeat across string values, variable names, constructors, module names, file names, and other scopes.

