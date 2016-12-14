---
layout: post
title: React and Redux Crash Course with Steven Nunez
---

### Instructor

[Steven Nunez](https://github.com/StevenNunez), Lead Instructor/Developer at Flatiron School

### Goals:

1. Learn through doing
2. Knowledge up on cool new tech

### Why Redux?

1. "Surprisingly simple" says Steven
2. Very specific purpose
3. Supes popular
4. Lots of high quality curriculum available from [Flatiron School](https://flatironschool.com/)
5. They hired [Dan Abramov](https://github.com/gaearon)

### What is Redux?

1. State Management

The end.

- Has nothing to do with the web.
- Has nothing to do with React.
- Nice and simple.

### Is it Flux?

Not really. Flux has a very specific set of parts: dispatcher, actions sent by dispatcher, etc. Redux has a lot of the same players, but follows slightly different pattern.

### Why Redux was hard for Steven

Seems like a lot of stuff at first.

The part that's Redux is actually fairly small. Redux covers Reducers, Dispatch, middleware. The rest is mostly Redux+React. And then conventions like ALL_THE_CAPS and Action Creators are separate conventions (oof).

### Exercise

Building a chat app

code lives at: [https://github.com/StevenNunez/redux_chat_hedgehog](https://github.com/StevenNunez/redux_chat_hedgehog)

1. `brew install elixir`
2. `mix phoenix.new redux_chat_hedgehog` (using phoenix bc it has nice tools for modern web workflow)
3. comment out db username and password in `config/dev.exs`
4. `mix deps.get`
5. `mix ecto.create`
6. rm boilerplate markup in `web/templates/page/index.html`
7. `mix phoenix.server`
8. `npm install --save react redux`
9. `npm install --save babel-preset-react`
10. `npm install --save react-dom`
11. `npm install --save axios` (for fetch requests)
12. `npm install --save redux-thunk`
13. wipe out boilerplace in `web/static/js/app.js`
14. update `brunch-config.js`
  - plugins config should look like this:

    ```javascript
    // Configure your plugins
    plugins: {
      babel: {
        // Do not use ES6 compiler in vendor code
        ignore: [/web\/static\/vendor/],
        presets: ['react', 'es2015']
      }
    },
    //...
    ```
15. in `web/static/js/app.js`: `import { createStore } from 'redux'`
  - with ES6 module system, couple ways to export from a class
  - can export as a constant
  - can export default
  - `import Something from 'aFile'` gets you the default
  - `import { thing1, thing2 } from 'aFile'` gets specific stuff (constants, functions, etc)
  - that way when you open a file, you can see up top what all the collaborators are
16. About Redux store:
  - In Redux, store is really stupid (aka simple)
  - Store can receive messages and do a thing
  - Store has method called dispatch: `store.dispatch({type: 'whatevs'})`
  - dispatch requires `type` key
  - we can add kinda like an event listener where when state changes, can re-run code:

    ```javascript
    store.subscribe(()=>{
      // whenever receives message, function will be called
      console.log(store.getState());
    })
    ```
  - note: you have to pass a function to createStore
  - chain of events: dispatch then call subscribe
  - function that you pass to store is known as the Reducer

    ```javascript
    function counter() { // this is our Reducer
      return 1;
    }

    const store = createStore(counter);
    ```
17. About the Reducer
  - first argument we receive in the reducer is state
  - common/best practice to define default state
  - second argument is the action
  - super simple example:

    ```javascript
    function counter(state=0, action) { // this is our Reducer
      switch (action.type) {
        case 'INCREMENT':
          return state + 1;
        case 'DECREMENT':
          return state - 1;
        default:
          return state;
      }
    }
    ```
18. Another thing: Combined Reducers
  - composing reducers
  - multiple reducers to handle different things, manage different parts of the page

    ```javascript
    function counter(state=0, action) { // this is our Reducer
      switch (action.type) {
        case 'INCREMENT':
          return state + 1;
        case 'DECREMENT':
          return state - 1;
        default:
          return state;
      }
    }

    function lastMessage(state='No Message Yet', action) { // Another reducer
      switch (action.type) {
        case 'INCREMENT':
          return 'Was incremented!';
        case 'DECREMENT':
          return 'Was decremented!';
        default:
          return state;
      }
    }

    const rootReducer = combineReducers({ // map reducers
      counter: counter,
      lastMessage: lastMessage
    });

    const store = createStore(rootReducer);
    ```
  - each reducer will now receive dispatched message and then execute accordingly

    ```jsx
    // state of web/static/js/app.js so far

    import { createStore, combineReducers } from 'redux'
    import React from 'react'
    import { render } from 'react-dom'

    function counter(state=0, action) { // this is our Reducer
      switch (action.type) {
        case 'INCREMENT':
          return state + 1;
        case 'DECREMENT':
          return state - 1;
        default:
          return state;
      }
    }

    function lastMessage(state='No Message Yet', action) { // Another reducer
      switch (action.type) {
        case 'INCREMENT':
          return 'Was incremented!';
        case 'DECREMENT':
          return 'Was decremented!';
        default:
          return state;
      }
    }

    const rootReducer = combineReducers({ // map reducers
      counter: counter,
      lastMessage: lastMessage
    });

    const store = createStore(rootReducer);

    const App = (props) => {
      return (
        <div>
          <div>The current value is: {props.counterValue}</div>
          <div>The last message was {props.lastMessage}</div>
          <div>
            <button onClick={()=> store.dispatch({type: 'INCREMENT'})}>
              Up
            </button>
            <button onClick={()=> store.dispatch({type: 'DECREMENT'})}>
              Down
            </button>
          </div>
        </div>
      )
    }

    store.subscribe(function () {
      render(
        <App
          counterValue={store.getState().counter}
          lastMessage={store.getState().lastMessage}
        />,
        document.querySelector('#root')
      );
    });

    store.dispatch({type: 'Banana'});
    ```
19. Concepts: immutability and functional purity
  - immutability helps w/ performance
  - pure function == function that if you give it same inputs, will always return same thing, not affected by outside world
  - reducers are always pure
  - no promise resolution inside reducers
20. How we handle async stuff in Redux:
  - can't emit a Promise, per se
  - but can take advantage of middleware
  - after action is dispatched, moment where you can transform message into format you want, make it a plain ol' object

### The End

More next week!
