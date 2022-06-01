# Angular Modules

## Feature Modules

In app.module.ts, the declaration [ ] lists all the components used in the module. The providers [ ] lists all the services used in the module. And the imports [ ] lists all the external modules used by that module. The bootstrap [ ] defines the root component in index.html

Instead of providing everything the application uses in the app module, its better to split the app across different feature modules so they become easier to maintain.

Components declared in one module cannot be used in another, unless the module is imported. The module being imported must have either exported that component or have a routing module with that component configured in it.

In the feature module, import all relevant components, and add to declarations [ ]. Make sure the feature module has CommonModule imported, and import any other Angular modules needed. Then in app module, import the feature module and add it to imports [].

For each module, also create its routing module, and import it into the feature's own module file. Use `RouterModule.forChild(routes)`

Place the wildcard route in app-routing-module, and in app.module.ts, import AppRoutingModule at the end

        import { NgModule } from '@angular/core';
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
        imports: [RecipesRoutingModule, RouterModule, SharedModule],
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
        { path: '', redirectTo: '/recipes', pathMatch: 'full' },
        {
            path: 'recipes',
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
        import { BrowserModule } from '@angular/platform-browser';
        import { HttpClientModule } from '@angular/common/http';
        import { HTTP_INTERCEPTORS } from '@angular/common/http';
        import { RecipesModule } from './recipes/recipes.module';
        import { ShoppingListModule } from './shopping-list/shopping-list.module';
        import { SharedModule } from './shared/shared.module';
        import { AppRoutingModule } from './app-routing.module';
        import { AppComponent } from './app.component';
        import { HeaderComponent } from './header/header.component';
        import { AuthComponent } from './auth/auth.component';
        import { AuthInterceptor } from './auth/auth.interceptor';

        @NgModule({
        declarations: [AppComponent, HeaderComponent, AuthComponent],
        imports: [
            BrowserModule,
            HttpClientModule,
            RecipesModule,
            ShoppingListModule,
            SharedModule,
            AppRoutingModule,
        ],
        providers: [
            { provide: HTTP_INTERCEPTORS, useClass: AuthInterceptor, multi: true },
        ],
        bootstrap: [AppComponent],
        })
        export class AppModule {}

---

        import { NgModule } from '@angular/core';
        import { RouterModule, Routes } from '@angular/router';
        import { AuthComponent } from './auth/auth.component';

        const routes: Routes = [
        { path: '', redirectTo: '/recipes', pathMatch: 'full' },
        { path: 'auth', component: AuthComponent },
        { path: '**', redirectTo: '/', pathMatch: 'full' },
        ];

        @NgModule({
        imports: [RouterModule.forRoot(routes)],
        exports: [RouterModule],
        })
        export class AppRoutingModule {}

## Shared Modules

No component, directive, or pipe can be declared in more than one module. Also, the same import shouldnt be repeated across multiple modules. Instead, create a shared module and import everything needed by the app, and import the shared module into all the other modules

        import { NgModule } from '@angular/core';
        import { CommonModule } from '@angular/common';
        import { FormsModule, ReactiveFormsModule } from '@angular/forms';
        import { MatSelectModule } from '@angular/material/select';
        import { MatFormFieldModule } from '@angular/material/form-field';
        import { MatInputModule } from '@angular/material/input';
        import { AlertComponent } from './alert/alert.component';
        import { LoadingSpinnerComponent } from './loading-spinner/loading-spinner.component';
        import { DropdownDirective } from './dropdown.directive';
        import {
        BrowserAnimationsModule,
        NoopAnimationsModule,
        } from '@angular/platform-browser/animations';

        @NgModule({
        declarations: [AlertComponent, LoadingSpinnerComponent, DropdownDirective],
        imports: [
            CommonModule,
            FormsModule,
            ReactiveFormsModule,
            MatFormFieldModule,
            MatSelectModule,
            MatInputModule,
            BrowserAnimationsModule,
            NoopAnimationsModule,
        ],
        exports: [
            AlertComponent,
            LoadingSpinnerComponent,
            DropdownDirective,
            CommonModule,
            FormsModule,
            ReactiveFormsModule,
            MatFormFieldModule,
            MatSelectModule,
            MatInputModule,
            BrowserAnimationsModule,
            NoopAnimationsModule,
        ],
        })
        export class SharedModule {}
