# Angular Route Guards

Code in service file thats evaluated just prior to entering or leaving a route.

## canActivate: used to allow or disallow access to a route.

CanActivateChild: used to allow or disallow access to child routes.

`ng g guard guards/auth`  
Import AuthGuard in app.module and add to providers []

In auth-guard.service.ts

    import { Injectable } from '@angular/core';
    import { CanActivate, canActivateChild, ActivatedRouteSnapshot, RouterStateSnapshot, Router, UrlTree } from '@angular/router';
    import { Observable } from 'rxjs';
    import { AuthService } from './auth/auth.service';

    @Injectable({
      providedIn: 'root'
    })
    export class AuthGuard implements CanActivate {
      constructor(private authService: AuthService, private router: Router){}

      canActivate(
        route: ActivatedRouteSnapshot,
        state: RouterStateSnapshot): Observable<boolean | UrlTree> | Promise<boolean | UrlTree> | boolean | UrlTree {

          return this.authService.isAuthenticated()
            .then( (authenticated: boolean) => {
              if (authenticated) {
                return true
              } else {
                return this.router.parseUrl("/");  // return to HomeComponent
                // can also use createUrlTree();  this.router.createUrlTree(['/']);
              }
            })
       }

      canActivateChild (
        route: ActivatedRouteSnapshot,
        state: RouterStateSnapshot): Observable<boolean | UrlTree> | Promise<boolean | UrlTree> | boolean | UrlTree {
          return this.canActivate(route, state);
      }
    }

Example AuthService

    export class AuthService {
      private loggedIn = false;

      login() {
        this.loggedIn = true;
      }
      logout() {
        this.loggedIn = false;
      }

      isAuthenticated() {
        const promise = new Promise(
          (resolve, reject) => {
            // http request with auth backend which updates this.loggedIn
            resolve(this.loggedIn)
          }
        )
        return promise;
      }
    }

Apply the guard to each protected route in app-routing.module

    ...
    import { AuthGuard } from  './auth.guard';

    const appRoutes: Routes = [
      {path: '', component: HomeComponent},
      {path: 'users', component: UsersComponent},
      {path: 'servers',
      component: ServersComponent,
      canActivate: [AuthGuard],    // to protect whole route
      canActivateChild: [AuthGuard],   // to protect only chile routes
      children: [
        {path: ':id', component: ServerComponent}
        {path: ':id/edit', component: EditServerComponent}
      ]}
      {path: '**', redirectTo: ''}  // wildcard route must be at bottom
    ]

## CanDeactivate: used to allow or deny exit from route.

'ng g g guards/can-abort'

    import { Injectable } from '@angular/core';
    import { CanDeactivate } from '@angular/router';
    import { Observable } from 'rxjs/Observable';

    export interface CanComponentDeactivate {
      canDeactivate: () => Observable<boolean | UrlTree> | Promise<boolean | UrlTree> | boolean | UrlTree;
    }

    @Injectable()
    export class CanDeactivateGuard implements CanDeactivate<CanComponentDeactivate> {

      canDeactivate(component: CanComponentDeactivate,
                    route: ActivatedRouteSnapshot,
                    state: RouterStateSnapshot,
                    nextState?: RouterStateSnapshot): Observable<boolean | UrlTree>
                    | Promise<boolean | UrlTree>
                    | boolean | UrlTree {
                      return component.canDeactivate();
                    }
    }

In app.module, import the guard and add in providers []

In app-routing.module, import the CanDeactivateGuard

    const appRoutes: Routes = [
      {path: '', component: HomeComponent},
      {path: 'users', component: UsersComponent},
      {path: 'servers', component: ServersComponent,
      children: [
        {path: ':id', component: ServerComponent}
        {path: ':id/edit', component: EditServerComponent, canDeactivate: [CanDeactivateGuard]}
      ]}
      {path: '**', redirectTo: ''}  // wildcard route must be at bottom
    ]

The component whose exit navigation is being protected must implement a canDeactivate() method

    import {CanComponentDeactivate} from './can-deactivate-guard.service';
    import { Observable } from 'rxjs/Observable';

    export class EditServerComponent implements CanComponentDeactivate {

      ...

      canDeactivate(): Observable<boolean | UrlTree> | Promise<boolean | UrlTree> | boolean | UrlTree {
        // Here add the logic of whether the navigation is allowed to exit
        // Usually this has to do with a form being incomplete
      }
    }

## Resolve: used for doing operations (preloading data) just before route is rendered. Pauses navigation until data is feched. Not used for deciding if a route will load.

    import { Injectable } from '@angular/core';
    import { Resolve, ActivatedRouteSnapshot, RouterStateSnapshot } from '@angular/router';
    import { APIService } from './api.service';
    import { IServer } from '../models/server.model.ts';
    import { Observable } from 'rxjs/Observable';

    @Injectable()
    export class ServerResolver implements Resolve<IServer> {  // add the structure of the data your fetching
      constructor(private apiService: APIService) {}

      resolve(route: ActivatedRouteSnapshot, state: RouterStateSnapshot):  Observable<IServer> | Promise<IServer> | IServer {
        return this.apiService.getItems(+route.params['id']);
      }
    }

In app.module, import ServerResolver and add it to providers []

In app-routing.module, import ServerResolver and add it to the routing config for the route being navigated to

    const appRoutes: Routes = [
      {path: '', component: HomeComponent},
      {path: 'servers', component: ServersComponent,
      children: [
        {path: ':id', component: ServerComponent, resolve: {server: ServerResolver}}
      ]}
      {path: '**', redirectTo: ''}  // wildcard route must be at bottom
    ]

In the component being routed to, you can obtain the pre-feched data with:

    import {ActivatedRoute, Data} from '@angular/router';
    ...
    constructor(private route: ActivatedRoute) { }
      ngOnInit() {
        this.errorMsssage = this.route.snapshot.data['server'];
        this.route.data.subscribe( (data: Data) => {
          this.server = data['server'];
        })
      }
