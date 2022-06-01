# Angular basics

## Data Presentation

0. Interpolation - one was data binding; class --> template  
   Use for displaying data that can be resolved to a string (ex: strings, methods, numbers)
   {{variable}}

1. Property Binding - one was data binding; class --> template

Used for changing template element attributes (on left) by binding them to component properties (on right);

    <button [disabled]="isDisabled"></button>
    <p [innerText]="!isDisabled"></p>

2.  Event Binding

Used for binding user actions on template elements to component methods;

      <button (click)="onButtonClick()"></button>
      <input type="text" (input)="onChange($event)" />

3. Two Way Binding

Need to have FormsModule installed
Used to provide the template element with data from the class and to update that data with user interaction with the element.

    <input type="text" [(ngModel)]="firstName" />

## Template Reference Variables

To referance an html element using a variable that can be used anywhere in that template (only in template)

    <input type="text" #serverInput>
    <button (click)="addServer(serverInput)">Add Server</button>

    ...
    addServer(input: HTMLInputElement) {
      this.server.create(input.value);
    }

## @ViewChild

To reference an html element (like a template reference variable) in the typescript class, use @ViewChild

    import {ViewChild, ElementRef} from '@angular/core';
    ...
    @ViewChild('serverInput') serverInput: ElementRef;    --> for #serverInput in template
    ...
    // use as serverInput.nativeElement.value

Best to use for read operations, not to change the element
Template referance variables are available after the view initializes - ngAfterViewInit

## Directives

Directives are instructions for DOM elements. They can be Attribute or Structural Directives

**Attribute Directives**
Uses property binding to only affect the element they're added to; ngClass, ngStyle

**Structural Directives**
Affect a section of the DOM by adding/removing elements; *ngIf, *ngFor, ngSwitch

Some built in directives are ngClass, ngStyle, *ngIf, *ngFor

_ngStyle_:

    <p [ngStyle]="{backgroundColor: getColor()}" > Status is {{status}}</p>

    getColor() {
      return this.serverStatus === 'online' ? 'green' : 'red';
    }

_ngClass_:

    .red {
      color: red;
    }

    <button [ngClass]="red: timeRemaining <=10">Submit</button>

**Custom Attribute Directive**

Create a directive by adding fileName.directive.ts, and add it to the module in declarations []
Or by entering ng g d file-name into cli

    import {Directive, ElementRef, Renderer2, OnInit} from '@angular/core';
    @Directive ({
      selector: '[appFileName]';
    })

    export class fileNameDirective implements OnInit {
      constructor(private elementRef: ElementRef, renderer: Renderer2) {}

      ngOnInit() {
        // write the change to the selected element
        this.renderer.setStyle(this.elementRef.nativeElement, 'background-color', 'green');
      }
    }

    --- alternative way with inputs from consumer

    import {Directive, HostListener, HostBinding, Input} from '@angular/core';
    @Directive ({
      selector: '[appFileName];
    })

    export class fileNameDirective {

      @Input() defaultColor: string;
      @Input() highlightColor: string;
      @HostBinding('style.backgroundColor') backgroundColor: string = this.defaultColor;

      constructor() {}

      // listens for user events to apply the proprty change
      @HostListener('mouseenter')
      mouseenter(eventData: Event)  {
        this.backgroundColor = this.highlightColor;
      }
      @HostListener('mouseleave')
      mouseleave(eventData: Event)  {
        this.backgroundColor = this.defaultColor;
      }
    }

---

use it in a component html as:

    <p appFileName>Styled with directive</p>
    // or if using the alternative way
    <p appFileName [defaultColor]="'transparent'" [highlightColor]="'green'">Styled with directive</p>

---

Another example using a css class
import { Directive, HostListener, HostBinding} from '@angular/core';

    @Directive({
      selector: '[appDropdown]'
    })
    export class DropdownDirective {

      @HostBinding('class.open') isOpen = false;

      constructor() { }

      @HostListener('click')
      toggleOpen() {
        this.isOpen = !this.isOpen;
      }

    }

## Input/Output property drilling

**Input**
For some variable pcomponent in parent, use property binding to feed data to child component attribute

parent.html:

    <app-child [PData]="pcomponent.value"></app-child>

Retrieve that property in child component

    import { Component, Input } from '@angular/core';
    ...
    @Input() PData: number;

    ...
    <p>Value from parent component is: {{ PData }}</p>

A child component can have multiple parents feeding it the same or different data
An alias can be setup for the shared variable in case the name changes, while keeping the local name the same

**Output**

Child component

    import {Component, Output, EventEmitter} from '@angular/core';
    ...
    @Output() childData = new EventEmitter<string>();
    _childData: string;  // gets populated someway

    ...
    sendItUp() {   // gets triggered someway
      this.childData.emit(_childData);
    }

Parent Component
Use event binding to bubble data up to parent

    <child (childData)="onChange($event)"></child>
    ...
    onChange($event) {
      // use $event
    }

**Child to parent flow using @ViewChild**  
 child component:

          @Component({
            selector: 'app-child',
            template:`
              <p>{{data}}</p>
            `
          })
        export class ChildComponent {
          data:string = "Message from child to parent";
        }

    parent:

     import { Component,ViewChild, AfterViewInit} from '@angular/core';
      import { ChildComponent } from './../child/child.component';

      @Component({
        selector: 'app-parent',
        template: `
          <p>{{dataFromChild}}</p>
        ` ,
      })
      export class ParentComponent implements AfterViewInit {
        @ViewChild(ChildComponent,{static:false}) child;
        dataFromChild: string;

        ngAfterViewInit(){
          this.dataFromChild = this.child.data;
        }
      }

## Overriding component style encapsulation

This make's the component's styling global

    import {ViewEncapsulation} from '@angular/core';

    @Component({
      ...
      encapsulation: ViewEncapsulation.None
    })

## ngContent

When rendering child components, html content can be projected onto the child.
In parent.html

    <child *ngfor="let item of list">
      <p *ngIf="item.active">{{item.content}}</p>
    </child>

In child.html, add the ng-content hook to recieve the template content from the parent

    <ng-content>
      // content will render here
    </ng-content>

Useful for reusable widgets

The passed html content can have a template reference variable;
In parent.html

    <child *ngfor="let item of list">
      <p *ngIf="item.active" #contentParagraph>{{item.content}}</p>
    </child>

In child.ts,

    @ContentChild('contentParagraph') contentParagraph: ElementRef;

Will be available after ngAfterContentInit runs

## Lifecycle Hooks

Constructor - Calle first
ngOnChanges - Called upon component initialization and whenever @Input properties change - pass (changes: SimpleChanges) to the method to see prev & current value of the input props
ngOnInit - Called upon component initialization, after ngOnChanges
ngDoCheck - Called on every change to any local property, and every user event
ngAfterContentInit - Called after the ng-content passed from the parent arrives
ngAfterContentChecked - Called every time ng-content passed from parent changes
ngAfterViewInit - Called after component's view has been rendered
ngAfterViewChecked - Called every time component's view changes
DoCheck - Called whenever any component data gets updated or for any user interaction
ngOnDestroy - Called everytime a component is removed from the DOM (for ex: by \*ngIf)

Import them and have the class implement them for use
