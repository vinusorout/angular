# angular
Angular Personal Learning

## Parent to child OR Child to parent interation:
https://angular.io/guide/component-interaction

### @Input in child
If you update the parent property, and that property is assigned to child inout paraneter, then if you update the parent then the child will also update.

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
// USE ngOnChanges METHOD IF YOU NEED TO EXECUTE ANY METHOD OF CHILD COMPONENT WHEN ANY INPUT VALYES CHANGES.
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

### Call child functions from Parent using ViewChild.
```js
import { ViewChild } from '@angular/core';
import { Component, OnInit } from '@angular/core';
import { ChildComponent } from './child.component';

@Component({
  selector: 'parent-part',
  templateUrl: './parent.component.html',
  styleUrls: ['./parent.component.css']
})

export class ParentnComponent implements OnInit {
@ViewChild(ChildComponent) child: ChildComponent | undefined;
constructor() { }

calChildMethod() {
    this.child.childMethod();
}

}
```

### Using @ViewChild to inject a reference to a DOM element
Instead of injecting a direct child component, we might want to interact directly with a plain HTML element of the template.

In order to do that, we need to first assign a template reference to the HTML tag that we want to inject. for eaxmple #fullScreen:
```htm
<div #fullScreen>
</div>
```

```ts
@Component({
  selector: 'app-root',
  templateUrl: './app.component.html'
})
export class AppComponent implements  AfterViewInit {
  .....
  
  @ViewChild('fullScreen') fullScreen: ElementRef;

  ngAfterViewInit() {
    console.log('Values on ngAfterViewInit():');
    console.log("title:", this.fullScreen.nativeElement);
  }
  openFullscreen() {
        const fullScreen = this.fullScreen?.nativeElement;

        if (fullScreen.requestFullscreen) {
            fullScreen.requestFullscreen();
        } else if (fullScreen.msRequestFullscreen) {
            fullScreen.msRequestFullscreen();
        } else if (fullScreen.mozRequestFullScreen) {
            fullScreen.mozRequestFullScreen();
        } else if (elem.webkitRequestFullscreen) {
            fullScreen.webkitRequestFullscreen();
        }
    }
  .....
}
```

### Using ViewChild with a component, that is used in a parent component.

```ts
export class AppComponent implements OnInit {
  .
  .
  .
  @ViewChild(ChildComponent) canvasMap: ChildComponent | undefined;
  .
  .
  .
}
```








