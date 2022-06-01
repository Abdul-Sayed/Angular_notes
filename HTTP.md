# Angular HTTP REST

The front end application makes Http requests to a server using the Rest or GraphQL protocol and the server sends back a response.

Http requests are made to a URL or API endpoint; www.domainName.com/posts
Requests are specified by Http verbs;
GET, POST, PUT, PATCH, DELETE
For Post, Put, and Patch, data has to be supplied in the body of the request.
The request also contains metadata stored in the header

## Example Http Calls

This pattern makes the http calls in a service file and returns the resulting Observable.
The componends subscribe the the service methods.

An alternative pattern is the for service to have a subject for each http request, make the http call, subscribe to it, and then emit the data for components to subscribe to.

In app.module, `import { HttpClientModule } from '@angular/common/http';`
And add to imports [ ]

Create an interface

        export interface IPost {
            title: string;
            content: string;
            id?: string;
        }

Then in a service file,

        import { Injectable } from '@angular/core';
        import {
        HttpClient,
        HttpHeaders,
        HttpParams,
        HttpEventType,
        } from '@angular/common/http';
        import { catchError, map, tap } from 'rxjs/operators';
        import { IPost } from './post.model';
        import { Observable, Subject, throwError } from 'rxjs';

        @Injectable({
        providedIn: 'root',
        })
        export class PostService {
        URL = 'https://fucks-85fea-default-rtdb.firebaseio.com/posts.json';
        // URL = 'https://jsonplaceholder.typicode.com/posts';

        constructor(private http: HttpClient) {}

        //GET request
        getPosts(): Observable<IPost[]> {
            // optional headers
            const userHeaders = new HttpHeaders({
            'content-type': 'application/json',
            authenticationToken: '123XYZ*!$abc',
            });
            // optional Query PArams which affects the request url
            const userParams = new HttpParams()
            .set('print', 'pretty')
            .set('pageSize', '1');

            // returns an Observable of type IUser. Specifying the type as a generic argument is required
            return this.http.get<IPost[]>(this.URL, {
                headers: userHeaders,
                params: userParams,
            })
            .pipe(
                map((responseData) => {
                const postsArr: IPost[] = [];
                for (const key in responseData) {
                    postsArr.push({ id: key, ...responseData[key] });
                }
                return postsArr;
                }),
                catchError((err) => {
                // Send report to analytics
                return throwError(() => err);
                })
            );
        }

        //POST request
        createPost(postData: IPost) {
            console.log(postData);
            const userHeaders = new HttpHeaders({
            'content-type': 'application/json',
            authenticationToken: '123XYZ*!$abc',
            });
            const userParams = new HttpParams().set('userRole', 'admin');

            // return this.http.post(this.URL, postData);
            // pass observe: response to obtain the entire http response, not just the body
            // return this.http.post(this.URL, postData, { observe: 'response' });
             return  this.http.post(this.URL, postData, {headers: userHeaders, params: userParams,});
        }

        //PUT request
        updatePost(postBody: IPost, id: string): Observable<IPost> {
            const putHeaders = new HttpHeaders({
            'content-type': 'application/json',
            authenticationToken: '123XYZ*!$abc',
            });

            return this.http.put<IPost>(`${this.URL}/id`, postBody, {
            headers: putHeaders,
            });
        }

        //Delete Request
        deleteAllPosts() {
            return this.http.delete(this.URL, { observe: 'events' }).pipe(
            // (Optional) for granular control of request status
            tap((event) => {
                switch (event.type) {
                case HttpEventType.Sent:
                    console.log('Request was sent successfully, awaiting response');
                    break;
                case HttpEventType.Response:
                    console.log('Full response recieved');
                    break;
                case HttpEventType.ResponseHeader:
                    console.log('Response status and headers recieved');
                    break;
                default:
                    console.log(event);
                }
            })
            );
        }

}

In the component using the service;

        import { Component, OnInit } from '@angular/core';
        import { HttpClient } from '@angular/common/http';
        import { IPost } from './post.model';
        import { PostService } from './post.service';

        @Component({
        selector: 'app-root',
        templateUrl: './app.component.html',
        styleUrls: ['./app.component.css'],
        })
        export class AppComponent implements OnInit {
        loadedPosts: IPost[] = [];
        isFetching = false;
        isServerDown = false;

        constructor(private http: HttpClient, private postService: PostService) {}

        ngOnInit() {
            this.onFetchPosts();
        }

        onFetchPosts() {
            // Send Http GET request
            this.isFetching = true;
            this.postService.getPosts().subscribe({
            next: (posts) => {
                this.loadedPosts = posts;
                console.log(this.loadedPosts);
            },
            error: (err) => {
                console.log(err);
                this.isFetching = false;
                this.isServerDown = true;
            },
            complete: () => {
                console.log('Completed fetching all posts');
                this.isFetching = false;
            },
            });
        }

        onCreatePost(postData: IPost) {
            // Send Http Post request
            this.postService.createPost(postData).subscribe({
            next: (responseData) => {
                console.log(responseData);
            },
            error: (err) => {
                console.log(`Data could not be posted: ${err}`);
            },
            complete: () => {
                console.log('Completed creating new post');
                this.onFetchPosts();
                // Do pessimistic rendering here
            },
            });
        }

        onUpdatePost(id: string) {
            const updatedPost = {
            title: 'New Post Title',
            content: 'New Post Body Blah blah',
            id: id,
            };

            // this.postService.updatePost(updatedPost, id).subscribe({
            //   next: (responseData) => {
            //     console.log(responseData);
            //   },
            //   error: (err) => {
            //     console.log(`Data could not be updated: ${err}`);
            //   },
            //   complete: () => {
            //     console.log('Completed editing the post');
            //     this.onFetchPosts();
            //   },
            // });
        }

        onClearPosts() {
            // Send Http Delete request for all data
            this.postService.deleteAllPosts().subscribe({
            next: (responseData) => {
                console.log(responseData);
            },
            error: (err) => {
                console.log(`Data could not be deleted: ${err}`);
            },
            complete: () => {
                console.log('Completed deleting the post');
                this.onFetchPosts();
            },
            });
        }
        }

Requests are only sent if you subscribe to the response.

The http method will take the supplied JS object and convert it to JSON.

## Interceptors

In case certain headers/params need to be re-used on different http methods, such as authentication tokens, an interceptor is written which runs before an http request is sent.
ex: In common-interceptor.ts

`ng g interceptor <name>`
In app.module.ts:
import {HTTP_INTERCEPTORS} from '@angular/common/http';
import { CommonInterceptor } from './common.interceptor';
in providers [ ] , add :

    providers: [
        { provide: HTTP_INTERCEPTORS, useClass: CommonInterceptor, multi: true },
        { provide: HTTP_INTERCEPTORS, useClass: LoggingInterceptor, multi: true },
    ],

For different interceptors, add other additional objects. They will execute in the order listed.
One can be for authentication, one can be for logging and error handling

On common.interceptor.ts:

        import { Injectable } from '@angular/core';
        import {
        HttpRequest,
        HttpHandler,
        HttpEvent,
        HttpInterceptor,
        HttpEventType,
        } from '@angular/common/http';
        import { Observable } from 'rxjs';

        @Injectable()
        export class CommonInterceptor implements HttpInterceptor {
            constructor() {}

            // Apply a header to all outgoing requests
            // Apply common standard error handling to all http requests
            intercept(request: HttpRequest<unknown>, next: HttpHandler): Observable<HttpEvent<unknown> {
                    const API_KEY = 'APIxyz123';
                    const USER_ROLE_KEY = 'Role123';
                    return next.handle(request.clone({ setHeaders: { API_KEY, USER_ROLE_KEY } }));
            }
        }

In logging interceptor:

        import { Injectable } from '@angular/core';
        import {
        HttpRequest,
        HttpHandler,
        HttpEvent,
        HttpInterceptor,
        HttpEventType,
        } from '@angular/common/http';
        import { catchError, Observable, of, retry, tap } from 'rxjs';

        @Injectable()
        export class LoggingInterceptor implements HttpInterceptor {
            constructor() {}

            intercept(
                request: HttpRequest<unknown>,
                next: HttpHandler
            ): Observable<HttpEvent<unknown>> {
                console.log(request.url);
                console.log(request.headers);
                return next.handle(request).pipe(
                tap((event) => {
                    console.log(event);
                    if (event.type === HttpEventType.Response) {
                    console.log(`Response arrived, body data: ${event.body}`);
                    }
                }),
                retry(2),
                catchError((error) => {
                    console.log(error);
                    return of(error);
                })
                );
            }
        }
