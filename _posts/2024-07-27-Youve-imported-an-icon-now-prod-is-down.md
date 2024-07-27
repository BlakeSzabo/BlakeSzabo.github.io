---
title: You've imported an icon, now prod is down.
date: 2024-07-27 19:00:00 +1000
categories: [Software Engineering]
tags: [React, Jest, TypeScript]
author: blake_szabo
description: The story of how a simple import change took down our build pipelines
---
There comes a time in every Software Engineer's career where they will do *something* that causes everything to break. This can range from ultimately benign to completely disastrous depending and I'm sure all of you reading have various stories of that one time someone (you?) did something and all hell broke loose. The story I want to share with you today didn't end up measuring quite high on the destructive scale, but the cause was so interesting to me I had to share.

# The calm before the storm

The project was a React/TypeScript app split across a micro-frontend, built with Create React App. To keep our designs consistent across multiple repos it was decided that a dedicated "component library" repo would be created that all the projects would depend upon. The component library was essentially a wrapper around Material UI v5, just apply some styles and layouts to the common components for our use case.

To start with, we simply imported all the components from Material UI, and then just exported them! Simple. No problems with that. We created a directory and index file that just exports them all, `/components/index.ts`. Because of entry points and having other directories in the future, there was also a root level `index.ts` that has to import and export those components.

However, as we started to develop more and more components and using more and more of the Material UI suite, we were completely oblivious to the problems that were about to plague us.

# Tremors

One day, during a daily stand-up, one of my co-workers off handedly mentioned something about our build. Apparently every once in a while our build pipeline would fail. I volunteered to look into it, knowing full well that a "sporadic failure" is a recipe for a bug that is not only, by its very nature, hard to reproduce but also by that same virtue difficult to confirm you've done anything at all to fix it.

The failure was happening on all our repos, except the component library, so I opened the logs of the build that failed and find the error. It wasn't hard to find:

```sh
FATAL ERROR: CALL_AND_RETRY_LAST Allocation failed - JavaScript heap out of memory
```

Our pipelines weren't running on any amazing hardware at the time, and the heap usage was just in excess of 2GB. Which, as a fun fact, is the default heap size limit for node! Now, I'll be the first to admit that JavaScript isn't the best language if you're optimizing for memory usage. So my first thought here and also likely the advice you'll see if you just google this error was to just increase the size of the heap allocation. 4GB should be plenty, right? Instead of calling `react-scripts test` directly for our test command, I changed it to this:

```sh
node --max_old_space_size=4096 ./node_modules/react-scripts/scripts/test.js
```

It says a lot about the state of JavaScript that this was the first solution I reached to but alas.

Now, this doesn't "fix" anything. Much like adding another lane to a freeway to fix traffic I've merely looked at the problem, given it space to grow, then gave myself a pat on the back. The problem is this worked! Our builds didn't fail for the whole week which was enough for us to be happy with the solution.

But surely just one more lane? One more lane and all our problems will be fixed forever!

# Tremors 2: Electric Boogaloo

It happened again. Of course. It didn't take long either, the very next week I was back in the logs again seeing that same... damn... error.

```sh
FATAL ERROR: CALL_AND_RETRY_LAST Allocation failed - JavaScript heap out of memory
```

This time it was happening every build. Something was wrong. Really really wrong.

2GB of memory usage I can hand waive. Clearly I'm a man of standards so I draw the line at 4GB. The thing I didn't mention about this failure last time was that the step of the pipeline that was failing was the unit test step. For some reason our unit tests were using over 4GB of memory. Another weird thing I noticed though is that our tests weren't even running! Not to mention that none of me or the other developers had this happening on our local machines, even when I force Jest to use one worker which should be similar to how our CI performs this step.

Then, like a whisper in the wind delivered to me at the perfect moment. It hit me.

Caching.

Our Jest runs had already cached _something_ that allowed us to avoid this problem for so long. I run `--clearCache` then my tests locally once again. It happens. I run out of memory.

**Oh no.**

I know now that this isn't a typical "haha JavaScript" memory problem. Something was leaking memory, and I had to find out what. I pulled out the big guns, the Chrome DevTools to inspect Node. This situation called for something greater than a mere `console.log` and "she'll be right".

I noticed that our heap usage just climbed forever, and my minimal experience with the node inspector was no help either. I spent the better half of a day running these test suites (which mind you took a while in and of themselves). It took a while but I managed to find a potential lead, and it manifested itself in the form of a library: `@mui/icons-material`

This was quite weird to me. Yes, Material UI Icons have ~5,000 SVG icons but the unpacked size according to npm is in the low 10s of MB, not GBs. Something else was afoot.

Have you ever heard of tree shaking? Not the literal act but rather as a technique to removing dead code. This is something that your bundler will do for you, which is awesome! This is also something that Jest doesn't do. Hey... remember those barrel files?

# Barrel files murdered my wife

Material UI allows you to import icons in multiple ways. Here's the first way, either a named export or a default export.

```ts
// Named Export
import { Delete } from '@mui/icons-material';

// Default Export
import Delete from '@mui/icons-material/Delete';
```

This named export method is using whats called a barrel file, essentially a file that exists only to import all the other stuff and export it in one spot. That's what we were doing in our component library, twice! There's one small problem with this approach though. When you import your single `Delete` icon like above from the named export, Jest will load *the entire module* before it can resolve the dependencies. That's right, for the low low price of loading 5,000 SVG icon components you too can have access to one of them!

Now that's still only, apparently, under 20MB of memory usage. Where's this 4GB+ usage coming from? Remember how our component library had its own barrel files? Brace yourself.

If you import a single component from the library, it would load _all_ of the components in memory, and guess what library a _lot_ of those components used? Material UI icons, of course! They all imported by named export because VSCode defaults to that. That means for each test whenever you loaded in a component, it would load every component, which a good portion of them would each load their own instance of EVERY MATERIAL UI ICON.

![An example graph of the imports](/assets/img/pages/Youve-imported-and-icon-now-prod-is-down/one.png)
_An example of how the import tree might look for a *single* test_

Imagine the above, but for every single test suite. It's bad.

# The Fix

Lucky for me, the immediate fix is simple. Just don't use named exports. So I searched for every icon import and fixed them all, voila problem solved! Until someone, anyone, imports with a named export. Which if you remember is the default behaviour for VSCode. Coolio. We slap a ESLint rule to prevent this import from happening again and call it a day.

Currently this is where we stand. There is probably more that I can do in this area, yes, but I also have other work I have to do and it's difficult to pitch working on a problem that's already "fixed" in favour of shipping an actual feature.

I hope you can find some humour in this ridiculous journey. I still think it's crazy how much memory you can use just running tests in this ecosystem, even without a module problem like we have. An interesting thing about Node is that it lazily garbage collects, meaning it will only garbage collect when it needs to. So unless you force Node to garbage collect it's probably going to use all the memory it can (2GB default) when you run Jest. That's maybe a story for another day.

Also, I know that this issue didn't _actually_ bring prod down, but it did prevent us from deploying patches that were fixing breaking issues on prod when we included a named import as part of a fix. Besides, you shouldn't let the truth get in the way of a good title.