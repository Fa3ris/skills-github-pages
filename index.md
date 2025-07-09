---
title: Welcome to my blog!
---

# 9 July 2025: it works if you don't respect the specs

The nightmare of trying to fix a performance issue in a npm library and find out it's caused by a change in browser

I received a task explaining that a feature had become slow when used with the new version of Chrome 138. It was not acting bad when used with Chrome 136.
The feature is exporting a set of HTML node to a PNG. We use `dom-to-image-more` to do that.

So it was definitely something wrong with the browser.

I first spent some time to run two versions of Chrome on my laptop, first I tried to use puppeteer to run a test version which I did once but this time it was horribly slow, in the end I found you can have two versions of the same app on Mac OS, you just need to give them different names and set the user-data folder to something other than the default.

``` zsh
#!/usr/bin/env zsh
open /Applications/Google\ Chrome\ 138\ test.app --args --user-data-dir="tmp/Google Chrome/138-test/"
```

Then I ran the application in dev mode on both browsers.

Then I spent A LOT OF TIME trying to use and make sense of the performance tab of the dev tools and compare the flame charts of both versions.

In the end, I can definitely see that the version 138 is spending much more time than the version 136 but I cannot find the root cause.
I don't know what the lib is doing and why it is doing that. In short, I'm not familiar with the code base and am nowhere near to find a solution.

In desperation, I turn to a LLM to ask what might be the cause of this issue. O joy when it tells me there have been multiple issues raised for the same problem in other similar lib.

For example https://github.com/niklasvh/html2canvas/issues/3249

Smart people before me have already figured out what is the issue and raised a bug directly in chromium https://issues.chromium.org/issues/429073017

The issue is that the lib is cloning each HTML node that it needs to export to put it inside a SVG element.
When cloning the node, it also needs to clone the CSS associated with it so that it renders the same. To do this it uses the native `getComputedStyle`.
Before I guess `getComputedStyle` returned only the style directly associated to the element. Now it also returns all the CSS custom properties, also called CSS variables (the --my-var thingy)!!!
This makes the list of properties to process for one node go from 400 to 2700+!!!

And it turns out, this is totally the behavior expected by the specs!!! This behavior is controlled by a config in chromium which was turned off before for performance concerns (and they were right) and was enabled recently.

So how to fix this? Either propose a fix to the library (again I'm not familiar with the code base so finding where to do it will take time) or use another one that does not experience this issue (need to decide which one to pick, maybe there are constraints I am not aware of).

Joy again, by some luck the last release of the library exposes a way to filter the style. Coincidence??? Maybe not, other browsers like firefox which were spec compliant were already having those performance issues (Urgh...).

Anyway the fix is just filtering out properties that starts with `--` like explained in the PR that introduces the change https://github.com/1904labs/dom-to-image-more/pull/200


