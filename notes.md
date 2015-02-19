# Flux Notes #

In flux pattern, a **controller** is a React component that gets state from a
store and passes it down to child components.

* Controllers have a ref to actionCreators and stores. They call methods on actionCreators (to update state) and stores (to retrieve state) and also listen for events on stores. They pass the state updates down to children (UI components) as props.

* Stores have a ref to dispatcher. They call the `register(cb)` method.

* ActionCreators have a ref to dispatcher. They call domain-specific methods implemented on the dispatcher.

**Stores** - manage application state for a specific domain. This is different
than the model pattern. One store has state for many models. "Models" here
are simple object literals (not objects with methods).

Stores have a ref to the dispatcher, and they register a CB with it to receive new state. They emit events that are listened  by controller-views (React components that act as controllers). The controller-views then update the UI.

Stores use the `type` attr of an action to figure out if/how they will use the action.

**Action** - an object literal containing new fields and a type attr. Create a
library of methods called `ActionCreators` that create the action object
and pass it to the dispatcher. The actionCreators are used by components in the chat example. Components have a ref to actionCreators, actionCreators have a ref to the dispatcher.

**Dispatcher** - organizes/resolves dependencies between stores. Stores have a
ref to the dispatcher and call the `Dispatcher.register(cb)` method.

----

```js
// Store example
// from https://scotch.io/tutorials/getting-to-know-flux-the-react-js-architecture
var AppDispatcher = require('../dispatcher/AppDispatcher');
var ShoeConstants = require('../constants/ShoeConstants');
var EventEmitter = require('events').EventEmitter;
var merge = require('react/lib/merge');  // merge = object-assign = _.extend

// Keep state here
var _shoes = {};

// Method to load shoes from action data
function loadShoes(data) {
  _shoes = data.shoes;
}

// Merge our store with Node's Event Emitter
var ShoeStore = merge(EventEmitter.prototype, {

  // Return hash of all shoes
  // The controller calls this to get state.
  getShoes: function() {
    return _shoes;
  },

  emitChange: function() {
    this.emit('change');
  },

  addChangeListener: function(callback) {
    this.on('change', callback);
  },

  removeChangeListener: function(callback) {
    this.removeListener('change', callback);
  }

});

// Register dispatcher callback
AppDispatcher.register(function(payload) {
  var action = payload.action;
  var text;
  // Define what to do based on action.type property
  switch(action.type) {
    case ShoeConstants.LOAD_SHOES:
      // Call internal method based upon dispatched action
      loadShoes(action.data);
      break;

    default:
      return true;
  }

  // If action was acted upon, emit change event. This is how we communicate
  // with the controllers.
  ShoeStore.emitChange();

  return true;

});

module.exports = ShoeStore;
```
