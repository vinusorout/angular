# angular
Angular Personal Learning

## Parent to child OR Child to parent interation:
https://angular.io/guide/component-interaction

### Intercept input property changes with a setter
se an input property setter to intercept and act upon a value from the parent.
Child:
```ts
import { Component, Input } from '@angular/core';

@Component({
  selector: 'app-name-child',
  template: '<h3>"{{name}}"</h3>'
})
export class NameChildComponent {
  @Input()
  get name(): string { return this._name; }
  set name(name: string) {
    this._name = (name && name.trim()) || '<no name set>';
  }
  private _name = '';
}
```

Parent:
```ts
import { Component } from '@angular/core';

@Component({
  selector: 'app-name-parent',
  template: `
  <h2>Master controls {{names.length}} names</h2>
  <app-name-child *ngFor="let name of names" [name]="name"></app-name-child>
  `
})
export class NameParentComponent {
  // Displays 'Dr IQ', '<no name set>', 'Bombasto'
  names = ['Dr IQ', '   ', '  Bombasto  '];
}
```
In Above example whenever we change names array, even after application initialization, the chnages will reflect in child component.


### Intercept input property changes with ngOnChanges()
Detect and act upon changes to input property values with the ngOnChanges() method of the OnChanges lifecycle hook interface.
```ts
import { Component, Input, OnChanges, SimpleChanges } from '@angular/core';

@Component({
  selector: 'hello',
  template: `
    <h1>Hello test {{ typeOfInput }}!</h1>
    <h1>Hello name {{ name }}!</h1>
  `,
  styles: [
    `
      h1 {
        font-family: Lato;
      }
    `
  ]
})
export class HelloComponent implements OnChanges {
// nubmer with ngOnChanges method
ngOnChanges(changes: SimpleChanges): void {
  console.log('change detected');
  for (let propName in changes) {
    console.log(propName);
    // if(propName === 'number') {
    //   this.typeOfInput = this.number;
    // }
  }
}
  @Input() number: number;
  typeOfInput: any;
  
  // name with setter
  @Input()
  get name(): string { return this._name; }
  set name(name: string) {
    this._name = (name && name.trim()) || '<no name set>';
  }
  private _name = '';

  ngOnInit() {
    this.typeOfInput = this.number;
  }
}

```


### Child to Parent should be done using EventEmmiters




