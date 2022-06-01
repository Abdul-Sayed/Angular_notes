# Angular Authentication

The validation of authentication status must happen on a server, not the client.
The client sends the authetication server the required authentication data (username and password), and if its valid, the server responds with a JSON token.
This token is then included by the client to any http requests to the server. The server will check if the token is valid before responding with the requested data.

        import { Injectable } from '@angular/core';
        import {
        HttpClient,
        HttpHeaders,
        HttpParams,
        HttpEventType,
        } from '@angular/common/http';
        import { Observable, Subject, throwError } from 'rxjs';
        import { environment } from '../../environments/environment';

        interface AuthResponseData {    // With angular http calls, you must pass a generic type for the response
        idToken: string;
        email: string;
        refreshToken?: string;
        expiresIn?: string;
        localId: string;
        registered?: boolean;
        kind?: string;
        displayName?: string;
        }

        @Injectable({
        providedIn: 'root',
        })
        export class AuthService {
        signUpURL = `https://identitytoolkit.googleapis.com/v1/accounts:signUp?key=${environment.apiKey}`;
        signInURL = `https://identitytoolkit.googleapis.com/v1/accounts:signInWithPassword?key=${environment.apiKey}`;

        constructor(private http: HttpClient) {}

        signUp(email: string, password: string) {
            const body = {
            email: email,
            password: password,
            returnToken: true,
            };
            return this.http.post<AuthResponseData>(this.signUpURL, body);

            /* Example Response
            {
            "idToken": "eyJhbGciOiJSUzI1NiIsImtpZCI6IjM1MDZmMzc1MjI0N2ZjZjk0Y2JlNWQyZDZiNTlmYThhMmJhYjFlYzIiLCJ0eXAiOiJKV1QifQ.eyJpc3MiOiJodHRwczovL3NlY3VyZXRva2VuLmdvb2dsZS5jb20vaW5ncmVkaWVudHMtc2hvcHBpbmciLCJhdWQiOiJpbmdyZWRpZW50cy1zaG9wcGluZyIsImF1dGhfdGltZSI6MTY0MTcwOTkyNCwidXNlcl9pZCI6Iko5TVdGSmY5YjNUdmlldlQ5SWJET29DT3R0MDIiLCJzdWIiOiJKOU1XRkpmOWIzVHZpZXZUOUliRE9vQ090dDAyIiwiaWF0IjoxNjQxNzA5OTI0LCJleHAiOjE2NDE3MTM1MjQsImVtYWlsIjoiZXJlcmVyZXJAZ21haWwuY29tIiwiZW1haWxfdmVyaWZpZWQiOmZhbHNlLCJmaXJlYmFzZSI6eyJpZGVudGl0aWVzIjp7ImVtYWlsIjpbImVyZXJlcmVyQGdtYWlsLmNvbSJdfSwic2lnbl9pbl9wcm92aWRlciI6InBhc3N3b3JkIn19.NRajo-aLK9wv93tgtmE6dE5v-6l-6LTUKkDfJo5gnhkXEcDUq9rab4DtednVLrE4It-_lqWMzXsZJmiHKXN-koq7TCGVqnMOxvjU4i3uVadUxnz2YSSxy7IqpgrsCdZ3vxVsmRuqpk_QuQ8NyZgQvk6138MB6cP3nex-lK-VPev9kYtUvv01xmJ7wnQlPP__3dPRxj1_1vzkVoV62YERW1m6c8W6BMje5ZeodLNiJtDmUt08e10DDN-EqjClV-TryscGbaBmG9ihgLYwbDqhLyKhPMT9O8UmVPejMluUThJUC6fp16orYSOJeBcDhBr2BY6xOFC9ZcV-A-XmYDUHKw",
            "email": "erererer@gmail.com",
            "refreshToken": "AFxQ4_qMGcRHL4vQ6j35PoFJwh8-OUYMDblIGgfOAOF47CkcOeZaDFpxBjVQlkA9jctkDuddRkZQxh-WZ-eZWB_BwS-O-tNFoJAlf_ZZZ3xG7ZyWgIXf12WqDgdVchcfemsU8En6RRbJSVVeI_TWW55WzJlk27770YKFDeeBD1p6193wgqujW7qU82_tup6ii1f02vNXCK0GQwbpHjbgR8pZfWYcMjHXJBPvhn4PxCZT5LDtsL6SALc",
            "expiresIn": "3600",
            "localId": "J9MWFJf9b3TvievT9IbDOoCOtt02"
            "kind": "identitytoolkit#SignupNewUserResponse",
            }
            */
        }

        signIn(email: string, password: string) {
            const body = {
            email: email,
            password: password,
            returnToken: true,
            };
            return this.http.post<AuthResponseData>(this.signInURL, body);

            /* Example response:
        {
            "idToken": "eyJhbGciOiJSUzI1NiIsImtpZCI6InRCME0yQSJ9.eyJpc3MiOiJodHRwczovL2lkZW50aXR5dG9vbGtpdC5nb29nbGUuY29tLyIsImF1ZCI6ImluZ3JlZGllbnRzLXNob3BwaW5nIiwiaWF0IjoxNjQxNzEwMzMwLCJleHAiOjE2NDI5MTk5MzAsInVzZXJfaWQiOiJRS3JIYkVmcGFEZTN4VklObHRHOUJkc0hyUnUyIiwiZW1haWwiOiJ0ZXN0QHRlc3QuY29tIiwic2lnbl9pbl9wcm92aWRlciI6InBhc3N3b3JkIiwidmVyaWZpZWQiOmZhbHNlfQ.JtAS6GSLaDOT4unywRSFm9ayYp_F1g6bioc8-LC_UOVwP6tSMXy-mHXy4fLeYqCXwHVDwCmMu7DUjCH9f-YLCCcWe-5p0ZFvfjleIFAkO13j-uJBrk6WOURjKMkjp2alHqJw914Ch06U1TClB7T-89l0aJBCaGYVJj5pCq5y0QT7Sgd0acdrcNzceKHVvg1VVAgEKQmqMzMaMk05rFYbzUnzXT6QAKwGnaggr0PUnnjnRQkp2atjRBrHaHFtBhy0CM8CAE5BYszsJ4poiKAplGo7facI0NTt3KawN-v9F0Fp33Gy_-vg_dQyMCwd1CV0eCR6VOZafZy7MPQBuaOn4Q",
            "email": "test@test.com",
            "localId": "QKrHbEfpaDe3xVINltG9BdsHrRu2",
            "registered": true,
            "kind": "identitytoolkit#VerifyPasswordResponse",
            "displayName": "",
        }
            */
        }
        }
