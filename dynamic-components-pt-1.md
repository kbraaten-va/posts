## Part One: Programatically Creating A Component

You can follow along by modifying the code [here](http://embed.plnkr.co/iwIyVbaBuuF5ObQetsaN/), as well as view the finished code [here](http://embed.plnkr.co/eRx5vyf1eAvGlV0u82Tp/).

In order to programatically create a component, we require three things:
1. ViewContainerRef
2. ComponentFactory
3. entryComponent

Let's walk through these in detail...

### 1. ViewContainerRef

We begin with a basic component in `src/app.ts`:
```
@Component({
  selector: 'my-app',
  template: `
  <h2>Hello World</h2>
  `,
})
export class App {}
```

Let's start by adding a `div` onto which we will be able to programatically create our component.

```
import {..., ViewChild} from '@angular/core'
@Component({
  selector: 'my-app',
  template: `
  <h2>Hello World</h2>
  <div #container></div>
  `
})
export class App implements AfterContentInit {
  @ViewChild('container') container; // 1
  
  ngAfterContentInit() { // 2
    console.log(this.container);
  }
}
```

1. [`ViewChild`](https://angular.io/docs/ts/latest/api/core/index/ViewChild-decorator.html) allows us to get the first element matching the selector from the view DOM, as referenced in our template by `#container`.

2. The `AfterContentInit` lifecycle hook is called after our content has been rendered, which means our `ViewChild` will be set.

If we open up DevTools we will see that `container` is currently of type `ElementRef`, this limits us to native-type interactions. As alluded to, what we want is a `ViewContainerRef`.

Luckily, `ViewChild` makes this easy for us to retrieve. `ViewChild` has a property `read` that allows us to get a different token from the queried element, in our case, `ViewContainerRef`. To make use of this, we simply change our `@ViewChild` declaration:

`@ViewChild('container', {read:ViewContainerRef}) container;`

Now in DevTools we will see our container is a `ViewContainerRef` ☺️
![alt text](http://i.imgur.com/RE8Bwvr.png?1)

[`ViewContainerRef` provides a series of methods](https://angular.io/docs/ts/latest/api/core/index/ViewContainerRef-class.html), but the one we are interested in is `createComponent`. As you can see in the signature, in order to call this method, we need to pass in a `ComponentFactory`. Which leads us to our second requirement...

### 2. ComponentFactory

In order to create a `ComponentFactory`, we will make use of the [`ComponentFactoryResolver`](https://angular.io/docs/ts/latest/api/core/index/ComponentFactoryResolver-class.html) service. I wouldn't bother on clicking the link to the documentation, there isn't much there 😉. Basically, we'll just pass in the type of component we want it to resolve. Speaking of which, let's quickly create (and declare) a component for that...

```
@Component({
  template: `My name is Lucky`
})
export class GreetingComponent {}
//...
declarations: [ App, GreetingComponent ],
```

Then in our app component:
```
export class App implements AfterContentInit {
  @ViewChild('container', {read:ViewContainerRef}) container;
  
  constructor(private resolver: ComponentFactoryResolver) {} // 1
  
  ngAfterContentInit() {
    const greetingFactory = this.resolver.resolveComponentFactory(GreetingComponent); // 2
    this.container.createComponent(greetingFactory); // 3
  }
}
```
1. Inject the `ComponentFactoryResolver` into our component.
2. Pass our `GreetingComponent` type into the resolver to get the `ComponentFactory` for our `GreetingComponent`
3. Call `createComponent` and pass in our newly created `greetingFactory`

We are almost there, but this will raise an error, which leads us to our third requirement.

### 3. entryComponent

If we open DevTools we will see the folling error:

`Error: No component factory found for GreetingComponent. Did you add it to @NgModule.entryComponents?`

The answer is yes.

```
@NgModule({
  imports: [ BrowserModule ],
  declarations: [ App, GreetingComponent ],
  entryComponents: [ GreetingComponent ]
  bootstrap: [ App ]
})
export class AppModule {}
```

Tada, everything works! So what exactly does declaring our component in `entryComponents` do?

When compiling, Angular doesn't include components that aren't referenced by their `selector` in the templates. So even though our `GreetingComponent` was declared, Angular was saving on bundle size by ignoring it since it didn't think it was in use.

Declaring a component as an entry component essentially tells Angular, somewhere in our code we will be using this component so please include it for us. Angular will also create us `ComponentFactory` and store it in the `ComponentFactoryResolver` to allow us to programatically create them.

### Summary

* The `ViewContainerRef` creates a component from a `ComponentFactory`
* The `CompnentFactory` for a component is resolved by the `ComponentFactoryResolver`
* The component we are resolving for must be declared as an `entryComponent`

In Part 2, we will further explore this technique by seeing how we leverage programatic component creation from within a `directive` in `va-table` to create create different types of cells.