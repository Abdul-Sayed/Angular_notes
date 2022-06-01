# Angular Routing

**Configuration**

In app-routing.module.ts

    import {NgModule} from '@angular/core';
    import { Routes, RouterModule } from '@angular/router';
    // import all routed to components

    const appRoutes: Routes = [
      {path: '', component: HomeComponent, pathMatch = 'full'},  // empty path needs pathMatch
      {path: 'users', component: UsersComponent},
      {path: 'servers', component: ServersComponent,     // route with child (nested) routes
      children: [
        { path: '', component: ServersComponent, pathMatch: 'full' },
        { path: 'new', component: NewServerComponent },
        {path: ':id', component: ServerComponent}   // dynamic routes are placed at bottom. Use : for dynamic
        {path: ':id/edit', component: EditServerComponent}
      ]}
      {path: '**', redirectTo: ''}  // wildcard route must be at the very bottom
    ]

    @NgModule({
      imports: [RouterModule.forRoot(appRoutes)],
      exports: [RouterModule]
    })

    export class AppRoutingModule {}

In app.module.ts,

    import {AppRoutingModule} from './app-routing.module';
    ...
    @NgModule({
      imports: [
        ...
        AppRoutingModule
      ],
    })

In app.component.html

    <router-outlet></router-outlet>

**Child (nested) routing**

Specify the child routes in the children array

    const appRoutes: Routes = [
      {path: '', component: HomeComponent},
      {path: 'servers', component: ServersComponent,
      children: [
        {path: ':id', component: ServerComponent}
        {path: ':id/edit', component: EditServerComponent}
      ]}
    ]

Then in the parent template; ServersComponent.html, add `<router-outlet></router-outlet>`
To render the child components there.

**Navigating**

To route to a component; use this directive on a button, <a>, etc

    routerLink="/"
    routerLink="/users"   // to go to root /users
    routerLink="users"    // to go to /users on top of the base path your already on
    [routerLink]="['/servers', 'daniel']"   // for a  hard coded nested  route, use a comma then the sub route
    [routerLink]="[index]"   // for a variable route, where index is a variable

To make the navigation from the typescript file (upon user event),

    import { Router, ActivatedRoute } from '@angular/router';
    ...
    constructor(private router: Router, private route: ActivatedRoute) {}
    ...

    onNavigate() {
      this.router.navigate(['/servers']);
      // For specifying relative paths, use:
      this.router.navigate(['server'], {relativeTo: this.route});
    }

To go up one level in the route; this.router.navigate(['../'], {relativeTo: this.route})

**Active Route Styling**
To make the link/button of the active route styled, add routerLinkActive="someClass" to the element
For the homepage link/button, add [routerLinkActiveOptions]="{exact: true}"

**Params** ; Dynamic route parameters

In app.module

    const appRoutes: Routes = [
      {path: '', component: HomeComponent},
      {path: 'users/:id/:name', component: UserComponent},  // dynamic route
    ]

In UserComponent, to retrieve the id, use:

    import { Router, ActivatedRoute, Params } from '@angular/router';
    ...
    constructor(private router: Router, private route: ActivatedRoute) {}
    ...

    this.route.snapshot.params['id']
    this.route.snapshot.params['name']

The above runs only once on component initialization.
To recieve route params that may later change, additionally add:

    this.route.params.subscribe(
      (params: Params) => {
        this.user.id = +params['id'];
        this.user.name = params['name'];
      }
    )

Angular manually unsubscribes when the component is closed

**Query Params**

Used to pass data in the route

    const appRoutes: Routes = [
      {path: '', component: HomeComponent},
      {path: 'users/:id/:name', component: UserComponent},
      {path: 'servers/:id/edit', component: EditServerComponent}
    ]

From another component, you can navigate to EditServerComponent, add
queryParams, route fragments via:

    <a
    [routerLink]="['/servers', 5, 'edit']"
    [queryParamsHandling]= 'preserve'
    [queryParams]="{allowEdit: server.id === 1 ? true : false}"
    [fragment]="'loading'">
    </a>

or

    import {ActivatedRoute, Router} from '@angular/core';

    constructor(private route: ActivatedRoute, private router: Router) {}

    this.router.navigate(
      ['/servers', id, 'edit'],
      {queryParamsHandling: 'merge'}
      {queryParams: {allowEdit: server.id === 1 ? true : false},
      fragment: 'loading'})

To use relative path navigation:

    this.router.navigate(
      ['edit'],
      {relativeTo: this.route, queryParamsHandling: 'merge'}
      {queryParams: {allowEdit: server.id === 1 ? true : false},
      fragment: 'loading'})

To get access to the passed queryParam and fragment in the EditServerComponent,

    import {ActivatedRoute, Params} from '@angular/router';
    ...
    constructor(private route: ActivatedRoute)

    this.route.snapshot.queryParams // To get static queryParams and fragment
    this.route.snapshot.fragment

    this.route.queryParams.subscribe(  // Use for dynamic params
      (params: Params) => {
        this.allowEdit = params['allowEdit'];
      }
    )

    this.route.fragment.subscribe(
      (frag) => {

      }
    )

**Passing Static Data**

Pass a data object to the component's route definition

    const appRoutes: Routes = [
      {path: '', component: HomeComponent},
      {path: 'users', component: UsersComponent},
      {path: 'not-found', component: ErrorPageComponent, data: {message: 'Page not found'} },
      {path: '**', redirectTo: 'not-found'}
    ]

Inside the component, extract the data

    import {ActivatedRoute, Data} from '@angular/router';
    export class ErrorPageComponent implements OnInit {
      constructor(private route: ActivatedRoute) { }

      ngOnInit() {
        this.errorMsssage = this.route.snapshot.data['message'];
        this.route.data.subscribe( (data: Data) => {
          this.errorMsssage = data['message'];
        })
      }
    }
