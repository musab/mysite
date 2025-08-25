---
title: "React Redux 1-2-3"
date: 2017-11-28
draft: false
---

*Originally published on [Medium](https://medium.com/@__haqq/react-redux-1-2-3-f116fd952c7)*

A lot of people I speak to say:

> I understand both the point and the reason for Redux… I just get lost when trying to implement Redux into my React app.

This article is going to break down the 3 steps I use to implement Redux into my React apps. Mostly following community standards plus little of my opinion sprinkled on to help organize directories.

**Disclaimer:**

- This assumes basic terminal and React knowledge
- All following quotes are directly from the Redux docs. Definitely re-read the docs after going through this article, and I'm sure the Redux docs will make much more sense
- Final project repo at the end
- I suggest going over the article once without following along, then re-read and code along

Let us start on the same page, begin by creating a new React app using create-react-app name it anything you would like.

Now overwrite `src/App.js` with the following code:

```javascript
import React, { Component } from "react";

class App extends Component {
  constructor(props) {
    super(props);
    this.state = {
      people: [{ id: 1, name: "Mesut Ozil" }],
      textInput: ""
    };
  }
  
  handleClick = () => {
    console.log(this.state.textInput);
  };
  
  handleChange = event => {
    this.setState({ textInput: event.target.value });
  };
  
  render() {
    return (
      <div className="App">
        <p>Add a New Player</p>
        <input type="text" onChange={this.handleChange} />
        <button onClick={this.handleClick}>Add</button>
        <ul>
          {this.state.people.map(p => {
            return <li key={p.id}>{p.name}</li>;
          })}
        </ul>
      </div>
    );
  }
}

export default App;
```

This will be our starting point. A list of players, a text input to add new players, and a button to append the new player. `npm start` your project and play around and get familiar with our starting point if you would like.

Now that we're all on the same page, time to introduce Redux.

Here is our global objectives for this article:

1. Take Mesut Ozil out of our App component, and into our Redux store (global state)
2. When we click on Add, new players will be added into our Redux store (global state), and then appended to our list of players when running the React app

## Step 1 out of 3

- Install Redux dependencies
- Begin our file structure
- Create our first action

We need to install two packages. In your terminal enter:

```bash
npm install --save react-redux redux
```

Now in your `/src/` directory create a new folder, `redux` in this folder we will contain all our Redux files and activities in one central place.

**Disclaimer:** majority of the Redux community do not create a redux directory. They tend to create the redux directories (ie. actions, reducers) right into the `/src/` directory. But I feel it helps me keep all Redux related data in one place to quickly navigate.

Now let's create our first action.

An action is how your global state is changed. You can think of an action as an event that is fired off (from a button click, when a component mounts).

> Actions are payloads of information that send data from your application to your store.

We'll create an action that will add a new player to our list, and the action is fired when we click on the Add button.

Create a new directory `/src/redux/actions` now in here let's create a new file and name it `player.js`. Open your `player.js` file and create the first action:

```javascript
export const ADD_PLAYER = "ADD_PLAYER";

export const addPlayer = player => ({
  type: ADD_PLAYER,
  data: player
});
```

The first line will allow us to use this action type in other files (which you will see in the next step). Additionally not using a string literal will make bugs easier to find. For example, If we mistype the action type in any files we'll receive a clear undefined error.

Beneath the action type is our action. Doesn't look too complicated right? Plain 'ol JavaScript function.

Takes the player we want to add, we input our action type and then we'll send it over to the next step in our assembly line. The object we are returning will make much more sense in our next reducer step.

Before wrapping step 1, let's test this new action we created in our app.

Head into your `/src/app.js` file and add `import { addPlayer } from "./redux/actions/player.js";` beneath `import React,...`

Now overwrite what's in the handleClick function with:

```javascript
handleClick = () => {
  console.log(addPlayer(this.state.textInput));
};
```

Now `npm start` your app, type anything into the text input and click Add. In your developer console you should now see what an action looks like:

```javascript
{type: "ADD_PLAYER", data: "Alexis Sanchez"}
data:"Alexis Sanchez"
type:"ADD_PLAYER"
```

This is what our Action looks like! This is what we're sending to Step 2 to determine what to do.

As you can see the Action is just logging and sending what happened. It's up to our next step to determine what to do.

This step we setup our environment, created our first action and logged it to the console.

## Step 2 out of 3

Create our reducers

> Actions describe the fact that something happened, but don't specify how the application's state changes in response. This is the job of reducers.

Let's create our reducer.

Stop and contemplate over the word reducer, this will help visualize this step. What we are doing is based off the action type (ADD_PLAYER) we will be REDUCING to the correct path to take to modify the state.

In your redux directory create a new folder and name it `reducers`. Now in your reducers directory create 2 files: 1) `index.js`, 2) `player.js`.

The directory structure should look like this so far:

```
-redux
--actions
---player.js
--reducers
---index.js
---player.js
```

Let's first open `reducers/index.js` and enter the following:

```javascript
import { combineReducers } from "redux";
import player from "./player";

export default combineReducers({
  player
});
```

In our case we only have one reducer that will handle our actions, the player reducer.

Imagine you had a large app with different actions coming from different places from your application. For example, signing in actions, changing default language actions, adding new player actions…

Instead of having 1 large reducer file, you can break them into their own their own respective file.

> As your app grows more complex, you'll want to split your reducing function into separate functions, each managing independent parts of the state.
> 
> The combineReducers helper function turns an object whose values are different reducing functions into a single reducing function you can pass to createStore.

The point of this file is to combine all our reducers. In our case just the single `player.js` reducer. The only time you would need to revisit this file is to add a new reducer you have created.

Now access our next file, `reducers/player.js` to begin writing our first reducer.

```javascript
import { ADD_PLAYER } from "../actions/player";

const initialState = {
  people: [{ id: 1, name: "Mesut Ozil" }]
};

function returnNameObj(name) {
  return {
    id: Date.now(),
    name
  };
}

const reducer = (state = initialState, action) => {
  switch (action.type) {
    case ADD_PLAYER:
      return {
        ...state,
        people: [...state.people, returnNameObj(action.data)]
      };
    default:
      return state;
  }
};

export default reducer;
```

Remember the action type we created in our `actions/player.js` file? This is where we are importing it, again to help with typo bugs.

We have a helper function `returnNameObj` whose job is to create our new player object that we can append to our people array of objects (see initial state). The function returns a JavaScript object with the name of the new player we're adding and also generating a unique ID using `Date.now()`. In the real world I'd recommend using something like UUID.

Then we are going to create our initial state, this is exactly how we setup our initial state in `/src/App.js` in our component state, but now we're going to bring the initial state into redux.

Now the reducer function itself, we take in two arguments:

1. Our initial state (which we created above)
2. The action that is coming in from our `actions/player.js` file

When we take the action, we delegate which changes to the state we are going to make based on the action type. Remember step 1 was to capture which event was fired off, now we are determining what to do based off that action.

For example, if the action that came into the reducer had an action type of `ADD_PLAYER` then we are going through a plain JavaScript switch statement to determine which changes to the state we should make.

> Things you should never do inside a reducer: Mutate its arguments

Hence why when we hit our `ADD_PLAYER` case, we first copy our existing state then go on to create our new state. Never modifying the current state. The 3 things happening, when an `action.type` matches a switch case:

1. Copy our existing state
2. Add our new player that came with the action — remember `data: player` when we were creating our actions? The action holds `action.type` and `action.data`
3. And finally return this to our store who holds our application state (we'll be going over store in the next step)

## Step 3 out of 3

Create our store

> In the previous sections, we defined the actions that represent "what happened" and the reducers that update the state according to those actions.
> 
> The Store is the object that brings them together. The store has the following responsibilities:
> 
> - Holds application state;
> - Allows access to state via getState();
> - Allows state to be updated via dispatch(action);

We're almost there! We now need to create our store. The glue to bring together the actions, reducers, and state.

The good thing about the store file is, you create it once and never really have to revisit much.

In the redux directory create a new file name it `store.js` with the following:

```javascript
import { createStore } from "redux";
import reducer from "./reducers";

const store = createStore(reducer);
export default store;
```

> createStore Creates a Redux store that holds the complete state tree of your app. There should only be a single store in your app.

Simple right? We are using the createStore function, and passing in the reducer function whose in charge of updating our state depending on which action it was sent. Steps, 1–2–3.

One last piece before we're done with Redux, we need to wrap our React app with our Redux store which is quite literally 2 lines of code.

Open `/src/index.js` start by importing Provider at the top:

```javascript
import { Provider } from "react-redux";
```

Then our Redux store:

```javascript
import store from "./redux/store";
```

Now we need to wrap our `<App />` with the provider so we can access our Store from any of our Components we create, you can visualize this step as we're sending the data down to all our components to access.

```jsx
<Provider store={store}>
  <App />
</Provider>
```

Can you see the pattern now?

1. Action fires off, sent to Reducer
2. Reducer determines how the state is affected
3. The single Redux store state tree is updated

**ARS — Action, Reducer, Store**

ARS — Action, Reducer, Store. Lock this acronym down. My favourite soccer team is ARSenal, that's how I memorize this flow.

Guess what folks? We're DONE!

The only piece that's left is connecting the React component to our Redux state.

Jump back into our `/src/App.js` file and let's connect our component to our Redux store.

Start by importing connect from react-redux:

```javascript
import { connect } from "react-redux";
```

Now scroll to the bottom and update your export by wrapping it with connect which looks like:

```javascript
export default connect(mapStateToProps)(App);
```

And the last step to have access to our Redux state in the component is to create our `mapStateToProps` function, which receives the Redux state due to our wrapping with `connect` then allows us to access the state from our component props.

Just above the export, let's create the function:

```javascript
const mapStateToProps = state => {
  const ourReduxState = state;
  return { ourReduxState };
};
```

We can name our redux state anything here and export for our component to access. I called it `ourReduxState` to help you visualize, we can now access this state from our component props.

Let's delete our state we created in our `/src/App.js` and loop through our Redux state to list our players. Update `this.state.people.map` with `this.props.ourReduxState.player.people.map` and everything should look the same. The difference? We are pulling data right from our Redux state instead.

The reason for `player.people` is because the way we setup our reducer, head back into `player.js` in your reducer and examine our Initial State.

And the last last step, (promise) update your `handleClick` function in `/src/App.js` file with:

```javascript
handleClick = () => {
  this.props.dispatch(addPlayer(this.state.textInput));
};
```

Now type into the text input and click Add, the new player should appear on screen — #mindblown. This player is going through our ARS flow. What does ARS stand for again?

When we click ADD, the data in our text input is being sent as an action, that goes to our reducer, which then updates our redux store.

**ACTION**  
**REDUCER**  
**STORE**

Since we wrapped our component with `connect` the component knows to re-render itself to display the newly added data to our Redux store.