---
layout: post
title: "High Performance JS Tip: Dimensions are not your friend"
date: 2013-09-03 10:24
comments: true
categories: 
- JavaScript
- High Performance
---

Several months back I was tasked with building an [interactive instagram scroller](https://twitter.com/NikeNYC/status/322955662013374464/photo/1) 
that was to be slotted into large display via a Safari 5.1 "widget".

The project was generally fairly simple, 3 columns scrolling infinitely in alternate directions. 
The individual pictures, or "chips" as I called them were to do a 3D rotate transition to keep 
the visual appeal up. 

I covered all the usual suspects to maximize performance, I used as many CSS transitions as possible
to harness the GPU, keept everything asynchronous as possible, kept the DOM cleaned up, use `requestAnimationFrame`. 
However the biggest problem I encountered was getting the scroll smooth.

Initially the page was fully dynamic and my anmiation code used the `scrollTop` property (while 
inserting new chips as needed to prevent stoppage). That proved problematic and terrible performance-wise.

Next I tried inserting chips in the top of their column container with a negative margin and slowly
animating the first chip to a margin of 0 (effecitvely constantly pushing all other chips down). This
also posed problems and I couldn't approach what looked like a fluid animation. It would always do a slight,
maybe 2-3ms stutter when a new chip was introduced and there was nothing I could do to eliminate it. I tried
optimizing my DOM interaction to use DOM DocumentFragment, dropping all jQuery code and going full-native.

Finally I moved to using absolutely positioned chips that I tracked one by one in my animation loop. However,
*even this* was insufficient and I was still getting a stutter.

After much forehead-meets-desk time, I remembered a conversation I had with [Tim Caswell](https://twitter.com/creationix)
a long while back (revolving around the HP TouchPad) where he told me about an interesting quirk of WebKit that
turned out to be the cause of my problems:

To make my display flexible I was using the `getBoundingClientRect` method to calculate when a chip rolls off-screen
so I can either remove it from the DOM and insert a new chip at the beginning (as well as to calculate initial position).

The **problem** is WebKit likes to recalculate the layout of the dom pretty much every single time you use something 
similar to `getBoundingClientRect`. (Even getting the window's `innerHeight`/`innerWidth` will force a recalculation).
The stutter was coming from WebKit constantly recalculating the layout!

So, a quick refactor to use static, precalculated dimensions for the chips (calculated once on window resize/document load)
and BOOM, performance issues gone. So, lesson learned?

> All calls to get any calculated dimension from the DOM should be cached or avoided.

