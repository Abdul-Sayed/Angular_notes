# Angular Typescript

TS is a superset of Javascript to extend it with static typing.
It is transpiled to JS. And adds static typing to Javascript. It revearls type errors during development instead of just at runtime.

npm init -y  
npm install typescript  
npm tsc file.ts // to run a ts file

## Basic Types:

numbers, strings, booleans, symbol, undefined, null, unknown,

        let age: number = 12.1   (assigning anything other than a number breaks it)

For primitive types, the type is inferred, you don't need to specify

## Union Types

For when it makes sense for a variable to be of more than one type;

        let course: string | number;

## Complex Types

        let hobbies: string[]    //   or <Array>string

        let person: { name: string, age: number } = { name: 'Max', age: 32 }

## Type Aliases

Besides interfaces, reusable types can be defined in an alias;

        type Person : {
                name: string;
                age: number
        }
        ...
        let person: Person = { name: 'Max', age: 32 }

Aliases cannot be implemented by classes

## Generic Types

Generic types are flexibly defined based on how they're used

        function insertAtBeginning<T>( array: T[], value: T) {
                const newArray = [ value, ...array];
                return newArray;
        }

        const demoArr = [1, 2, 3];
        const updatedArr = insertAtBeginning(demoArr, -1);  // [-1, 1, 2, 3]

        T will be assigned the number type on the fly during usage

If passed strings, T would be a string

## Classes and Interfaces

        class Student {
                constructor(public firstName: string, public lastName: string, public age: number, private courses: string[ ]) { }
        }

Properties/methods with public access modifier can be used outside the class while private ones can only be used inside the class (and protected ones can be used in the class and its instances)

### Interfaces

Interfaces are object type definitions that (unlike aliases) can be implemented by classes.

        interface Human {
                firstName: string;
                age: number;

                greet: () => void;
        }
        ...
        let max: Human;

        max = {
                name: 'Max',
                age: 32,
                greet() {
                        console.log(`Hello, I'm ${this.name}`);
                }
        }

When a class implements an interface, its forced to have the structure defined by the interface.

        class Instructor implements Human {
                firstName: string;
                age: number;
                constructor(public firstName: string, public age: number ) {}
                greet() {
                        console.log(`Hello, I'm ${this.name}`);
                }
        }
