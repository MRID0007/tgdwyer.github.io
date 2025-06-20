---
layout: chapter
title: "Lambda Calculus"
---
## Learning Outcomes

* Understand that the lambda calculus provides a complete model of computation
* Relate the lambda calculus to functional programming
* Apply conversion and reduction rules to simplify lambda expressions

## Introduction

The Lambda Calculus is a model of computation developed in the 1930s by the mathematician Alonzo Church.  You are probably aware of the more famous model for computation developed around the same time by Alan Turing: the Turing Machine.  However, while the Turing Machine is based on a hypothetical physical machine (involving tapes from which instructions are read and written) the Lambda Calculus was conceived as a set of rules and operations for function abstraction and application.  It has been proven that, as a model of computation, the Lambda Calculus is just as powerful as Turing Machines, that is, any computation that can be modelled with a Turing Machine can also be modeled with the Lambda Calculus.

The Lambda Calculus is also important to study as it is the basis of functional programming.  The operations we can apply to Lambda Calculus expressions to simplify (or reduce) them, or to prove equivalence, can also be applied to pure functions in a programming language that supports [higher-order functions](/higherorderfunctions).

## Lambda Expressions

Lambda Calculus expressions are written with a standard system of notation.  It is worth looking at this notation before studying haskell-like languages because it was the inspiration for Haskell syntax.  Here is a simple Lambda Abstraction of a function:

```lambdacalc
λx.x
```

The `λ` (Greek letter Lambda) simply denotes the start of a function expression. Then follows a list of parameters (in this case we have only a single parameter called `x`) terminated by `.`.  After the `.` is the function body, an expression returned by the function when it is applied. A variable like `x` that appears in the function body and also the parameter list is said to be *bound* to the parameter.  Variables that appear in the function body but not in the parameter list are said to be *free*.  The above lambda expression is equivalent to the JavaScript expression:

```javascript
x => x
```

---

### Exercise

When we discussed combinators in JavaScript, we gave this function a name.  What was it?

### Solutions

It was the [I-combinator](/higherorderfunctions#identity-i-combinator).

---

Some things to note about such lambda expressions:

* A lambda expression has no name, it is anonymous.  Note that anonymous functions in languages like JavaScript and Python are also frequently called lambda expressions, or just lambdas.  Now you know why.
* The only values that Lambda Calculus variables can take on is other functions (i.e. lambda expressions).  It’s lambdas all the way down!  However, to actually model and perform useful computations we say that certain expressions represent values.  See the discussion of [Church Encodings](#church-encodings), below, to see how this is done.
* The names of variables bound to parameters in a lambda expression are only meaningful within the context of that expression.  Thus, `λx.x` is semantically equivalent (or *alpha* equivalent) to `λy.y` or any other possible renaming of the variable.
* Lambda functions can have multiple parameters in the parameter list, e.g.: `λxy. x y`, but they are implicitly curried (e.g. a sequence of nested univariate functions).  Thus the following are all equivalent:

```lambdacalc
λxy.xy
= λx.λy.xy
= λx.(λy.xy)
```

## Combinators

We have already discussed combinators in JavaScript, now we can give them a more formal definition:

<div class="alert-box alert-info" markdown="1">
A *combinator* is a lambda expression (function) with no free variables.
</div>

Thus, the expression `λx.x` is a combinator because the variable `x` is bound to the parameter.  The expression `λx.xy` is not a combinator, because `y` is not bound to any parameter, it is *free*.

The [`K` combinator](/higherorderfunctions/#k-combinator) which we wrote as `x=>y=>x` in JavaScript, is written `λxy.x`.

## Application

What can we do with such a lambda expression?  Well we can *apply* it to another expression (The same way we can *apply* anonymous functions to an argument in JavaScript).  Here, we apply the lambda `(λx.x)` to the variable `y`:

```lambdacalculus
(λx.x)y
```

Note that while in JavaScript application of a function `(x=>x)` to an argument `y` requires brackets around the argument: `(x=>x)(y)`, in the Lambda Calculus application of some expression `f` to some other expression `x` is indicated simply `fx`.  Brackets are required to delineate the start and end of an expression, e.g. in `(λx.x)y`, the brackets make it clear that `y` is not part of the lambda `λx.x`, but rather the lambda is being applied to `y`.

We can reduce this expression to a simpler form by a substitution, indicated by a bit of intermediate notation.  Two types of annotations are commonly seen, you can use either (or both!):

```lambdacalculus
x [x:=y]         -- an annotation on the right of the lambda body showing the substitution that will be applied to the expression on the left
(λx [x:=y].x)    -- an annotation inside the parameter list showing the substitution that will be performed inside the body (arguments have already been removed)
```

Now we perform the substitution in the body of the expression and throw away the head, since all the bound variables are substituted, leaving only:

```lambdacalc
y
```

This first reduction rule, substituting the arguments of a function application to all occurrences of that parameter inside the function body, is called *beta reduction*.

The next rule arises from the observation that, for some lambda term `M` that does not involve `x`:

```lambdacalc
λx.Mx
```

is just the same as M.  This last rule is called *eta conversion*.

Function application is left-associative except where terms are grouped together by brackets.  This means that when a Lambda expression involves more than two terms, BETA reduction is applied left to right, i.e.,

```(λz.z) (λa.a a)  (λz.z b) = ( (λz.z) (λa.a a) ) (λz.z b)```.

<div class="cheatsheet" markdown="1">

## Lambda Calculus Cheatsheet

Three operations can be applied to lambda expressions:

**Alpha Equivalence**: variables can be arbitrarily renamed as long as the names remain consistent within the scope of the expression.

```lambda
λxy.yx = λwv.vw
```

**Beta Reduction**: functions are applied to their arguments by substituting the text of the argument in the body of the function.

```lambda
(λx. x) y
= (λx [x:=y]. x)     - we indicate the substitution that is going to occur inside []
= x [x:=y]           - an alternative way to show the substitution
= y
```

**Eta Conversion**: functions that simply apply another expression to their argument can be substituted with the expression in their body.

```lambda
λx.Mx
= M
```

</div>

One thing to note about the lambda calculus is that it does not have any such thing as a global namespace.  All variables must be:

* Parameters from some enclosing lambda expression (note, below we start using labels to represent expressions - these are not variables, just placeholders for an expression that can be substituted for the label).
* Immutable - there is no way to assign a new value to a variable from within a lambda expression.

This makes the language and its evaluation very simple.  All we (or any hypothetical machine for evaluating lambda expressions) can do with a lambda is apply the three basic alpha, beta and eta reduction and conversion rules.  Here’s a fully worked example of applying the different rules to reduce an expression until no more Beta reduction is possible, at which time we say it is in *beta normal form*:

```lambdacalc
(λz.z) (λa.a a) (λz.z b)
⇒
((λz.z) (λa.a a)) (λz.z b)    => Function application is left-associative
⇒
(z [z:=λa.a a]) (λz.z b)      => BETA Reduction
⇒
(λa.a a) (λz.z b)
⇒
a a [a:=λz.z b]               => BETA Reduction
⇒
(λz.z b) (λz.z b)
⇒
z b [z:=(λz.z b)]             => BETA Reduction
⇒
(λz.z b) b
⇒
z b [z:=b]                    => BETA Reduction
⇒
b b         => Beta normal form, cannot be reduced again.
```

Note, sometimes I add extra spaces as above just to make things a little more readable - but it doesn’t change the order of application, indicate a variable is not part of a lambda to its left (unless there is a bracket) or have any other special meaning.

## Church Encodings

And yet, this simple calculus is sufficient to perform computation.  Alonzo Church demonstrated that we can model any of the familiar programming language constructs with lambda expressions.  For example, Booleans:

```lambda
TRUE = λxy.x    = K-combinator
FALSE = λxy.y   = K I
```

Note that we are making use of the K and I combinators here as we did for the head and rest functions for our [cons list](/higherorderfunctions/#k-combinator), i.e. returning either the first or second parameter to make a choice between two options.  Now we can make an IF expression:

```lambda
IF = λbtf.b t f
```

`IF TRUE` returns the expression passed in as `t` and `IF FALSE` returns the expression passed in as `f`.  Now we can make Boolean operators:

```lambda
AND = λxy. IF x  y FALSE
OR = λxy. IF x TRUE y
NOT = λx. IF x FALSE TRUE
```

And now we can evaluate logical expressions with beta reduction:

```lambda
NOT TRUE
= (λx. IF x FALSE TRUE) TRUE       - expand NOT
= IF x FALSE TRUE [x:=TRUE]        - beta reduction
= IF TRUE FALSE TRUE
= (λbtf.b t f) TRUE FALSE TRUE     - expand IF
= b t f [b:=TRUE,t:=FALSE,f:=TRUE] - beta reduction
= TRUE FALSE TRUE
= (λxy.x) FALSE TRUE               - expand TRUE
= x [x:=FALSE]                     - beta reduction
= FALSE
```

Alonzo Church also demonstrated an encoding for natural numbers:

```lambda
0 = λfx.x = K I
1 = λfx.f x
2 = λfx.f (f x)
```

In general a natural number `n` has two arguments `f` and `x`, and iterates `f` `n` times. The successor of a natural number `n` can also be computed:

```lambda
SUCC = λnfx.f (n f x)

SUCC 2
= (λnfx.f (n f x)) 2
= (λfx.f (n f x)) [n:=2]
= (λfx.f (2 f x))
= (λfx.f ((λfx.f (f x)) f x))
= (λfx.f ((f (f x)) [f:=f,x:=x]))
= (λfx.f (f (f x)))
= 3
```

---

### Exercises

1. Try using beta reduction to compute some more logical expressions.  E.g. make an expression for XOR.
2. Our JavaScript [cons list](/higherorderfunctions/#k-combinator) was based on the Church Encoding for linked lists.  Try writing the `cons`, `head` and `rest` functions as Lambda expressions.
3. Investigate [Church Numerals](https://en.wikipedia.org/wiki/Church_encoding) and try using lambda calculus to compute some basic math.

#### Solutions

1. First, let’s recall the definition of XOR: If either, but not both, of the inputs is true, then the output is true.

    ```lambda
    XOR = λxy. IF x (NOT y) y
    ```

    ```lambda
    XOR TRUE FALSE

    = (λxy. IF x (NOT y) y) TRUE FALSE - expand XOR
    = IF x (NOT y) y [x:=TRUE, y:=FALSE] - beta reduction
    = IF (TRUE) (NOT FALSE) FALSE
    = (λbtf.b t f) TRUE (NOT FALSE) FALSE - expand IF
    = b t f [b:=TRUE,t:=(NOT FALSE),f:=FALSE] - beta reduction
    = TRUE (NOT FALSE) FALSE
    = (λxy.x) (NOT FALSE) FALSE - expand TRUE
    = x [x:=(NOT FALSE), y:=FALSE] - beta reduction
    = NOT FALSE
    = (λx. IF x FALSE TRUE) FALSE - expand NOT
    = IF x FALSE TRUE (x:=FALSE)
    = IF FALSE FALSE TRUE
    = (λbtf.b t f) FALSE FALSE TRUE - expand IF
    = b t f [b:=FALSE,t:=FALSE,f:=TRUE] - beta reduction
    = FALSE FALSE TRUE
    = (λxy.y) FALSE TRUE - expand FALSE
    = y [x:=FALSE, y:=TRUE] - beta reduction
    = TRUE
    ```

    ```lambda
    XOR TRUE TRUE

    = (λxy. IF x (NOT y) y) TRUE TRUE - expand XOR
    = IF x (NOT y) y [x:=TRUE, y:=TRUE] - beta reduction
    = IF (TRUE) (NOT TRUE) TRUE
    = (λbtf.b t f) TRUE (NOT TRUE) TRUE - expand IF
    = b t f [b:=TRUE,t:=(NOT TRUE),f:=TRUE] - beta reduction
    = TRUE (NOT TRUE) TRUE
    = (λxy.x) (NOT TRUE) TRUE - expand TRUE
    = x [x:=(NOT TRUE), y:=TRUE] - beta reduction
    = NOT TRUE
    = (λx. IF x FALSE TRUE) TRUE - expand NOT
    = IF x FALSE TRUE (x:=TRUE)
    = IF TRUE FALSE TRUE
    = (λbtf.b t f) TRUE FALSE TRUE - expand IF
    = b t f [b:=TRUE,t:=FALSE,f:=TRUE] - beta reduction
    = TRUE FALSE TRUE
    = (λxy.x) FALSE TRUE - expand TRUE
    = x [x:=FALSE, y:=TRUE] - beta reduction
    = FALSE
    ```

2. We can define these just like we did [in JavaScript](/functionaljavascript/#computation-with-pure-functions):

    ```lambda
    CONS = λhrf.f h r
    HEAD = λl.l(λhr.h)
    REST = λl.l(λhr.r)
    ```

3. Recall that a number `n` in lambda calculus can be represented by a function that takes in a function `f` and applies it to a value `n` times. Hence, a number `m+n` in lambda calculus should apply a function `f` `m+n` times.

    We can implement `ADD` by applying `f` `n` times and then applying `f` `m` times:

    ```lambda
    ADD = λmnfx.m f (n f x)
    ```

    For example,

    ```lambda
    ADD 1 2
    = (λmnfx.m f (n f x)) (λfx.f x) (λfx. f (f x))
    = (λfx.m f (n f x)) [m:=(λfx.f x), n:=(λfx. f (f x))]
    = λfx.(λfx.f x) f ((λfx. f (f x)) f x) - this is the same as λfx.1 f (2 f x)
    = λfx.(λfx.f x) f ((λx. f (f x)) x)
    = λfx.(λfx.f x) f (f (f x))
    = λfx.(λx.f x) (f (f x))
    = λfx.f (f (f x))
    = 3
    ```

    For multiplication, we need a function that applies `f` `m*n` times. We can do this by creating a function that applies `f` `n` times (`n f`), and then applying that function `m` times (`m (n f)`):

    ```lambda
    MULTIPLY = λmnfx.m (n f) x
    ```

    For example,

    ```lambda
    MULTIPLY 2 3
    = (λmnfx.m (n f) x) (λfx. f (f x)) (λfx. f (f (f x)))
    = (λfx.m (n f) x) [m:=(λfx. f (f x)), n:=(λfx. f (f (f x)))]
    = λfx.(λfx. f (f x)) ((λfx. f (f (f x))) f) x
    = λfx.(λfx. f (f x)) (λx. f (f (f x))) x - this is the same as λfx.2 3 x
    = λfx.(λx. (λx. f (f (f x))) ((λx. f (f (f x))) x)) x
    = λfx.(λx. f (f (f x)) ((λx. f (f (f x))) x))
    = λfx.(f (f (f ((λx. f (f (f x))) x))))
    = λfx.(f (f (f (f (f (f x))))))
    = 6
    ```

---

## Divergent Lambda Expressions

Despite the above demonstration of evaluation of logical expressions, the restriction that lambda expressions are anonymous makes it a bit difficult to see how lambda calculus can be a general model for useful computation.  For example, how can we have a loop?  How can we have recursion if a lambda expression does not have any way to refer to itself?

The first hint to how loops might be possible with lambda calculus is the observation that some expressions do not simplify when beta reduced.  For example:

```lambda
( λx . x  x) ( λy. y y)   - (1)
x x [x:= y. y y]
( λy . y  y) ( λy. y y)   - which is alpha equivalent to what we started with, so goto (1)
```

Thus, the reduction would go on forever.  Such an expression is said to be divergent.  However, if a lambda function is not able to refer to itself it is still not obvious how recursion is possible.

The answer is due to the American mathematician Haskell Curry and is called the fixed-point or Y combinator:

```lambda
 Y = λf. ( λx . f (x x) ) ( λx. f (x x) )
```

When we apply `Y` to another function `g` we see an interesting divergence:

<pre class="code">
Y g = (λf. ( λx . f (x x) ) ( λx. f (x x) ) ) g
    = ( λx . f (x x) ) ( λx. f (x x) ) [f:=g]  <i>- beta reduction</i>
    = <b>( λx . g (x x) ) ( λx. g (x x) )</b>         <i>- a partial expansion of Y g, remember this…</i>
    = g (x x) [ x:= λx. g (x x)]               <i>- beta reduction</i>
    = g ( <b>(λx. g (x x) ) (λx. g (x x) )</b> )      <i>- bold part matches Y g above, so now…</i>
    = g (Y g)
<i>  … more beta reduction as above
  … followed by substitution with Y g when we see the pattern above…</i>
    = g (g (Y g))
    = g (g (g (Y g)))
<i>  … etc</i>
</pre>

If we directly translate the above version of the Y-combinator into JavaScript we get the following:

```javascript
const Y = f=> (x => f(x(x)))(x=> f(x(x))) // warning infinite recursion ahead!
```

So now `Y` is just a function which can be applied to another function, but what sort of function do we pass into `Y`?  If we are to respect the rules of the Lambda calculus we cannot have a function that calls itself directly.  That is, because Lambda expressions have no name, they can’t refer to themselves by name.

Therefore, we need to wrap the recursive function in a lambda expression into which a reference to the recursive function itself can be passed as parameter.  We can then in the body of the function refer to the parameter function by name.
It’s a bit weird, let me just give you a JavaScript function which fits the bill:

```javascript
// A function that recursively calculates “n!”
//  - but it needs to be reminded of its own name in the f parameter in order to call itself.
const FAC = f => n => n>1 ? n * f(n-1) : 1
```

Now we can make this function compute factorials like so:

```js
FAC(FAC(FAC(FAC(FAC(FAC())))))(6)
```

> 720

Because we gave FAC a stopping condition, we can call too many times and it will still terminate:

```js
FAC(FAC(FAC(FAC(FAC(FAC(FAC(FAC(FAC()))))))))(6)
```

> 720

From the expansion of `Y g = g (g (g (…)))` it would seem that `Y(FAC)` would give us the recurrence we need. But will the JavaScript translation of the Y-combinator be able to generate this sequence of calls?  

```js
console.log(Y(FAC)(6))
```

> stack overflow

Well we got a recurrence, but unfortunately the JavaScript engine’s strict (or eager) evaluation means that we must completely evaluate Y(FAC) before we can ever apply the returned function to (6).  
Therefore, we get an infinite loop - and actually it doesn’t matter what function we pass in to Y, it will never actually be called and any stopping condition will never be checked.  
How do we restore the laziness necessary to make progress in this recursion?

(**Hint:** it involves wrapping some part of `Y` in another lambda)

Did you get it?  If so, good for you!  If not, never mind, it is tricky and in fact was the subject of research papers at one time, so I’ll give you a bigger hint.

**Bigger hint:** there’s another famous combinator called `Z` which is basically `Y` adapted to work with strict evaluation:

```lambda
Z=λf.(λx.f(λv.xxv))(λx.f(λv.xxv))
```

---

### Exercises

* Note the similarities between `Y` and `Z` and perform a similar set of Beta reductions on `Z FAC` to see how it forces FAC to be evaluated.
* Write a version of the Z-Combinator in JavaScript such that `Z(FAC)(6)` successfully evaluates to `720`.

#### Solutions

Key ideas:

* The Z-combinator introduces an additional lambda (`λv.xxv`) to delay the evaluation of the recursive call. Note that eta-reducing `λv.xxv` into `xx` would give us the Y-combinator.
* This ensures that each step of the recursion only evaluates when needed, preventing infinite immediate recursion.

```javascript
const Z = f => (x => f(v => x(x)(v)))(x => f(v => x(x)(v)));
const FAC = f => n => n > 1 ? n * f(n - 1) : 1;
console.log(Z(FAC)(6)); // Should print 720
```

Let’s see how `Z FAC 2` would evaluate in lambda calculus:

```lambda
Z FAC 2
= (λf. (λx. f (λv.xxv)) (λx. f (λv.xxv))) FAC 2
= (λx. FAC (λv.xxv)) (λx. FAC (λv.xxv)) 2
= FAC (λv. (λx. FAC (λv.xxv)) (λx. FAC (λv.xxv)) v) 2
```

Notice how the `(λx. FAC (λv.xxv)) (λx. FAC (λv.xxv))` has not been evaluated yet since it is inside the `λv` lambda. (If we had used the Y-combinator, we would have infinite recursion here.)

Since 2 > 1, `FAC` returns `n * f(n - 1)`, and it is only in the `f(n - 1)` call that the `(λx. FAC (λv.xxv)) (λx. FAC (λv.xxv))` is evaluated to `FAC (λv.(λx. FAC (λv.xxv)) (λx. FAC (λv.xxv)) v)`:

```lambda
= 2 * (λv. (λx. FAC (λv.xxv)) (λx. FAC (λv.xxv)) v) 1
= 2 * (λx. FAC (λv.xxv)) (λx. FAC (λv.xxv)) 1
= 2 * FAC (λv.(λx. FAC (λv.xxv)) (λx. FAC (λv.xxv)) v) 1
```

Now, since we have reached the base case (n ≤ 1), `FAC` simply returns 1. `(λx. FAC (λv.xxv)) (λx. FAC (λv.xxv))` will not evaluate here since `FAC` returns 1 immediately:

```lambda
= 2 * 1
= 2
```

Here’s the same thing in JavaScript:

```js
Z(FAC)(2)
= (f => (x => f(v => x(x)(v)))(x => f(v => x(x)(v))))(FAC)(2)
= (x => FAC(v => x(x)(v)))(x => FAC(v => x(x)(v)))(2)
= FAC(v => (x => FAC(v => x(x)(v)))(x => FAC(v => x(x)(v)))(v))(2)
= 2 * (v => (x => FAC(v => x(x)(v)))(x => FAC(v => x(x)(v)))(v))(1)
= 2 * (x => FAC(v => x(x)(v)))(x => FAC(v => x(x)(v)))(1)
= 2 * FAC(v => (x => FAC(v => x(x)(v)))(x => FAC(v => x(x)(v)))(v))(1)
= 2 * 1
= 2
```

---

## Conclusion

If you want to dig deeper there is much [more written about Lambda Calculus encodings](https://www.seas.harvard.edu/courses/cs152/2015sp/lectures/lec07-encodings.pdf) of logical expressions, natural numbers, as well as the `Y` and `Z` combinators, and also [more about their implementation in JavaScript](https://benestudio.co/fixed-point-combinators-in-javascript/).  

However, the above description should be enough to give you a working knowledge of how to apply the three operations to manipulate Lambda Calculus expressions, as well as an appreciation for how they can be used to reason about combinators in real-world functional style curried code.  The other important take away is that the Lambda Calculus is a turing-complete model of computation, with Church encodings demonstrating how beta-reduction can evaluate church-encoded logical and numerical expressions and the trick of the Y-combinator giving us a way to perform loops.

## Glossary

*Lambda Calculus*: Model of computation developed in the 1930s by Alonzo Church, providing a complete model of computation similar to Turing Machines.

*Lambda expressions*: Functions written using the λ notation, e.g., λx.x, which are anonymous and can only take on other functions as values.

*Alpha equivalence*: Renaming variables in lambda expressions as long as the names remain consistent within the scope.

*Beta reduction*: Substituting the arguments of a function application into the function body.

*Eta conversion*: Substituting functions that simply apply another expression to their argument with the expression in their body. This is a technique in Haskell and Lambda Calculus where a function f x is simplified to f, removing the explicit mention of the parameter when it is not needed.

*Combinator*: A lambda expression with no free variables.

*Divergent lambda expressions*: Expressions that do not simplify when beta reduced, leading to infinite loops.
