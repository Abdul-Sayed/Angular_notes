# Angular Forms

## Reactive Forms

Form is created programatically in the TS file and syncs with the DOM. It uses FormGroup to make a form and FormControl to make a form element. FormGroup groups together one or more FormControls.

import {ReactiveFormsModule} from '@angular/forms', and add to imports [ ] of the module the component is in

In the component code,

        import {FormGroup, FormControl, Validators} from '@angular/forms';
        ...
        signupForm: FormGroup;
        genders = ['male', 'female']
        ...
        ngOnInit() {
            this.signupForm = new FormGroup({
                'userData': new FormGroup({  // nested FormGroups appear for related form elements
                    'username': new FormControl(null, Validators.required),  // 1st arg of FormControl is the initial value of the element
                    'email': new FormControl(null, [Validators.required, Validators.email])
                })
                'gender': new FormControl('male', Validators.required),
            });
        }

        onSubmit() {
            console.log(this.signupForm);
        }

Alternatively, a form can be initialized using setValue;

        ngOnInit() {
            const formObj = new FormGroup{
                'userData': new FormGroup({
                    'username': new FormControl(null, Validators.required),
                    'email': new FormControl(null, [Validators.required, Validators.email])
                })
                'gender': new FormControl('male', Validators.required),
            }
            this.signupForm = new FormGroup(formObj)
        }

In the template

        <form [formGroup]="signupForm" (ngSubmit)="onSubmit()">
            <div formGroupName="userData">
                <label for="username">Username</label>
                <input
                    type="text"
                    id="username"
                    formControlName="username">
                <span *ngIf="!signupForm.get('userData.username').valid && signupForm.get('userData.username').touched">Please enter a valid username</span>

                <label for="email">Email</label>
                <input
                    type="email"
                    id="email"
                    formControlName="email">
                <span *ngIf="!signupForm.get('userData.email').valid && signupForm.get('userData.email').touched">Please enter a valid email</span>
            </div>

            <div *ngFor="let gender of gender">
                <label>
                    <input type="radio" [value]="gender" formControlName="gender">
                    {{gender}}
                </label>
            </div>
            <span *ngIf="!signupForm.get('gender').valid && signupForm.get('gender').touched">Please enter a valid gender</span>

            <button
                class="btn btn-primary"
                type="submit"
                [disabled]="!f.valid">
                Submit
            </button>
            <span *ngIf="!signupForm.valid && signupForm.touched">Please enter valid data</span>
        </form>

...

        input.ng-invalid.ng-touched {
            border: 1px solid red;
        }

For more complex data, use FormBuilder;

        import {FormBuilder} from '@angular/forms';  // import to the component
        ...
        newLoanForm: any;
        constructor(private formBuilder: FormBuilder)

        ngOnInit() {
            this.newLoanForm = this.formBuilder.group({
                'loanName': ['Car loan', Validators.required],
                'loanType': ['Personal loan', Validators.pattern('^[a-zA-Z]+$')],
                'loanDescription': ['For transport', [Validators.required, Validators.minLength(10)]]
            })
        }

        addLoan() {
            // To get entire form; `this.newLoanForm.value`
            // To get a specific form element; `this.newLoanForm.get('loanName').value`
            // observable that emits an event on any form change; `this.newLoanForm.valueChanges.subscribe(data => console.log(data)`
            this.formName.get('email').valueChanges.subscribe(data => {  // for listening to specific form field
                console.log(data);
            })
            this.newLoanForm.reset();
        }

        ...

        // Alternativly declare the form with setValue. This is useful when resetting the form to an intial state
        // All keys must match for setValue. For patchValue, only some need to be present

        const newLoanObj = {
                'loanName': ['Car loan', Validators.required],
                'loanType': ['Personal loan', Validators.pattern('^[a-zA-Z]+$')],
                'loanDescription': ['For transport', [Validators.required, Validators.minLength(10)]]
        }
        this.newLoanForm.setValue(newLoanObj);


        // To track the status change of the form; touched, dirty, pristine, valid, etc
        this.formName.statusChanges.subscribe(data=> console.log(data))
        this.formName.get('email').statusChanges.subscribe(data => {  // for listening to validation status of a specific form field
            console.log(data);
        })

In Template

        <form [formGroup]="newLoanForm">
            <label>Loan Name</label>
            <input type="text" formControlName="loanName">
            <span *ngIf="newLoanForm.get('loanName').hasError('required') && newLoanForm.get('loanName').touched">Loan name is required</span>

            <label>Loan Type</label>
            <input type="text" formControlName="loanType">

            <label>Loan Description</label>
            <textarea type="text" formControlName="loanDescription"></textarea>
            <span *ngIf="newLoanForm.get('loanDescription').hasError('minLength') && newLoanForm.get('loanDescription').touched">Minimum of 10 chars required</span>

            <button (click)="addLoan()" [disabled]="!newLoanForm.valid ">Add Loan</button>
        </form>

Form Arrays
Used to add multiple form fields dynamically, such as another education form field appearing on the click of a button in a resume entry site.
Programatically adds new formControls or formGroups. Uses FormArray. If the status of any one formControl in the formArray is invalid, the entire formArray becomes invalid.

        'users': new FormArray([
            new FormControl(''),
            new FormControl('')
        ])

        ...

        <div formArrayName="users">
            <label>Users</label>
            <div *ngFor="let control of myForm.controls.users['controls']; let i = index">  // also can use  myForm.get(users).controls;
                <input type="text" [FormControlName]="i">
            </div>
        </div>

...

        'users': new FormArray([
            this.fb.group({
                'name': new FormControl('')
                'age': new FormControl(''),
                'dept': new FormControl('')
            })
        ])

        ...

        <div formArrayName='users'>
            <label>Users</label>
            <div *ngFor="let control of myForm.controls.users['controls']; let i = index;" [formGroupName]='i'>  //  // also can use  myForm.get(users).controls;
                <input type="text" FormControlName="name">
                <input type="text" FormControlName="age">
                <input type="text" FormControlName="dept">
            </div>
        </div>

To add or remove elements (FormControls or FormGroups) from a FormArray;
Assuming the users FormArray was part of a larger FormGroup, myForm

        addUser() {
            let userArray = this.myForm.get('users') as FormArray;
            const newUser = this.fb.group({
                'name': '',
                'age': '',
                'dept': ''
            })
            userArray.push(newUSer)
        }

        removeUser(name) {
            let userArray = this.myForm.get('users') as FormArray;
            userArray = userArray.filter(user => user.name !== name)
        }

        moveUser(name) {
            let userArray = this.myForm.get('users') as FormArray;
            let user = userArray.find(user = > user.name === name);
            if (user.age > 55) {
                user.dept = 'admin';
            }
            this.users.setValue(user);
        }



## Working Example

        import { Component, OnInit } from '@angular/core';
        import { FormControl, FormGroup, Validators } from '@angular/forms';

        import { CustomValidators } from './custom-validators';

        @Component({
        selector: 'app-root',
        templateUrl: './app.component.html',
        styleUrls: ['./app.component.css']
        })
        export class AppComponent implements OnInit {
        projectForm: FormGroup;

        ngOnInit() {
            this.projectForm = new FormGroup({
            'projectName': new FormControl(
                null,
                [Validators.required, CustomValidators.invalidProjectName],
                CustomValidators.asyncInvalidProjectName
            ),
            'email': new FormControl(null, [Validators.required, Validators.email]),
            'projectStatus': new FormControl('critical')
            });
        }

        onSaveProject() {
            console.log(this.projectForm.value);
        }
        }

..

        import { FormControl } from '@angular/forms';
        import { Observable } from 'rxjs/Observable';

        export class CustomValidators {
        static invalidProjectName(control: FormControl): {[s: string]: boolean} {
            if (control.value === 'Test') {
            return {'invalidProjectName': true};
            }
            return null;
        }

        static asyncInvalidProjectName(control: FormControl): Promise<any> | Observable<any> {
            const promise = new Promise((resolve, reject) => {
            setTimeout(() => {
                if (control.value === 'Testproject') {
                resolve({'invalidProjectName': true});
                } else {
                resolve(null);
                }
            }, 2000);
            })
            return promise;
        }
        }

...

        <div class="container">
        <div class="row">
            <div class="col-xs-12 col-sm-10 col-md-8 col-sm-offset-1 col-md-offset-2">
            <form [formGroup]="projectForm" (ngSubmit)="onSaveProject()">
                <div class="form-group">
                <label for="name">Project Name</label>
                <input
                    type="text"
                    id="name"
                    formControlName="projectName"
                    class="form-control">
                </div>
                <div class="form-group">
                <label for="email">Mail</label>
                <input
                    type="email"
                    id="email"
                    formControlName="email"
                    class="form-control">
                </div>
                <div class="form-group">
                <label for="status">Projectstatus</label>
                <select
                    id="status"
                    formControlName="projectStatus"
                    class="form-control">
                    <option value="stable">Stable</option>
                    <option value="critical">Critical</option>
                    <option value="finished">Finished</option>
                </select>
                </div>
                <button class="btn btn-primary" type="submit">Create Project</button>
            </form>
            </div>
        </div>
        </div>
