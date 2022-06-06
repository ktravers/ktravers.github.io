---
layout: post
title: RenderATL Notes
tags: [react, conference]
---

June 1-4, 2022  
[renderatl.com](https://www.renderatl.com/schedule)

# Day 1

## Debugging React Applications workshop

Cecelia Martinez  
- Did bootcamp in Atlanta (Georgia Tech)
- Used to be at Cypress
- Now at Replay.io
- GitHub Star
- @ceceliacreates

Slides: slides.com/ceceliamartinez/debugging-react-apps-workshop

Common approach: "debugging dartboard"  
- Classics: console.log, debuggers, etc.
- Random

Process + tools => better experience

### Process

3 step process:

1. **What** is the problem?
  - Reproduce bug
  - Understand expected behavior ("what does working look like?")
  - Define problem
2. **Why** is it broken?
  - Multiple hypotheses: the more hypothesis, the better
  - Collaborate early (phone a friend)
  - Isolate the problem
3. **How** did it happen?
  - Ask yourself: could a test have caught this?
  - Are you using the framework correctly?
  - Other areas of the codebase with same implementation? (i.e. is it working/broken elsewhere?)

Find a process that works for you.

### Tools

How you execute your process

Two categories:

- Inspecting
  - Browser/IDE
  - Elements
  - State
- Evaluating
  - Control flow
  - Execution
  - Performance

Activity: https://hot.opensauced.pizza/ (by @bdougie 🎉)

- Source maps shipped to the browser (non-minified original source code)
- Built with React
- React DevTools also has a performance profiler

Replay.io demo  
- Replay is a browser, fork of Firefox
- Nextjs app (frontend)
- Replay protocol (backend) ([examples](https://github.com/replayio/protocol-examples))
- Record session or automated tests
- Code heat maps
- replay.io/examples
- Drop in replacement for automated tests
- Practice with Replay:
  - replayable.dev
  - replay.io/examples

### Experience

"The best debugging tool is knowing something well" - Jenn Creighton (@gurlcode), "Debugging async JS"

Practice  
- Reinforce process
- Recognize patterns

Patterns  
- Component (re)rendering
  - Rendering before data is there
  - Check useEffects
- State and hooks
  - Competing sources of truth (local vs global data store)
  - Race conditions (event loop shenanigans)

Pro tip: use beta React docs

### Random observations

- "TipTapEditor" --> great component name
- Comments in side panel, not inline. Both narrating recording and code review


### Web Components Workshop

Sponsored by Vonage

ft. live transcription (!!)

lol @ https://arewebcomponentsathingyet.com/

Examples:
- video element
- audio element
- details element
- input (same element, so many flavors)
- [model-viewer](modelviewer.dev)
- AR/VR tic tac toe (with Vonage + WC): https://xoxr.games/
- dwane-made
  - pulls in reusable code for each project
  - ex. "about", "contact", etc.
- dwane-timer
  - eventually can be replaced by temporal timer
- See more:
  - mdn/web-components-examples
  - web-components.org

What are they?
- Custom elements
  - kebab case, no single words
  - class object
  - extends, if any
- Shadow DOM
  - encapsulation
  - "contain the gross" - [Paul Lewis](https://www.youtube.com/watch?v=plt-iH_47GE)
  - remaps events (i.e. custom events)
- HTML templates
  - `<template>`
- Can be styled with CSS, same as anything else.
  - Helpful to leverage CSS variables
  - `::part` pseudoclass

Demo: https://codepen.io/conshus/pen/NWPzwGO

Built-in functions:
- `connectedCallback` - fires when added to DOM
- `attributeChangedCallback` - fires when observed attribute(s) change
- `disconnectedCallback` - fires when removed from DOM
- operate on Shadow DOM, not document

More examples  
- webcomponents.dev
  - Play around with components in the browser
  - Push to GitHub/Lab optionally
  - Lots of starter templates
- open-wc.org
  - best practices
  - CLI tool that allows you to scaffold WCs

### Build

`<vc-keypad>` WC
- scaffolded via [open-wc](https://open-wc.org) and [Lit](https://lit.dev)
- [Try it out](https://webcomponents.dev/edit/NlZFNRKfCK2EdBYbfHmU/src/index.js?p=stories)
- NPM: @wcd/conshus.vc-keypad-workshop
- Installed in [starter project](https://stackblitz.com/edit/vc-keypad-vanilla)

WC + React  
- React doesn't play super nice with WCs ([explainer](https://custom-elements-everywhere.com/#react))
- custom-elements-everywhere.com --> "caniuse" for WCs
- Experimental branch has [full support](https://custom-elements-everywhere.com/#react-experimental) (!!)

Example with React  
- https://stackblitz.com/edit/vc-keypad-react-start?file=src%2FApp.js
- https://developer.vonage.com/blog/2020/10/07/using-web-components-in-a-react-application-dr

## Day 2

### Bria Sullivan, "Making Fun: Building Your First Mobile Game On The Side"

- Goal: launch in 3 months
- Used Firebase analytics
- Advice: don't do it all yourself, know your expertise (ex. use fivvr for design assets)

### Angie Jones, "Refactoring the Web"

- Web was meant to be decentralized
  - No central authority originally
  - No identity layer (oops)
- Currently
  - Auth via fb, google, so no agency
  - You do not own your own data
- Future: what if identity was native?
  - Self owned ids
  - Based on Open standards
- Proposal
  - New layer for the web
  - 1. DIDs : [decentralized identifiers](https://www.w3.org/TR/did-core/)
    - Identity stored on the blockchain
    - Specifically, only DID string stored on blockchain
  - 2. Verifiable credentials
  - 3. Decentralized web nodes
    - Personal data stores
    - Decouple data from apps we use
    - Nodes are not on the blockchain, stored locally
    - Similar to progressive web apps (PWAs)

### Scott Hanselman, "Beyond Mentorship - Storytelling And Sponsorship"

- Do you have 20 yrs experience, or the same year, 20 times? (i.e. are you learning? or "asleep"?)
- Privilege doesn't run out. You can give it away without losing any
- Sponsors create luck (where luck === opportunity + being prepared)
- Advice: don't answer emails/DMs. Write it somewhere public and shareable, then share that link with the person(s) who inspired it. That way you reach more people.

### Pariss Athena, "Beauty Meets Web3"

- Examples
  - Olay: simulator that ages you 20 years (then suggests products)
  - [Opte printed makeup](https://opte.com/)
  - E-makeup, aka filters
  - Valde beauty NFTs
  - MetaOptimist NFT contest by Clinique

### Danny Thompson, "Error Boundaries Save You From Crashes! Here's How To Use It!"

- Create trust through better customer facing error messages
- Error boundary components catch error anywhere in child tree and handle it gracefully
- Wrap individually (one option)
- Does not catch a whole bunch of kinds of errors. Not a catch all

### Kent C. Dodds, "Bringing Back Progressive Enhancement"

- Remix is built as a browser emulator
- Works with JS disabled
- Progressive enhancement is not new, coined in 2003
- You don't need Relay or Apollo when using Remix
- Replace your state management code with Remix
- Form submissions are navigation
