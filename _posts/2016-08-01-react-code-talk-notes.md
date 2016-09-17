---
layout: post
title: React Code Talk Notes
---

## Overview

- library that renders data to the DOM
- can add Flux architecture after you're comfortable with React

## Setting up a React app in our codebase

### General Notes

- avoid using jQuery so we can remove that dependency
- use vanilla JS where possible instead
- Gulpfile is not watching these files, so will need to re-compile manually after making changes

### Setup

##### 1. Add `_src` file (`_src/code-reading.js`)

```javascript
// _src/code-reading.js

var React = require('react')
  , ReactDom = require('react-dom')
  , AppComponent = require('code-reading/app.jsx')
  ;

document.addEventListener('DOMContentLoaded', function () {
  ReactDom.render(<AppComponent/>, document.getElementById('js-main'));
});
```

##### 2. Create manifest (`javascripts/code-reading.js`) and require src file in via sprockets

```javascript
// javascripts/code-reading.js

//= require _bundles/code-reading
```

##### 3. Add to asset pipeline (`application.rb`) and blacklist

##### 4. Create AppComponent (`/javascripts/code-reading/app.jsx`)
  - `React.createClass(){}` requires render function
  - **React can only return one top level node**

```javascript
// javascripts/code-reading/app.jsx

var AppComponent = React.createClass({
  render(){
    return(
      <div>Hello World!</div>
    )
  }
});

module.exports = AppComponent;
```

## Components Overview

Two things to pay attention to: state & props

Resource: https://github.com/uberVU/react-guide/blob/master/props-vs-state.md

### Pass prop to component

```javascript
// _src/code-reading.js

var React = require('react')
  , ReactDom = require('react-dom')
  , AppComponent = require('code-reading/app.jsx')
  ;

document.addEventListener('DOMContentLoaded', function () {
  ReactDom.render(<AppComponent text={'hello world 2'}/>, document.getElementById('js-main'));
  // You can pass html by using `dangerouslySetInnerHTML={{__html:'<div>stuff<div>'}}`
});

// javascripts/code-reading/app.jsx

var AppComponent = React.createClass({
  render(){
    return(
      <div>{text}</div>
    )
  }
});

module.exports = AppComponent;
```

## Managing State at the App level

Use `getInitialState` and `onChange` events inside elements.

```javascript
// javascripts/code-reading/app.jsx

var AppComponent = React.createClass({
  getInitialState() {
    return {
      listCollection: {},
      inputText: ''
    }
  },

  render(){
    var text = this.props.text
      , inputValue = this.state.inputText
      ;

    return(
      <div>
        // React has more complext Form Components
        <input
          type='text'
          value={inputValue}
          onChange={this.updateInputText}
        />
        <input
          type='submit'
          value='Add'
          onClick={this.submitItem}
        />
      </div>
    )
  },

  updateInputText(e){
    var text = e.target.value;

    // this.state.input = text; // BAD PRACTICE, DO NOT USE

    this.setState({
      inputText: text
    });
  },

  submitItem(e){
    this.state.listCollection.push(this.state.inputText);

    this.setState({
      listCollection: this.state.listCollection,
      inputText: ''
    });
  }
});

module.exports = AppComponent;
```

## Rendering List Items

Normally separate out into ListCollection and ListItem components.

```javascript
// javascripts/code-reading/app.jsx

var AppComponent = React.createClass({
  getInitialState() {
    return {
      listCollection: {},
      inputText: ''
    }
  },

  render(){
      var inputValue = this.state.inputText
      , listItems = this.state.listCollection
      ;

    return(
      <div>
        // React has more complext Form Components
        <input
          type='text'
          value={inputValue}
          onChange={this.updateInputText}
        />
        <input
          type='submit'
          value='Add'
          onClick={this.submitItem}
          disabled={!this.isValid()}
        />
        <ol>
          {
            listItems.map((item, i) => { // ES6 fanciness
              return (
                <li key={i}>{item}</li> // use index as key for item
              )
            });
          }
        </ol>
      </div>
    )
  },

  updateInputText(e){
    var text = e.target.value;

    // this.state.input = text; // BAD PRACTICE, DO NOT USE

    this.setState({
      inputText: text
    });
  },

  isValid() {
    return !!this.state.inputText.length;
  },

  submitItem(e){
    this.state.listCollection.push(this.state.inputText);

    this.setState({
      listCollection: this.state.listCollection,
      inputText: ''
    });
  }
});

module.exports = AppComponent;
```

## Debugging

Use Chrome React extension to inspect DOM

