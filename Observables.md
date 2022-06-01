# Angular Observer Pattern / RxJs reactive Programming

Reactive programming is a design pattern concerned with asynchronous code and change propagation. It is commonly implemented with RxJs, a library which provides utilities such as observable data streams that push emissions to observers which consume, react to, and transform those emissions using various pipeable operators. RxJs is tightly integrated with Angular and allows handling async operations such as user events or http calls declaratively.

The observer/observable pattern handles asynchronous tasks such as pending user events or an http request. Observables sends the client application any updates to the data requested from the server, without the client having to make another http call.
It consists of an Observable which is the data source, and an Observer which listens to and handles the data coming from the Obserable. The observer handles the data when the data arrives successfully, handles the error in the case the data doesn't arrive, and handles what to do in case the data stream ceases.

Observables are objects that change over time, and that update listeners on those changes.

To prevent memory leaks, observable subscriptions must be unsubscribed when the data is no longer needed;

        import { Observable, Subscription } from 'rxjs';
        ...
        myObservable$: Observable;
        dataSubscription: Subscription;
        ...
        ngOnInit() {
                this.myObservable$ = Observable.create(observer => {
                let count = 0;
                setInterval( () => {
                    observer.next(count);  // send an emission
                    if (count === 7) {
                        observer.complete();
                    }
                    if (count > 8) {
                        observer.error(new Error('Count has surpassed 8 !!!))
                    }
                    count = Math.ceil(Math.random() * 10);
                }, 1000 )
                });

                this.dataSubscription = this.myObservable.subscribe(
                    data => console.log(data),
                    error => console.log(error),
                    () => console.log('Completed!')
                );
        }

        ...
        ngOnDestroy() { this.dataSubscription.unsubscribe }

## toPromise

Used to make an asynchronous call, then stop listening once data arrives.

In service file:

        getUserByID(id) {
            return this.http.get( `${ this.URL }/id` )
        }

In component:

        this.myService.getUserByID(this.id).toPromise()
        .then( user => {   // the promise resolves as soon as data arrives, and stops listening any further
            this.user$ = user;
        }).catch(error => {
            console.log(error);
        }).finally( () => {   // In the success scenario, after .then runs, finally will run
            console.log('User retrieved from server');
        })

## Subject

A subject is an Observable where next() can be called from the outside, similar to calling .emit() on an Event Emitter. Aslo like EventEmitter(), Subjects can cast emissions to more than one subscriber, unlike an Observable.

        import {EventEmitter} from 'rxjs';
        ...
        activatedEmitter = new EventEmitter<boolean>();  // in user.service.ts
        this.userService.activatedEmitter.emit(true);  // in one component
        ...
        this.userService.activatedEmitter.subscribe(didActivate => ... )   // in another component

While normal observables are set up to passivley respond to asynchronous tasks like http requests or user events, Subjects are meant to actively respond to the triggers your supply.

        import {Subject} from 'rxjs';
        ...
        activatedEmitter = new Subject<boolean>();    // in user.service.ts
        ...
        this.userService.activatedEmitter.next(true);  // in one component
        ...
        this.userService.activatedEmitter.subscribe(didActivate => ...)    // in another component

Subject is more efficient than event emitter and rxjs operators can be used on the data at the point of subscription. EventEmitter should only be used with @Output() for child to parent prop lifting.
Just store the subscription in a variable and unsubscibe it in ngOnDestory

## Behavior Subject

Behaves just like a Subject (can call .next() to emit a value and .Subscribe() to be informed about new emissions), but gives subscribers the previously emitted value as well as the current value.
Behavior Subjects need to be initialized with a value; `activatedEmitter = new Subject<boolean>(true);`

## RXJS Operators

Array methods to operate on the data arriving from an observable, then pass the transformed observable data to the observer/subscriber.
These methods are chained wihin the .pipe method and are pure methods (for a given input, always produce the same output). They take in an observable stream, and output a modified observable stream.

         import { map } from 'rxjs/operators';

         this.myObservable.pipe( map( (n) => { return n + 1 } ), filter( (n) => { return n % 2 === 0 } )  )
        .subscribe(
            data => console.log(data),
            error => console.log(error),
            () => console.log('Completed!')
        )

**Of**  
This operator wraps and returns whatever you supply it into an observable;
`studentList = ['Mark', 'Dave', 'Bob'];`  
`students$: Observable<string[]> = of(studentList)`

**From**  
Creates an observable from a supplied array or collection only;
`studentList = ['Mark', 'Dave', 'Bob'];`  
`students$: Observable<string> = from(studentList)`

**FromEvent**  
Creates an observable from a user event

        <button #validate (click)="handleClick()"></button>
        ...
        @ViewChild('validate')
        validate: ElementRef

        handleClick() {
            fromEvent(validate?.nativeElement, 'click').subscribe(event => {
                ...
            })
        }

**Interval**  
Creates an observable that emits sequential numbers every specified interval of time  
`const sequence$ = interval(500);` // starting from n=0, every 500 ms it will emit n+1
`sequence$.subscribe(num => ... );

**debounceTime**  
Emits from the source observable only after a certain time has passed without another source emission.
Useful for live api calls as user types in a search box

        searchForm: FormGroup;
        ...
        this.searchForm = new Formgroup({
            name: new FormControl('', Validators.required);
        })
        ...
        this.searchForm.get('name').valueChanges.pipe(
            debounceTime(3000)
        ).subscribe(input => {
            ...
        })

**First**  
The first() operator obtains the first emission then unsubscribes itself

         this.dataStorageService.fetchRecipes().pipe(first()).subscribe();

**Take**  
The take(n) operator takes the first n emissions then unsubscribes

        this.dataStorageService.fetchRecipes().pipe(take(1)).subscribe();

**TakeWhile**  
Emits values from the source observable as long as each value satisfies a condition, but completes as soon as a value fails to meet the condition;

        let numbers$ = from([1, 2, 3, 4, 5]);

        numbers$.pipe(
            takeWhile(num => num < 4)
        ).subscribe(data => {
            // 1, 2, 3
        })

**TakeLast**  
Waits for the source to complete, then emits the last specified N values from the source

        const source = of('Ignore', 'Ignore', 'Hello', 'World!');
        // take the last 2 emitted values; Hello, World!
        source.pipe(takeLast(2)).subscribe(val => console.log(val)

**ElementAt**  
elementAt returns an Observable that emits the item at the specified index in the source Observable

        import { fromEvent } from 'rxjs';
        import { elementAt } from 'rxjs/operators';

        const clicks = fromEvent(document, 'click');
        const result = clicks.pipe(elementAt(2));
        result.subscribe(x => console.log(x));

        // Results in:
        // click 1 = nothing
        // click 2 = nothing
        // click 3 = MouseEvent object logged to console

**Map**  
Apply a transformation to each value from the source

        import { from } from 'rxjs';
        import { map } from 'rxjs/operators';

        //emit (1,2,3,4,5)
        const source = from([1, 2, 3, 4, 5]);
        //add 10 to each value
        const example = source.pipe(map(val => val + 10));
        //output: 11,12,13,14,15

**Filter**  
Emit values that pass the provided condition

        import { from } from 'rxjs';
        import { filter } from 'rxjs/operators';

        //emit (1,2,3,4,5)
        const source = from([1, 2, 3, 4, 5]);
        //filter out non-even numbers
        const example = source.pipe(filter(num => num % 2 === 0));
        //output: "Even number: 2", "Even number: 4"
        const subscribe = example.subscribe(val => console.log(`Even number: ${val}`));

**Reduce**

Recap: JS reduce method

        const items = [
            {name: 'Desktop', price: 300},
            {name: 'Monitor', price: 100}
        ]

        let totalPrice = 0;
        items.forEach(item => {
            totalPrice += item.price
        })

reduce achieves the same thing. Its an iterator that works on a collection and takes an
accumulator and the current element in the iteration as arguments in a callback function. The second argument to reduce is the starting count;

        const totalPrice = items.reduce((total, item) => {
            return total += item.price;
        } , 0);

In RxJs, it reduces the values from a source observable to a single value that's emitted when the observable completes

        import { of } from 'rxjs';
        import { reduce } from 'rxjs/operators';

        const source = of(1, 2, 3, 4);
        const example = source.pipe(reduce((acc, val) => acc + val));
        //output: Sum: 10'
        const subscribe = example.subscribe(val => console.log('Sum:', val));

**ExhaustMap** - to avoid a que of requests
Ignores any subsequesnt requests until current request completes. Used to avoid a que of requests
Useful when you need to return an observable within an observable.
It wraps the second observable within the first one and waits for the 1st one to complete before sending the stream to the 2nd observable

**switchMap** - detach from previus emission and subscribes to new emission as new emission arrives from source stream
Cancels current subscription if a new value is emitted and subscribes to the new observable. Use for get requests or cancelable requests or it will lead to race conditions.

**concatMap** - run requests in order
Runs requests/subscriptions in order. Waits for last request to complete before subscribing to next request. Less performant. Use for post/put requests or any operation where order is important.

**mergeMap** - used when order of requests is not important
Runs requests in paralell and merges inner observables to return a single observable stream. More performant. Use for operations where order is not important.
