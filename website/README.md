I spent the first few years of my career regarding `this` as yet another aspect of my primarily hunch-driven approach to web development—which is to say, sometimes it *felt* like `this` *might* be what I wanted, but if so, it was for reasons I couldn't possibly begin to understand. I was occasionally right, to my very slim credit, but not because I knew *why* or anything. Now, after years of muddling through on vibes, trial and error, and squinting at Stack Overflow for as long as impending deadlines would allow, I think I've gotten a pretty good handle on `this`. I mean, give or take an emotional support `console.log` every so often; I'm only human.

For me, the trouble was that `this` is contextual, but that context isn't always meaningful to us developers so much as it's meaningful to *JavaScript*. We're used to telling JavaScript what things are: this is the identifier we typed, where we typed it, scoped to the pair of brackets we typed around it, with the value we typed. That's not the case with `this`, which is—brace for it—**a keyword that refers to the object bound to the function where `this` is invoked at the time when that function is called.** 

That is an absolute nightmare of a sentence, I know, but bear with me. It breaks down to two important concepts: `this` references **the object bound to a function**, and the object referenced by `this` is determined **at the time a function is called**.  Once you've got a handle on those two aspects, `this` is— well, still a little weird, but the *knowable* kind of weird.

At a high level, "the object bound to a function" makes sense when you frame it in terms of the essential nature of JavaScript: objects with methods referencing objects with methods, all the way down. Almost any code we execute anywhere in a script relates to an object in one way or another, either explicitly or implicitly. I wouldn't say that's an *uncomplicated* topic, strictly speaking, but we'll get to that in a bit.

For my money, "at the time a function is called" is what makes `this` particularly tricky to get a handle on: the context that determines the value of this isn't the writing, but the *calling* of a function—meaning that the value of `this` inside a function could be different every time that function is called. It requires us to think more like JavaScript executing a function than the person at the keyboard writing a function.

## Execution Contexts and the Call Stack

To better understand how JavaScript "thinks," we need to better understand JavaScript's execution model and the nature of the *call stack*—the "first in, last out" data structure that a JavaScript interpreter uses to execute code.

Again, you and I are used to thinking about the JavaScript we write in terms of *the JavaScript we're writing*. We're calling the shots; when, where, and how we declare a variable and assign it a value communicates its relevancy to JavaScript.

```jsx
function theFunction() {
    const theVariable = true;
};
```

When this code is encountered, a JavaScript engine creates a *lexical environment* for `theFunction`—a data structure that represents the code as written.

All the context needed to execute that function—our variable, in this example—is added to the *environment record* for the function. If you've ever wondered how JavaScript can be aware of variables *prior* to declaring them (but gives you a hard time about it), those environmental records are why:

```jsx
function theFunction() {
    console.log( theVariable );

    let theVariable;
}
theFunction();
// result: Uncaught ReferenceError: can't access lexical declaration 'hoistedVariable' before initialization
```

The environmental record created for this function's execution context contains a `theVariable` property, because it's right there in the lexical environment for the function; JavaScript is *aware* of our variable when it executes the function, but we're doing something pretty weird with it, so JavaScript throws an error to save us from ourselves. That environmental record-based "awareness" is how variable hoisting works, but *that* is a topic for another time.

After making lexical sense of the function and creating the environmental record, the JavaScript engine creates an *execution context* (sometimes called a "frame") for our function. That execution context includes the function and everything required to execute it.

You can think of the call stack as a sort of playlist made up of these function execution contexts: it determines what code is executed and when. This probably goes without saying, but there's a *lot* going on in the call stack—if you're interested in learning more about how it works and how it impacts the code we write, well, [do I ever have a course for you](https://piccalil.li/javascript-for-everyone). 

For now, and purposes of understanding `this`, here's the short and synchronous version:

When a script is executed, the JavaScript interpreter creates a "global execution context" and pushes it to the call stack—the script and all the environmental factors required to execute it, like variables and function declarations. Any statements inside that global context are then executed one at a time, from top to bottom. When the interpreter encounters a function call within the global execution context, it creates a *function* execution context for that call at the top of the stack, complete with anything required to run that function, and immediately executes it. If that function execution context contains a function call, a function execution context for *that* call is added to the top of the stack and executed immediately. Once a function execution context concludes, it gets removed from the stack, and JavaScript continues executing the function that called it; that concludes, gets removed, and the function that called it continues, winding all the way back down to the global execution context eventually. Once that concludes, well, the script is finished. First in, last out—execution contexts all the way down.

A number of things happen when a JavaScript interpreter creates an execution context, one of which is setting a value for the keyword `this` within the scope of that execution context. Except in one specific case, this doesn't depend on the *lexical* environment—we don't dictate the the value for `this` within the scope of the execution context for the function where we reference it. The value referenced by `this` within the execution context depends on how that function was *called*: whether a function is called as a standalone function, a method, or a constructor can change that value.

## `this` Binding

So, the "determined *at the time a function is called*" part of the equation: check. Not the most intuitive thing in the world, but the "when" makes a kind of sense, deep down in the JavaScript machinery. For us to start putting `this` to use, though, we need to understand the *what*: what value is bound to `this`?

### Global Binding (formerly "default binding")

Any code outside of a defined function lives in the global execution context. Within the scope of the global execution context, `this` references `globalThis`—a property that itself references a JavaScript environment's global object. In a document in your browser, the global object is the `window` object:

```jsx
globalThis;
// result: Window {...}
```

Now, remember that the value of `this` is a reference to the object bound to a function call. `this` referencing the global object—`window`, for our purposes—makes sense in terms of how JavaScript has historically handled global scope: when you create a function using the `function` constructor, that function is created in the global scope and made available as a method on the `window` object:

```jsx
function theFunction() { }

theFunction;
// result: theFunction()

window.theFunction;
// result: function theFunction()
```

Given that behavior, the value of `this` within the execution context created for `myFunction` is `window`—the implied object bound to that function at the time the function is called.

```jsx
function theFunction() {
    console.log( this );
}

theFunction();
// result: Window { … }
```

I know; a little weird. For what it's worth, you're not likely to run into this particular use of `this` very often, and you're even less likely to have occasion to use it. We don't tend to need a dedicated reference to `window` when we can access `window` directly, and while it makes sense in terms of the anatomy of JavaScript, there aren't many use cases for "act on the object bound to this function, even if that means messing with `window`"—that sounds dangerous. Still, altering the rules of JavaScript as-it-is-played would potentially break untold numbers of websites out in the wild, so changing this behavior had to be [opt-in via strict mode](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Strict_mode).

In that strict mode context, this behavior is changed to something a little more predictable: when we call a function by identifier within the global scope, `this` gets the value `undefined`, full stop:

```jsx
function theFunction() {
    "use strict";
    console.log( this );
}

theFunction();
// result: undefined
```

"`this` references an object explicitly or implicitly bound to this function, and `undefined` otherwise" makes for much more predictable conditional logic around acting on the object bound to a function.

That function is still available as a method on `window`, naturally—the nature of JavaScript and all. So, what happens if we explicitly invoke it as one?

```jsx
function theFunction() {
    "use strict";
    console.log( this );
}

theFunction();
// result: undefined

window.theFunction();
// result: Window { … }
```

Ah-hah. In this case, we're calling the function as a method of `window`, so `window` is the object bound to our function *at the time the function is executed*, so `this` is a reference to `window`. Through `this`, we then get access to all the parent properties and methods of the object it references:

```jsx
function theFunction() {
    "use strict";
    if ( this !== undefined ) {
        console.log( this.getComputedStyle  );
    } else {
        return this;
    }
}

theFunction();
// result: undefined

window.theFunction();
// result: function getComputedStyle()
```

Is this useful, as seen here? Well, no, not especially; we don't need to stop for `this` when we already have `window` at home. In fact, I’d caution against deliberately using `this` to refer to `globalThis`—better to use `window`, `self` , or even `globalThis` itself, which will give you exactly what you need, independent of surrounding context. 

For our purposes, this snippet of code is *illustrative*, even if not copy-and-paste *useful*: this is a perfect example of "implicit binding," and that’s something you’ll definitely end up using.

### Implicit binding

As you just saw, when a function is called as an object's method, the value of `this` within the context of that function's execution is a reference to to the object that contains the method. That gives you access to all the methods and properties that sit alongside the method.

```jsx
const theObject = {
    theString: "This is a string!",
    theMethod() {
        console.log( this.theString );
    }
};

theObject.theMethod();
// result: This is the string!
```

The `theObject` object is calling the `theMethod` method, so within the function execution context that gets created, the value for `this` is a reference to `theObject` (and I bet you thought "`this` is a keyword that refers to the value of the object bound to the function where `this` is invoked at the time when the function is called" was the worst sentence you would read in this post).

This is the heart of the confusion around `this` , so far as I’m concerned: even knowing all this, that snippet above probably still reads a lot like “`this` in the method refers to the object because, well, look at it—the method is right there *in that object*. However:

```jsx
const theObject = {
    theString: "This is a string!",
    theMethod() {
        console.log( this.theString );
    }
};
const theFunctionIdentifier = theObject.theMethod;

theObject.theMethod();
// result: This is a string!

theFunctionIdentifier();
// result: undefined
```

That’s the same method, on the same object—but if we invoke it a little differently, the value of `this` inside `theMethod` changes:

```jsx
const theObject = {
    theMethod() {
        console.log( this === globalThis );
    }
};
const theFunctionIdentifier = theObject.theMethod;

theObject.theMethod();
// result: false

theFunctionIdentifier();
// result: true
```

“*At the time the function is called*.”

At this point, I have some good news and some bad news. Tthe good news is that, with what you now know of implicit binding, you’re covered for the vast majority of use cases for `this`. The bad news, of course, is that there's an exception: arrow functions.

### `this` in arrow functions

Arrow functions don't have `this` bindings of their own. Instead, the value of `this` in an arrow function resolves to a binding in a "lexically enclosing environment." That's right: *lexical* environment—as in, where it sits in the code we've written—not the *execution context*. Inside an arrow function, `this` works very differently: it refers to the value of `this` in that function's closest enclosing context *as written*:

```jsx
const theObject = {
    theMethod() { console.log( this ); },
    theArrowFunction: () => console.log( this )
};

myObject.theMethod();
// result: Object { myMethod: myMethod(), myArrowFunction: myArrowFunction() }

myObject.theArrowFunction();
// result: Window {...}
```

Here, `myObject.theMethod()` is classic implicit binding. Just like the earlier examples: when `theMethod` is called and a function execution context is created for it, the value of `this` is set to a reference to the object containing the method.

That isn't the case when we call `myObject.theArrowFunction()`, however: `this` inherits the value of `this` from the *lexically* enclosing environment—the value of `globalThis`. Because we're outside of strict mode, `this` ends up being a reference to `window`—if we were in strict mode, it would be `undefined` :

```jsx
const theObject = {
    theInnerMethod() {
        this.theArrowFunction()
    },
    theArrowFunction: () => console.log( this )
};

myObject.theMethod();
// result: Object { myMethod: myMethod(), myArrowFunction: myArrowFunction() }

myObject.theArrowFunction();
// result: Window {...}
```

Arrow functions are exceptionally useful in a lot of ways, and not having their own `this` bindings *can* be one of them if used very carefully. For this reason (among others), it's generally considered a good practice to avoid using arrow functions as a method—a [function defined as a property of an object](https://developer.mozilla.org/en-US/docs/Glossary/Method)—unless you can make a strong case for it.

### Explicit binding

Implicit binding handles most use cases for working with `this`, by a huge margin. However, in the rare event you need `this` to represent a specific execution context instead of the implicit context, invoking a function using the `call()` or `apply()` methods will allow you to specify the value referenced by `this` to the object provided as an argument.

```jsx
function theFunction() {
    "use strict";
    console.log( this );
}

const theObject = {
    theValue: "This is a string!"
};

theFunction();
// result: undefined

theFunction.call( theObject );
// result: Object { theValue: "This is a string!" }
```

Just like it says on the tin, the`bind()` method can net you the same end result when it comes to `this`, though the usage is a little different. When you invoke `bind()` on a function, a new, wrapped version of that function is created. The function created using `bind()` is usually referred to as a "bound function," predictably enough.

Calling that bound function then executes the inner function with a value for `this` that references the object specified as an argument. 

```jsx
function theFunction() {
    "use strict";
    console.log( this );
  }
const theObject = {
    theValue: "This is a string!"
};

const boundFunction = theFunction.bind( theObject );

theFunction();
// result: undefined

boundFunction();
// result: Object { theValue: "This is a string!" }
```

When we call `theFunction`, `this` is a reference to `globalThis`—because we're in strict mode, that means `this` is `undefined`. However, when we invoke that function using `call()` (or use `bind()` to create a bound function) with `theObject` as an argument, `this` contains a reference to that object instead. The explicit binding overrides the implicit binding:

```jsx
const theObject = {
    theValue : "This string sits alongside myMethod.",
    theMethod() {
        console.log( this.theValue );
    }
};
const someOtherObject = {
    theValue : "This is a string in another object entirely!",
};

theObject.theMethod();
// result: This string sits alongside myMethod.

theObject.theMethod.call( someOtherObject );
// This is a string in another object entirely!
```

We're veering a little academic, but explicit binding does come with one interesting complication: like so many dogs playing so much basketball, there's no rule that says you *can't* bind something other than an object—a [primitive](https://developer.mozilla.org/en-US/docs/Glossary/Primitive), for example.

```jsx
function theFunction() {
    "use strict";
    console.log( this );
}

theFunction.call( "How'd this string get here?" );
// result: How'd this string get here?
```

Now, needing to use explicit binding in the first place is rare, and I can't imagine a case where you'd do so with a non-object *on purpose*, but you just never know with JavaScript—so, just in case, there are some unique behaviors worth knowing.

A passed `this` value isn't coerced in strict mode—`this` just takes on that value:

```jsx
"use strict";
function theFunction(){
    "use strict";
    console.log( typeof this, this );
}

theFunction.call( 7 );
// result: number 7

theFunction.call( "A string." );
// result: string A string.

theFunction.call( undefined );
// result: undefined undefined

theFunction.call( null );
// result: object null
// Believe it or not, `typeof null` is `object`! Check out <https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/typeof#typeof_null>
```

As is often the case, things get a little weirder outside of strict mode. If a function is called in a way that would provide `this` with a value of either `undefined` or `null`, that value is replaced by `globalThis`.

```jsx
function theFunction() {
    console.log( this );
}

theFunction.call( null );
// result: Window {...}
```

If a function is called in a way that would provide `this` with a primitive value, that value is substituted with the primitive value's wrapper object outside strict mode:

```jsx
function theFunction() {
    console.log( this );
}
theFunction.call( 10 );
// result: Number { 10 }
```

You may very well be asking "in what way would that be useful?" You would be right to wonder this! I tell you, I have for *years* now. The thing is, the nature of the web is such that we can't know whether someone out there had some wildly obscure use case (or made some mistake) where the JavaScript running on their website relies on this obscure behavior, and so the nature of the standards that govern JavaScript is that they can't be *changed* without being opt-in. So outside of strict mode, this behavior persists to this day. If over the course of your travels in JavaScript you’ve ever run into this behavior buried in a legacy codebase somewhere, [I](https://www.notion.so/JavaScript-what-is-this-1d51c83839de80d6877ff8bf1fa6969f?pvs=21) absolutely want to hear about it in the [JavaScript for Everyone](https://piccalil.li/javascript-for-everyone) Discord.

All told, explicit binding is part of the language, and you shouldn’t hesitate use it when and where you need it. That said, when writing something brand new, I try to avoid *depending* on explicit binding for anything load-bearing; it breaks with the contextual usefulness of `this` , and turns it into an argument that wants desperately to be a little too clever for its own good.

### `new` Binding

When a class is used as a constructor by way of the `new` keyword, `this` refers to the newly-created object:

```jsx
class TheClass {
    theString;
    constructor() {
        this.theString = "A string.";
    }
    logThis() {
        console.log( this );
    }
}
const thisClass = new TheClass();

thisClass.logThis();
// result: Object { theString: "A string.", ... }
```

Nothing too surprising right out of the gate—when `logThis` is invoked this way, the value of `this` reflects the clear relationship between the class and the objects it creates. But remember, the `value` of this is set at the time the `logThis` method is executed, and that means that value can be overridden by the way a method is invoked *within* a class:

```jsx
class TheClass {
    successMsg = "You clicked the button!";
    constructor( theElement ) {
        theElement.addEventListener('click', this.logThis );
        // result: <button>
    }
    logThis() {
        console.log( this );
    };
}
const theButton = document.querySelector( "button" )
const theConstructedObject = new TheClass( theButton );
```

Imagine we wanted to do something with the instance property containing our `You clicked the button!` string in response to a button press. Well, spoilers for a little later on in the course, but `this` takes on a different value in the context of an event listener—because of the execution scope of the `logThis` method, `this` doesn’t have the value we’re looking for.

Well, there's an argument to be made that this is the exact use case for arrow functions inheriting their `this` value from lexical scope rather than execution scope. If we write the `logThis` method as an arrow function, the event handler execution scope doesn’t matter anymore—and the lexical scope for our method is the class instance:

```jsx
class TheClass {
    successMsg = "You clicked the button!";
    constructor( theElement ) {
        theElement.addEventListener('click', this.logThis );
        // result: Object { successMsg: "You clicked the button!", logThis: logThis() }
    }
    logThis = () => {
        console.log( this );
    };
}
const theButton = document.querySelector( "button" )
const theConstructedObject = new TheClass( theButton );
```

As with classes, when a function is called with `new`, `this` within that function is an instance of that constructor function:

```jsx
function TheConstructorFunction() {
  this.theString = "A string.";
  this.logThis = function() {
    console.log( this );
  }
}
const theConstructedObject = new TheConstructorFunction();

theConstructedObject.logThis();
// result: Object { theString: "A string.", ... }
```

Nothing too surprising there—there's a pretty clear relationship between those objects and functions. There's *one* little thing to keep in mind when you're working with `this` and constructor functions, however: in a well-written constructor function like the example above, `this` will always reference the instance of that constructor function:

```jsx
function TheConstructorFunction() {
  this.logThis = function() {
    console.log( this.constructor.name );
  }
}
const theConstructedObject = new TheConstructorFunction();

theConstructedObject.logThis();
// result: TheConstructorFunction
```

If that constructor function were written using a `return`—not a good practice, but I’ve certainly seen it done—the object that is explicitly returned might not be an instance of the constructor function:

```jsx
function TheConstructorFunction() {
  return {
    logThis: function() {
      console.log( this.constructor.name );
    }
  };
}
const theConstructedObject = new TheConstructorFunction();

theConstructedObject.logThis();
// result: Object
```

We’re veering a little academic again, but that doesn't mean you *couldn't* make use of `this` in the example above, using our terse but surprisingly complicated friend the arrow function.

```jsx
function TheConstructorFunction() {
  return {
    logThis: function() {
      console.log( this.constructor.name );
    },
    logThat: () => console.log( this.constructor.name ),
  };
}

const thing = new TheConstructorFunction();

thing.logThis();
// result: Object

thing.logThat();
// result: TheConstructorFunction
```

In this example, `logThat` inherits a `this` value from the *lexical* scope of `TheConstructorFunction`.

For those of you keeping score, though: that's us using an arrow function as a method, which should usually be avoided, to address the failing of a constructor function that returns an object, which should definitely be avoided—we're two layers deep in "questionable practice" territory. Still, I know better than anyone that we don't always have full control over the codebases we inherit; it's worth knowing the edge cases, just in case you find yourself faced with one someday.

### Event handler binding

Ah, you never forget your first `this`. An old favorite. A *classic*.

Inside an event handler's callback function, `this` references the element associated with the handler. That's it!

```jsx
document.querySelector( "button" ).addEventListener( "click", function( event ) {
    console.log( this );
    // result: <button class="btn">
});
```

When a user fires this click event on the `button` element, the resulting value of `this` is a reference to the element object for the `<button>` itself.

Explicit binding still applies, of course, and overrides the default behavior: when you use `bind()` to create a bound function for use as a callback, `this` will reference the object you specify:

```jsx
const button = document.querySelector( "button" );
const theObject = {
    "theValue" : true
};
function handleClick() {
    console.log( this );
}
button.addEventListener( "click", handleClick.bind( theObject ) );
// result: Object { theValue: true }
```

Just like everywhere else, when an arrow function is used as an event listener's callback, the value of `this` is provided by the closest enclosing *lexical* environment. For example, at the top level, `this` inside an event handler callback *arrow* function will reference `globalThis`:

```jsx
let button = document.querySelector( "button" );

button.addEventListener( "click", ( event ) => { console.log( this ); } );
// result: Window { … }
```

Now, again: that's not to say you *can't* use an arrow function as an event handler's callback, but you should only reach for one in a situation where you want to rely on lexical scope to determine the value of `this`, like an event handler inside a method attached to an object that you’ll need to reference from your event handler’s callback function. That’s a pretty specific use case when laid out that way, but better to know it and not need it, than to need it but not know it.