# Deep JavaScript Foundations

In this series, we'll do a deep dive into the foundations of JavaScript. This series is geared towards the intrepid web developer who has a fair understanding of using JavaScript and wants to delve a bit more into its internals to see how it _really_ works. If you've been able to follow my [Mastering Hard Parts of JavaScript](https://dev.to/ryanameri/mastering-hard-parts-of-javascript-callbacks-i-3aj0) series, you're good to go!

- What exactly is the difference between `==` and `===`? _(hint: if you think that `==` doesn't check the types, you are wrong!)_
- Why was `let` and `const` introduced in JavaScript and what do they actually do?
- What's the difference between a function declaration and function expression? What is this "hoisting" you keep seeing and why is it different between a function declaration and a function expression?
- Where should you use arrow functions and where should you avoid them?

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
- null? (it's a type but the historical bug...) (perhaps write a note about JavaScript backwards compatibility here)

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

## Exercise 1: Polyfill Object.is()

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

### Exercise 1 Solution

{% codepen https://codepen.io/ryanameri/pen/LYNRVBV default-tab=js %}

### Exercise 1 Kyle's Solution

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

## Exercise 2: Coercion

> In this exercise, you will define some validation functions that check user inputs (such as from DOM elements). You'll need to properly handle the coercions of the various value types.
>
> 1. Define an `isValidName(..)` validator that takes one parameter, `name`. The validator returns `true` if all the following match the parameter (`false` otherwise):
>
> - must be a string
> - must be non-empty
> - must contain non-whitespace of at least 3 characters
>
> 2. Define an `hoursAttended(..)` validator that takes two parameters, `attended` and `length`. The validator returns `true` if all the following match the two parameters (`false` otherwise):
>
> - either parameter may only be a string or number
> - both parameters should be treated as numbers
> - both numbers must be 0 or higher
> - both numbers must be whole numbers
> - `attended` must be less than or equal to `length`

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

### Exercise 2 Solution

{% codepen https://codepen.io/ryanameri/pen/BaKLzLV default-tab=js %}

### Exercise 2 Kyle's Solution

{% codepen https://codepen.io/ryanameri/pen/QWNKEGz default-tab=js %}

## Double and Triple Equals

Most of us have heard that `==` (or so called loose equality) checks the value whereas `===` (or so called strict equality) checks both value and the type. Unfortunately, that's not exactly true!

If you're trying to understand your code, it's critical to learn how JavaScript thinks!

Both double and triple equality check the type, it's just that they do different things with that information.

TODO: expand by pointing to the [spec](https://www.ecma-international.org/ecma-262/11.0/index.html#sec-abstract-equality-comparison).

- If the types are the same, the double equal and triple equal do exactly the same thing.
- With triple equal, if the types are different, it will always return false without even checking the value. It will only check the value only if the types are the same.(remember that triple equal lies about NaN and -0).
- The double equal will allow coercion when the types are not equal.
- Like every other situation, we have to consider whether implicit coercion is helpful or not.
- null can be coerced to undefined and vice versa, so double equal will treat them the same. It can be helpful to have one "empty" type, as opposed to two. In which case the use of double equal can be helpful.

So given:

```js
let item1 = { topic: null };
let item2 = {};
if (
  (item1.topic === null || item1.topic === undefined) &&
  (item2.topic === null || item2.topic === undefined)
) {
  //Do something
}
```

If we are to restrict ourselves to only using the triple equal operator, we'd have to write something like above to check to see if our two objects have a property `topic`. If however we understand our types, we can use the implicit coercion that the double operator gives us to write a much more readable version such as:

```js
let item1 = { topic: null };
let item2 = {};
if (item1.topic == null && item2.topic == null) {
  //Do something
}
```

- Another thing to remember is that double equal general prefers numbers, so if one operand is not a number, it will try and coerce it into one.

This implicit coercion can be useful in writing readable code such as above. It can also be unhelpful when the coercion is almost "by accident". Let's consider this:

```js
let item1 = 42;
let item2 = [42];
if (item1 == item2) {
  // this will execute but don't write code like this!
}
```

This conditional statement will return true, but why? Why does it work?

The double equal is only going to coerce primitives. If it is given a fundamental object (such as an array), it will use abstract operations to turn it into a primitive.

How is an array turned into a primitive type? It's turned into a string. So `[42]` becomes `"42"`. And then as discussed, the double equality prefers to coerce values into numbers, so `"42"` becomes `42`. Now that both types are the same (numbers) it compares the values, and since they are equal, it returns true.

But this type of coercion is almost an accident of JS's heritage and certainly does not help in readability of code. The JavaScript spec could have been written to stringify an array as `"[42]"` and then the statement would have been `false` (and many would argue that that form of stringifying would have been a better behaviour, alas we cannot go back and change how JavaScript works; refer to section on backwards compatibility).

Implicit coercion should not be avoided at all costs, instead we need to make our types clear in our code, and then leverage coercion, like any other abstraction, when it helps our code's readability and maintainability.

Kyle Simpson argues that double equals is preferable to triple equals in almost all cases. I'm not going to go that far, but I do believe that we need to know our types when coding, and when we do know our types and our code is structured so that we take care of corner cases, then double equals can live happily side by side triple equals and should be used when its implicit coercion helps readability.

## Exercise 3: Equality

> In this exercise, you will define a `findAll(..)` function that searches an array and returns an array with all coercive matches.
>
> 1. The `findAll(..)` function takes a value and an array. It returns an array.
>
> 2. The coercive matching that is allowed:
>
> - exact matches (`Object.is(..)`)
> - strings (except "" or whitespace-only) can match numbers
> - numbers (except `NaN` and `+/- Infinity`) can match strings (hint: watch out for `-0`!)
> - `null` can match `undefined`, and vice versa
> - booleans can only match booleans
> - objects only match the exact same object

```js
// TODO: write `findAll(..)`

// tests:
var myObj = { a: 2 };

var values = [
  null,
  undefined,
  -0,
  0,
  13,
  42,
  NaN,
  -Infinity,
  Infinity,
  "",
  "0",
  "42",
  "42hello",
  "true",
  "NaN",
  true,
  false,
  myObj,
];

console.log(setsMatch(findAll(null, values), [null, undefined]) === true);
console.log(setsMatch(findAll(undefined, values), [null, undefined]) === true);
console.log(setsMatch(findAll(0, values), [0, "0"]) === true);
console.log(setsMatch(findAll(-0, values), [-0]) === true);
console.log(setsMatch(findAll(13, values), [13]) === true);
console.log(setsMatch(findAll(42, values), [42, "42"]) === true);
console.log(setsMatch(findAll(NaN, values), [NaN]) === true);
console.log(setsMatch(findAll(-Infinity, values), [-Infinity]) === true);
console.log(setsMatch(findAll(Infinity, values), [Infinity]) === true);
console.log(setsMatch(findAll("", values), [""]) === true);
console.log(setsMatch(findAll("0", values), [0, "0"]) === true);
console.log(setsMatch(findAll("42", values), [42, "42"]) === true);
console.log(setsMatch(findAll("42hello", values), ["42hello"]) === true);
console.log(setsMatch(findAll("true", values), ["true"]) === true);
console.log(setsMatch(findAll(true, values), [true]) === true);
console.log(setsMatch(findAll(false, values), [false]) === true);
console.log(setsMatch(findAll(myObj, values), [myObj]) === true);

console.log(setsMatch(findAll(null, values), [null, 0]) === false);
console.log(setsMatch(findAll(undefined, values), [NaN, 0]) === false);
console.log(setsMatch(findAll(0, values), [0, -0]) === false);
console.log(setsMatch(findAll(42, values), [42, "42hello"]) === false);
console.log(setsMatch(findAll(25, values), [25]) === false);
console.log(
  setsMatch(findAll(Infinity, values), [Infinity, -Infinity]) === false
);
console.log(setsMatch(findAll("", values), ["", 0]) === false);
console.log(setsMatch(findAll("false", values), [false]) === false);
console.log(setsMatch(findAll(true, values), [true, "true"]) === false);
console.log(setsMatch(findAll(true, values), [true, 1]) === false);
console.log(setsMatch(findAll(false, values), [false, 0]) === false);

// ***************************

function setsMatch(arr1, arr2) {
  if (
    Array.isArray(arr1) &&
    Array.isArray(arr2) &&
    arr1.length == arr2.length
  ) {
    for (let v of arr1) {
      if (!arr2.includes(v)) return false;
    }
    return true;
  }
  return false;
}
```

### Exercise 3 Solution

{% codepen https://codepen.io/ryanameri/pen/OJNbbQR default-tab=js %}

### Exercise 3 Kyle's Solution

{% codepen https://codepen.io/ryanameri/pen/abNBBqP default-tab=js %}

Full disclosure: I struggled for over an hour on this exercise. I ended up using the [Array.prototype.filter](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/filter) method to cycle through the array items, Kyle constructs the array manually and loops through it. Both methods are very similar and the the logic is pretty much identical, but that's due to the corner cases involved and the structure of the tests.

The most important lesson to take away from this exercise is that, if you want to be absolutely sure that `Y` is equal to `X` and only `X`, your best option is to use `Object.is()` because any other comparison operator, including `===` has edge cases. Going through this exercise makes you aware of pretty much all these corner cases in JavaScript when dealing with coercing values from one type to the other. This should build up your confidence to not be afraid of using the double equality `==` operator, when it makes sense.

## TypeScript, Flow and Type-aware linting

If you are coding and you don't know your types, that is a problem. There are many different ways of solving this problem, we could solve it using style guides and linting, or we can use tools such as TypeScript.

Benefits of TypeScript

- Catch type-related mistakes
- Communicate type intent
- Allow the tooling ecosystem including VS Code to provide great IDE feedback

Caveats:

- Inferencing is best-guess, not a guarantee (unless you disable `any` type), so not a guarantee; which can lead to false sense of security
- Annotations are optional (again unless you disable `any` type)
- Any part of the codebase that isn't typed reduces uncertainty

For a list of similarities and differences between TypeScript and flow, have a look at [this article](https://github.com/niieani/typescript-vs-flowtype).

### Static typing pros

- Make the types much more obvious
- They look like other languages with static type system such as Java
- Extremely popular these days and hav strong corporate backing. The investment to learn and get into them is an investment that will pay off for a long time.
- Very sophisticated and good at inferring types

### Static typing cons

- They use a non-js-standard syntax (or code comments). Maybe types would be added someday to JavaScript, but if they are, it would probably look very different to TypeScript. This could be similar to ES Modules vs Common JS.
- They require (yet another) build process, which raises the barrier to entry.
- The type system can be intimidating to those without prior formal types experience. Once you get into advanced TypeScript with generics, the syntax will look closer to Haskell than to traditional JavaScript, again raising the barrier to entry.
- Fundamentally against the _grain_ of JavaScript, focusing on static types instead of value types.

There is a big divide there, between the things that TypeScript gives you vs how much it raises the barrier to entry and adds to tooling requirements. Static typing is not the _only_ option to knowing our types, it is _a_ solution to making sure that we know our types.

## Summary of Types

Whether you achieve that using static type system such as TypeScript, or we could achieve it using a strong style guide with the help of a configurable linter that enforces that strong type and embracing JavaScript's types and coercion.

Code with clear types is more readable and more robust, for experienced and new developers. **tl;dr You cannot write quality JS programs without knowing the types involved**.

Next up, we'll delve into _scope_, and all that _scope_ enables in JavaScript: hoisting, closure, modules.

## Scope

This chapter will delve more deeply into:

- Nested Scope
- Hoisting
- Closure
- Modules

But first, what does scope mean?

**Scope: where to look for things**.

Every time we reference a variable, we are either reading its value, or assigning a new value to it. But first we need to figure out where to find the variable! And that requires a level of understanding of how JavaScript is _parsed_, and how it is _executed_.

### JavaScript is a Compiled Language

It is very common for people to think of JavaScript as a "scripting" language, i.e., statements are executed line by line. This is actually incorrect; JavaScript is a _parsed_ language, which is a form of _compilation_.

To test this, execute the following JS code:

```js
let n = 10;
if (n < 10>)
    console.log("it is a one digit number");
else
    console.log("it is a two digit number");
consle.log("Typo here!")
```

Now run the following equivalent bash script:

```sh
#!/bin/bash
n=10
if [ $n -lt 10 ];
then
echo "It is a one digit number"
else
echo "It is a two digit number"
fi
ech "Typo here!"
```

The JavaScript example won't run at all! It will throw a syntax error due to the typo. However the equivalent program, written in a traditional scripting language will print `"It is a two digit number"` first before running into the typo and showing an error as it executes statements one by one and won't see the error until it encounters it.

How can JavaScript see the error even before it has run a single statement? Because the whole program is _parsed_ first by the JS engine, making JavaScript a form of a _compiled_ language. Yes, as it happens, both parts of the name of the language, the _Java_ part and the _Script_ part are misnomers. Think of the name really as another historical bug, similar to `typeof null` which returns `"object"` 😉.

### JavaScript's Execution Model

We won't delve into the details of JavaScript execution too much, but a simplified overview of it can be thought of as:

1. JS engine is given the source code in a .js file
2. JS engine parses the source code and turns it into an "Abstract Syntax Tree" (AST). This is the first processing.
3. The AST is given to a virtual machine, which translates it to machine code in some fashion, either directly compiling it or using JIT. This is the second processing.
   ![Sample V8 Execution Path](assets/v8-execution.png)

So when we think of JavaScript execution, we have to think of our code being processed twice. Once it is parsed and turned into AST, and then the AST is executed and turned into machine code.

It's pretty important to keep this mental model in mind in order to understand how scope works.

When the JS engine is parsing our code (first processing), it effectively divides our code into _chunks_. Traditionally in JavaScript, only functions were broken into chunks. ES6 also introduced _blocks_ (represented by `{}`) which are also broken into chunks. So in modern JavaScript, each block or functions is a chunk, and each chunk is a unit of scope.

Next when the JS engine translates the AST into machine code (second processing) and encounters a variable, it can only make sense of it if the variable is defined in the current chunk (i.e., the variable is in scope). If it encounters a variable that is not in the current chunk, i.e., it is not in scope, it throws an error (in strict mode) or results in unintended behaviour (non-strict mode). <small>(We'll talk about strict vs. non-strict mode, and about scope nesting in later sections.)</small>

### Scope Demo

To understand this execution model, let's look at this simple demo:

```js
function firstFn() {
  var name = "Alice";
  console.log(name);
}

function secondFn() {
  var name = "Ali";
  console.log(name);
}

var name = "Ryan";
console.log(name);
firstFn();
secondFn();
```

The first thing to note is that in JavaScript, everything is put into a chunk called _global_. So when the JS engine parses our code (first processing), it divides our code into three chunks: the global chunk, the firstFn chunk and the secondFn chunk.

Next when the JS engine executes our code (second processing), it looks into the global chunk and encounters the variable `name`. It checks to see if the variable is defined in the current chunk. It is, so it reads its value `"Ryan"` and prints it.

In the next statement, we tell the JS engine to go and look into the firstFn chunk. It runs the firstFn chunk and encounters the variable `name` there. It checks to see if it is defined in the current chunk. It is, so it reads its value `"Alice"` and prints it.

When the execution of firstFn chunk is finished, the JS engine returns back to the global chunk and remembers where it was and continues from where it left off. Next we instruct it to go and execute the chunk secondFn. It goes into the secondFn chunk and encounters the variable `name` there. It checks to see if the variable is defined in the current chunk. It is, so it takes its value `"Ali"` and prints it.

So the output is:

```
Ryan
Alice
Ali
```

We'll cover other cases (for example what happens if the JS engine doesn't find the variable in the current chunk, and what effects `let` and `const` have vs `var`) soon, but for now, if you can follow how this code is divided into three chunks when it is parsed, and how each chunk is then executed by the JS engine, you've effectively got the fundamentals of scope.

### Scope Nesting

We hinted at nesting of scopes (chunks) in our previous example when we looked the global scope. In JavaScript (as in most other languages) chunks don't have to follow each other serially, they can be inside one another. So a bigger chunk can contain many smaller chunks, each of which could contain smaller chunks.

The biggest chunk of them all is _global_ inside which everything else resides. In the previous example, we saw that `firstFn` and `secondFn` were chunks inside global.

With this mental model in mind, it's simple to explain scope nesting: What does the JS engine do if it encounters a variable and it's not been defined in that chunk? It looks up to the outside/container chunk to see if the variable was defined there. If it was, great, it can access the variable. If not, it will continue looking at the container chunk all the way until it reaches _global_ and checks there.

So let's look at this example:

```js
function firstFn() {
  console.log(name);
}

var name = "Ryan";
console.log(name);
firstFn();
```

When the code is parsed, it is divided into two chunks, a _global_ chunk and a `firstFn` chunk that resides inside _global_. Then when the program is executed, the JS engine runs the _global_ chunk and encounters the variable `name`. It checks to see if `name` is defined in the current chunk. It is so it reads its value `"Ryan"` and prints it. Next it executes the `firstFn` chunk. Inside the firstFn chunk the JS engine encounters the variable `name`. It checks to see if it is defined in the current chunk. It is not, so the JS engine looks at the container chunk (which here is _global_) to see if `name` was defined there. It is, so it reads its value and prints it. So the output is:

```
Ryan
Ryan
```

### Automatic _Global_ variable creation

Here we encounter one of JavaScript's really _bad parts_, automatic global variable creation. To understand it, let's look at this example:

```js
function firstFn() {
  name = "Alice";
  console.log(name);
  profession = "doctor";
  console.log(profession);
}

var name = "Ryan";
console.log(name);
firstFn();
console.log(name);
console.log(profession);
```

As with the previous example, when the code is parsed, it is divided into two chunks, a _global_ chunk and a `firstFn` chunk that resides inside _global_. Then when the program is executed, the JS engine runs the _global_ chunk and encounters the variable `name`. It checks to see if `name` is defined in the current chunk. It is so it reads its value `"Ryan"` and prints it. Next it executes the `firstFn` chunk. Inside the firstFn chunk the JS engine encounters the variable `name`. It checks to see if it is defined in the current chunk. It is not, so the JS engine looks at the container chunk (which is _global_) to see if `name` was defined there. It is, so it changes its value to `"Alice"` and prints it.

Next it encounters the variable `profession`. It checks to see if it is declared in the current chunk, it is not. It then looks outside into the container chunk to see if it was defined there. It is not. Here is where things get messy! When JavaScript was devised in the early 90s, its designers wanted programmers to face as few errors as possible, so instead of throwing an error here, they said, once you are in the global scope and if you are accessing a variable that has not been defined, we'll helpfully create it for you (behind the scenes) so you can use it. As a result of this line, a variable called `profession` is created in _global_ scope, and the value `"doctor"` is assigned to it, and it is printed.

We then head back into global chunk. The variable `name` is read, and since its value was changed to `"Alice"`, this time `"Alice"` is printed. Then the variable `profession` is read, and since it was created in global scope and its value set to `"doctor"`, that is printed. So the output is:

```
Ryan
Alice
doctor
Alice
doctor
```

This automatic global variable creation is rather unfortunate, since instead of being helpful as the original designers intended, it leads to a lot of bugs and a lot of unintended side effects. But unfortunately we can't fix it since fixing it we'll break JavaScript's backwards compatibility. **NEVER** intentionally declare a variable in this method.

We'll see how to tame this behaviour by using _strict mode_ in the next section.

### Strict Mode

Let's consider the example in the previous section, with one tiny modification:

```js
"use strict";
function firstFn() {
  name = "Alice";
  console.log(name);
  profession = "doctor";
  console.log(profession);
}

var name = "Ryan";
console.log(name);
firstFn();
console.log(name);
console.log(profession);
```

Strict mode was added to JavaScript in ES5. When we add the string `"use strict"` as the first line of our program, we instruct the JS engine to disable this automatic global variable creation behaviour.

The parsing and execution chain is exactly the same as in the previous section so I won't repeat it here. The program is divided into two chunks, _global_ and `firstFn`.And the first two lines, `"Ryan"` and `"Alice"` are printed exactly as before. However next we come to the line `profession = "doctor";`. Here the JS engine encounters the variable `profession`. It checks to see if has been declared in the current chunk. It isn't so it moves to the container chunk _global_ to see if it is defined there. It isn't. Now instead of automatically creating the variable in global scope, the JS engine will throw an error.

This behaviour is much preferred and you should always use strict mode when creating new code. ES6 enabled the JS engine to use strict mode in some other contexts as well even without the explicit `"use strict"` directive. Some tools such as babel will also implicitly insert "use strict" into your code so if you are using such transpilers, your code will most probably run in strict mode. But JavaScript is itself run in non-strict (a.k.a sloppy) mode, and that behaviour will probably never change to preserve backwards compatibility with old code.

### Function Expression vs. Declaration

Up to this point, when we've been talking about scope, we've only addressed variables. But in JavaScript, functions are also identifiers so the rules of scope, i.e., what is defined where apply to functions as well in exactly the same way. However there the rules of scope differ when a function is declared vs. when a function is expressed. Let's have a look at what each one means.

#### Function Declaration

The examples we have used up to this point have all used function declaration. To demonstrate, the following example:

```js
function firstFn() {
  console.log(name);
}

var name = "Ryan";
console.log(name);
firstFn();
```

The functions `firstFn` is here declared and it gets its own scope, as we have discussed.

#### Function Expression

```js
var fun = function firstFn() {
  firstFn(); // Works
  console.log(name);
};

var name = "Ryan";
console.log(name);
fun();
firstFn(); /// Error
```

This style of writing functions (which is increasingly popular and generally used with the `const` keyword as opposed to `var`) is called function expression. The key thing to note here is that while `fun` is in _global_ scope, `firstFn` gets its own scope and is not in _global_ scope. So while we can access `furstFn` from inside its own scope, if we were to reference `firstFn` in global scope, we would get an error.

This method of expressing a function is called a _Named Function Expression_. Since we seldom use the name of the expressed function, we can also omit it and create an _Anonymous Function Expression_ such as:

```js
var fun = function () {
  console.log(name);
};

var name = "Ryan";
console.log(name);
fun();
```

As our function is now doesn't have a name, we can't reference it even from within its own scope. ES6 introduced the arrow syntax which allows us to write anonymous functions as:

```js
var fun = () => {
  console.log(name);
};

var name = "Ryan";
console.log(name);
fun();
```

Dropping the `function` keyword altogether <small>(arrow functions do change some other behaviours in terms of binding and the `this` keyword which we'll cover in a later section)</small>.

While this style is shorter and increasingly popular, there are benefits to naming our functions:

1. Allows us to self-reference the function (used in recursion or to access properties of the function).
2. More debuggable stack traces (though modern debugging tools do a lot of inferencing)
3. More self-documenting code. Makes the purpose of the function explicit.

## Exercise 4: Function Expressions

> ### Part 1
>
> In this exercise, you will be writing some functions and function expressions, to > manage the student enrolment records for a workshop.
>
> **Note:** The spirit of this exercise is to use functions wherever possible and > appropriate, so consider usage of array utilities `map(..)`, `filter(..)`, `find(..)`, > `sort(..)`, and `forEach(..)`.
>
> **Note:** In Part 1, use only function declarations or named function expressions.
>
> You are provided three functions stubs -- `printRecords(..)`, `paidStudentsToEnroll()>`, and `remindUnpaid(..)` -- which you must define.
>
> At the bottom of the file you will see these functions called, and a code comment > indicating what the console output should be.
>
> 1. `printRecords(..)` should:
>
>    - take a list of student Ids
>    - retrieve each student record by its student Id (hint: array `find(..)`)
>    - sort by student name, ascending (hint: array `sort(..)`)
>    - print each record to the console, including `name`, `id`, and `"Paid"` or `"Not > Paid"` based on their paid status
>
> 2. `paidStudentsToEnroll()` should:
>
>    - look through all the student records, checking to see which ones are paid but > **not yet enrolled**
>    - collect these student Ids
>    - return a new array including the previously enrolled student Ids as well as the > to-be-enrolled student Ids (hint: spread `...`)
>
> 3. `remindUnpaid(..)` should:
>    - take a list of student Ids
>    - filter this list of student Ids to only those whose records are in unpaid status
>    - pass the filtered list to `printRecords(..)` to print the unpaid reminders
>
> ### Part 2
>
> Now that you've completed Part 1, refactor to use **only** `=>` arrow functions.
>
> For `printRecords(..)`, `paidStudentsToEnroll()`, and `remindUnpaid(..)`, assign these > arrow functions to variables of such names, so that the execution still works.
>
> As the appeal of `=>` arrow functions is their conciseness, wherever possible try to > use only expression bodies (`x => x.id`) instead of full function bodies (`x => { > return x.id; }`).

```js
function printRecords(recordIds) {
  // TODO
}

function paidStudentsToEnroll() {
  // TODO
}

function remindUnpaid(recordIds) {
  // TODO
}

// ********************************

var currentEnrollment = [410, 105, 664, 375];

var studentRecords = [
  { id: 313, name: "Frank", paid: true },
  { id: 410, name: "Suzy", paid: true },
  { id: 709, name: "Brian", paid: false },
  { id: 105, name: "Henry", paid: false },
  { id: 502, name: "Mary", paid: true },
  { id: 664, name: "Bob", paid: false },
  { id: 250, name: "Peter", paid: true },
  { id: 375, name: "Sarah", paid: true },
  { id: 867, name: "Greg", paid: false },
];

printRecords(currentEnrollment);
console.log("----");
currentEnrollment = paidStudentsToEnroll();
printRecords(currentEnrollment);
console.log("----");
remindUnpaid(currentEnrollment);

/* Output Should be: 
	Bob (664): Not Paid
	Henry (105): Not Paid
	Sarah (375): Paid
	Suzy (410): Paid
	----
	Bob (664): Not Paid
	Frank (313): Paid
	Henry (105): Not Paid
	Mary (502): Paid
	Peter (250): Paid
	Sarah (375): Paid
	Suzy (410): Paid
	----
	Bob (664): Not Paid
	Henry (105): Not Paid
*/
```

### Exercise 4 Solutions

#### My Solution Part 1

{% codepen https://codepen.io/ryanameri/pen/mdPOvVZ default-tab=js %}

#### My Solution Part 2

{% codepen https://codepen.io/ryanameri/pen/NWNboNN default-tab=js %}

#### Kyle's Solution Part 1

{% codepen https://codepen.io/ryanameri/pen/poyNGye default-tab=js %}

#### Kyle's Solution Part 2

{% codepen https://codepen.io/ryanameri/pen/Vwamgag default-tab=js %}

As can be seen, each solution has its own pros and cons. Using arrow functions and anonymous functions certainly condenses the code and more importantly it inserts the _utility_ function exactly where the action is happening, so our eyes don't have to dart across different parts of the page to follow the execution.

However giving an explicit name to our functions allows us to take a moment to ponder what our function's job actually is, and that can lead to us writing more succinct and single-purpose functions. It also aids in debugging by allowing us to follow the stack trace easier. Finally, declaring functions can decouple the logic of the function from the nitty gritty details of its implementation. For example if you compare the `printRecords` function in my two solutions, you can see that in solution 1 it's much easier to follow the logic since there are only a few high level instructions and the details of for example how each student record is printed is hidden away (in a function in the utility section). This allows us to be able to follow the logic of our program easier, and also enables us to refactor our utility functions later on in an easier fashion since they stand by themselves.

Each method has its own benefits and drawbacks, and as always the _right_ solution is probably somewhere in between. But the increasing popularity of arrow functions, where some people think that functions _only_ have to be written as arrow functions and using the _function_ keyword is somehow deprecated or discouraged is also incorrect. Declaring our functions separately and naming them explicitly has certain merits. As always, engineering is a series of making decisions between various trade offs, and it's upon us as authors of our code to make the right call on where the balance is in each case.

## Lexical & Dynamic Scope

### Lexical Scope

In computer science, there are two types of scope: _Lexical Scope_ (a.k.a static scope) and _Dynamic Scope_. Most programming languages including JavaScript have implemented scope as lexically, which means that scope is determined at compile time. As the author of code, you dictate scope by where you declare your variables and how you nest functions (or blocks) inside each other. When the parser goes through the source code (first processing), it determines the scope of each variable based on this information. At runtime (second processing) a variable's scope is already pre-determined.

Lexical scope has the advantage that scope is generally much easier to follow (and debug) in source code, as well as performance benefits. By being able to determine the scope beforehand by parsing the code, significant optimisations can be made by the JS engine to our code.

### Lexical Scope Demo

Let's have a look at a simple example:

```js
let str = "begin";
console.log(`start: str=${str}`);

function inner() {
  console.log(`inner: str=${str}`);
}

function outer() {
  let str = "outer";
  inner();
}

outer();
console.log(`end: str=${str}`);
```

This should look familiar to us as there's nothing fundamentally new. What do you think will be the output of this code? It is:

```
"start: str=begin"
"inner: str=begin"
"end: str=begin"
```

The scope of the variable `str` is determined by where the variable is declared. So when we reference the variable in the `inner` function, the JS engine looks in the scope of inner to see if it can find `str`. It can't, so it looks at the outer scope which is _global_ and finds the variable there and takes its value which is "begin" and prints it.

In the case of lexical scoping, we have already pre-determined the value of our variables based on where they are declared in our code, and the rules of nested scoping.

### Dynamic Scope

Some computer languages (such as bash or early versions of Lisp) have implemented scope dynamically, where scope is determined at run time not based on where a variable has been declared, but based on the value of the variable in the function that has called the current function that is being executed. This sounds complicated but a simple example should clarify it.

### Dynamic Scope Demo

The following bash script is equivalent to the previous JavaScript code, so don't worry if you don't know bash/shell scripts, you should be able to follow it:

```sh

str="begin"

echo "start: str=$str"

function inner {
	echo "inner: str=$str"
}

function outer {
	local str="outer"
	inner
}

outer

echo "end: str=$str"
```

The output would be:

```
start: str=begin
inner: str=outer
end: str=begin
```

Because the value of `str` in the `inner` function is determined not based on what its value is in the _global_ scope, but based on what its value is in the function that called `inner`.

Dynamic Scope has the advantage that it gives our functions a lot more flexibility. Our function can basically behave differently based on where it was called from, making them much more reusable. Dynamic scope however is generally harder to debug since most programmers aren't accustomed to its intricacies. It is also pretty impossible to optimise therefore suffers performance penalties.

JavaScript is a lexically scoped language (unlike what some might say). However it does have mechanisms (namely closure) that enable it to have many of the benefits of dynamically scoped languages while still being optimisable and performant. We'll cover closure more in a future section. For those who want to know about dynamic vs lexical scope, Wikipedia has a [good easy explanation](<https://en.wikipedia.org/wiki/Scope_(computer_science)#Lexical_scope_vs._dynamic_scope>).

## Function Scoping

Traditionally in JavaScript, the scope of a variable was only bound by the function it was in. Take this simple example:

```js
var name = "Ryan";
// Do some other things
console.log(name);
```

Since everything is in global scope, of course when we access name, it takes its value from the current scope and prints "Ryan". However, say many years later, another developer looks at this code, and since our function is long, can't see that we have already declared a variable name. So they declare it again for something they need to use it and leave the rest of the code untouched:

```js
var name = "Ryan";
///200 lines later another developer comes and adds
var name = "Sarah";
console.log(name); /// prints sarah
/// 200 lines later, we reach our original console.log
console.log(name); //except now it prints Sarah not Ryan as the original developer intended... oops!
```

As we can see, their additions in the middle of our function have changed the behaviour of our code later on. Before ES6, the only way our developer could avoid introducing errors like this was by declaring another function:

```js
var name = "Ryan";
///200 lined later, developer comes and adds this function
function anotherName() {
  var name = "Sarah";
  console.log(name);
}
anotherName(); //then they have to call their function to run it;
/// Now we get to our original code, and it works as intended
console.log(name); //prints Ryan
```

The second developer is implementing a computer science principle here called "The Principle of Least Privilege" by basically hiding the internal workings of their code from the outside world. This is great, but our second developer had to write a lot of additional code. Furthermore, they had to create and declare a function that they have no intention of reusing again. They just had to create `anotherName` to avoid name collisions.

### IFFE

Around 2011 or so, JavaScript developers came up with the concept of _Immediately Invoked Function Expression_ or IFFE. IFFE in effect uses the rules of scope of JavaScript, wraps a function in parentheses (turning it from a function declaration into a function expression) and immediately executes it. So the previous example becomes:

```js
var name = "Ryan";
///Now the 2nd developer creates an IFFE instead
(function anotherName() {
  var name = "Sarah";
  console.log(name);
})(); //and calls it strait away.
/// Finally when we get to our original code, and it works as intended
console.log(name); //prints Ryan
```

This patten was very common in the JavaScript world from around 2011 until 2015 or so and you might still run into it especially in older codebases (you'll probably see it more often as an anonymous function without a name). There is nothing wrong with this pattern, it achieves the benefits of hiding the implementation details from the outside code, but it certainly is not elegant. Thankfully block scoping was introduced in ES6, providing us much better tools for this purpose.

## Block Scoping

ES6 introduced block scoping to JavaScript. Simply put, block scoping puts the boundaries of the scope where `{}` are located.

It's important to note that putting `{}` by itself doesn't make a block. The _chunk_ of code only becomes a scope if either the keywords `let` or `const` are located inside the chunk.

Because `var` was previously only bound by function scoping, and JavaScript designers did not want to change its behaviour and break old code (refer to section on backwards compatibility), var still does not observe block scoping and is only bound by function scope.

And this difference in scope is the main difference between `var` with `let` and `const`. Now we know why these two new ways of declaring variables were added to JavaScript. With these new tools, we can write our previous IFFE example as:

```js
var name = "Ryan";
///Now the 2nd developer add this block
{
  let name = "Sarah";
  console.log(name); //prints Sarah
}
/// Finally when we get to the old code, it works as intended
console.log(name); //prints Ryan
```

No need to create a function and execute it straightaway anymore now that we have block scoping! So much more elegant.

Do take note that if the second developer had used `var` instead of `let`, a block scope would not have been created and our code would have had the original error:

```js
var name = "Ryan";
///Now the 2nd developer add this block
{
  var name = "Sarah";
  console.log(name); //prints Sarah
}
/// Finally when we get to the old code
console.log(name); //prints Sarah... oops!
```

This pattern of adding extra `{}` just to create a block scope is not super popular in the JavaScript world. Most developers observe the effects of block scoping when they create a `for` loop or when they use `try` and `catch` which all have `{}`. However in some other programming environments such as Java, this pattern is pretty popular, and if we are to follow the principle of least privilege, it makes sense and is best practice to wrap variables that are only needed for a few lines in `{}` so that they are hidden from the outside code and we reduce the chances of introducing bugs in our code.

### `let` vs. `var`

If your first introduction to JavaScript has been in the past few years, you've probably seen and heard that `let` is the new `var` and that you should avoid using `var` and you'll only encounter it in old code.

While `let` (and `const`, we'll cover `const` more in the next section) are definitely a lot more hot than `var`, as we have seen, they have fundamentally different behaviour when it comes to scope. They are not replacements of `var`, rather they are new tools that can augment our usage of `var`. As authors of our code, we can leverage these tools to communicate our intention better in our code. `let` and `var` are tools that can effectively be used side by side. Consider the following example:

```js
function lookupRecord(searchStr) {
  try {
    let id = getRecord(searchStr);
  } catch (err) {
    let id = -1;
  }
  return id;
}
```

This doesn't work! The variable `id` is created by either the `try` statement or the `catch` statement, but it is bound to the block and so when we try and return it, it is out of scope, resulting in an error. In order to use `let`, we'd have to declare the variable first and then use it:

```js
function lookupRecord(searchStr) {
  let id;
  try {
    id = getRecord(searchStr);
  } catch (err) {
    id = -1;
  }
  return id;
}
```

This works, but now we have separated our declaration from our assignment. Imagine if there are a few dozen lines between the declaration and assignment, now our eyes have to dart across the page to know whether we've declared it previously or not, which makes our code less readable and more error prone.

Imagine if we had used `var`:

```js
function lookupRecord(searchStr) {
  try {
    var id = getRecord(searchStr);
  } catch (err) {
    var id = -1;
  }
  return id;
}
```

This works, because unlike `let`, `var` is not bound by block scope but by function scope, so when are returning it, it's still in scope. And we have managed to keep our variable declaration and assignment together, making it easier to follow the execution path.

There is also, a semantic difference between `let` and `var`. `var` is and has always been bound to function scope, whereas `let` is bound to block scope. We can use this difference to make our intention for the usage of our variable clearer to someone who will be reading our code in the future. If there is a variable that will be used throughout the function, use `var` to declare it. If there is a variable that is only going to be used for a few lines, such as the increment variable of a `for` loop, use `let` to ensure that it's only used in that block. That way when someone reads our code in future, our intentions of where we intend to use this variable are instantly communicated, making our code more self-documenting.

Like everything else in this series, like `==` vs `===`, like function expression vs declaration, `let` and `var` are tools that provide us with different capabilities. Instead of making blanket statements of only using one or the other, we can learn how our tools work and use them to communicate our intentions effectively.

### The case for and against `const`

There are two schools of thought when it comes to `const`:

- Those who believe we should declare all our variables by default as `const`, and leave `let` only for variables that will need to change value during execution, such as the incriminator of a `for` loop
- Those who believe that we should adhere to the semantic meaning of `const`, that `const` is short for _constant_ and we should reserve it for variables whose value will absolutely never change, i.e., are _constants_. Let's see the pros and cons of each school here.

The always `const` camp argues that `using` const to restrict unavoidable reassignment of a variable is adherence to the principle of least privilege. They argue that this prevents the coder from unintentionally reassigning for example an object to a different object. For example:

```js
const hello = { a: "foo" };
hello = { b: "bar" }; // Error!
```

Would throw a type Error since hello is a `const`.

The second camp argue that `const` doesn't really prevent you from _mutating_ the object. For example in the previous example, the author could have written:

```js
const hello = { a: "foo" };
hello.b = "bar"; //This works since we are adding a property to an object
delete hello.a;
```

Which would have achieved the exact same result that `const` was supposedly trying to prevent.

In order to fully understand what can and cannot be changed with `const` requires a good understanding of what is passed by value and what is passed by reference in JavaScript, and that is a topic that is rife with confusion especially amongst newcomers to the language.

Briefly, primitive types are passed by value, for example:

```js
let firstNum = 5;
let secondNum = firstNum;
firstNume = 10;
console.log(firstNum); //prints 10
console.log(secondNum); // prints 5
```

Whereas built-in objects are passed by reference, for example:

```js
const obj1 = { first: "5" };
const obj2 = obj1;
obj2.first = 10;
console.log(obj1);
```

Prints:

```
Object {
  first: 10
}
```

Of course this difference is well known to seasoned programmers. Anyone who has programmed in JavaScript for any significant amount of time (or any other programming language that has pass-by-reference for that matter) would be intimately familiar with these details. JavaScript programmers know that in order to prevent an object from mutating, they need to freeze it using the Object.freeze method (which is shallow freeze) or use a deep freeze method provided by one of the many third party libraries.

This nuance means that semantically, `const` is a misnomer. A JavaScript newbie will find a simple example such as:

```js
const a = [5];
a.push(10);
```

confusing, and they'd be very well within their rights to be confused! _Semantically_, if the array `a` is _constant_, why are we adding numbers to it? This confusion has lead to some programming languages (such as Java) even avoiding using the `const` keyword in their language (though using `static final` results in a similar behaviour in Java.)

This group, and Kyle Simpson is certainly among them, argue that in order to avoid this confusion, `const` should only be used where it semantically makes sense, i.e., on primitive variables whose value will never change during the runtime of the program. Kyle Simpson argues that using `const` for example for an API URL, or using it for a number that has a significant _magical_ value in our program, makes perfect sense. In these instances, the author should use `const` to achieve what `const` was designed for: prevent a primitive from changing its value. Using it on objects is nonsensical.

As always, I believe that the choice of which school of thought to follow is yours as an engineer/author of the code. What I think is most important is for these decisions to be made at a company/project level early on, and for these decisions to be codified in a style guide (and perhaps enforced with a linter). Both approaches have certain merits, and using the same approach throughout the codebase will make the code clearer and less error prone.

## Hoisting

The term _Hoisting_ is thrown around a lot in the JavaScript world to explain various phenomena that programmers find hard to explain. In reality, the term _hoisting_ only appeared in the JavaScript specification a few years ago for the first time, and that's because the JS engines actually don't do _hoisting_ per se. Hoisting is an English language term that has been used to define various side effects of JavaScript's lexical scope and its two-step execution model (i.e. parsing).

But first, let's look at what is meant by hoisting:

```js
catName("Chloe");

function catName(name) {
  console.log(`My cat's name is ${name}`);
}
/*
This code works. "My Cat's name is Chloe" is printed.
*/
```

`catName` is referenced before it is declared, and this code works. It is claimed that the reason this works is due to _hoisting_, i.e., function and variable declarations are put to the top of the function when the program is run. What is actually happening, if you followed the section on JavaScript's execution model, is that declarations are kept exactly where they are on the code, however a reference to them is kept in memory during _parsing_, i.e., the first processing of the code, so in the second process, the JS engine knows where `catName` is and goes and looks for it when we reference it, even when that reference appears before the declaration in our code.

Though hoisting is a by-product of JavaScript's execution model and not something JavaScript engines explicitly do, it can be a useful _mental model_. It can help make sense of the rules of lexical scoping and execution model by simply thinking that the JS engine puts those declarations at the top of the scope. We can then use hoisting to make our code more readable; for example we can put high level logic of our program at the very top of the source code and put the utility functions that it utilises near the bottom. This makes our code easier to read without being cluttered by utility functions.

But it's important to keep a few caveats in mind

### Only declarations are Hoisted

JavaScript only hoists declarations, not initialisations. If a variable is initialised after declarations, its value will be undefined before initialisation. For example:

```js
console.log(num); //prints undefined as only declaration was hoisted not the initialisation
var num; // Declaration
num = 10; // Initialization
```

This also means that functions that are written as function declaration are hoisted, however those that are written as function expression (assigned to a variable) are not. So if we had written our `catName` example as:

```js
catName("Chloe"); // Error! Cannot use catName before initialisation

const catName = function catName(name) {
  console.log(`My cat's name is ${name}`);
};
```

or as an anonymous arrow function

```js
catName("Chloe"); // Error! Cannot use catName before initialisation

const catName = (name) => {
  console.log(`My cat's name is ${name}`);
};
```

### `let` and `const` and hoisting

You may have heard that `let` and `const` don't hoist, i.e., a variable declared with these two keywords isn't hoisted. This is technically incorrect, but practically correct! Remember when we mentioned that `undeclared` and `undefined` are different states, and that they in turn are different from `TDZ` (Temporal Dead Zone)? What's TDZ you asK? Let's dive a little into this.

`const` and `let` are hoisted the same way as `var` to the top of their scope (obviously `let` and `const` are only hoisted to a block whereas `var` is hoisted to its function). However the designers of JavaScript had a problem. If the value of a `const` was set to `undefined` prior to initialisation, and then that value changed after initialisation, technically that broke the rules of `const` (can't change its value!). So in ES6 they added another state: Temporal Dead Zone (TDZ). When a variable is declared with `let` or `const`, their value prior to initialisation is set to TDZ instead of `undefined`. And if you try and access a variable with the value TDZ, you'll get an error! So technically speaking `let` and `const` are hoisted similar to `var`, but practically speaking you can't access them when they are in TDZ. So you might as well think that they aren't hoisted.

Those interested can find more on `let` and `const` hoisting [in the spec](https://www.ecma-international.org/ecma-262/11.0/index.html#sec-let-and-const-declarations).

Let's practice Hoisting with an exercise.

## Exercise 5: Hoisting

### Part 1 Function Declaration & Hoisting

In this exercise, refactor this to take advantage of function hoisting. Refactor all inline function expressions to be function declarations. Place function declarations at the bottom (that is, below any executable code) of their respective scopes.
Also, pull function declarations to outer scopes if they don't need to be nested.

### Part 2: Arrow Functions

In Part 2, refactor the code to use only function expressions and arrow functions wherever possible. Remember that expressions are not hoisted. Compare and contrast the two styles:

```js
function getStudentFromId(studentId) {
  return studentRecords.find(function matchId(record) {
    return record.id == studentId;
  });
}

function printRecords(recordIds) {
  var records = recordIds.map(getStudentFromId);

  records.sort(function sortByNameAsc(record1, record2) {
    if (record1.name < record2.name) return -1;
    else if (record1.name > record2.name) return 1;
    else return 0;
  });

  records.forEach(function printRecord(record) {
    console.log(
      `${record.name} (${record.id}): ${record.paid ? "Paid" : "Not Paid"}`
    );
  });
}

function paidStudentsToEnroll() {
  var recordsToEnroll = studentRecords.filter(function needToEnroll(record) {
    return record.paid && !currentEnrollment.includes(record.id);
  });

  var idsToEnroll = recordsToEnroll.map(function getStudentId(record) {
    return record.id;
  });

  return [...currentEnrollment, ...idsToEnroll];
}

function remindUnpaid(recordIds) {
  var unpaidIds = recordIds.filter(function notYetPaid(studentId) {
    var record = getStudentFromId(studentId);
    return !record.paid;
  });

  printRecords(unpaidIds);
}

// ********************************

var currentEnrollment = [410, 105, 664, 375];

var studentRecords = [
  { id: 313, name: "Frank", paid: true },
  { id: 410, name: "Suzy", paid: true },
  { id: 709, name: "Brian", paid: false },
  { id: 105, name: "Henry", paid: false },
  { id: 502, name: "Mary", paid: true },
  { id: 664, name: "Bob", paid: false },
  { id: 250, name: "Peter", paid: true },
  { id: 375, name: "Sarah", paid: true },
  { id: 867, name: "Greg", paid: false },
];

printRecords(currentEnrollment);
console.log("----");
currentEnrollment = paidStudentsToEnroll();
printRecords(currentEnrollment);
console.log("----");
remindUnpaid(currentEnrollment);

/*
	Bob (664): Not Paid
	Henry (105): Not Paid
	Sarah (375): Paid
	Suzy (410): Paid
	----
	Bob (664): Not Paid
	Frank (313): Paid
	Henry (105): Not Paid
	Mary (502): Paid
	Peter (250): Paid
	Sarah (375): Paid
	Suzy (410): Paid
	----
	Bob (664): Not Paid
	Henry (105): Not Paid
*/
```

### Exercise 5 Solution

#### Using Function Declaration and Hoisting

{% codepen https://codepen.io/ryanameri/pen/KKzWepw default-tab=js %}

#### Using Arrow Functions

{% codepen https://codepen.io/ryanameri/pen/xxVqWGQ default-tab=js %}

It can easily be seen that the two styles are different but do exactly the same thing. Using function declaration gives us hoisting, and this allows us to write executable part of our program, the _logic_ of our code at the top and easily follow its execution without being encumbered by various utility functions.

On the other hand, the second example using arrow functions is a lot less verbose. It allows us to achieve the same outcome using fewer lines of code. However we do have to be conscious that we can't use our utility functions for example `getStudentFromId` before initialising them first.
