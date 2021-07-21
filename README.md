# angular
Angular Personal Learning

## Life Cycles Hooks:

* **ngOnChanges**: When an input/output binding value changes.
	* Called before ngOnInit() (if the component has bound inputs) and whenever one or more data-bound input properties change.
	* Note that if your component has no inputs or you use it without providing any inputs, the framework will not call ngOnChanges()
* **ngOnInit**: After the first ngOnChanges.
	* Initialize the directive or component
	* ngOnInit() is still called even when ngOnChanges() is not (which is the case when there are no template-bound inputs)
* **ngDoCheck**: Developer's custom change detection.
	* Detect and act upon changes that Angular can't or won't detect on its own
	* Called immediately after ngOnChanges() on every change detection run, and immediately after ngOnInit() on the first run
* **ngAfterContentInit**: After component content initialized.
* **ngAfterContentChecked**: After every check of component content.
* **ngAfterViewInit**: After a component's views are initialized.
	* Respond after Angular initializes the component's views and child views, or the view that contains the directive.
```ts
...
// Query for a VIEW child of type `ChildViewComponent`
  @ViewChild(ChildViewComponent) viewChild!: ChildViewComponent;

  ngAfterViewInit() {
    // viewChild is set after the view has been initialized
    this.logIt('AfterViewInit');
    this.doSomething();
  }
  ...
```
* **ngAfterViewChecked**: After every check of a component's views.
* **ngOnDestroy**: Just before the directive is destroyed.
	* To unsbscribe from the promises, to minimize the memory leak.

## Services
* Injectable need to know where to make the class available, like root of the application or feature module
* And a token, the class name is used as Token.
```ts
@Injectable({
	providedIn: 'root' // at root level, singleton instance
})
export class Storage {
}
```
* if you need to change the token:
```ts
@Injectable({
	providedIn: 'root', // at root level, singleton instance,
	useClass: OtherStorage // change the token
})
export class Storage {
}
```
* to scope a service to a feature module:
```ts
@Injectable({
	providedIn: MyModule // feature module name
})
export class Storage {
}
```
* lazy loading is better when service provideIn with feature module
* To inject service in any module with a **different instance**
```ts
@Injectable({
	providedIn: 'any' // feature module name
})
export class Storage {
}
```


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

### ExpressionChangesAfterItHasBeenCheckedError
* Occured when a value was modified AFTER change detection finished.
* Potential causes:
* Code update after ngAfterViewInit
```ts
ngAfterViewInit() {
	this.loading = false; // will throw this error
}
```
```html
<span *ngIf="loading">Show Me!</span>
```

What if you have to update the value in After View, let say you need to update it as per the any item, and that will be avilable only after  ngAfterViewInit, use either settime out or promise, this will make angular the detect the change in next change detection cycle.

```ts

@ViewChild('item') item;
ngAfterViewInit() {
	if(item) {
		Promise.resolve().then(() => this.loading = false;  )
	}
}
```
```html
<span *ngIf="loading">Show Me!</span>
```

Or by manually enable change detection:
```ts
constructor(private cd: ChangeDetectorRef) {}
ngAfterViewInit() {
	this.loading = false; 
	this.cd.detectChanges();
}
```
```html
<span *ngIf="loading">Show Me!</span>
```


* When change detection trigger itself in infinite loop, like a method that return diff value each time
```ts
get randomValue() {
	return Math.random();
}
```
```html
<p> This is a {{ randomValue }}</p>
```
* Or when child component tries to update Parent properties
Change detection runs from Parent to Child, first Parent then first level child then second level and so on...

Parent Component:
```ts
export class AppComponnet implements AfterViewInit {
	loading = true;
	ngAfterViewInit() {
	}
}
```

Child Component:
```ts
export class ChildComponent implements OnInit {
	@Output() ready = new EvevntEmmiter();
	ngOnInit() {
		this.ready.emit(true);
	}
}
```
Parent Template:
```html
<span *ngIf="loading">Show Me!</span>
<app-item (ready)="loading = false"></app-item>
```

To solve this move the code to a shared service.

### Cant bind to {prop} since it isnt a known property of {node}

```html
<my-component [name]> </my-component> // when name is not a attribute or property of my component, missing @input
```

### Multiple components match node with tagname {selector}
* the selector name inside a module should be unique

### Export of name {directive} not found
* example: Export of name 'ngForm' not found
```html
<form #form="ngForm"></form>
```
Can be fixed by importing the required module:
```ts
import {FormsModule } ...
.
.
.
imports: [
	FormsModule
]
.
.
.
```



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

## RxJS

### Observable
* of('Word'),
* map like the array .map => of([0,1,23]).map(x => x+1).filter(x > 10)
* map is an operator that transforms data by applying a function
* pipe composes operators (like map, filter, etc)
* Unlike **map**, which is an **operator**, **pipe** is a method on Observable which is used for **composing operators**. pipe was introduced to RxJS in v5.5 to take code that looked like this
```ts
of(1,2,3).map(x => x + 1).filter(x => x > 2);
```
to
```ts
of(1,2,3).pipe(
  map(x => x + 1),
  filter(x => x > 2)
);
```

* **forkJoin**, to call APis/observables parallely and clud results
```ts
forkJoin([
      this.service.method1(param1),
      this.service.method2(param2)
    ])
      .subscribe(response => {
        console.log(response[0]);
        console.log(response[1]);
      });
```
* ForkJoin operator faces some problems with delayed requests too. The order will be preserved but if one request is delayed all the others have to wait for its resolution
* Another issue with forkJoin() is related to the way it handles failed requests. If any of the executed requests fails it will fail for the whole collection. Instead of items we will receive first encountered exception
* MergeMap() executes requests in parallel and it is fault tolerant so we still display most of the posts even if some of the requests fail. Order is not maintained in MergeMap.

* **concatMap**
* concatMap(): Projects each source value to an Observable which is merged in the output Observable, in a serialized fashion waiting for each one to complete before merging the next
* No parallel calls
* In our case we want to perform a HTTP request for every ID and Angular HttpClient returns observable of a response by default
```ts
getItems(ids: number[]): Observable<Item> {
  return from(ids).pipe(
     concatMap(id => <Observable<Item>> this.httpClient.get(`item/${id}`))
  );
}
```

* **mergeMap**
* it executes all nested observables immediately as they pass through the stream. This is a major performance win because all our requests are executed in parallel
```ts
getItems(ids: number[]): Observable<Item> {
  return from(ids).pipe(
    mergeMap(id => <Observable<Item>> this.httpClient.get(`item/${id}`))
  );
}
```

## Directive (Attribute directives)
* With attribute directives, you can change the appearance or behavior of DOM elements and Angular components.
```sh
ng generate directive highlight
```
* highlight directive ex:
```ts
import { Directive, ElementRef } from '@angular/core';

@Directive({
  selector: '[appHighlight]'
})
export class HighlightDirective {
    constructor(el: ElementRef) {
       el.nativeElement.style.backgroundColor = 'yellow';
    }
}
```

```html
<p appHighlight>Highlight me!</p>
```

### Handling user events Using HostListener
```ts
import { Directive, ElementRef, HostListener } from '@angular/core';

@Directive({
  selector: '[appHighlight]'
})
export class HighlightDirective {
    constructor(el: ElementRef) {
       el.nativeElement.style.backgroundColor = 'yellow';
    }
    @HostListener('mouseenter') onMouseEnter() {
	  this.highlight('yellow');
	}

	@HostListener('mouseleave') onMouseLeave() {
	  this.highlight('');
	}

	private highlight(color: string) {
	  this.el.nativeElement.style.backgroundColor = color;
	}
}
```

### Passing values into an attribute directive, Using Input
```ts
import { Directive, ElementRef, HostListener, Input } from '@angular/core';

@Directive({
  selector: '[appHighlight]'
})
export class HighlightDirective {
    @Input() appHighlight = '';
    constructor(el: ElementRef) {
       el.nativeElement.style.backgroundColor = appHighlight;
    }
}
```
```html
<p [appHighlight]="color">Highlight me!</p>
```
### Passing multiple values
```ts
import { Directive, ElementRef, HostListener, Input } from '@angular/core';

@Directive({
  selector: '[appHighlight]'
})
export class HighlightDirective {
    @Input() appHighlight = '';
    @Input() defaultColor = '';
    constructor(el: ElementRef) {
       el.nativeElement.style.backgroundColor = appHighlight;
    }
}
```
```html
<p [appHighlight]="color" defaultColor="violet">
  Highlight me too!
</p>
```

### ngNonBindable, directive
* To prevent expression evaluation in the browser, add ngNonBindable to the host element. ngNonBindable deactivates interpolation, directives, and binding in templates
```html
<p ngNonBindable>This should not evaluate: {{ 1 + 1 }}</p>
```
* If you apply ngNonBindable to a parent element, Angular disables interpolation and binding of any sort, such as property binding or event binding, for the element's children.

## Structural Directive, like *ngIf
https://angular.io/guide/structural-directives


## ngTemplate and ngContent
* Used for creating Dynamic Template Creation
* **ng-template** : Like the name indicates, the ng-template directive represents an Angular template: this means that the content of this tag will contain part of a template, that can be then be composed together with other templates in order to form the final component template
* Angular is already using ng-template under the hood in many of the structural directives that we use all the time: ngIf, ngFor and ngSwitch.
```ts
@Component({
  selector: 'app-root',
  template: `      
<ng-template #estimateTemplate let-lessonsCounter="estimate">
    <div> Approximately {{lessonsCounter}} lessons ...</div>
</ng-template>
<ng-container 
	[ngTemplateOutlet]="estimateTemplate"
   	[ngTemplateOutletContext]="{ $implicit: ctx }">
</ng-container>
`})
export class AppComponent {

    totalEstimate = 10;
    ctx = {estimate: this.totalEstimate};
  
}
```


## UNIT TESTING

### Service with HttpClient:
```ts
import { TestBed } from '@angular/core/testing';
import { HttpClientTestingModule, HttpTestingController } from '@angular/common/http/testing';
import { MyService } from './my.service';
import { CommonService } from '../common.service';
import { Observable, of } from 'rxjs';
import { HttpErrorResponse } from '@angular/common/http';
import { MyData } from '../../model/mydata.model';
import { AppConstants } from 'src/app/app.constants';

let mockCommonService: Partial<CommonService>;

describe('MyService', () => {
  let service: MyService;
  let httpMock: HttpTestingController;
  let commonService: CommonService;

  beforeEach(() => {
    mockCommonService = {
      handleError(error: HttpErrorResponse): Observable<never> {
        console.log('Error handled');
        return of();
      },
      getbaseUrl() {
        return 'http://abc.com/';
      }
    };
    TestBed.configureTestingModule({
      imports: [HttpClientTestingModule],
      providers: [
        MyService,
        { provide: CommonService, useValue: mockCommonService }
      ]
    });
    service = TestBed.inject(MyService);
    commonService = TestBed.inject(CommonService);
    httpMock = TestBed.inject(HttpTestingController);
  });

  afterEach(() => {
    httpMock.verify();
  });

  it('should be created', () => {
    expect(service).toBeTruthy();
  });

  it('get my data success', () => {
    const myData: Partial<MyData>[] = [
      { Id: 1 }
    ];

    service.getMyData().subscribe(result => {
      expect(result.length).toBe(1);
      expect(result[0].Id).toEqual(1);
    }, error => {
      expect(true).toBe(false);
    });

    const req = httpMock.expectOne(`http://abc.com/` + AppConstants.MyData_URL);
    expect(req.request.method).toBe('GET');
    req.flush(myData);

  });

  it('get mydata faliure', () => {

    service.getMyData().subscribe(result => {
      expect(true).toBe(false);
    }, error => {
      expect(true).toBe(true);
      expect(commonService.handleError).toHaveBeenCalled();
    });

    const req = httpMock.expectOne(`http://abc.com/` + AppConstants.MyData_URL);
    expect(req.request.method).toBe('GET');

    // const errorEvent: ErrorEvent = new ErrorEvent('Not Found');
    // req.error(errorEvent);
    // or
    req.flush(null, { status: 400, statusText: 'Bad Request' });

  });
});

```

### Components
```ts
import { DebugElement } from '@angular/core';
import { ComponentFixture, TestBed } from '@angular/core/testing';
import { FormsModule } from '@angular/forms';
import { By } from '@angular/platform-browser';
import { ButtonModule } from 'primeng/button';
import { Dropdown, DropdownModule } from 'primeng/dropdown';
import { InputNumberModule } from 'primeng/inputnumber';
import { InputTextModule } from 'primeng/inputtext';
import { BehaviorSubject } from 'rxjs/internal/BehaviorSubject';
import { Observable } from 'rxjs/internal/Observable';
import { of } from 'rxjs/internal/observable/of';
import { MyService } from 'src/app/shared/my.service';

import { MyComponent } from './my.component';
  // My Service
  let mockedMyService: Partial<MyService>;

describe('MyComponent', () => {
  let component: MyComponent;
  let fixture: ComponentFixture<MyComponent>;

  beforeEach(async () => {
    mockedMyService = {
      onMyData: new BehaviorSubject<any>(null),
    };
    await TestBed.configureTestingModule({
      declarations: [ MyComponent ],
      providers: [ 
        { provide: MyService, useValue: mockedMyService }
      ],
      imports:[
        InputTextModule,
        InputNumberModule,
        DropdownModule,
        ButtonModule,
        FormsModule
      ]
    })
    .compileComponents();
  });

  beforeEach(() => {
    fixture = TestBed.createComponent(MyComponent);
    component = fixture.componentInstance;
    fixture.detectChanges();
  });

  it('should create', () => {
    expect(component).toBeTruthy();
  });

  it('verify input values on change ', async () => {
    let myMockedService = fixture.debugElement.injector.get(MyService);
    spyOn(myMockedService, 'addData').and.callFake(() => of([]));
    
    await fixture.whenStable();
    const data: Data = {'CC':10,'FI':263,'PC':0,'PR':0,'TI':69,'TN':'J-10','TT':'Storage', 'LC': 0};

    const dropdown: Dropdown = fixture.debugElement.query(By.css('p-dropdown')).componentInstance;
    
    component.editData = data;
    fixture.detectChanges();
    await fixture.whenStable();

    const name: DebugElement = fixture.debugElement.query(By.css('input[id=tN]'));
    expect(name.nativeElement.value).toBe(data.TN);
    // name.triggerEventHandler('input', { target: { value: data.TN } });
    fixture.detectChanges();
    await fixture.whenStable();
    const widthEl: DebugElement = fixture.debugElement.query(By.css('p-inputNumber[id=tL] input'));
    expect(widthEl.nativeElement.value).toBe(data.CC.toString());
    const verticalEl: DebugElement = fixture.debugElement.query(By.css('p-inputNumber[id=v] input'));
    expect(verticalEl.nativeElement.value).toBe(component.editData.PR.toString());
    const horizontalEl: DebugElement = fixture.debugElement.query(By.css('p-inputNumber[id=h] input'));
    expect(horizontalEl.nativeElement.value).toBe(component.editData.PC.toString());
    
    // horizontalEl.triggerEventHandler('blur', null);
    // dropdown.selectItem(new Event('change'), data.TT);
    
    expect(dropdown.selectedOption).toBe(component.editData.TT.toString());
    
    fixture.detectChanges();
    await fixture.whenStable();

    // get the Cancel Button Elemet element
    const saveButtonEl: DebugElement = fixture.debugElement.query(By.css('button[label=Save]'));

    // subscribe to the emmiter
    let dataReturned = 0;
    myMockedService.onDataChange?.subscribe(value => facilityReturned = value);

    // trigger the event
    saveButtonEl.triggerEventHandler('click', null);

    expect(dataReturned).toBe(1);
    expect(myMockedService.addTrack).toHaveBeenCalled();
  });

});

```










