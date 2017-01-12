---
layout: post
title: Code Reading - Learn.co Shared React Form Library
---

Presenter: [Seiji](https://github.com/snags88)

## Problem

React is not an opionated library  
  - too many options for building forms  
  - we ended up with a bunch of different implementations / approaches

Forms are omnipresent in our app  
  - lots of repeated code  
  - need to DRY this up

## Solution

- Build out library to manage form state + validation + submission
- Build reusable form component specific to our domain

## Process

- Reviewed all our current approaches
- Melded into single, shareable component

## Shared Component Library

Can be used for controlled + uncontrolled inputs  
  - examples at `/admin/react/forms`  
  - documentation on ironboard wiki

Robust test coverage

Default styles can be overridden

Acts as building block / starting point for new form  
  - won't just magically build your entire form for you  
  - use lego blocks to build lego set


Alt Form: existing library, but outdated and not super functional, so Seiji rebuilt one specific to our app

Uses Curring pattern  
  - Referred to in React community as "high order component"

3 Objects:

1. Form Object
2. Form Component
3. Form Connector


```
Form Object      Form Component
     \                 /
      \               /
        Form Connector
            ||
            \/
        React Form
```

## Demo

Goal: build form with first name, last name submittable inputs

#### Walkthrough (propriety code omitted)

1. Create FormComponent
  - updates UI on state changes
  - holds Input and Button components
2. Build FormObject
  - stores field configuration
  - underlying implementation: AltStore
  - dispatch actions through Alt dispatcher
  - uses global Alt instance to dispatch events, so will need to namespace events
  - object that'll have field names (keys)
  - values are the validation functions
  - to declare something invalid, throw an Error
3. Import FormConnector
  - connector syncs FormObject and FormComponent states
  - render ConnectedForm into DOM
  - connector gives us lotsa good props to use
  - passes FormObject into FormComponent, returns React component w/ everything needed to render
4. Import individual input components from shared react components (ex. TextFieldInput, etc)
  - Use REST spread syntax to pass fields into props:

    ```javascript
    // generic example
    var props = {
      a: 'hello',
      b: 'world'
    }
    var {a, ...rest} = props

    // input component example
    const {fields} = this.props

    <TextFieldInput
      {...fields}
      labelText='Cool label'
    />
    ```
5. Question: how to validate server-side?
  - might need more configuration, but FormObject does handle `onFormSubmitFailure`
6. Sending data to server
  - configure `onSubmit` function on FormObject
  - takes in current form state
  - that function makes a POST request to whatever endpoint


## Conclusion

Bit of a learning curve, but should help standardize our forms.

Handles decision making for you

Abstracts out state management

Goal: open source this project  
  - good resource for Alt users  
  - nothing else out there as good as ReactForms
