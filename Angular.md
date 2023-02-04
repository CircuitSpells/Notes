# Angular Notes

- [Angular Notes](#angular-notes)
  - [Misc](#misc)
  - [Routing](#routing)
  - [Modules](#modules)
  - [Angular CLI](#angular-cli)

---
## Misc
- index.html is the "Single Page" in "SPA".
  
- The `@function({...})` above a class is called a <ins>decorator</ins>, you can think of it like a C# attribute that takes in an object.

- The <ins>selector</ins> in the `@Component` decorator is used as an html <ins>element</ins> tag, called a <ins>directive</ins> in that context. The Angular <ins>module</ins> that owns the component that is calling the selector / directive must contain the selectors component class in the module's `declarations` array, OR import the module which references the selector.

- To specify a custom prefix for selectors when creating a new app:
```console
ng new project -name --prefix myprefix
```

- <ins>Interpolation</ins>: one <ins>binding</ins> method that Angular provides. It is one-way from the class to the html:
```html
<h1>{{ pageTitle }}</h1>
<!-- or -->
<h1>{{ 1 + 2 }}</h1>
```

- <ins>Template Expression</ins>: The value between the curly braces when using <ins>interpolation</ins> (e.g. "pageTitle" from the last example).

- <ins>Directive</ins>: Custom html <ins>element</ins> or <ins>attribute</ins>. These can be created by you, but Angular provides built-in directives as well.

- <ins>Structural Directive</ins>: Modifies the structure of an html layout (e.g. `*ngIf` and `*ngFor`). The asterisk marks the directive as a structural directive. Note: `*ngIf` and `*ngFor` are referenced in `BrowserModule`, which is automatically pulled into `AppModule`.

- <ins>Property Binding</ins>: Enclosing an attribute in square brackets and referencing the <ins>template expression</ins> in quotes, e.g.:
```html
<img [src]='product.imageUrl'>
```
Note the difference to <ins>interpolation</ins>:
```html
<img src={{ product.imageUrl }}>
```
Note: <ins>Interpolation</ins> always assigns a string while <ins>property binding</ins> returns the actual property type. <ins>Interpolation</ins> is still needed for things such as inlined concatenation.

Note: Property binding, like <ins>interpolation</ins>, is one-way data binding from the component to the DOM.

- <ins>Event Binding</ins>: Enclose event name in parentheses, then reference the method to call when that event occurs on the html element it has been added to, e.g.:
```html
<button (click)='doSomething()'>
```
A list of possible events can be found [here.](https://developer.mozilla.org/en-US/docs/Web/Events)

Note: `(click)` is not the same as `onclick`. The former is an <ins>event listener</ins>, while the ladder is an <ins>event handler content attribute</ins>. Using event listeners is recommended.

- Two-way binding: Consider the following:
```html
<input type='text' [(ngModel)]='foo'/>
```
This combines multiple things learned so far:

`[]` --> <ins>property binding</ins>

`()` --> <ins>event binding</ins>

`ngModel` --> Built in Angular <ins>directive</ins> (recall that a directive can either be an html element or an html attribute)

- `ngModel`: Used for template two-way data binding (not reactive forms). It is apart of `FormsModule` (as opposed to `BrowserModule`), and needs to be added to the `imports` list of the component's parent module.

- Pipes: Transforms properties just before displaying them. Pipes include: date, number, decimal, percent, currency, json, etc. e.g.:
 ```html
{{ myTitle | lowercase }}
```

Pipes  can be chained:
```html
{{ price | currency | lowercase }}
```
In the above example, `currency` optionally adds "USD" to the end, then lowercase changes that to "usd".

Pipe parameters are colon delimited, e.g.:
```html
{{ price | currency:'USD':'symbol':'1.2-2' }}
```
`1-2.2` explained:

first digit is min digits before decimal

second digit is min digits after decimal

third digit is max digits after decimal

- Component <ins>Lifecycle Hooks</ins>:

`ngOnInit()` --> component init

`ngOnChanges()` --> perform action after a change to <ins>input properties</ins>

`ngOnDestroy()` --> perform cleanup

To use a <ins>lifecycle hook</ins> your component must implement the corresponding hook's interface, e.g.:
```ts
export class myComponent implements OnInit { ... }
```

- To pass data from an outer component to its nested component:

In the child component, add the `@Input` <ins>decorator</ins> to the variable that will be passed in from the parent:
```ts
//...
selector: 'example-child'
//...
export class ChildComponent {
    @Input foo: number;
}
```

Then, in the parent html, use <ins>property binding</ins> to send the data to the child:
```html
<example-child [foo]='bar'></example-child>
```

Note: any time the `foo` variable is updated the `ngOnChanges()` <ins>lifecycle hook</ins> is called and can be defined in the child component.

- To pass data from a nested component to its outer component: The only way to pass data out is to emit an event. This can be achieved with an `EventEmitter` object and the `@Output` <ins>decorator</ins>.

In the child typescript:
```ts
export class ChildComponent {
    @Output foo: EventEmitter<string> = new EventEmitter<string>();

    onClick() {
        this.foo.event('clicked');
    }
}
```

In the child html:
```html
<button (click)='onClick()'>
```

In the parent typescript:
```ts
export class ParentComponent {
    onFoo(message: string): void { ... }
}
```

In the parent html:
```html
<example-child (foo)='onFoo($event)'></example-child>
```

- <ins>Service</ins>: A class used for features that are independent from any particular component, share data/logic across components, or does external work like data access, e.i. loggers, database access, etc.
  - Services are accessed with dependency injection in a components constructor, similar to services in C#.
  - Services use the `@Injectable` <ins>decorator</ins>.

- Registering a Service: You can register a service globally with the root injector (most common), or locally to a component and its nested components with the component injector. The latter allows for multiple service instances.

Example root injector:
```ts
@Injectable({
    providedIn: 'root'
})
export class ExampleService { ... }
```

Example Component injector:
```ts
@Component({
    templateUrl: ...
    providers: [exampleService]
})
export class exampleComponent { ... }
```

Both root/component dependency injection:
```ts
export class MyComponent {
    private _exampleService;

    constructor(exampleService: ExampleService) {
        this._exampleService = exampleService;
    }
}
```

Note: there is a shorter way to implement dependency injection that will automatically assign the service to a private local variable:
```ts
constructor(private exampleService: ExampleService) { }
```

- Observable pipes allow for sequential data transformation, much like pipes used in <ins>interpolation</ins>:
```ts
source$.pipe(
    map(x => x*3),
    filter(x => x % 2 === 0)
).subscribe(x => console.log(x));
```
Note: Ending observables with `$` is convention.

Note: `map()` transforms data and `filter()` filters values that meet a condition. `map()` and `filter()` are rxjs pipe <ins>operators</ins>. Other useful operators include `tap()` and `catchError()`.

- http requests:
  - Add `HttpClientModule` to the `AppModule` imports list.
  - Create a service object. Dependency inject an `HttpClient` into the constructor:
```ts
constructor(private http: HttpClient) { }
```

GET request:
```ts
this.http.get<IProduct[]>(this.apiUrl);
```
Note: The above will return an `Observable<IProduct[]>`. The return from the API call is automatically deserialized to an `IProduct[]`.

- The following automatically converts an object into JSON:
```ts
JSON.stringify(myObj);
```

- The `!` suffix after a variable name tells the compiler that the initial value will be set at a later time, e.g.:
```ts
sub!: Subscription;
ngOnInit() {
    this.sub = ...
}
```

---
## Routing
- Routing: a route is configured for each component that wants to display as a webpage.
- Routes are tied to options/actions the user takes to navigate to a separate page. Activating a route then displays that components view.
- Tie a route to a link:
```html
<a routerLink="/products">Products</a>
```
This will navigate to `mywebsite.com/products`.

Note: this is an html5 style url, and requires you configure your web server for url rewriting.

- When the url changes, Angular router looks for a route matching the path segment (`products` in this case), e.g.:
```ts
{ path: 'products', component: MyComponent }
```

The template is then displayed wherever the router's directive is defined:
```html
<router-outlet></router-outlet>
```

- An Angular application has one router.
  - In AppModule, add `RouterModule` to imports. Then, configure the routes in `AppModule` in the `RouterModule.forRoot()` method:
```ts
@NgModule({
    imports: [
        ...
        RouterModule.forRoot([
            { path: 'products', component: ... },
            { path: ... , component: ... }
        ])
    ]
    ...
})
```

- path options/examples:

Hard path (e.g. `foo.com/products`):
```ts
path: 'products'
```

Hard path with parameters (e.g. `foo.com/products/5`):
```ts
path: 'products/:id'
```

Default route for `foo.com`:
```ts
{ path: '', redirectTo: 'products', pathMatch: 'full' }
```

Wildcard path--the component to go to if an undefined url is entered (e.g. `foo.com/fail`):
```ts
path: '**'
```

Note: Order of paths matters! Therefore `'products'` should appear before `'products/:id'` and everything should appear before `'**'`.

- index.html will need a `base` tag:
```html
<head>
    ...
    <base href="/">
</head>
```

- Tie route to navbar link:
```html
<ul class='nav navbar-nav'>
    <li><a [routerLink]="['/welcome']">Home</a></li>
    <li><a [routerLink]="['/products']">Products</a></li>
</ul>
```
Note: `['/welcome']` and `['/products']` are actually lists whose first parameter is the path. You can omit the brackets if the array only contains one argument.

- Use `ActivatedRoute` to get parameter values from the url:
```ts
constructor(private route: ActivatedRoute) { }

ngOnInit(): void {
    const id = Number(this.route.snapshot.paramMap.get('id'));
}
```
Note: The argument passed into `get()` must be the same as is defined in the `RouterModule` definition. 

- Routing can also be called programmatically:
```ts
constructor(private router: Router) { }

onBack(): void {
    this.router.navigate(['/products']);
}
```

- Route guards can restrict access to only certain users, require confirmation before navigating away, and more. Some guard examples: `CanActivate`, `CanDeactivate`, `Resolve`, `CanLoad`.

- A guard is a class:
```ts
@Injectable({providedIn: 'root'})
export class exampleGuard implements CanActivate {
    canActivate(): boolean { ... }
}
```

The guard then needs to be added to the `RouterModule` config in `.forRoot()`:
```ts
RouterModule.forRoot([
    ...
    {
        path: 'products/id',
        canActivate: [exampleGuard],
        component: fooComponent
    }
])
```

---
## Modules
- `bootstrap[]` defines the entry component to the program. The `bootstrap[]` array should only ever be declared once, specifically in `AppModule`. 

- A component should only ever be declared in one module. If another module needs that component, then that component will have to be exported in the `exports[]` array.

- You can chain exports between modules. A module can export a component that is not in its `declarations` or `imports` arrays.

- To create a new module:
```console
ng g m path/moduleName --flat -m app
```
`--flat`: use if folder exists.

`-m app`: imports automatically to `AppModule`.

- In a feature model (not the root module), use `.forChild()` instead of `.forRoot()`.

- Create a module `SharedModule` for functionality that is shared across multiple features.

---
## Angular CLI

- CLI Syntax:
```console
ng <command> <args> --<options>
```

- Install Angular globally:
```console
npm install -g @angular/cli
```
The global install is the default, but calling from individual project folders will reference the local project version.

- Help:
```console
ng help
```

- Update (Updates your Angular version and its dependencies):
```console
ng update
```

- Check Angular CLI version:
```console
ng v
```

- New project:
```console
ng new proj-name --prefix myPrefix
```
This creates a folder in the calling directory with the same name as the project name.

- Run application:
```console
ng serve
```
Compiles project and spins up a web server on port 4200 by default.

The `-o` flag auto opens your default browser.

- Generate:
```console
ng g <schematicType> <name>
```
Note: A <ins>schematic</ins> is an individual piece of an Angular project, i.e. components, directives, route guards, interfaces, modules, pipes, services.
By default, generating components will create a new folder with all the newly generated files. Use the `--flat` flag to prevent this.

- Unit testing:
```console
ng test
```
This builds, runs the browser, then runs the karma test runner. The tests immediately run as files are changed.

Note: Closing the browser won't work, you'll have to terminate the test runner in the console with `Ctrl+C`.

- Build:
```console
ng build
```
The artifacts are placed in the `dist` folder.

The build automatically minifies the files and performs "tree shaking" (cleaning out unused/unneeded code).

Each build adds a new hash to the file names so that user browsers don't cache old files when new code is deployed.
