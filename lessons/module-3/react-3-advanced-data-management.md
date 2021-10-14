---
title: "React: Advanced Data Management"
length: 180
tags: react, ideabox, lifecycle methods, prop types, conditional rendering
module: 3
---

## Learning Goals

* Understand the component lifecycle
* Understand class component lifecycle methods
* Implement conditional rendering
* Implement `PropTypes` and understand why they are useful

## Vocabulary

- `lifecycle method` a set of methods found in the parent class Component that fire at different points during the component lifecycle
- `PropTypes` a library we use to validate the data type of props coming into a component; allows for more specific, helpful error messages

<section class="call-to-action">
### Warm Up

You're going to learn about some important lifecycle methods today: `componentDidMount`, `componentDidUpdate`, and `componentWillUnmount`. Before we jump too far into the concepts for today, let's do some exploring.  

First, follow the following instructions:
```bash
git clone git@github.com:turingschool-examples/react-iii-ideabox.git
cd react-iii-ideabox
git checkout lifecycle-warmup
npm start
```

Then, follow these steps:
1. Uncomment lines 15-21 in the `App.js` file.
2. Open up the `Console` in your Dev Tools.
3. Refresh the page. What logged to the console?
4. Add an idea. What logged to the console?
5. Add another idea. What logged to the console?
6. Delete an idea. What logged to the console?
7. Now, comment lines 15-21 in the `App.js` file so they are no longer active.
8. Uncomment lines 6-16 in the `Card.js` file.
9. Refresh the page. What logged to the console?
10. Add an idea. What logged to the console?
11. Add another idea. What logged to the console?
12. Delete an idea. What logged to the console?
  
Now, answer these questions:
*(It's okay if you're wrong! Just make your best guess!)*
- When does `componentDidMount` run? How many times does it run?
- When does `componentDidUpdate` run? How many times does it run?
- When does `componentWillUnmount` run? How many times does it run?
</section>

## The Component Lifecycle

Did you ever learn about the butterfly lifecycle when you were a kid? It starts out as an egg, hatches into a caterpillar, eats a bunch, wraps itself into a cocoon, emerges as a butterfly? Well, React components have lifecycles, too.

A React component goes through 3 distinct phases during its lifecycle:   

* Mounting (aka Birth)
* Updating (aka Growth and Change)
* Unmounting (aka Death)

Dan Abramov [tweeted](https://twitter.com/dan_abramov/status/981712092611989509) a diagram that helps explain some of the React 16 lifecycle methods.

![React 16 component lifecycle methods chart](https://pbs.twimg.com/media/DZ-97vzW4AAbcZj?format=jpg)

We'll take a deeper look into some of these methods in this lesson. There are a few that you will probably use throughout your time at Turing. This simplified diagram of the same lifecycle is what we will talk about in depth ([created here](http://projects.wojtekmaj.pl/react-lifecycle-methods-diagram/)):

![React 16 component lifecycle methods chart simplified](/assets/images/react-lifecycle-simplified.png)

## Mounting Phase

### constructor() && super()

Let's talk about the first methods we see in a class-based React component.  

```js
constructor() {
  super();
  this.state = {
    name: ''
  };
}
```

Per [the docs](https://facebook.github.io/react/docs/react-component.html#constructor), the `constructor()` method is called once, and before the component is mounted onto the DOM. It is the first and only function called automatically whenever a `class`-based component is created.  

Any class in JavaScript has this rule: Within the constructor, it's important to immediately call `super()` in cases where our class extends any other class that also has a defined constructor.

Invoking `super()` will run the constructor method of the parent class and allows it to initialize itself. This invocation allows `this` to have a defined value **within the constructor**. Constructors are great for setting up our component and initializing state. However, this is **NOT** a place you should consider making a network request.

This does not mean that every class NEEDS a constructor. The [default constructor](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Classes/constructor#Default_constructors) is used if you aren't modifying it. There is even an [eslint rule](http://eslint.org/docs/rules/no-useless-constructor) for detecting the use of a default constructor. Basically, if you don't need to initialize state or bind any methods, you don't need to implement a constructor for your component! It will get called automatically behind the scenes. Magic!

### Where are my props?  

If, in your constructor, you need access to that component's `props`, you _must_ pass these as an argument down through `super()`.

```js
constructor(props) {
  super(props);
  this.state = {
    name: props.initialName
  };
}
```

Without passing the props down into this method, `this.props` will return as `undefined` **within the `constructor` method**.  
This is bad because the constructor method is the first function called when your component is instantiated as a class - knowing the context of `this` from the get go can be a big deal.   

<section class="note">
### Note  

Whether or not you have a constructor method has no effect on `this` or `this.props` within the `render()` method (or any other method you create for that matter) - the `render()` method defines its own context.  
</section>

Per [the docs](https://reactjs.org/docs/state-and-lifecycle.html#adding-local-state-to-a-class), class components should always pass props to the base constructor (this is why we pass them into the `super()` method). However there is some debate as to why this is suggested, since React will automatically set the props for you once the constructor has fired.

### render()

Now that our component has been initialized and configured, we can begin rendering some content/elements on the page. This is the lifecycle method that React developers are most familiar with. It is the one lifecycle method that exists across all phases of a React component.

<section class="call-to-action">
### Turn and Talk

It's important to remember to always keep `render()` a **pure method**. This means we should never call `this.setState()` within `render()`. Why do you think this is?  
</section>

#### Managing Children Components and Mounting

The React Element that has been rendered by the initial `render()` may possibly have a number of children that need to be rendered as well. This is where each of those children kickoff their own lifecycle methods... `constructor()`, `render()`, and `componentDidMount()`. Once all the children have been successfully created, initialized and mounted, the parent's `componentDidMount()` method will finally be called.

### componentDidMount()

Per [the docs](https://reactjs.org/docs/react-component.html#componentdidmount), `componentDidMount()` is invoked "immediately after a component is mounted." When `componentDidMount()` is called, this signalizes that the component - and all its sub-components - have rendered properly.  This is also the go-to location to fire off an API call or network request (**BEST PRACTICE**). This is a great [blog post](https://www.robinwieruch.de/react-fetching-data/) that discusses how and where to fetch data in React.

<section class="note">
### Note

Setting state in this method **WILL** trigger a re-render - but will not cause an infinite re-rendering loop, because `componentDidMount` only fires after the initial mount of the component.
</section>

## Updating Phase

### componentDidUpdate()

This method is invoked *after* updating occurs.  It is **NOT** called for the initial render.  Per the [docs](https://reactjs.org/docs/react-component.html#componentdidupdate): "Use this as an opportunity to operate on the DOM when the component has been updated. This is also a good place to do network requests as long as you compare the current props to the previous props (e.g. a network request may not be necessary if the props have not changed)."  For example, if the `userId` in your new props is not the same as the one that is being currently displayed, you could fetch the new user.

<section class="note">
### Note

You are able to call `setState()` but only if it's wrapped in a conditional.  Otherwise, an infinite loop will happen just like it does in the `render()` method.
</section>

## Unmounting Phase

### componentWillUnmount()

This is invoked just once, before the component is unmounted and destroyed. According to the [docs](https://reactjs.org/docs/react-component.html#componentwillunmount), it's a good place to cancel any open network requests, kill any timers, and do any other necessary cleanup tasks.  You should **NOT** called `setState` here because the component will not re-render.

## Workshop On Our IdeaBox

Okay. Why do we care about any of this?

To be honest, you probably won't be using much more than `constructor()`, `super()`, `render()`, and `componentDidMount()` in Mod 3. However, it's good to be aware that other lifecycle methods exist.

We're going to implement `componentDidMount()` in our IdeaBox.

### Setup Your Front End

We are going to use the same repo we used for the warm up, but you'll need to switch branches:

```bash
git checkout main
```

### Setting Up Your Back End

We're also going to be running a back end server, which will function as our API! Lucky you, we've written up the backend for you, complete with documentation and friendly error messages.

Clone down [the repo](https://github.com/turingschool-examples/ideabox-api) - but NOT inside your existing IdeaBox repository!

```bash
git clone https://github.com/turingschool-examples/ideabox-api.git ideabox-api
cd ideabox-api
npm i
node server.js
```

Note that the frontend should be running on `localhost:3000` and the backend should be running on `localhost:3001`.

Once you are set up, you can visit `http://localhost:3001/api/v1/ideas` and you should see a brief list of ideas!

### Implement Getting Data from the Backend

<section class="call-to-action">
### Your Turn -- Review `.fetch()`
 Take some time to look into the <a href="https://developer.mozilla.org/en-us/docs/web/api/fetch_api/using_fetch" target="_blank">fetch API documentation</a>.
 Answer the following questions:
  <section class="answer">
### What does `.fetch()` do?
  `.fetch()` is a method provided by the fetch api. It allows us to make a network request.     
  </section>
  <section class="answer">
### What does `.fetch()` return?
`.fetch()` returns a promise.
  </section>
  <section class="answer">
### What does `.then()` do? What is the method called on? What does it return?
`.then()` is a promise prototype method that runs when the promise object it is chained to successfully resolves. It returns a new promise object.
  </section>
  <section class="answer">
### What does `.catch()` do? What is the method called on? What does it return?
`.catch()` is a method called on a Promise object (aka a Promise prototype method). It runs when the first promise it is chained to fails / rejects. It returns a promise object.
  </section>
</section>

Okay. Let's refactor our IdeaBox so that we are consuming data from our API!

<section class="call-to-action">
### Your Turn

Pair up and work together on implementing the functionality to `fetch` our ideas from the API!  Use the docs for the API to figure out what endpoint you need to hit.  Also think about what lifecycle method this should live in.  Once you have the ideas being rendered, give your partner a high five!
</section>

As we've noted before, we should use our catch somehow in order to display an error if something goes wrong.  Try stopping your backend server and see what happens.  Your catch should before firing (because the Promise rejected).  Let's do something with that error.

Let's edit our intial state in our `App.js`:

```js
// App.js

this.state = {
  ideas: [],
  error: ''
};
```

Then inside of your catch, set the error message in state. Now, let's _do_ something with that information! Such as displaying it to the user.

### Conditional Rendering

We don't want an error message showing all the time. So ... let's make use of conditional rendering! Let's take a look at our current `render()` method:

```jsx
// App.js

render() {
  return (
    <main className="App">
      <h1>IdeaBox</h1>
      <Form addIdea={this.addIdea} />
      <Ideas
        ideas={this.state.ideas}
        removeIdea={this.removeIdea}
      />
    </main>
  );
}
```

If we wanted to add an `h2` that would show up if we had an error, what would we write?

```jsx
// App.js

render() {
  return (
    <main className="App">
      <h1>IdeaBox</h1>
      <Form addIdea={this.addIdea} />
      {this.state.error && <h2>{this.state.error}</h2>}
      <Ideas
        ideas={this.state.ideas}
        removeIdea={this.removeIdea}
      />
    </main>
  );
}
```

Wowwwww okay. What does that syntax even mean?

First of all, this is JavaScript (we can tell because of the curly brackets). The first statement of `this.state.error` will evaluate to a truthy or a falsy value (an empty string is falsy). The code after the double ampersand is what will render if we get past the first statement.

It's a shorthand way of saying, "If there is an error in state, render the error inside h2 tags!"

Neato!

Notice that we are only conditionally rendering the one part of the render that is contingent upon whether or not an error is in state.

We're not rendering two different versions of the App. We just have the one, and one line will show up only if there is an error stored in state.

<section class="call-to-action">
### Your Turn

Imagine we have a slow connection or need to load A LOT of data.  We might want to implement some kind of loading icon.  Using conditional rendering, display a loading icon while the fetch is retrieving the data.
</section>

## PropTypes

PropTypes allow you to specify what type of props you are expecting in a certain component. This is also known as "typechecking".

Many people have moved to implementing languages like [TypeScript](https://www.typescriptlang.org/) or [Flow](https://flowtype.org/) for this exact purpose, but even without any additional technologies, the `prop-type` module lets you establish a safety net with very little effort.

Let's say you declare a component `<Card title={this.state.title}/>`. Here, your component is expecting a prop called `title` and you (probably) expect it to be a string. If you define that value within your `propTypes` object and it comes in as something else - say for example the API you are consuming has changed and now you have an array of strings - you will get a helpful warning message in your console.  

Previously, PropTypes was built into the React library itself, however it has since been extracted into its own module. You can install it like any other module.

```bash
$ npm install prop-types
```

In React, `PropTypes` are declared like this:

```jsx
// Card.js

import React from 'react';
import PropTypes from 'prop-types';
import './Card.css';

const Card = ({ id, title, description, removeIdea, isFavorite }) => {
  const favoriteClass = isFavorite ? 'favorite' : 'card';

  return (
    <section className={favoriteClass}>
      <h3>{ title }</h3>
      <p>{ description }</p>
      <button onClick={() => removeIdea(id)}>🗑</button>
    </section>
  );
};

export default Card;

Card.propTypes = {
  title: PropTypes.string
};
```

The error you will see if the component gets something besides a string would look something like this:  

```
Warning: Failed prop type: Invalid prop `name` of type `array` supplied to `Card`, expected `string`.
```

Without using PropTypes, you would have seen a vague error when the Component failed to render. With PropTypes, we can see that the error was that we were receiving an array when we expected a string.

### Class propTypes

Check out a complete list of `propTypes` and examples of usage [here](https://facebook.github.io/react/docs/typechecking-with-proptypes.html#react.proptypes).  

By default, all props specified within the `Class.propTypes` object will be considered optional. There are many instances where your component will not function correctly without that particular prop. To add a validation that will fire an error message if a prop does not show up at all, simply add `.isRequired` to the end of the propType declaration.  

```js
Card.propTypes = {
  title: PropTypes.string.isRequired
};
```

You can also be more generic - let's say you need a prop to come in but it doesn't matter what type it is as long as it's there. Instead of specifying a particular JS primitive you can use `.any`.

```js
Card.propTypes = {
  title: PropTypes.any.isRequired
};
```

<section class="call-to-action">
### Your Turn

Take a few minutes and finish writing up the rest of the propTypes for our `Card` component.  

</section>

<section class="call-to-action">
### Your Turn

Take the next 5 minutes to look up the following prop types and understand what they do. We will circle back to talk about
these particular methods when you are done.  

- `PropTypes.oneOf()`  
- `PropTypes.arrayOf()`  
- `PropTypes.objectOf()`  
- `PropTypes.shape()`

</section>

## DefaultProps

Just like when writing functions, React also allows us to provide a default value for props. [defaultProps](https://facebook.github.io/react/docs/typechecking-with-proptypes.html#default-prop-values) let you ensure that a value will be passed through. This helps eliminate some of the incessant ternaries that either render the prop or an empty string, for instance.  

```jsx
const Card = ({ id, title, description, removeIdea, isFavorite }) => {
  const favoriteClass = isFavorite ? 'favorite' : 'card'

  return (
    <section className={favoriteClass}>
      <h3>{ title }</h3>
      <p>{ description }</p>
      <button onClick={() => removeIdea(id)}>🗑</button>
    </section>
  );
};

export default Card;

Card.propTypes = {
  id: PropTypes.number.isRequired,
  title: PropTypes.string.isRequired,
  description: PropTypes.string.isRequired,
  removeIdea: PropTypes.func.isRequired,
  isFavorite: PropTypes.bool.isRequired,
};

Card.defaultProps = {
  isFavorite: true
};
```  

<section class="note">
### Note

The `propTypes` typechecking validations fire AFTER `defaultProps` have been resolved. This allows for the default props to fill themselves in before any warning messages are thrown.
</section>

<section class="checks-for-understanding">
### Reflect

Take a few minutes to journal to write notes to each of these questions:

* What "aha" moments did you have?
* Which concepts are the fuzziest for you right now?
* Which concepts are the clearest for you right now?
* What do you know about `componentDidMount`?
* What's the purpose of propTypes?
</section>

## Required Homework
<section class="call-to-action">
### Life Cycles Diagram
Diagram out the react component lifecycle. Feel free to follow the diagram at the top of the lesson.
Your diagram should include a description of the following methods, an example of what you might use them for, and an indication of when in the lifecycle they run:
- `constructor`
- `render`
- `componentDidMount`
- `componentDidUpdate`
- `componentWillUnmount`
</section>

## Suggested homework

<section class="call-to-action">
### Extra Reading on PropTypes

Now that we've talked about the most obvious use cases of propTypes to preemptively debug your code, read the following two articles - you are highly encouraged to take notes:  
- [Better Prop Validation](https://medium.com/@MoeSattler/better-prop-validation-in-react-cc83590d311f#.8z6wszfzn).  
- [Writing A Good React Component](https://thoughts.travelperk.com/writing-a-good-react-component-59624ed40b8e#.64wzjk4qc)
</section>

<section class="call-to-action">
### Additional Async Practice

Right now, our IdeaBox will work as expected. However, you'll notice that our `addIdea` and `deleteIdea` methods are only manipulating our application state, rather than interacting at all with our back-end.

Go take a look at the BE repo's README. There is documentation for adding and deleting ideas.

Refactor IdeaBox to use these endpoints. Hint: you might need to write more fetches...

When you have completed this functionality, compare it to the solution on the `react-iii-complete` branch.
</section>

<section class="call-to-action">
### Additional Steps

Take a look ahead to our async testing lesson and pull the fetches into their own files and try to write out a few async tests!
</section>

## References

- [React Lifecycle Docs](https://reactjs.org/docs/react-component.html)
- [The React Life Cycle](https://developmentarc.gitbooks.io/react-indepth/content/life_cycle/introduction.html)
- [Understanding Component Lifecycle](https://medium.com/@baphemot/understanding-reactjs-component-life-cycle-823a640b3e8d)
- [Lifecycle Simulators](https://reactarmory.com/guides/lifecycle-simulators)
- [Dan Abramov's Twitter chart/convo on modern React lifecycle methods](https://twitter.com/dan_abramov/status/981712092611989509)
- [Error handling in React 16](https://reactjs.org/blog/2017/07/26/error-handling-in-react-16.html)
- [Update on Async Rendering](https://reactjs.org/blog/2018/03/27/update-on-async-rendering.html)
- [React v16.3.0: New Lifecycles](https://reactjs.org/blog/2018/03/29/react-v-16-3.html)
