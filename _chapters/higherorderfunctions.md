---
layout: chapter
title: "Higher-Order Functions"
---


## Learning Outcomes

- Understand that [Higher-Order Functions](#higher-order-functions) are those that take other functions as input parameters or return functions
- Understand that [curried functions](#curried-functions) support partial application and therefore creation of functions that are partially specified for reuse scenarios
- Understand that a [Combinator](#combinators) is a higher-order function that uses only function application and earlier defined combinators to define a result from its arguments
- Use simple Combinator functions to manipulate and compose other functions

## Introduction

The really exciting aspect of higher-order function support in languages like JavaScript is that it allows us to combine simple reusable functions in sophisticated ways.  We’ve already seen how functions like map, filter and reduce can be chained to flatten the control flow of data processing.  In this section we will look at some tricks that allow us to use functions that work with other functions in convenient ways.

<div class="alert-box alert-info" markdown="1">
**Type annotations in this section**
If you are following the reading order given by the [index for these notes](/), then you have already read our [introduction to TypeScript](/typescript1/).  Therefore, below we frequently use TypeScript type annotations to be precise about the intended use of the functions.  However, as we start to rely more and more heavily on curried higher-order functions in this chapter, TypeScript type annotations start to become a bit cumbersome, and for the purposes of concisely representing use of combinators to create new functions, we abandon them entirely.  As an exercise, you may like to think about what the TypeScript annotations for some of these functions should be.  This is one of the reasons why we later in these notes move away from JavaScript and TypeScript entirely to instead focus on a real functional language, [Haskell](/haskell1/).
</div>

## Higher-Order Functions

<div class="alert-box alert-info" markdown="1">
Functions that [take other functions as parameters](/javascript1#functions-as-parameters-to-other-functions) or [return functions](/javascript1#closures) are called *higher-order functions*. They are called “higher-order” because they are functions that operate on other functions.
Higher-order functions are a very powerful feature and central to the functional programming paradigm.
</div>
We’ve seen many examples of functions that take functions as parameters, for example, operations on arrays:

```javascript
[1,2,3].map(x=>x*x)
```

>[1,4,9]

Being able to pass functions into other functions enables code customisability and reuse.  For example, a sort function that allows the caller to pass in a comparison function can easily be made to sort in increasing or decreasing order, or to sort data elements on an arbitrary attribute.

We also saw a simple example of a function that returns a new function:

```javascript
const add = x => y => x + y
const add9 = add(9)

add9(3)
add9(1)
```

>12  
>10

Functions that can create new functions give rise to all sorts of emergent power, such as the ability to customise, compose and combine functions in very useful ways.  We will see this later when we look at [function composition](/higherorderfunctions#composition) and [combinators](/higherorderfunctions#combinators).

## Curried Functions

Higher-order functions that take a single parameter and return another function operating on a single parameter are called *curried functions*.  The `add` function above is one example.  You can either call it twice immediately to operate on two parameters:

```javascript
add(3)(2)
```

>5

or call it once, leaving one parameter left unspecified, to create a reusable function, like `add9` above.
Such use of a curried function with only a subset of its parameters is called *partial application*.  Partial application returns a function for which further parameters must be supplied before the body of the function can finally be evaluated, e.g.:

```js
add9(1)
```

>10

Here’s a practical example of a curried function. Let’s say we want a function for computing the volume of cylinders, parameterised by the approximation for π that we plan to use:

```javascript
function cylinderVolume(pi: number, height: number, radius: number): number {
   return pi * radius * radius * height;
}
```

And we invoke it like so:

```javascript
cylinderVolume(Math.PI, 4, 2);
```

Now consider another version of the same function:

```javascript
function cylinderVolume(pi: number) {
  return function(height: number) {
    return function(radius: number) {
      return pi * radius * radius * height;
    }
  }
}
```

This one, we can invoke like so:

```javascript
cylinderVolume(Math.PI)(4)(2)
```

But we have some other options too.  For example, we are unlikely to change our minds about what precision approximation of PI we are going to use between function calls. So let’s make a local function that fixes PI:

```javascript
const cylVol = cylinderVolume(Math.PI);
```

Which we can invoke when we are ready like so:

```javascript
cylVol(4)(2)
```

What if we want to compute volumes for a whole batch of cylinders of fixed height of varying radii?

```javascript
const radii = [1.2,3.1,4.5, ... ],
      makeHeight5Cylinder = cylVol(5),
      cylinders = radii.map(makeHeight5Cylinder);
```

Or we can make it into a handy function to compute areas of circles:

```javascript
const circleArea = cylVol(1)
```

Such functions are called *curried functions*, named after a mathematician named Haskell Curry.  This gives you a hint as to what functions look like in the Haskell programming language and its variants.
We can also create a function to make curried versions of conventional multi-parameter JavaScript functions:

```typescript
function curry2<T,U,V>(f: (x:T, y:U) => V): (x:T) => (y:U) => V {
   return x => y => f(x,y);
}
```

Now, given a function like `plus = (x,y) => x + y`, we can create the curried add function above, like so:

```javascript
const add = curry2(plus)
add(3)(4)
```

> 7

We can also create curried versions of functions with more than two variables, but the TypeScript syntax for functions with arbitrary numbers of arguments gets a bit scary, requiring advanced use of [conditional types](https://www.typescriptlang.org/docs/handbook/2/conditional-types.html). This is one of the many reasons we will shortly [switch to Haskell](/haskell1/) for our exploration of more advanced functional programming topics.

```javascript
// A type for a regular Uncurried function with arbitrary number of arguments
type Uncurried = (...args: any[]) => any;

// Warning - advanced TypeScript types!
// A type for curried functions with arbitrary numbers of arguments.
// Full disclosure: I needed ChatGPT to help figure this out.
type Curried<T> = T extends (...args: infer Args) => infer R
  ? Args extends [infer First, ...infer Rest]
    ? (arg: First) => Curried<(...args: Rest) => R>
    : R
  : never;

// now the curry function returns a function which, when called in curried style,
// will build the argument list until it has enough arguments to call the original fn
function curry<T extends Uncurried>(fn: T): Curried<T> {
  const curried = (...args: any[]): any =>
    args.length >= fn.length ? fn(...args)
                             : (...moreArgs: any[]) => curried(...args, ...moreArgs);
  return curried as Curried<T>;
}

const weirdAdd = (a:number,b:boolean,c:string) => a + (b?1:0) + parseInt(c)

// type of curriedWeirdAdd is:
//  (arg: number) => (arg: boolean) => (arg: string) => number
const curriedWeirdAdd = curry(weirdAdd);
```

## Composition

Consider the following function which takes two functions as input. Note the way the types line up:

```javascript
function compose<U,V,W>(f:(x:V)=>W,g:(x:U)=>V) {
  return (x:U)=> f(g(x))
}
```

This function lets us combine two functions into a new reusable function.  For example, given a messy list of strings representing numbers of various precision:

```javascript
const grades = ['80.4','100.000','90','99.25']
```

We can define a function to parse these strings into numbers and then round them to the nearest whole number purely through composition of two existing functions:

```javascript
const roundFloat = compose(Math.round, Number.parseFloat)
```

And then apply it to the whole set:

```javascript
grades.map(roundFloat)
```

> [80, 100, 90, 99]

Note that `compose` lets us define `roundFloat` without any messing around with anonymous functions and explicit wiring-up of return values to parameters.  We call this *tacit* or *point-free* style programming.

---

## Exercise

- Create a `compose` function in JavaScript that takes a variable number of functions as arguments and composes (chains) them.  Using the spread operator (`...`) to take a variable number of arguments as an array and the `Array.prototype.reduce` method, the function should be very small.

- Create a `pipe` function that composes its arguments in the opposite order to the `compose` function above.  That is, left-to-right.  Note that in [RxJS](https://www.learnrxjs.io/), such a `pipe` function is an important way to create chains of operations (over Observable streams).

### Solutions

The `compose` function takes multiple functions and returns a new function that applies these functions from right to left to an initial argument using `reduceRight`. In the example, `add1` adds 1 to a number, and `double` multiplies a number by 2. `compose(double, add1)` creates a function that first adds 1 and then doubles the result. Calling this function with 5 results in 12.

```javascript
const compose = (...funcs) => (initialArg) =>
  funcs.reduceRight((arg, fn) => fn(arg), initialArg);

// Example usage
const add1 = x => x + 1;
const double = x => x * 2;

const add1ThenDouble = compose(double, add1);

console.log(add1ThenDouble(5)); // Output: 12
```

Using TypeScript to create a flexible compose function requires accommodating varying input and output types for each function in the chain. To achieve this flexibility without losing type safety, we need to ensure that the output type of one function matches the input type of the next function. However, if we try to type this correctly without using any, we face significant complexity with the lack of expressiveness in typescripts type system. `pipe` in RxJS solves this issue by [hardcoding up to 9 functions inside the pipe](https://rxjs.dev/api/index/function/pipe), any functions larger then this will not be typed correctly.

```javascript
const pipe = (...funcs) => (initialArg) =>
  funcs.reduce((arg, fn) => fn(arg), initialArg);
```

The pipe function is similar to the compose function, but it applies its functions in the opposite order—from left to right.

---

## Combinators

Combinators are higher-order functions that perform pure operations on their arguments to produce a result.  They may seem very basic, but as their name suggests, they provide useful building blocks for manipulating and composing functions to create new functions.  The [`compose`](#composition) function is a combinator.  Some more examples follow.

### Identity I-Combinator

The following may seem trivial:

```javascript
function identity<T>(value: T): T {
   return value;
}
```

But it has some important applications:

- Higher-order functions that take a user-specified function to apply in some context ([such as our sumTo from earlier](/javascript1/#functions-as-parameters-to-other-functions)) can be passed `identity` to restore the default behaviour.
- For extracting data from encapsulated types (e.g. by passing `identity` into map).
- The above scenarios are also indicative of a useful way to test such higher-order functions, broadly: “does passing the `identity` operator really give us back what we started with?”.
- For composition with other combinators, as below.

### K-Combinator

The curried K-Combinator looks like:

```javascript
const K = x=> y=> x
```

So it is a function that ignores its second argument and returns its first argument directly.  Note the similarity to the `head` function of our [cons list](/functionaljavascript#computation-with-pure-functions).  In fact, we can derive curried versions of both the `head` and `rest` functions used earlier from `K` and `I` combinators (renaming `rest` to `tail`; see below):

```javascript
const
   K = x=> y=> x,
   I = x=> x,
   cons = x=> y=> f=> f(x)(y),
   head = l=> l(K),
   tail = l=> l(K(I)),
   forEach = f=> l=> l?(f(head(l)),forEach(f)(tail(l))):null;

const l = cons(1)(cons(2)(cons(3)(null)));

forEach(console.log)(l)
```

> 1  
> 2  
> 3

The definition of `head` is by straightforward, like-for-like substitution of `K` into a curried version of our previous definition for `head`.  Note, the following is not code, just statements of equivalence (≡):

```javascript
head ≡ l=>l((h,_)=>h) -- previous uncurried definition of head
     ≡ l=>l(curry2((h,_)=>h))
     ≡ l=>l(h=>_=>h)
```

Where the expression in brackets above we notice is equivalent to `K`:

```javascript
K  ≡  x=> y=> x  ≡  h=> _=> h
```

Of course, this definition is not unique to JavaScript; we mainly use this language to explore this idea, but the equivalent can be completed in Python, with a slightly more verbose syntax:

```python
K = lambda x : lambda y : x
I = lambda x : x
cons = lambda x: lambda y : lambda f : f(x)(y)
head = lambda l : l(K)
tail = lambda l : l(K(I))
forEach = lambda f : lambda l : (f(head(l)),forEach(f)(tail(l))) if l is not None else None
l = cons(1)(cons(2)(cons(3)(None)))

forEach(print)(l)
```

> 1
> 2
> 3

In the context of the [Lambda Calculus](/lambdacalculus), we will see that such a renaming is called *Alpha Conversion*.

We are gradually changing our terminology to be more Haskell-like, so we have named the curried version of `rest` to `tail`.  The new definition of `tail`, compared to our previous definition of `rest`, is derived as follows:

```javascript
K(I)  ≡  K(i=> i)             -- expand I := i=> i
      ≡  (x=> y=> x)(i=> i)   -- expand K := x=> y=> x
      ≡  y=> i=> i
```

Where the last line above is the result of applying `x=>y=>x` to `i=>i`.  Thus, we substitute `x:=i=>i` in the body of the first function (the expansion of `K`).  When we explore the [Lambda Calculus](/lambdacalculus), we will see that this operation (simple evaluation of function application by substitution of expressions) is called *Beta reduction*.

Now we could derive `tail` from `rest` using our `curry2` function:

```javascript
rest ≡ l=>l((_,r)=>r)
tail ≡ l=>l(curry2((_,r)=>r))
     ≡ l=>l(_=>r=>r)
```

Where `_=> r=> r  ≡  y=> i=> i` and therefore `tail ≡ l=>l(K(i))`.  QED!!!

FYI it has been shown that simple combinators like K and I (at least one other is required) are sufficient to create languages as powerful as lambda calculus without the need for lambdas, e.g. see [SKI Combinator Calculus](https://en.wikipedia.org/wiki/SKI_combinator_calculus).

In previous sections, we have seen a number of functions that transform lists (or other containers) into new lists like `map`, `filter` and so on.  We have also introduced the `reduce` function as a way to compute a single value over a list.  If we realise that the value we produce from `reduce` can also be a list, we can actually use `reduce` to implement all of the other list transformations.  Instead of returning a value from `reduce`, we could apply a function which produces only side effects, thus, performing a `forEach`.  We’ll use this as an example momentarily.

First, here’s another implementation of `reduce` for the above formulation of cons lists - but we rename it `fold` (again, as our JavaScript becomes more and more Haskell-like we are beginning to adopt Haskell terminology).

```js
const fold = f=> i=> l=> l ? fold(f)(f(i)(head(l)))(tail(l)) : i
```

Now, for example, we can define `forEach` in terms of `fold`:

```javascript
const forEach = f=>l=>fold(_=>v=>f(v))(null)(l)
```

Now, the function `f` takes one parameter and we don’t do anything with its return type (in TypeScript we could enforce the return type to be `void`).
However, `fold` is expecting as its first argument a curried function of two parameters (the accumulator and the list element).  Since in `forEach` we are not actually accumulating a value, we can ignore the first parameter, hence we give `fold` the function `_=>v=>f(v)`, to apply `f` to each value `v` from the list.

But note that `v=>f(v)` is precisely the same as just `f`.
So we can simplify `forEach` a bit further:

```javascript
const forEach = f=>l=>fold(_=>f)(null)(l)
```

But check out these equivalences:

```javascript
K(f)  ≡  (x=> y=> x)(f) -- expand K
      ≡  y=> f          -- apply the outer function to f, hence we substitute x:= f
      ≡  _=> f          -- rename y to _
```

where in the last line above, since `y` doesn’t appear anywhere in the body of the function, we don’t care what it’s called anymore and rename it to `_`.

Therefore, we can use our `K` combinator to entirely avoid defining any functions in the body of `forEach`:

```javascript
const forEach = f=>l=>fold(K(f))(null)(l)
```

---

### Fold Exercise

- Write `map` and `filter` for the above cons list definition in terms of `fold`.

#### Solutions

A naive implementation of `map` using `fold` would be:

```javascript
const map = f => l =>
  fold(acc => v => cons(f(v))(acc))(null)(l);

const l = cons(1)(cons(2)(cons(3)(null)));
const mappedList = map(x => x * 2)(l);
forEach(console.log)(mappedList);

```

> 6
>
> 4
>
> 2

This constructs a new cons every time, applying the function `f` to the current item in `v`. However, this will reverse the list because `fold` processes the list from head to tail, and constructs the new list by *prepending* each element to the accumulator.

```javascript
const reverse = l =>
  fold(acc => v => cons(v)(acc))(null)(l);

const map = f => l =>
  fold(acc => v => cons(f(v))(acc))(null)(reverse(l));

const filter = pred => l =>
  fold(acc => v => pred(v) ? cons(v)(acc) : acc)(null)(reverse(l));
```

Therefore, we need to reverse the list using a separate function to ensure that we apply the functions in the correct order. However, the preferred way around this would be to reduce in the other direction, e.g., using [reduceRight](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/reduceRight), to fold through the list tail to head.

---

### Alternation (OR-Combinator)

A function that applies a first function.  If the first function fails (returns undefined, false or null), it applies the second function.  The result is the first function that succeeded.

```javascript
const or = f=> g=> v=> f(v) || g(v)
```

Basically, it’s a curried if-then-else function with continuations. Imagine something like the following data for student names in a unit, then a dictionary of the ids of students in each class:

```javascript
const students = ['tim','sally','sam','cindy'],
      class1 = { 'tim':123, 'cindy':456},
      class2 = { 'sally':234, 'sam':345};
```

We have a function that lets us lookup the id for a student in a particular class:

```javascript
// 'class' is a reserved keyword in JavaScript
const lookup = class_=> name=> class_[name]
```

Now we can try to find an id for each student, first from `class1` but fall back to `class2` if it isn’t there:

```javascript
const ids = students.map(or(lookup(class1))(lookup(class2)))
```

---

## End Note

### Unary versus Binary Functions in JavaScript

Uncurried functions of two parameters can be called Binary functions.  Functions of only one parameter can therefore be called Unary functions.  Note that all of our curried functions are unary functions, which return other unary functions.
We’ve seen situations now where curried functions are flexibly combined to be used in different situations.

Note that in JavaScript you sometimes see casual calls to binary functions but with only one parameter specified.  Inside the called function the unspecified parameter will simply be `undefined`, which is fine if the case of that parameter being `undefined` is handled in a way that does not cause an error or unexpected results, e.g.:

```javascript
function binaryFunc(x,y) {console.log(`${x} ${y}`) }
binaryFunc("Hello", "World")
binaryFunc("Hello")
```

> Hello World  
> Hello undefined

Conversely, JavaScript allows additional parameters to be passed to unary functions which will then simply be unused, e.g.:

```javascript
function unaryFunc(x) { console.log(x) }
unaryFunc("Hello")
unaryFunc("Hello", "World")
```

> Hello  
> Hello

But, here’s an interesting example where mixing up unary and binary functions in JavaScript’s very forgiving environment can go wrong.

```javascript
['1','2','3'].map(parseInt);
```

We are converting an array of strings into an array of int.  The output will be `[1,2,3]` right?  WRONG!

```javascript
['1','2','3'].map(parseInt);
```

> [1, NaN, NaN]

What the …!

But:

```javascript
parseInt('2')
```

> 2

What’s going on?
HINT: [parseInt is not actually a unary function](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/parseInt).

The point of this demonstration is that curried functions are a more principled way to support partial function application, and also much safer and easier to use when the types guard against improper use.  Thus, this is about as sophisticated as we are going to try to get with Functional Programming in JavaScript, and we will pick up our discussion further exploring the power of FP in the context of the [Haskell functional programming language](/haskell1/).  However, libraries do exist that provide quite flexible functional programming abstractions in JavaScript.  For example, you might like to investigate [Ramda](http://ramdajs.com).

---

## Exercises

1. From the docs for `Array.map` and `parseInt`  can you figure out why the above is happening?
2. Write a function called unary that takes a binary function and a value to bind to its first argument, and returns a unary function.  What is its fully specified TypeScript type signature?
3. Flip - e.g. applied to `map(Iterable,fn)` to create `mapApplyFn(Iterable)`.

### Solutions

1. When `parseInt` is used as the callback for `map`, it is called with **three** arguments: currentValue, index, and array. `parseInt` expects the second argument to be the radix, but `map` provides the index of the current element as the second argument. This leads to incorrect parsing.

2. The unary function is defined as:

  ```typescript
function unary<T, U, V>(binaryFunc: (arg1: T, arg2: U) => V, boundValue: T): (arg2: U) => V {
      return function(secondValue: U): V {
        return binaryFunc(boundValue, secondValue);
    };
  }
  ```

 1. `T`: Type of the first argument of the binary function (the value to bind).
 2. `U`: Type of the second argument of the binary function.
 3. `V`: Return type of the binary function.

```typescript
function flip<T, U, V>(binaryFunc: (arg1: T, arg2: U) => V): (arg2: U, arg1: T) => V {
  return function(arg2: U, arg1: T): V {
    return binaryFunc(arg1, arg2);
  };
}
```

---

## Glossary

*Combinator*: A higher-order function that uses only function application and earlier defined combinators to define a result from its arguments.

*Function composition:* The process of combining two or more functions to produce a new function.

*Point-free style*: A way of defining functions without mentioning their arguments.

*Curried functions*: Functions that take multiple arguments one at a time and return a series of functions.

*Partial application*: The process of fixing a number of arguments to a function, producing another function of smaller arity.

*Identity function (I-combinator)*: A function that returns its argument unchanged.

*K-combinator*: A combinator that takes two arguments and returns the first one.
