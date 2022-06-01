# Angular NGRX (old version)

NGRX (old version)
State example
To configure a data model for use in state, first create an interface for it;

    export interface IUser {
      id: number;
      name: string;
      age: string;
    }

For user state in store/state/user.state.ts:

    import { IUser } from '../../models/user.interface';

    export interface IUserState {
      users: IUser[];
      selecterUser: IUser;
    }
    export const initialUserState: IUserState = {
      users: null,
      selecterUser: null;
    }

For the app state (store/state/app.state.ts) import, combine, and return the inital state of all modules:

    import { IUserState, initialUserState } from './user.state';
    // also include properties for the other slices of state

    export interface IAppState {
      users: IUserState;
    }
    export const initialAppState: IAppState = {
      users: InitialUserState
    }

    export function getInitialState(): IAppState {
      return initialAppState;
    }
    Actions example
    (store/actions/user.actions.ts):

---

    import { Action } from '@ngrx/store';
    import { IUser } from '../../models/user.interface';

    export enum EUserActions {
      GetUser = '[User] Get User',
      GetUserSuccess = '[User] Get User Success',
      GetUserFailure = '[User] Get User Failure'
    }

    export class GetUser implements Action {
      public readonly type = EUserActions.GetUser;
      constructor(public payload: number) {}
    }

    export class GetUserSuccess implements Action {
      public readonly type = EUserActions.GetUserSuccess;
      constructor(public payload: IUser) {}
    }

    export class GetUserFailure implements Action {
      public readonly type = EUserActions.GetUserFailure;
      constructor(public payload: string) {}
    }

    export type UserActions = GetUser | GetUserSuccess | GetUserFailure;
    Reducers example
    (store/reducers/user.reducers.ts):

---

    import { UserActions, EUserActions } from '../actions/user.actions';
    import { initialUserState, IUserState } from '../state/user.state';

    export const userReducers = (
      state = initialUserState,
      action: UserActions
    ): IUserState => {
      switch (action.type) {
        case EUserActions.GetUserSuccess: {
          return {
            ...state,
            selectedUser: action.payload
          };
        }
        case EUserActions.GetUserFailure: {
          return {
            ...state,
            error: action.payload
          };
        }

        default:
          return state;
      }
    };

app reducer
(store/reducers/app.reducers.ts):
Add the reducers of all modules to the app action-reducer-map

    import { ActionReducerMap } from '@ngrx/store';

    import { IAppState } from '../state/app.state';
    import { userReducers } from './user.reducers';

    export const appReducers: ActionReducerMap<IAppState, any> = {
      users: userReducers,
    };
    Effects example
    (store/effects/user.effects.ts):

---

    import { Injectable } from '@angular/core';
    import { Effect, ofType, Actions } from '@ngrx/effects';
    import { Store, select } from '@ngrx/store';
    import { of } from 'rxjs';
    import { switchMap, map, withLatestFrom } from 'rxjs/operators';

    import { IAppState } from '../state/app.state';
    import {
      EUserActions,
      GetUserSuccess,
      GetUser,
    } from '../actions/user.actions';
    import { UserService } from '../../services/user.service';
    import { IUserHttp } from '../../models/http-models/user-http.interface';
    import { selectUserList } from '../selectors/user.selector';

    @Injectable()
    export class UserEffects {
      @Effect()
      getUser$ = this._actions$.pipe(
        ofType<GetUser>(EUserActions.GetUser),
        map(action => action.payload),
        withLatestFrom(this._store.pipe(select(selectUserList))),
        switchMap(([id, users]) => {
          const selectedUser = users.filter(user => user.id === +id)[0];
          return of(new GetUserSuccess(selectedUser));
        })
      );

      @Effect()
      getUsers$ = this._actions$.pipe(
        ofType<GetUsers>(EUserActions.GetUsers),
        switchMap(() => this._userService.getUsers()),
        switchMap((userHttp: IUserHttp) => of(new GetUsersSuccess(userHttp.users)))
      );

      constructor(
        private _userService: UserService,
        private _actions$: Actions,
        private _store: Store<IAppState>
      ) {}
    }

Selectors example
Return slices of state for use by other files;
(store/selectors/user.selectors.ts):

---

    import { createSelector } from '@ngrx/store';

    import { IAppState } from '../state/app.state';
    import { IUserState } from '../state/user.state';

    const selectUsers = (state: IAppState) => state.users;

    export const selectUserList = createSelector(
      selectUsers,
      (state: IUserState) => state.users
    );

    export const selectSelectedUser = createSelector(
      selectUsers,
      (state: IUserState) => state.selectedUser
    );

Component example

    import { GetUsers } from './../../store/actions/user.actions';
    import { Component, OnInit } from '@angular/core';
    import { Store, select } from '@ngrx/store';

    import { IAppState } from '../../store/state/app.state';
    import { selectUserList } from '../../store/selectors/user.selector';
    import { Router } from '@angular/router';

    @Component({
      templateUrl: './users.component.html',
      styleUrls: ['./users.component.css']
    })
    export class UsersComponent implements OnInit {
      users$ = this._store.pipe(select(selectUserList));

      constructor(private _store: Store<IAppState>, private _router: Router) {}

      ngOnInit() {
        this._store.dispatch(new GetUsers());
      }

      navigateToUser(id: number) {
        this._router.navigate(['user', id]);
      }
    }
