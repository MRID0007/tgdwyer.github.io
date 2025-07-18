---
layout: chapter
title: "Functional Programming in JavaScript"
---

### Learning Outcomes

* Create programs in JavaScript in a functional style
* Understand the definitions of [function purity and referential transparency](#function-purity-and-referential-transparency)
* Explain the role of pure functional programming style in managing side effects
* See how pure functions can be used to [model sophisticated computation](#computation-with-pure-functions)

## Introduction

The elements of JavaScript covered in [our introduction](/javascript1), specifically:

* [Binding functions to variables](/javascript1#functions-are-objects)
* [Anonymous functions](/javascript1#anonymous-functions)
* [Higher-order functions](/higherorderfunctions)

are sufficient for us to explore a paradigm called *functional programming*.  In the functional programming paradigm, the primary model of computation is through the evaluation of functions.  Functional Programming is highly inspired by the [Lambda Calculus](/lambdacalculus/), a theory which develops a model for computation based on the application and evaluation of mathematical functions.

While JavaScript (and many---but not all, as we shall see---other languages inspired by the functional paradigm) do not enforce it, true functional programming mandates the functions be pure in the sense of not causing *side effects*.

## Side Effects

<div class="alert-box alert-info">
Side effects of a function are changes to state outside of the result explicitly returned by the function.
</div>
Examples of side effects from inside a function:

* changing the value of a variable declared outside the function scope
  * mutating global state in this way can cause difficult-to-diagnose bugs: for example, an effective debugging strategy is dividing the program up into little pieces which can easily be proven correct or unit tested---sources of side effects deep inside functions are a hidden form of coupling making such a strategy very difficult.
* printing to the console---changing the state of the world in such a way can also be dangerous
  * for example, filling a disk with log messages is a good way to crash the whole computer!

In languages without compilers that specifically guard against them, side effects can occur:

* intentionally through sloppy coding practices, where a misguided programmer may think it’s more convenient to have a function do multiple things at once;
* unintentionally, for example by accidentally setting a global variable instead of a local one.

We’ll see more examples in actual code below.

## Function Purity and Referential Transparency

<div class="alert-box alert-info" markdown="1">
A *pure function*:

* has no *side effects*: i.e. it has no effects other than to create a return value;
* always produces the same result for the same input.

</div>
In the context of functional programming, [*referential transparency*](https://soundcloud.com/tim-dwyer-17038309/referentialtransparency-lessboringmix) is:

* the property of being able to substitute an expression that evaluates to some value, with that value, without affecting the behaviour of the program.

### Trivial Example

Imagine a simple function:

```javascript
const square = x=>x*x
```

And some code that uses it:

```javascript
const four = square(2)
```

The expression `square(2)` evaluates to `4`.  Can we replace the expression `square(2)` with the value `4` in the program above without changing the behaviour of the program?  YES!  So is the expression `square(2)` referentially transparent? YES!

But what if the `square` function depends on some other data stored somewhere in memory and may return a different result if that data mutates? (An example might be using a global variable or reading an environment variable from IO or requiring user input.)  What if `square` performs some other action instead of simply returning the result of a mathematical computation?  What if it sends a message to the console or mutates a global variable?  That is, what if it has side effects (is not a pure function)?  In that case is replacing `square(2)` with the value `4` still going to leave our program behaving the same way?  Possibly not!

Put another way, if the `square` function is not pure---i.e. it produces side effects (meaning it has hidden output other than its return value) or it is affected by side effects from other code (meaning it has hidden inputs)---then the expression `square(2)` would not be referentially transparent.

### Realistic Examples

A real-life example where this might be useful would be when a cached result for a given input already exists and can be returned without further computation.  Imagine a pure function that computes π (pi) to a specified number of decimal points of precision.  It takes an argument `n`, the number of decimal points, and returns the computed value.  We could modify the function to instantly return a precomputed value for π from a lookup table if `n` is less than `10`.  This substitution is trivial and is guaranteed not to break our program, because its effects are strictly locally contained.

Pure functions and referential transparency are perhaps most easily illustrated with some examples and counterexamples.
Consider the following *impure* function:

```javascript
   function impureSquares(a) {
       let i = 0
       while (i < a.length) {
*         a[i] = a[i] * a[i++];
       }
   }
```

Since the function modifies `a` in place, we get a different outcome if we call it more than once.

```javascript
const myArray=[1,2,3]
impureSquares(myArray)
// now myArray = [1,4,9]
impureSquares(myArray)
// now myArray = [1,16,81]
```

Furthermore, the very imperative style computation in `impureSquares` at the line marked with `*` is not pure.
It has two effects: incrementing `i` and mutating `a`.
You could not simply replace the expression with the value computed by the expression and have the program work in the same way.
This piece of code does not have the property of *referential transparency*.

True pure functional languages (such as Haskell) enforce referential transparency through immutable variables (*note: yes, “immutable variable” sounds like an oxymoron---two words with opposite meanings put together*).  That is, once any variable in such a language is bound to a value, it cannot be reassigned.

In JavaScript we can opt in to immutable variables by declaring them `const`, but it is only a shallow immutability.  Thus, the variable `myArray` above cannot be reassigned to reference a different array.  However, we can change the contents of the array as shown above.

A more functional way to implement the `squares` function would be more like the examples we have seen previously:

```javascript
function squares(a) {
    return a.map(x=> x*x)
}
```

The above function is pure.  Its result is a new array containing the squares of the input array, and the input array itself is unchanged.  It has no side effects changing values of variables, memory or the world, outside of its own scope.  You could replace the computation of the result with a different calculation returning the same result for a given input and the program would be unchanged.  It is *referentially transparent*.

```javascript
const myArray = [1,2,3]
const mySquares = squares(myArray)
const myRaisedToTheFours = squares(mySquares)
```

We could make the following substitution in the last line with no unintended consequences:

```javascript
 const myRaisedToTheFours = squares(squares(myArray))
```

An impure function typical of something you may see in OO code:

```javascript
let messagesSent = 0;
function send(message, recipient) {
   let success = recipient.notify(message);
   if (success) {
       ++messagesSent;
   } else {
       console.log("send failed! " + message);
   }
   console.log("messages sent " + messagesSent);
}
```

This function is impure in three ways:

* it mutates the state of the count variable `messagesSent` from the enclosing scope
* it (likely) does something (what exactly is unclear) to `recipient`
* finally, it sends output to the console.  Outputting to a device (although only for display purposes) is definitely a side effect.

Side effects are bad for transparency (knowing everything about what a function is going to do) and maintainability.  When state in your program is being changed from all over the place bugs become very difficult to track down.

### Exercises

Do the following functions have side effects?

1. Yes/No and why?

    ```javascript
    let counter = 0;

    function incrementCounter() {
        counter++;
    }
    ```

2. Yes/No and why?

   ```javascript
   function greet(name) {
       return `Hello, ${name}!`;
   }
   ```

3. Yes/No and why?

   ```javascript
   function multiplyArray(numbers) {
       for (let i = 0; i < numbers.length; i++) {
           numbers[i] = numbers[i] * 2;
       }
       return numbers
   }
   ```

4. Yes/No and why?

   ```javascript
   function logMessage(message) {
       console.log(message);
   }
   ```

5. Yes/No and why?

   ```javascript
   function doubleNumbers(numbers) {
       return numbers.map(x => x * 2);
   }
   ```

6. Yes/No and why?

   ```javascript
   function getRandomNumber() {
       return Math.random();
   }
   ```

#### Solutions

1. Yes, it modifies the global variable counter.

2. No, it simply returns a new string and does not modify any external state.

3. Yes, it modifies the input array numbers in place.

4. Yes, it writes to the console, which is an external action.

5. No, it returns a new array without modifying the input array.

6. Yes, it relies on and modifies a global seed for random number generation.

## Functional Patterns

Passing functions around, anonymous or not, is incredibly useful and pops up in many practical programming situations.

### Eliminating Loops

Loops are the source of many bugs: fence-post errors, range errors, typos, incrementing the wrong counter, etc.

A typical for loop has four distinct places where it’s easy to make errors that can cause critical problems:

```javascript
for ([initialisation]; [condition]; [final-expression])
   statement
```

* The **initialisation** can initialise to the wrong value (e.g. n instead of n-1, 1 instead of 0) or initialise the wrong variable.
* The **condition** test can use = instead of ==, <= instead of < or test the wrong variable, etc.
* The **final-expression** can (again) increment the wrong variable
* The **statement** body might change the state of variables being tested in the termination condition since they are in scope.

For many standard loops, however, the logic is the same every time and can easily be abstracted into a function.  Examples: `Array.map`, `Array.reduce`, `Array.forEach`, etc.  The logic of the loop body is specified with a function which can execute in its own scope, without the risk of breaking the loop logic.

#### Examples

1. Consider this loop to multiply each item in the `someArray` by two. Try to find the mistake in this code:

    ```javascript
    const someArray = [1,2,3,4,5];
    let newArray = [];
    for (let i = 0; i <= someArray.length; i++) {
        newArray.push(someArray[i] * 2);
    }
    ```

    <p class="spoiler" markdown="1">The condition should be `<`, not `<=`.</p>

    To avoid the likelihood of errors, we can replace the `for` loop with the use of `.map`.

    We use the `map` function since we want to apply a function to every element in the list.

    ```javascript
    const someArray = [1,2,3,4,5];
    const newArray = someArray.map(x => x * 2);
    ```

    We use an arrow function here to allow our function definition to be short and to the point!

2. Consider this code which aims to compute the product of a list. Try to find the mistake in this code:

    ```javascript
    const someArray = [1,2,3,4,5];
    let result = 1;
    for (let i = 0; i < someArray.length; i++) {
        result *= i;
    }
    ```

    <p class="spoiler" markdown="1">We should multiply by `someArray[i]`, not `i`.</p>

    Again, to avoid the likelihood of errors, we can replace the `for` loop with the use of `.reduce`.

    We use the `reduce` function since we want to reduce the list to a singular value.

    ```javascript
    const someArray = [1,2,3,4,5];
    const result = someArray.reduce((acc, el) => el * acc, 1);
    ```

---

#### Exercises

1. Refactor this code to use `map` and `filter` instead of a loop:

    ```javascript
    const numbers = [2, 6, 3, 7, 10];
    const result = [];
    for (let i = 0; i < numbers.length; i++) {
        if (numbers[i] % 2 === 0) {
            result.push(numbers[i] / 2);
        }
    }
    ```

2. Refactor this code to remove the loop:

    ```javascript
    const words = ["apple", "banana", "cherry"];
    let totalLength = 0;
    for (let i = 0; i < words.length; i++) {
        totalLength += words[i].length;
    }
    ```

3. Refactor this code to remove the loop:

    ```javascript
    const people = [
        {name: "Alice", age: 20},
        {name: "Bob", age: 15},
        {name: "Charlie", age: 30},
        {name: "David", age: 10},
    ];
    let firstChildName = undefined;
    for (let i = 0; i < people.length; i++) {
        if (people[i].age < 18) {
            firstChildName = people[i].name;
            break;
        }
    }
    ```

4. Refactor this code to remove the loop:

    ```javascript
    const people = [
        {name: "Alice", age: 20},
        {name: "Bob", age: 15},
        {name: "Charlie", age: 30},
        {name: "David", age: 10},
    ];
    let result = "";
    for (let i = 0; i < people.length; i++) {
        result += `Person #${i + 1}: ${people[i].name} is ${people[i].age} years old`;
        if (i != people.length - 1) {
            result += "\n";
        }
    }
    ```

5. Identify what is wrong with this code and correct it:

    ```javascript
    // Don’t worry about this line
    const value = document.getElementById('some-input-element').value.toLowerCase();

    const filteredNames = [];

    roleNames.forEach(role => {
        if (role.slice(0, value.length).toLowerCase() === value)
            filteredNames.push(role);
    });
    ```

#### Solutions

1. We can use `filter` to get the even numbers, then `map` to divide each of those numbers by 2:

    ```javascript
    const numbers = [2, 6, 3, 7, 10];
    const result = numbers.filter(x => x % 2 === 0).map(x => x / 2);
    ```

2. We need to sum up the lengths of each word. One way to do that would be to use `reduce`:

    ```javascript
    const words = ["apple", "banana", "cherry"];
    const totalLength = words.reduce((acc, x) => acc + x.length, 0);
    ```

    Here, we initialise the accumulator to 0, and then add the length of the string `x` to the accumulator for each string in the `words` array.

3. We want to get the name of the first person under 18 years old. We can get the person with the `find` method and then access the `name` property if the person exists:

    ```javascript
    const people = [
        {name: "Alice", age: 20},
        {name: "Bob", age: 15},
        {name: "Charlie", age: 30},
        {name: "David", age: 10},
    ];
    const firstChild = people.find(x => x.age < 18);
    const firstChildName = firstChild !== undefined ? firstChild.name : undefined;
    ```

    We could also do the last part more succinctly using [optional chaining](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Optional_chaining) (`?.`):

    ```javascript
    const firstChildName = people.find(x => x.age < 18)?.name;
    ```

4. We can use the second parameter in the function passed to `map` to get the index of each element, and then combine the resulting array of strings into one string with `join`:

    ```javascript
    const people = [
        {name: "Alice", age: 20},
        {name: "Bob", age: 15},
        {name: "Charlie", age: 30},
        {name: "David", age: 10},
    ];
    const result = people
        .map((x, i) => `Person #${i + 1}: ${x.name} is ${x.age} years old`)
        .join("\n");
    ```

5. We should use the `filter` method to get a new array with only the roles that match `value`:

    ```javascript
    // Don’t worry about this line
    const value = document.getElementById('some-input-element').value.toLowerCase();

    const filteredNames = roleNames
        .filter(role => role.slice(0, value.length).toLowerCase() === value);
    ```

    We should reuse existing methods and functions wherever possible, and `filter` does exactly what we want to do without having to manually and impurely `push` each matching element into a new array.

    Generally, you should only use `forEach` when you don’t care about the result and thus want to do something that will cause a side effect, such as printing each element (e.g. `roleNames.forEach(role => console.log(role))`). While the function passed to `forEach` in the original code does technically cause a side effect (appending to the `filteredNames` array), the overall effect of the lines below

    ```javascript
    const filteredNames = [];

    roleNames.forEach(role => {
        if (role.slice(0, value.length).toLowerCase() === value)
            filteredNames.push(role);
    });
    ```

    is to simply create a new array, which can be done purely with `filter`.

---

### Callbacks

In JavaScript and HTML5, events trigger actions associated with all mouse clicks and other interactions with the page.  You subscribe to an event on a given HTML element as follows:

```javascript
element.addEventHandler('click',
e=>{
// do something when the event occurs,
// maybe using the result of the event e
});
```

Note that callback functions passed as event handlers are a situation where the one semantic difference between the arrow syntax and regular anonymous function syntax really matters.  In the body of the arrow function above, `this` will be bound to the context of the *caller*, which is probably what you want if you are coding a class for a reusable component.  For functions defined with the `function` keyword, the object that `this` refers to will depend on the context of the *callee*, although the precise behaviour of `this` for functions defined with `function` [may vary with the particular JavaScript engine and the mode of execution](https://www.codementor.io/@dariogarciamoya/understanding--this--in-javascript-du1084lyn).

Here’s a situation where this makes a difference.  Recall that JavaScript functions are just objects.  Therefore, we can also assign properties to them.  We can do this from within the function itself using the `this` keyword:

```javascript
function Counter() {
    this.count = 0;
    setInterval(function increment() {
        console.log(this.count++)
    }, 500);
}
const ctr = new Counter();
```

But, if I run this program at a console, I get the following, each line emitted 500 milliseconds apart:

> NaN  
> NaN  
> NaN  
> ...

This occurs because the `this` inside the function passed to `setInterval` is referring to the first function enclosing its scope, i.e. the `increment` function.  Since `increment` has no count property, we are trying to apply `++` to `undefined` and the result is `NaN` (Not a Number).

Arrow functions have different scoping rules for `this`. That is, they take the `this` of the enclosing scope (outside the arrow function), so in the following we get the expected behaviour:

```javascript
function Counter() {
    this.count = 0;
    setInterval(()=>console.log(this.count++), 500);
}
new Counter();
```

> 0  
> 1  
> 2  
> ...

### Continuations

Continuations are functions that, instead of returning the result of a computation directly to the caller, pass the result on to another function, specified by the caller.  
We can rewrite basically any function to pass their result to a user-specified continuation function instead of returning the result directly.  The parameter `done` in the `continuationPlus` function below will be a function specified by the caller to do something with the result.

```javascript
function simplePlus(a, b) {
    return a + b;
}
function continuationPlus(a, b, done) {
    done(a + b);
}
```

An example of using this to log the result:

```javascript
continuationPlus(3, 5, console.log)
```

This will output “8” to the console.

We can also rewrite tail-recursive functions to end with continuations, which specify some custom action to perform when the recursion is complete:

Consider a tail-recursive implementation of factorial:

```javascript
function tailRecFactorial(a, n) {
    return n <= 1 ? a : tailRecFactorial(n * a, n - 1);
}
```

The function `tailRecFactorial` is tail recursive because the final operation in the function is the recursive call to itself, with no additional computation after this call. We can convert this function into a continuation version by adding an extra parameter `finalAction`:

```javascript
function continuationFactorial(a, n, finalAction) {
    if (n <= 1) finalAction(a);
    else continuationFactorial(n * a, n - 1, finalAction);
}
```

The `continuationFactorial` function uses a continuation by passing a `finalAction` callback that gets called with the result when the recursion reaches the base case (`n <= 1`), allowing further actions to be specified and executed upon completion.

```javascript
continuationFactorial(1, 5, console.log);
```

> 120

<div class="alert-box" markdown="1">
**Optional reading: Making functions tail-recursive**

Consider this non-tail-recursive version of factorial:

```javascript
function factorial(n) {
    return n <= 1 ? 1 : n * factorial(n - 1);
}
```

We can rewrite it to take in a continuation `done` continuation/callback function and to return the result of the callback:

```javascript
function factorialCPS(n, done) {
    return n <= 1 ? done(1) : factorialCPS(n - 1, result => n * result);
}
```

Functions written this way are said to be written in *continuation-passing style (CPS)*. Notice how `factorialCPS` is a tail-recursive function. We just got a tail-recursive function for free without having to rewrite the logic of the function or add an accumulator parameter!

This might not seem that useful for `factorial`, but imagine you had a more complex recursive function, such as `fibonacci` which calculates the `n`th Fibonacci number (ignoring that there are more efficient ways of computing this without recursion):

```javascript
function fibonacci(n) {
    if (n <= 1) {
        // 0th Fibonacci number is 0
        // 1st Fibonacci number is 1
        return n;
    }
    const x = fibonacci(n - 1);
    const y = fibonacci(n - 2);
    return x + y;
}
```

This is harder to convert to a tail-recursive function like we did with `tailRecFactorial` above. However, we can rewrite it with CPS to get a tail-recursive function for free:

```javascript
function fibonacciCPS(n, done) {
    if (n <= 1) {
        return done(n);
    }
    return fibonacciCPS(n - 1, x =>
        fibonacciCPS(n - 2, y =>
            done(x + y)));
}
```

In languages that support [tail call optimisation](/purescript/#tail-call-optimisation), the compiler will be able to optimise the tail call `fibonacciCPS(n - 1, x => ...)` into a loop so that it doesn't add a frame to the call stack, avoiding stack overflow errors. However, it's important to note that CPS doesn't magically make a function use less space; even though a tail-call-optimised `fibonacciCPS` will use less space on the stack than `fibonacci`, it will instead use space on the heap due to the continuation functions/closures capturing their variables.

If you are interested in learning more, [this article](https://www.gresearch.com/news/continuation-passing-style) goes through a more complex example.
</div>

Continuations are essential in asynchronous processing, which abounds in web programming.  For example, when an HTTP request is dispatched by a client to a server, there is no knowing precisely when the response will be returned (it depends on the speed of the server, the network between client and server, and load on that network).  However, we can be sure that it will not be instant and certainly not before the line of code following the dispatch is executed by the interpreter.  Thus, continuation-style callback functions are typically passed through to functions that trigger such asynchronous behaviour, for those callback functions to be invoked when the action is completed.  A simple example of an asynchronous function invocation is the built-in `setTimeout` function, which schedules an action to occur after a certain delay.  The `setTimeout` function itself returns immediately after dispatching the job, e.g. to the JavaScript event loop:

```javascript
setTimeout(()=>console.log('done.'), 0);
// the above tells the event loop to execute
// the continuation after 0 milliseconds delay.
// even with a zero-length delay, the synchronous code
// after the setTimeout will be run first…
console.log('job queued on the event loop…');
```

> job queued on the event loop…  
> done.

### Function and Method Chaining

*Chained functions* are a common pattern.  
Take a simple linked-list data structure as an example.  We’ll hard-code a list object to start off with:

```javascript
const l = {
    data: 1,
    next: {
        data: 2,
        next: {
            data: 3,
            next: null
        }
    }
};
```

We can create simple functions similar to [those of Array](/javascript1#array-cheatsheet), which we can chain:

```javascript
const
  map = (f,l) => l ? ({data: f(l.data), next: map(f,l.next)}) : null
, filter = (f,l) => !l ? null :
                    (next =>
                        f(l.data) ? ({data: l.data, next})
                                  : next
                    ) (filter(f,l.next))
                    // the above is using an immediately invoked
                    // function expression (IIFE) such that the function
                    // parameter `next` is used like a local variable for
                    // filter(f,l.next)
, take = (n,l) => l && n ? ({data: l.data, next: take(n-1,l.next)})
                         : null;
```

(An IIFE is an immediately invoked function expression.)

We can chain calls to these functions like so:

```javascript
take(2,
   filter(x=> x%2 === 0,
       map(x=> x+1, l)
   )
)
```

> { data: 2, next: { data: 4, next: null }}

#### Aside: Dive in to the definition of Map

The definition of `map` (and friends) may look scary, but let’s break it down. We will write it in a more verbose way for now:

```javascript
function map(func, list) {
    if (list !== null) {
        return {
            data: func(list.data),
            next: map(func, list.next)
        };
    }
    else {
        return null;
    }
}
```

The map function will recursively apply the given `func` to each element of the linked list `list`, constructing and returning a new linked list where each element is the result of applying `func` to the corresponding data element in the original list.

Try to expand and step through the `filter` and `take` and see if you can understand how they work.

## Fluent Interfaces (pure vs impure)

In the chained function calls above, you have to read them inside-out to understand the flow.  Also, keeping track of how many brackets to close gets a bit annoying.  Thus, in object-oriented languages, you will often see class definitions that allow for method chaining by providing methods that return an instance of the class itself or another chainable class.

```javascript
class List {
    constructor(private head) {}
    map(f) {
         return new List(map(f, this.head));
    }
  ...
```

Then the same flow as above is possible without the nesting and can be read left-to-right, top-to-bottom:

```javascript
new List(l)
    .map(x=>x+1)
    .filter(x=>x%2===0)
    .take(2)
```

This is called *fluent* programming style.
Interfaces in object-oriented languages that chain a sequence of method calls (as above) are often called *fluent interfaces*.  One thing to be careful about fluent interfaces in languages that do not enforce purity is that the methods may or may not be pure.

That is, the type system does not warn you whether the method mutates the object upon which it is invoked and simply returns `this`, or creates a new object, leaving the original object untouched.  We can see,  however, that `List.map` as defined above creates a new list and is pure.

## Computation with Pure Functions

Pure functions may seem restrictive, but in fact pure function expressions and higher-order functions can be combined into powerful programs.  In fact, anything you can compute with an imperative program can be computed through function composition. Side effects are required eventually, but they can be managed and the places they occur can be isolated. Let’s do a little demonstration; although it might be a bit impractical, we’ll make a list processing environment with just functions:

```javascript
const cons = (_head, _rest)=> selector=> selector(_head, _rest);
```

With just the above definition we can construct a list (the term “cons” dates back to LISP) with three elements, terminated with null, like so:

```javascript
const list123 = cons(1, cons(2, cons(3, null)));
```

The data element and the reference to the next node in the list are stored in the closure returned by the `cons` function.  Created like this, the only side effect of growing the list is the creation of new cons closures.  Mutation of more complex structures such as trees can be managed in a similarly “pure” way, and surprisingly efficiently, as we will see later in this course.

`cons` is a function that takes two parameters `_head` and `_rest` (the `_` prefix is just to differentiate them from the functions I create below), and returns a function that itself takes a function (selector) as argument.  The selector function is then applied to `_head` and `_rest`.

The `selector` function that we pass to the list is our ticket to accessing its elements:

```javascript
list123((_head, _rest)=> _head)
```

> 1

```javascript
list123((_,r)=>r)((h,_)=>h) // we can call the parameters whatever we like
```

> 2

We can create accessor functions to operate on a given list (by passing the list the appropriate selector function):

```javascript
const
    head = list=> list((h,_)=>h),
    rest = list=> list((_,r)=>r)
```

Now, `head` gives us the first data element from the list, and `rest` gives us another list.  Now we can access things in the list like so:

```javascript
const one = head(list123), // ===1
    list23 = rest(list123),
    two = head(list23), // ===2
    ... // and so on
```

Now, here’s the ubiquitous map function:

```javascript
const map = (f, list)=> !list ? null
                              : cons(f(head(list)), map(f, rest(list)));
```

We can now apply our map function to `list123` to perform some transformation of the data

```javascript
const list234 = map(x => x + 1, list123);
```

> `cons(2, cons(3, cons(4, null)));`

In the above, we are using closures to store data.  It’s just a trick to show the power of functions and to put us into the right state of mind for the Lambda Calculus, which provides a complete model of computation using only anonymous functions like those above.  In a real program I would expect you would use JavaScript’s class and object facilities to create data structures.

### Towards Lambda Calculus and Church Encoding

Thus, with only pure function expressions and JavaScript conditional expressions (`?:`), we can begin to perform complex computations.  We can actually go further and eliminate the conditional expressions with more functions! Here’s the gist of it: we wrap list nodes with another function of two arguments, one argument, `whenempty`, is a function to apply when the list is empty, the other argument, `notempty`, is applied by all internal nodes in the list.  An empty list node (instead of `null`) applies the `whenempty` function when visited; a non-empty node applies the `notempty` function. The implementations of each of these functions then form the two conditions to be handled by a recursive algorithm like `map` or `reduce`.  See [“Making Data out of Functions” by Braithwaite](https://leanpub.com/javascriptallongesix/read#leanpub-auto-making-data-out-of-functions) for a more detailed exposition of this idea.

These ideas, of computation through pure function expressions, are inspired by Alonzo Church’s *lambda calculus*.   We’ll be looking again at the lambda calculus later.  Obviously, for the program to be at all useful, you will need some sort of side effect, such as outputting the results of a computation to a display device.  When we begin to explore PureScript and Haskell later in this course we will discuss how such languages manage this trick while remaining “pure”.

---

## Exercises

* Implement a `fromArray` function to construct a `cons` list from an array.
* Implement a `filter` function, which takes a function and a cons list, and returns another cons list populated only with those elements of the list for which the function returns true.
* Implement a `reduce` function for these cons lists, similar to JavasScript’s `Array.reduce`.
* Implement a `reduceRight` function for these cons lists, similar to JavaScript’s `Array.reduceRight`.
* Implement a `concat` function that takes two lists as arguments and returns a new list of their concatenation.
* How can we update just one element in this list without mutating any data and what is the run-time complexity of such an operation?

The solutions for this exercise will be discussed in class.

---

## Updating Data Structures With Pure Functions

We saw in the [introduction to JavaScript](javascript1) that one can create objects with a straightforward chunk of JSON:

```javascript
const studentVersion1 = {
  name: "Tim",
  assignmentMark: 20,
  examMark: 15
};
```

> studentVersion1  
> {name: "Tim", assignmentMark: 20, examMark: 15}

Conveniently, one can copy all of the properties from an existing object into a new object using the “spread” operator `...`, followed by more JSON properties that can potentially overwrite those of the original.  For example, the following creates a new object with all the properties of the first, but with a different assignmentMark:

```javascript
const studentVersion2 = {
    ...studentVersion1,
    assignmentMark: 19
};
```

> studentVersion2  
> {name: "Tim", assignmentMark: 19, examMark: 15}

One can encapsulate such updates in a succinct pure function:

```javascript
function updateExamMark(student, newMark) {
    return {...student, examMark: newMark};
}

const studentVersion3 = updateExamMark(studentVersion2, 19);
```

> studentVersion3  
> {name: "Tim", assignmentMark: 19, examMark: 19}

Note that when we declared each of the variables `studentVersion1-3` as `const`, these variables are only constant in the sense that the object reference cannot be changed.  That is, they cannot be reassigned to refer to different objects:

```javascript
studentVersion1 = studentVersion2;
```

> VM430:1 Uncaught TypeError: Assignment to constant variable.

However, there is nothing in these definitions to prevent the properties of those objects from being changed:

```javascript
studentVersion1.name = "Tom";
```

> studentVersion1  
> {name: "Tom", assignmentMark: 20, examMark: 15}

We will see later how the [TypeScript compiler](/typescript1) allows us to create deeply immutable objects that will trigger compile errors if we try to change their properties.

You may wonder how pure functions can be efficient if the only way to mutate data structures is by returning a modified copy of the original.  There are two responses to such a question, one is: “purity helps us avoid errors in state management through wanton mutation effects---in modern programming correctness is often a bigger concern than efficiency”, the other is “properly structured data permits log(n) time copy-updates, which should be good enough for most purposes”.  We’ll explore what is meant by the latter in later sections of these notes.

*Callback*: A function passed as an argument to another function, to be executed after some event or action has occurred.

*Chained Functions*: A programming pattern where multiple function calls are made sequentially, with each function returning an object that allows the next function to be called.

*Continuation:* A function that takes a result and performs some action with it, instead of returning the result directly. Used extensively in asynchronous programming.

*Fluent Interface*: A method chaining pattern where a sequence of method calls is made on the same object, with each method returning the object itself or another chainable object.

*Pure Function*: A function that always produces the same output for the same input and has no side effects.

*Referential Transparency*: An expression that can be replaced with its value without changing the program’s behaviour, indicating no side effects and consistent results.

*Side Effects*: Any state change that occurs outside of a function’s local environment or any observable interaction with the outside world, such as modifying a global variable, writing to a file, or printing to a console.
