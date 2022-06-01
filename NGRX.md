# Angular NGRX

**Redux Pattern**

- Store: central place where application state is held. Components and services read state from the store
- Actions: Components and Services dispatch actions to change state. The action object (which has a class) contains an identifier of what state to change, and a payload to add to that slice of state
- Reducer: Function that takes in the current state, and an action to update the relevant slice of state specified by the action. The entire state object is replaced with the updated one.
- Effects: Used for async operations that produce side effects (HTTP calls), where the same input can yield different potential outputs. For each potential output, the effects file dispatches a different action.

## Store Setup

`npm install --save @ngrx/store @ngrx/effects @ngrx/store-devtools @ngrx/router-store`

In app.module.ts;

`import { StoreModule } from '@ngrx/store';` , add to imports []

In the component, you don't need to unsubscribe from the store state selection observable if you are using an async pipe
However, if you subscribe to the store selection observable in the component, you should also unsubscribe

## Actions

An action identifier must be unique throughout the application

## Reducers

Reducer functions must by default return the initial state

// Both the LOGIN and SIGNUP can be grouped since they change state the same way

## NGRX Side Effects

Writing async code in reducer files is forbidden. For http events like user signin, the part that handles the http request is handled by an effect. And the success and/or failure response is handled by the reducer.

// actions$ gives access to all dispatched actions
// use ofType() to specify the different types of effects you want to handle

// New (success/failure) actions are always dispatched once the async code is done
// They are dispatched a little differently in the effects (actions are returned in order to be dispatched);
return new AuthActions.LoginSuccess(user); instead of
this.store.dispatch(new AuthActions.LoginSuccess(user));

// Unlike in the success block, in catchError block, the returned error data must be wrapped in of( )

// pass {dispatch: false} as a second arg to createEffect for success/fail actions where there will be no further action to be dispatched
