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

## Importants

### Angular Zone

Angular Zone: Angular error handler runs out of angular zone, if you initiate a material pop-up from error handler, it will not close, using the close button. To close it first you need to initiate the pop-up using zone, only than you can close it.

* NZZone injected in component and then use. this.ngzonevariable.runOutsideAngula.

### PURE and IMPURE Pipes, https://indepth.dev/posts/1447/how-pure-and-impure-pipes-work-in-angular-ivy
* Impure pipes's transform function, runs every time any change is detected on the page
* Pure pipes's transform function, runs only when its own data value changed
```ts
	@Pipe({
	  name: 'myCustomPipe', 
	  pure: false/true        <----- here (default is `true`)
	})
export class MyCustomPipe {}
```
```html
<span>{{v1 | customPipe}}</span>
<span>{{v2 | customPipe}}</span>

// pass parameters to a pipe:
<span>{{v1 | customPipe:param1:param2}}</span>
```
* Since the pipe is pure it means that there’s no internal state and the pipe can be shared. How can Angular leverage that? **Even though there are two usages in the template Angular can create only one pipe instance which can be shared between the usages**.
* A good example of impure pipe is the AsyncPipe from @angular/common package. This pipe has internal state that holds an underlying subscription created by subscribing to the observable passed to the pipe as a parameter. Because of that Angular has to create a new instance for each pipe usage to prevent different observables affecting each other. And also has to call transform method on each digest because even thought the observable parameter may not change the new value may arrive through this observable that needs to be processed by change detection
```ts
// Async Pipe use
@Component({
  selector: 'async-promise-pipe',
  template: `<div>
    <code>promise|async</code>:
    <button (click)="clicked()">{{ arrived ? 'Reset' : 'Resolve' }}</button>
    <span>Wait for it... {{ greeting | async }}</span>
  </div>`
})
export class AsyncPromisePipeComponent {
  greeting: Promise<string>|null = null;
  arrived: boolean = false;

  private resolve: Function|null = null;

  constructor() {
    this.reset();
  }

  reset() {
    this.arrived = false;
    this.greeting = new Promise<string>((resolve, reject) => {
      this.resolve = resolve;
    });
  }

  clicked() {
    if (this.arrived) {
      this.reset();
    } else {
      this.resolve!('hi there!');
      this.arrived = true;
    }
  }
}
```


### HttpClient/HttpBackend
* If you have HttpClient at submodule level then it will over write the HttpClinet at app level
* If you want to neglact HttpInterceptors, use HttpBackend

### WHY IVY
* From version 9
* AOT compilation with Ivy is faster and should be used by default. In the angular.json workspace configuration file, set the default build options for your project to always use AOT compilation
* Ivy applications can be built with libraries that were created with the View Engine compiler. This compatibility is provided by a tool known as the Angular compatibility compiler (**ngcc**). CLI commands run ngcc as needed when performing an Angular build.
* With Ivy, you can compile components more independently of each other. This improves development times since recompiling an application will only involve compiling the components that changed
* Tree Shaking: It means that unused code is not included in a build during the build process, for example if you don’t need "views queries" you don’t need the Angular Framework code that updates these queries on browsers. "This is where the bundle size reduction comes from"
* Bootstrapping: Ivy provides a simpler API now to bootstrap a component. In existing implementation we need to use NgModule, that defines a component to bootstrap an application with.
```ts
		@NgModule({
			…
			bootstrap: [AppComponent]
		})
		export class AppModule {}
		platformBrowserDynamic().bootstrapModule(AppModule);
		
		In Ivy, the bootstrap function takes this bootstrap component directly:
		renderComponent(AppComponent)
```










