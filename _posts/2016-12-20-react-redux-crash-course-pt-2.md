---
layout: post
title: React and Redux Crash Course with Steven Nunez PART DEUX
---

### Instructor

[Steven Nunez](https://github.com/StevenNunez), Lead Instructor/Developer at Flatiron School

### Goals:

1. Build on what we learned [last week](http://blog.kate-travers.com/react-redux-crash-course/)
2. Set up some tests

### Exercise: Hacking on Our Chat App

Last time: closed out talking about Middleware (Thunk, specifically). Today we'll make some more progress towards a nice lil chat app.

We'll start out with by building some functional components.

1. create new files:
  - `./components/app`
  - `./components/RoomList`
  - `./components/RoomDetails`
2. render RoomList and RoomDetails from App
3. render an unordered list of Rooms (aka channels) in RoomList
4. RoomDetails will render an individual room:
  - messages back and forth from each room member
  - also the textarea input where you can compose new messages
5. Add some styling (lil flexbox, lil custom fonts, etc.)

Next up, set groundwork for testing. Use this post for reference: [https://keita.blog/2016/02/08/javascript-unit-tests-in-a-phoenix-application/](https://keita.blog/2016/02/08/javascript-unit-tests-in-a-phoenix-application/)

1. `npm install --save-dev mocha`
2. `npm install --save-dev chai`
3. `npm install --save-dev babel-register`
4. in .babelrc, give preset:
  ```
  {
    "presets": ["es2015"]
  }
  ```
5. in package.json:
  ```
  {
    "dependencies": {
      ...
    },
    "scripts": {
      "test": "mocha --compilers js:babel-register test/js/**/*.js"
    }
  }
  ```
6. mkdir -p test/js/reducers
7. touch test/js/reducers/RoomReducerSpec.js
8. write some tests (see [example repo](https://github.com/StevenNunez/redux_chat_hedgehog))

Let's pause and talk about **immutability**.

Redux is trying to optimize whether or not to redraw things, so it's paying attention to objects and their state. When updating data, **changes should always flow through reducers and reducers shouldn't mutate state**. Instead, reducers will return new objects that are essentially copies of old ones that incorporate whatever changes you need (adding an object to array, incrementing a count, etc).

For example, don't push to an array. That mutates the original array object. Instead, set a variable to store a new array that contains old array children + new children. In our example app, this could be a new array that represents all current messages in a room.

You'll prolly find yourself using `Object.assign()` when creating these new objects.

```javascript
// pretend we're inside a Reducer
let roomToBeUpdated = {...}
  , newMessageList = [...]
  ;

let updatedRoom = Object.assign({}, roomToBeUpdated, {
  messages: newMessageList
});

let updatedRooms = Object.assign({}, state, {
  // brackets interpolate the value of room to dynamically get keys
  [room]: updatedRoom
});
```

Check out this [other post](http://reactkungfu.com/2015/08/pros-and-cons-of-using-immutability-with-react-js/) for a more thorough breakdown.

_tl;dr:_ get to know `Object.assign()` really well.


Also, here's a neat thing:

```javascript
// two ways to do the same thing
let newMsgList1 = roomToBeUpdated.concat([newMessage])
let newMsgList2 = [...roomToBeUpdated, newMessage] // uses Object Spread operator
```

Next step, incorporating Redux.

1. Let's think about what kind of data our app will need.
2. Don't worry about API too much at this point. Start building, and we'll work out the kinks.




