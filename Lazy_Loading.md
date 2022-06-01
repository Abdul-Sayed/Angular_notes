# Angular Lazy Loading

LAzy Loading of unvisited parts of the application can be setup once feature modules are made with their own routing configuration

Use loadChildren router property to only load a module when the specified route is visited. Also remove the lazy loaded module from app module imports [ ]. In the lazy loaded router config, set path to '' for its main component.

In app routing module, configure a preloading strategy so that lazy loaded modules load in the background while user is using the app instead of when they hit a route.

        import { NgModule } from '@angular/core';
        import { BrowserModule } from '@angular/platform-browser';
        import { HttpClientModule } from '@angular/common/http';
        import { HTTP_INTERCEPTORS } from '@angular/common/http';
        import { SharedModule } from './shared/shared.module';
        import { AppRoutingModule } from './app-routing.module';
        import { AppComponent } from './app.component';
        import { HeaderComponent } from './header/header.component';
        import { AuthInterceptor } from './auth/auth.interceptor';
        import {
        BrowserAnimationsModule,
        NoopAnimationsModule,
        } from '@angular/platform-browser/animations';

        @NgModule({
        declarations: [AppComponent, HeaderComponent],
        imports: [
            BrowserModule,
            HttpClientModule,
            SharedModule,
            AppRoutingModule,
            BrowserAnimationsModule,
            NoopAnimationsModule,
        ],
        providers: [
            { provide: HTTP_INTERCEPTORS, useClass: AuthInterceptor, multi: true },
        ],
        bootstrap: [AppComponent],
        })
        export class AppModule {}

---

        import { NgModule } from '@angular/core';
        import { PreloadAllModules, RouterModule, Routes } from '@angular/router';

        const routes: Routes = [
        { path: '', redirectTo: '/recipes', pathMatch: 'full' },
        {
            path: 'recipes',
            loadChildren: () =>
            import('./recipes/recipes.module').then((m) => m.RecipesModule),
        },
        {
            path: 'shopping-list',
            loadChildren: () =>
            import('./shopping-list/shopping-list.module').then(
                (m) => m.ShoppingListModule
            ),
        },
        {
            path: 'auth',
            loadChildren: () => import('./auth/auth.module').then((m) => m.AuthModule),
        },
        { path: '**', redirectTo: '/', pathMatch: 'full' },
        ];

        @NgModule({
        imports: [
            RouterModule.forRoot(routes, { preloadingStrategy: PreloadAllModules }),
        ],
        exports: [RouterModule],
        })
        export class AppRoutingModule {}

---

        import { NgModule } from '@angular/core';
        import { CommonModule } from '@angular/common';
        import { SharedModule } from '../shared/shared.module';
        import { RouterModule, Routes } from '@angular/router';
        import { RecipesRoutingModule } from './recipes-routing.module';
        import { RecipeDetailComponent } from './recipe-detail/recipe-detail.component';
        import { RecipeEditComponent } from './recipe-edit/recipe-edit.component';
        import { RecipeItemComponent } from './recipe-list/recipe-item/recipe-item.component';
        import { RecipeListComponent } from './recipe-list/recipe-list.component';
        import { RecipeStartComponent } from './recipe-start/recipe-start.component';
        import { RecipesComponent } from './recipes.component';

        @NgModule({
        declarations: [
            RecipesComponent,
            RecipeStartComponent,
            RecipeListComponent,
            RecipeItemComponent,
            RecipeDetailComponent,
            RecipeEditComponent,
        ],
        imports: [CommonModule, RecipesRoutingModule, RouterModule, SharedModule],
        exports: [],
        })
        export class RecipesModule {}

---

        import { NgModule } from '@angular/core';
        import { RouterModule, Routes } from '@angular/router';
        import { RecipesComponent } from './recipes.component';
        import { RecipeDetailComponent } from './recipe-detail/recipe-detail.component';
        import { RecipeStartComponent } from './recipe-start/recipe-start.component';
        import { RecipeEditComponent } from './recipe-edit/recipe-edit.component';
        import { RecipesResolverGuard } from './recipes-resolver.guard';
        import { AuthGuard } from '../auth/auth.guard';

        const routes: Routes = [
        {
            path: '',
            component: RecipesComponent,
            canActivate: [AuthGuard],
            children: [
            { path: '', component: RecipeStartComponent, pathMatch: 'full' },
            { path: 'new', component: RecipeEditComponent },
            {
                path: ':id',
                component: RecipeDetailComponent,
                resolve: { recipes: RecipesResolverGuard },
            },
            {
                path: ':id/edit',
                component: RecipeEditComponent,
                resolve: { recipes: RecipesResolverGuard },
            },
            ],
        },
        ];

        @NgModule({
        imports: [RouterModule.forChild(routes)],
        exports: [RouterModule],
        })
        export class RecipesRoutingModule {}

---

        import { NgModule } from '@angular/core';
        import { CommonModule } from '@angular/common';
        import { RouterModule } from '@angular/router';
        import { AuthGuard } from '../auth/auth.guard';
        import { ShoppingListComponent } from './shopping-list.component';
        import { ShoppingEditComponent } from './shopping-edit/shopping-edit.component';
        import { SharedModule } from '../shared/shared.module';

        @NgModule({
        declarations: [ShoppingListComponent, ShoppingEditComponent],
        imports: [
            CommonModule,
            SharedModule,
            RouterModule.forChild([
            {
                path: '',
                component: ShoppingListComponent,
                canActivate: [AuthGuard],
            },
            ]),
        ],
        exports: [],
        })
        export class ShoppingListModule {}
