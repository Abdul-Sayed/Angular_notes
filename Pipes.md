# Angular Pipes

**Angular's built in pipes**

https://angular.io/api/common#pipes

        {{ value_expression | date:'medium'  | uppercase }}
        {{ value_expression | uppercase }}
        {{ value_expression | lowercase }}
        Decimal: {{ value_expression | number }}
        Currency: {{ value_expression | currency }}
        Percent: {{ value_expression | percent }}

**Custom Pipes**

Create a shorten.pipe.ts file or enter: ng g p shorten

Import and add it to the declarations array in the module its being used.

        import {Pipe, PipeTransform} from '@angular/core'

        @Pipe({
            name: 'shorten'
        })
        export class ShortenPipe implements PipeTransform {

        transform(value: string) {
            if (value > 10) {
                return value.substr(0, 10) + '...';
            }
            return value;
        }

Then in any component use it in the template as {{ data | shorten }}

Parameterize a custom pipe with additional arguments

        transform(value: string, limit: number) {
            if (value.length > number) {
                return value.substr(0, limit) + '...';
            }
            return value;
        }

Use it as as {{ data | shorten:10 }}

**async pipe**

For some value that resolves after some time, use the async pipe

{{ value_expression | aync }}
