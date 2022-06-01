# Angular NGRX (new version)

npm install @ngrx/core @ngrx/store @ngrx/effects @ngrx/store-devtools @ngrx/router-store --save
or ng add @ngrx/store@latest

Basic Flow
For each user event, we dispatch an action to the reducer
The reducer takes the action (and its payload) along with the relevant slice of state
to produce new state. The store is replaced with this new state.

**State**

Readonly global object serving as the single source of truth for the application's data.
The store is an observable of state, and also an observer of actions

In the parent module (or in app module) create a 'store' directory. In it have folders for state, actions, effects, reducers

**Actions**

Identifiers of operations for reducers or effects. Actions are dispatched by other files (components and effects).
If the operation is synchronous (no side effects), the action is handled by a reducer. Otherwise, its handled by an effects.
The Action object has a type and a payload property. The type describes what the action does.
The optional payload defines the type of data to be sent to the reducer.
The Action string and payload are specified as inputs to the CreateAction method.
Use [Source] Event to describe what the action does

createAction( '[Product Page] Update Product', props<{ product: Product }>() );

**Reducers**

Pure functions (for the same input, always return the same output) that take the previous state and an action as inputs
and execute an operation related to that action to return new state

**Selectors**

Selectors are pure functions used to select, derive and compose pieces of state.

Component Example

Inject the store in the constructor
`constructor(private store: Store<State>){}`

Type the observable
`products$: Observable<Product[]>;`

Dispatch an action to load the data from the api into state
`this.store.dispatch(ProductActions.loadProducts());`

Select the slice of state to read and populate a local variable
`this.products$ = this.store.select(getProducts);`

In template;
`*ngIf = "products$ | async as products"`

With use of async pipes, there is no need to subscribe and unsubscribe

## Actions CRUD example

    import { Product } from '../../product';
    import { createAction, props } from '@ngrx/store';
 
/* Read Operation */
 
    export const loadProducts = createAction(
      '[Product Page] Load'
    );
    
    export const loadProductsSuccess = createAction(
      '[Product API] Load Success',
      props<{ products: Product[] }>()
    );
    
    export const loadProductsFailure = createAction(
      '[Product API] Load Fail',
      props<{ error: string }>()
    );
 
/* Update Operation */
 
    export const updateProduct = createAction(
      '[Product Page] Update Product',
      props<{ product: Product }>()
    );
    
    export const updateProductSuccess = createAction(
      '[Product API] Update Product Success',
      props<{ product: Product }>()
    );
    
    export const updateProductFailure = createAction(
      '[Product API] Update Product Fail',
      props<{ error: string }>()
    );
 
/* Create Operation */
 
    export const createProduct = createAction(
      '[Product Page] Create Product',
      props<{ product: Product }>()
    );
    
    export const createProductSuccess = createAction(
      '[Product API] Create Product Success',
      props<{ product: Product }>()
    );
    
    export const createProductFailure = createAction(
      '[Product API] Create Product Fail',
      props<{ error: string }>()
    );
 
/* Delete Operation */
 
    export const deleteProduct = createAction(
      '[Product Page] Delete Product',
      props<{ productId: number }>()
    );
    
    export const deleteProductSuccess = createAction(
      '[Product API] Delete Product Success',
      props<{ productId: number }>()
    );
    
    export const deleteProductFailure = createAction(
      '[Product API] Delete Product Fail',
      props<{ error: string }>()
    );

**Effects**
For async operations such as http requests, use effects combined with the actions
that have side effects such as api calls (don't always return the same output for the same input).
Effects call on service methods where the actual http requests are made.

`ng add @ngrx/effects`
In the feature module, add:

    import {EffectsModule} from '@ngrx/effects';
    import {ProductEffects} from './state/products.effects';
    ...
    EffectsModule.forFeature([ProductEffects])

Effects select an action, do some work that creates a side effect (a service api call), and dispatch a new action depending on the result of the side effect (success, error). This new action will have no side effects, and will be handled by a reducer.

Ex: effects file

    import { Injectable } from '@angular/core';
    
    import { mergeMap, map, catchError } from 'rxjs/operators';
    import { of } from 'rxjs';
    import { ProductService } from '../product.service';
 
    import { Actions, createEffect, ofType } from '@ngrx/effects';
    import * as ProductActions from './product.actions';
 
    @Injectable()
    export class ProductEffects {
    
      constructor(private actions$: Actions, private productService: ProductService) { }   // inject actions$ and services
    
      loadProducts$ = createEffect(() => {
        return this.actions$
          .pipe(
            ofType(ProductActions.loadProducts),   // select the action for this effect
            mergeMap(() => this.productService.getProducts()  // merges the observables output from productService into a single stream
              .pipe(
                map(products => ProductActions.loadProductsSuccess({ products })),    // call the success action on the result
                catchError(error => of(ProductActions.loadProductsFailure({ error })))
              )
            )
          );
      });
    }

Reducer

    import { createReducer, on } from "@ngrx/store";
    import { ProductApiActions, ProductPageActions } from "./actions";
    
    import { ProductState } from "./index";
    import { initialState } from "./index";
    
    export const productReducer = createReducer<ProductState>(
      initialState,
    
      on(ProductApiActions.loadProductsSuccess, (state, action): ProductState => {
        return {
          ...state,
          products: action.products,
          error: ''
        };
      }),
      on(ProductApiActions.loadProductsFailure, (state, action): ProductState => {
        return {
          ...state,
          products: [],
          error: action.error
        };
      }),
    
      on(ProductApiActions.updateProductSuccess, (state, action): ProductState => {
        const updatedProducts = state.products.map(
          item => action.product.id === item.id ? action.product : item);
        return {
          ...state,
          products: updatedProducts,
          currentProductId: action.product.id,
          error: ''
        };
      }),
      on(ProductApiActions.updateProductFailure, (state, action): ProductState => {
        return {
          ...state,
          error: action.error
        };
      }),
    
      // After a create, the currentProduct is the new product.
      on(ProductApiActions.createProductSuccess, (state, action): ProductState => {
        return {
          ...state,
          products: [...state.products, action.product],
          currentProductId: 