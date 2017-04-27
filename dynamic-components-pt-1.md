# Part One: Dynamically Creating An Angular Component

## Overview

In order to dynamically create a component, we require three things:
1. ViewContainerRef
2. ComponentFactory
3. entryComponent

We will access a `ViewContainerRef` and use it to create our component using a `ComponentFactory`. We get a component's `ComponentFactory` using the `ComponentFactoryResolver` service. We'll need to declare our component as an `entryComponent` so Angular knows to include it in our bundle.

What does all that mean? Let's walk through it in detail...

Note:
You can follow along by modifying the code [here](https://github.com/kbraaten-va/dynamic-comp-1/tree/seed)
You can view the finished code [here](http://embed.plnkr.co/eRx5vyf1eAvGlV0u82Tp/)
Edit the post [here](https://github.com/kbraaten-va/posts/blob/master/dynamic-components-pt-1.md)

## 1. ViewContainerRef

We begin with a basic component in `src/app.ts`:
```
@Component({
  selector: 'my-app',
  template: `
  <h2>Hello World</h2>
  `
})
export class App {}
```

Let's start by adding a `div` onto which we will be able to create our component.

```
import {..., ViewChild, AfterContentInit} from '@angular/core'
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

2. The [`AfterContentInit`](https://angular.io/docs/ts/latest/api/core/index/AfterContentInit-interface.html) lifecycle hook is called after our content has been rendered, which means our `ViewChild` will be set.

If we open up DevTools we will see that `container` is currently of type [`ElementRef`](https://angular.io/docs/ts/latest/api/core/index/ElementRef-class.html), this limits us to native-type interactions. As alluded to, what we want is a `ViewContainerRef`.

Luckily, `ViewChild` makes this easy for us to retrieve. `ViewChild` has a property `read` that allows us to get a different token from the queried element, in our case, `ViewContainerRef`. To make use of this, we simply change our `@ViewChild` declaration (don't forget to import `ViewContainerRef`):

`@ViewChild('container', {read:ViewContainerRef}) container;`

Now in DevTools we will see our container is a `ViewContainerRef` ☺️

![alt text](http://i.imgur.com/RE8Bwvr.png?1)

[`ViewContainerRef`](https://angular.io/docs/ts/latest/api/core/index/ViewContainerRef-class.html) provides a series of methods, but the one we are interested in is `createComponent`. As you can see in the signature, in order to call this method, we need to pass in a `ComponentFactory`. Which leads us to our second requirement...

## 2. ComponentFactory

In order to create a [`ComponentFactory`](https://angular.io/docs/ts/latest/api/core/index/ComponentFactory-class.html), we will make use of the [`ComponentFactoryResolver`](https://angular.io/docs/ts/latest/api/core/index/ComponentFactoryResolver-class.html) service. Basically, we'll just pass in the type of component we want it to resolve. Speaking of which, let's quickly create (and declare) a component for that...

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
import {..., AfterContentInit, ComponentFactoryResolver} from '@angular/core'
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

## 3. entryComponent

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

Declaring a component as an entry component essentially tells Angular, somewhere in our code we will be using this component so please include it for us. Angular will also create a `ComponentFactory` and store it in the `ComponentFactoryResolver` allowing us to dynamically create them.

## Summary

* The `ViewContainerRef` creates a component from a `ComponentFactory`
* The `CompnentFactory` for a component is resolved by the `ComponentFactoryResolver`
* The component we are resolving for must be declared as an `entryComponent`

In Part 2, we will further explore this technique by seeing how we leverage dynamic component creation from within a `directive` in `va-table` to dynamically create different types of cells.

## References
1. https://angular.io/docs/ts/latest/api/core/index/ViewChild-decorator.html
2. https://angular.io/docs/ts/latest/api/core/index/AfterContentInit-interface.html
3. https://angular.io/docs/ts/latest/api/core/index/ElementRef-class.html
4. https://angular.io/docs/ts/latest/api/core/index/ViewContainerRef-class.html
5. https://angular.io/docs/ts/latest/api/core/index/ComponentFactory-class.html
6. https://angular.io/docs/ts/latest/api/core/index/ComponentFactoryResolver-class.html
