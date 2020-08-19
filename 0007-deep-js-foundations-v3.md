# Deep JavaScript Foundations

In this series, we'll do a deep dive into the foundations of JavaScript. This series is geared towards the intrepid web developer who has a fair understanding of using JavaScript and wants to delve a bit more into its internals to see how it _really_ works. If you've been able to follow my [Mastering Hard Parts of JavaScript](https://dev.to/ryanameri/mastering-hard-parts-of-javascript-callbacks-i-3aj0) series, you're good to go!

The blog series is based on [Kyle Simpson's](https://twitter.com/getify?lang=en) excellent [Deep JavaScript Foundations v3](https://frontendmasters.com/courses/deep-javascript-v3/introduction/) course on Frontend Masters. The slides for the course are freely available [here](https://static.frontendmasters.com/resources/2019-03-07-deep-javascript-v2/deep-js-foundations-v2.pdf).

Though the content and the presentation are my own, this blog series owes a heavy debt to Getify for his excellent structure of material and for obviously being the _source of truth_ when it comes to JavaScript. I'd highly encourage everyone take the course if they can and to read his [You Don't Know JS](https://github.com/getify/You-Dont-Know-JS) book series.

<h3 align="center"> Whenever there's a divergence between what your brain thinks is happening, and what the computer does, that's where bugs enter code <em> - getify's law #17</em></h3><br>

If we want our code to have less bugs, we need to get better at understanding what the computer does. For web developers, that means understanding how JS works.

We'll cover the following "Pillars of JavaScript" in this series:

1. Types
   - Primitive Types
   - Abstract Operations
   - Coercion
   - Equality
   - Statically typed supersets e.g., TypeScript
2. Scope
   - Nested Scope
   - Hoisting
   - Closure
   - Modules
3. Objects
   - this
   - class {}
   - Prototypes
   - Object Oriented vs. OLOO

## Types

When it comes to JavaScript and types, misconceptions are aplenty. Two of the most common misconceptions that many newbies to JavaScript have are:

### ~~JavaScript has no types~~

Many newbies to JavaScript have heard that _"JavaScript has no types"_. This might confuse some people but this is actually incorrect. JavaScript has types, and types are an extremely important concept to understand properly. It's just that like most other dynamically types languages, _variables_ don't have types. It's _values_ that have types.

```js
let time = "15:30";
```

Here, the variable `time` does not have a type. However `"15:30"` is a _value_ of type _string_. Variables can hold values of any type.

### ~~In JavaScript everything is an object~~

This also is a common misconception (perhaps it was more common in earlier days) but in fact JavaScript has primitive types that are not objects, such as number and string. A feature of JavaScript is _boxing_, the ability for primitive types to _act_ like objects. This is what allows us to write a statement such as `"A common string".toUpperCase()` and have it return `"A COMMON STRING"` to us, it makes a primitive type temporarily act as an object. However it's important to know the difference between primitive types, so let's go over them quickly.

### Primitive Types

- undefined
- string
- number
- boolean
- object
- symbol
- BigInt
- null? (it's a type but the historical bug...)

- the `typeof` operator always returns a string.
- `undefined` and `undeclared` are entirely different concepts.
- There is also another concept (TDZ) or `uninitialised`. It was added in ES6.

### Special Values

#### NAN

- NaN: It's not "not a number". It's "invalid number".
- There is nothing you can do with NaN that will result in anything other than NaN.
- NaN is the only value in JS that is not equal to itself.
- isNaN() is first going to coerce the input into a number before determining if it is NaN
- Number.isNaN() does not perform that coercion. It was added in ES6. Better to check for NaN with this:

```js
isNaN("cat"); //true
Number.isNaN("cat"); // false
```

#### Negative Zero

```js
let trendRate = -0;
trendRate === -0; //true
trendRate.toString(); //returns "0", oops
trendrate === 0; //true; another historical bug.
// So now we have this method of checking for -0
Object.is(trendRate, -0); //true
Object.is(trendRate, 0); //false
//Object.is() was added in ES6 and is better way of checking
```

### Fundamental Objects

This used to be called "native functions" or "built-in objects".

Use the `new` keyword for:

- Object
- Array
- Function
- Date
- RegExp
- Error

Don't use `new` with:

- String
- Number
- Boolean

## Exercise 1 Polyfill Object.is()

### Instructions

1. `Object.is(..)` should take two parameters.
2. It should return `true` if the passed in parameters are exactly the same value (not just `===` -- see below!), or `false` otherwise.
3. For `NaN` testing, you can use `Number.isNaN(..)`, but first see if you can find a way to test without usage of any utility?
4. For `-0` testing, no built-in utility exists, but here's a hint: `-Infinity`.
5. If the parameters are any other values, just test them for strict equality.
6. You cannot use the built-in `Object.is(..)` -- that's cheating!
7. You have implemented the solution correctly if all of the console.log() print `true`

```js
Object.is = function ObjectIs() {};

// tests: Everything should output true
console.log(Object.is(42, 42) === true);
console.log(Object.is("foo", "foo") === true);
console.log(Object.is(false, false) === true);
console.log(Object.is(null, null) === true);
console.log(Object.is(undefined, undefined) === true);
console.log(Object.is(NaN, NaN) === true);
console.log(Object.is(-0, -0) === true);
console.log(Object.is(0, 0) === true);

console.log(Object.is(-0, 0) === false);
console.log(Object.is(0, -0) === false);
console.log(Object.is(0, NaN) === false);
console.log(Object.is(NaN, 0) === false);
console.log(Object.is(42, "42") === false);
console.log(Object.is("42", 42) === false);
console.log(Object.is("foo", "bar") === false);
console.log(Object.is(false, true) === false);
console.log(Object.is(null, undefined) === false);
console.log(Object.is(undefined, null) === false);
```

### Solution

My Solution:

{% codepen https://codepen.io/ryanameri/pen/LYNRVBV default-tab=js %}

Kyle's Solution:

{% codepen https://codepen.io/ryanameri/pen/JjXRdxo default-tab=js %}

## Abstract Operations: Turning Fundamental Objects into Primitives

Many built-in functions in JavaScript expect a primitive type as value, so when for example we `console.log` an object or we alert a number, these functions need to have a way to turn them into strings. This is where abstract operations come in.

There are 3 types (hints) of abstract operations:

- "string" (for alert and other operations that need a string)
- "number" (for maths)
- "default"

The specification describes explicitly which operator uses which hint. There are very few operators that “don’t know what to expect” and use the "default" hint. Usually for built-in objects "default" hint is handled the same way as "number", so in practice the last two are often merged together.

The conversion algorithm is:

- If the object has a `obj.toPrimitive(hint)` method, call that.
- Otherwise, if hint is "string", call `obj.toString()` and `obj.valueOf()`, in that order.
- Otherwise if hint is "number" or "default", call `obj.valueOf()` and `obj.toString()`, in that order.
- toBoolean is a simple lookup table. If a [falsy value](https://developer.mozilla.org/en-US/docs/Glossary/Falsy) is encountered, it will convert to `false`, everything else is converted to `true`. All objects are truthy.

There are many corner cases with abstract operations so we have to be extremely careful using them in production.

You can find more information on abstract operations [here](https://javascript.info/object-toprimitive).

## Boxing

Boxing can be thought of as the opposite of Abstract Operations. You can access properties of primitives and JavaScript makes them act like objects. In effect, boxing is a form of implicit coercion (primitive to object).

Example of boxing:

```js
console.log("testing".toUpperCase());
```

## Intentional/Explicit Coercion

We'll use the terms coercion and conversion interchangeable here, though from a CS point of view there can be minor differences between the two. This concept is also called "casting" in some other languages.

Unlike what some others advocate, as a web developer, you cannot simply avoid coercion and conversion. You have to embrace it, making sure that the types involved in any operation are clear. Yes there are corner cases, but they can be safely managed by learning them, instead of avoiding them.

The `===` is not a magic bullet that fixes all conversions. Implicit coercion is everywhere in JavaScript, for example:

```js
let num = 4;
console.log(`This number is ${num}`);
```

Can you guess what kind of coercion is happening in this simple console.log? As it turns out, console.log only outputs string, so `num` is being implicitly converted to a string.

You could of course write:

```js
let num = 4;
console.log(`This number is ${String(num)}`);
```

To be more explicit, but this level of being explicit adds verbosity without adding much value. There is a balance of clarity vs. verbosity that needs to be struck.

- Implicit does not mean "magical". Implicit is not in itself bad. We need to think of implicitness as an abstraction. Not all abstractions are good, but some abstractions _are_.

Those familiar with functional programming know that abstractions allow us to hide unnecessary details by re-focusing the reader and increasing clarity. Implicit coercion is very similar, a level of abstraction.

However let's take a look at a different example:

```js
let item1 = document.getElementById(item1);
let item2 = document.getElementById(item2);

if (item1 < item2>){
    //Do something
}
```

Here, interfacing with the outside word (the DOM), you might not have a perfect idea of what types item1 or item2 might be. If they are input tags in an html form, their type might be string, or it might be `null` or something else. If they are string, the `<` operator will perform an alphabetic comparison, which is probably not what we want and will lead to unintentional consequences and bugs. So it would be helpful to write:

```js
let item1 = document.getElementById(item1);
let item2 = document.getElementById(item2);

if (Number(item1) < Number(item2)>){
    //Do something
}
```

To ensure that our comparison is working as intended.

The question we have to ask about when to use explicit coercion, is does it add clarity to the reader? If the type is absolutely clear and adding something like `String()` just adds verbosity, we should avoid it. However if the type is not always clear, adding explicit type conversion will aid in readability and reducing bugs.

## Exercise 2

### Instructions

In this exercise, you will define some validation functions that check user inputs (such as from DOM elements). You'll need to properly handle the coercions of the various value types.

1. Define an `isValidName(..)` validator that takes one parameter, `name`. The validator returns `true` if all the following match the parameter (`false` otherwise):

- must be a string
- must be non-empty
- must contain non-whitespace of at least 3 characters

2. Define an `hoursAttended(..)` validator that takes two parameters, `attended` and `length`. The validator returns `true` if all the following match the two parameters (`false` otherwise):

- either parameter may only be a string or number
- both parameters should be treated as numbers
- both numbers must be 0 or higher
- both numbers must be whole numbers
- `attended` must be less than or equal to `length`

```js
// TODO: write the validation functions

// tests:
console.log(isValidName("Frank") === true);
console.log(hoursAttended(6, 10) === true);
console.log(hoursAttended(6, "10") === true);
console.log(hoursAttended("6", 10) === true);
console.log(hoursAttended("6", "10") === true);

console.log(isValidName(false) === false);
console.log(isValidName(null) === false);
console.log(isValidName(undefined) === false);
console.log(isValidName("") === false);
console.log(isValidName("  \t\n") === false);
console.log(isValidName("X") === false);
console.log(hoursAttended("", 6) === false);
console.log(hoursAttended(6, "") === false);
console.log(hoursAttended("", "") === false);
console.log(hoursAttended("foo", 6) === false);
console.log(hoursAttended(6, "foo") === false);
console.log(hoursAttended("foo", "bar") === false);
console.log(hoursAttended(null, null) === false);
console.log(hoursAttended(null, undefined) === false);
console.log(hoursAttended(undefined, null) === false);
console.log(hoursAttended(undefined, undefined) === false);
console.log(hoursAttended(false, false) === false);
console.log(hoursAttended(false, true) === false);
console.log(hoursAttended(true, false) === false);
console.log(hoursAttended(true, true) === false);
console.log(hoursAttended(10, 6) === false);
console.log(hoursAttended(10, "6") === false);
console.log(hoursAttended("10", 6) === false);
console.log(hoursAttended("10", "6") === false);
console.log(hoursAttended(6, 10.1) === false);
console.log(hoursAttended(6.1, 10) === false);
console.log(hoursAttended(6, "10.1") === false);
console.log(hoursAttended("6.1", 10) === false);
console.log(hoursAttended("6.1", "10.1") === false);
```

### Solutions

My Solution:

{% codepen https://codepen.io/ryanameri/pen/BaKLzLV default-tab=js %}

Kyle's Solution:

{% codepen https://codepen.io/ryanameri/pen/QWNKEGz default-tab=js %}
