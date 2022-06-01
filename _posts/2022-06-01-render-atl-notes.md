# RenderATL conference

June 1-3, 2022
renderatl.com

## Day 1

### Debugging React Applications workshop

Cecelia Martinez  
- Did bootcamp in Atlanta (which one?)
- Used to be at Cypress
- Now at Replay.io
- GitHub Star
- @ceceliacreates

Slides: slides.com/ceceliamartinez/debugging-react-apps-workshop

Common approach: "debugging dartboard"  
- Classics: console.log, debuggers, etc.
- Random

Process + tools => better experience

#### Process

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

#### Tools

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
- Record session or automated tests
- Code heat maps
- replay.io/examples
- Drop in replacement for automated tests
