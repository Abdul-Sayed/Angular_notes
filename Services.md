# Angular Services

Services are central locations of code used throughout the app. The same instance (singleton) of the service will be available for all components in the application if using `providedIn: root` in the @Injectable decorator. Because of this feature, data in it is treated as live across components for communication. The service is imported and injected as a dependency in the constructor of the client components using it. This allows components to focus on component specific logic and the shared logic, especially side effect producing logic, can be externalized to a service, allowing modularity and avoiding redundant code.
An example use case for them is http calls.

Create a folder to contain all services. A service needs to be imported and added to the providers array in the module its being used OR be declared as providedIn: root in the service file itself;

    @Injectable({
      providedIn: 'root'
    })

    // In the module

        import { CrudService } from './crud.service';

        @NgModule({
          declarations: [...],
          imports: [...],
          providers: [CrudService],
          bootstrap: [...]
        })

You can skip adding the service to the module if the service decorator has providedIn: root
Eitherway, there will be the same singleton instance of the service available everywhere the service is used.

If you provide a service in the providers array of an eagerly loaded module, a singleton instance of that service also becomes available application wide. But doing so in a lazy loaded module will create a seperate instance of the service there. Do this only if you deliberatley want the service to be scoped only to the lazy loaded module.

Don't provide services in shared modules, as that can create unintended multiple instances of the services.

The service file itself should import {Injectable} from '@angular/core', and the service class should have the @Injectable() decorator. The @Injectable decorator also allows the service need to use another service.

    // Service Class Example
      import { Injectable } from '@angular/core';

      @Injectable({
        providedIn: 'root'
      })

      export class CrudService {

        constructor() { }

      }

A component consuming a service must import it, and inject it in the constructor;  
'private someService: SomeService'

**To use**

Example: passing live data between components;

In service  
 `updatedStatus = new Subject<string>();`

In comp1

    onUpdate(status: string) {
      this.myService.updatedStatus.next(status);
    }

In comp2

    this.myService.updatedStatus.subscribe(
      (status: string) => {
        alert(status)
      }
    )
