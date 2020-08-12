---
series: Mastering Hard Parts of JavaScript
description: An introduction to JavaScript asynchronicity using 10 exercises
---

# Mastering Hard Parts of JavaScript: Asynchronicity

## The Event Loop

Understanding asynchronicity in JavaScript requires understanding one fundamental concept: what will the JS engine execute next? This is a very simplified overview of how to answer this question, more formally known as the _Event Loop_.

JavaScript is (for the most part) single threaded, so if everything in JavaScript was synchronous, The JS engine would executive every statement one by one as they appear in the source code, wait for the execution to finish, and go to the next line.

However that would be incredibly limiting when it comes to web development. To solve this problem, some APIs that the browser/node.js provide are asynchronous, which basically means that they don't execute when the JS engine first runs into them. Instead they get put in a queue, to be executed once all the synchronous statements have finished. Let's consider:

```js
function printHello() {
  console.log("Hello");
}
setTimeout(printHello, 0);
console.log("Me first!");
```

Because `setTimeout` is told to execute `printHello` at 0 milliseconds, one could reason that the output should be:

```js
Hello
Me first!
```

But in fact the the output is

```js
Me first!
Hello
```

This is because setTimeout is an asynchronous API (a callback function), so its execution gets placed in the "task queue". Anything in the task queue is only executed after all synchronous code has already ran.

Note: `console.log` is in fact itself an asynchronous function but I'm glossing over that detail for the sake of simplicity and clear demonstration of the concept.

## Promises

_Promises_, introduced in ES6, add one additional queue to the mix. Consider:

```js
function display(data){console.log(data)}
function printHello(){console.log("Hello");}
function blockForLong(){
    const arr = [];
    for (let i = 0; i < 3_000_000_000; i++>){
        arr.push(i)
    }
 }
setTimeout(printHello, 0);
const futureData = fetch('https://twitter.com/AmeriRyan/status/1291935897076641792')
futureData.then(display)
blockForLong()
console.log("Me first!");
```

This code won't run correctly as this is not exactly how fetch() works, but for the sake of simplicity, let's assume that `fetch` is a function that takes a URL as a string and returns a Promise. `blockForLong` is a function that doesn't do anything important for our purposes but is a synchronous function that takes a long a long time to execute. We first call `setTimeout` that runs `printHello` at 0 milliseconds. Then we handle the Promise and pass it to a function `display` that just prints it to console. Then we execute `blockForLong` and finally we execute `console.log`. Can you guess what gets printed first?

First, all synchronous code is executed. That means `blockForLong` is called first, and then `Me first!` is printed to console. Promises get placed in a queue called the "micro task queue", which has priority over the "task queue" where callback functions are placed. So even though `setTimeout` appears first in the source code, we first call the `display` function with the returned data, and only call the `printHello` function last.

So, the _Event Loop_ in JavaScript, in a nutshell, is:

1. Synchronous code
2. Anything in the micro task queue (Promises)
3. Anything in the task queue (callback functions)

If you can follow the execution order in the this example, you should be able to solve all the upcoming exercises (perhaps with a bit of help from MDN).

In the next section, we'll practice 10 exercises that should help us master asynchronicity as well as introduce us to Promises.

## Exercise 1

> Thinking point (no writing code necessary for this challenge): Inspect the code given to you in Challenge 1. In what order should the console logs come out? Howdy first or Partnah first?

```js
function sayHowdy() {
  console.log("Howdy");
}

function testMe() {
  setTimeout(sayHowdy, 0);
  console.log("Partnah");
}
testMe();
```

### Solution 1

The output is `Partnah` first, followed by `Howdy`. As discussed, setTimeout is a callback function so its execution gets places in the task queue, it is invoked only after everything in the call stack is executed.

## Exercise 2

> Create a function delayedGreet that console logs welcome after 3 seconds.

```js
function delayedGreet() {}

delayedGreet();
// should log (after 3 seconds): welcome
```

### Solution 2

```js
function delayedGreet() {
  setTimeout(() => console.log("welcome"), 3000);
}
```

A gentle introduction, but an important foundation. This is how to wrap a callback function (which in reality is not a JS function but a browser API) in our own function.

## Exercise 3

> Create a function helloGoodbye that console logs hello right away, and good bye after 2 seconds.

```js
function helloGoodbye() {}

helloGoodbye();
// should log: hello should also log (after 3 seconds): good bye
```

### Solution 3

```js
function helloGoodbye() {
  console.log("hello");
  setTimeout(() => console.log("good bye"), 3000);
}
```

Notice that we could have also written

```js
function helloGoodbye() {
  setTimeout(() => console.log("good bye"), 3000);
  console.log("hello");
}
```

The order doesn't matter in this example, as `console.log` will always get executed first before `setTimeout` is called.

## Exercise 4

> Create a function brokenRecord that console logs hi again every second. Use the End Code button to stop the console logs when you are satisfied that it is working.

```js
function brokenRecord() {}
brokenRecord();
// should log (every second): hi again
```

### Solution 4

```js
function brokenRecord() {
  function printHi() {
    console.log("hi again");
  }
  setInterval(printHi, 1000);
}
```

If you haven't seen `setInterval` before, you might be tempted to use a loop here but that won't get you the desired output. `setInterval` is a method of the Web APIs that the browser/environment provide. As always [MDN](https://developer.mozilla.org/en-US/docs/Web/API) is a fantastic way of delving into them all.

## Exercise 5

> Create a function limitedRepeat that console logs hi for now every second, but only for 5 seconds. Research how to use clearInterval if you are not sure how to do this.

```js
function limitedRepeat() {}
limitedRepeat();
// should log (every second, for 5 seconds): hi again
```

### Solution 5

```js
function limitedRepeat() {
  function printHi() {
    console.log("hi again");
  }
  function clear() {
    clearInterval(id);
  }
  printHi();
  const id = setInterval(printHi, 1000);
  setTimeout(clear, 5000);
}
```

And here is setInterval's counterpart: clearInterval. When we call setInterval, it returns an Interval ID which uniquely identifies the interval. We can pass that to clearInterval to stop the interval.

## Exercise 6

> Write a function called everyXsecsForYsecs that will accept three arguments: a function func, a number interval, and another number duration. everyXsecsForYsecs will execute the given function every interval number of milliseconds, but then automatically stop after duration milliseconds. Then pass the below sayHi function into an invocation of everyXsecsForYsecs with 1000 interval time an 5000 duration time. What do you expect to happen?

```js
function everyXsecsForYsecs() {}
function theEnd() {
  console.log("This is the end!");
}
everyXsecsForYsecs(theEnd, 2, 20);
// should invoke theEnd function every 2 seconds, for 20 seconds): This is the end!
```

### Solution 6

```js
function everyXsecsForYsecs(func, interval, duration) {
  const id = setInterval(func, interval * 1000);
  function clear() {
    clearInterval(id);
  }
  setTimeout(clear, duration * 1000);
}
```

This turns out to be very similar to the previous exercise, another way of practicing setInterval and clearInterval. Here the function to be executed is passed on as an argument, but other than that everything should look familiar.

## Exercise 7

> Write a function delayCounter that accepts a number (called 'target') as the first argument and a number of milliseconds (called 'wait') as the second argument, and returns a function.

> When the returned function is invoked, it should log to the console all of the numbers between 1 and the target number, spaced apart by 'wait' milliseconds.

```js
function delayCounter() {}

const countLogger = delayCounter(3, 1000);
countLogger();
//After 1 second, log 1
//After 2 seconds, log 2
//After 3 seconds, log 3
```

### Solution 7

```js
function delayCounter(target, wait) {
  function closureFn() {
    let i = 1;
    const id = setInterval(() => {
      console.log(i);
      i++;
      if (i > target) clearInterval(id);
    }, wait);
  }
  return closureFn;
}
```

We're putting all the concepts we've practiced on callbacks, closure and asynchronicity to good use here! The description demands that our function should return another function, so we're talking closure. We're also calling clearInterval in the callback function given to setInterval. Every time setInterval is invoked, we increment our counter `i` that's declared in the outside scope (our memory). We check to make sure that our counter is still lower than our target, and when it goes above that, we execute clearInterval.

## Exercise 8

> Write a function, promised, that takes in a value. This function will return a promise that will resolve after 2 seconds.

```js
function promised() {}

const createPromise = promised("wait for it...");
createPromise.then((val) => console.log(val));
// will log "wait for it..." to the console after 2 seconds
```

### Solution 8

```js
function promised(val) {
  return new Promise((resolve) => {
    setTimeout(() => resolve(val), 2000);
  });
}
```

If you are not familiar with the syntax of a Promise (hint: there's always [MDN](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise)) this can look a bit confusing. The important thing to remember is that a promise can take one or two parameters, the first is the function to be called when the promise is resolved, and the second (optional, not shown here) is the function to be called when the operation fails.

So in this exercise, we are creating a Promise and returning it. The resolve function is given to the Promise when the .then method is called on it. Here we just execute that function with a setTimeout set to 2 seconds.

## Exercise 9

> Write a SecondClock class, with two methods: start and reset.​
> start: upon invocation, invokes a callback (this.cb, defined in the constructor) on an argument every second, with the argument to the callback being the current seconds "value".

> In other words, the callback gets invoked every second on the "seconds hand" of the clock. Always start with 1 and don't utilize the seconds value of the current computer clock time.

> The first "tick" with value 1 occurs 1 second after the initial "secondClock" invocation.
> The second "tick" with value 2 occurs 2 seconds after the initial "secondClock" invocation.

> The sixtieth "tick" with value 60 occurs 60 seconds after the initial "secondClock" invocation.
> The sixty-first "tick" with value 1 occurs 61 seconds after the initial "secondClock" invocation. The sixty-second "tick" with value 2 occurs 62 seconds after the initial "secondClock" invocation.
> etc.

> reset: upon invocation, completely stops the "clock". Also resets the time back to the beginning Hint: look up setInterval and clearInterval.

```js
class SecondClock {}

const clock = new SecondClock((val) => {
  console.log(val);
});
console.log("Started Clock.");
clock.start();
setTimeout(() => {
  clock.reset();
  console.log("Stopped Clock after 6 seconds.");
}, 6000);
```

### Solution 9

```js
class SecondClock {
  constructor(cb) {
    this.cb = cb;
    this.seconds = 0;
    this.id = undefined;
  }
  start() {
    this.id = setInterval(() => {
      this.seconds++;
      this.cb(this.seconds % 60);
    }, 1000);
  }
  reset() {
    this.seconds = 0;
    clearInterval(this.id);
  }
}
```

The description again looks a bit daunting, but as always the challenge in solving the problem requires breaking it down into simpler parts. Solving this exercise requires a bit of a knowledge of the class syntax as well, which we will practice a lot in the next section of this series.

What this exercise is trying to show us is how to implement something very similar to exercise 7, but here using class structure instead of closure. So instead of having an outer variable that acts as our memory, here our memory is a class field. We've got two class methods, start and reset that basically manipulate our counter using a callback function that's first given to us in the constructor.

## Exercise 10

> Write a function called debounce that accepts a function and returns a new function that only allows invocation of the given function after interval milliseconds have passed since the last time the returned function ran.

> Additional calls to the returned function within the interval time should not be invoked or queued, but the timer should still get reset.

> For examples of debouncing, check out [this CSS Tricks article](https://css-tricks.com/debouncing-throttling-explained-examples/).

```js
function debounce() {}

function giveHi() {
  return "hi";
}
const giveHiSometimes = debounce(giveHi, 3000);
console.log(giveHiSometimes());
// should output 'hi'
setTimeout(function () {
  console.log(giveHiSometimes());
}, 2000);
// should output undefined
setTimeout(function () {
  console.log(giveHiSometimes());
}, 4000);
//should output undefined
setTimeout(function () {
  console.log(giveHiSometimes());
}, 8000);
// should output 'hi'
```

### Solution 10

```js
function debounce(callback, interval) {
  let counter = 0;
  let hasRan = false;
  function closureFn() {
    let id = undefined;
    if (!hasRan) {
      ///this is the first run
      id = setInterval(() => counter++, 1);
      hasRan = true;
      return callback();
    } else {
      //for subsequent runs
      if (counter < interval) {
        // Not enough time has elapsed
        counter = 0;
        clearInterval(id);
        id = setInterval(() => counter++, 1);
        return undefined;
      } else {
        //Enough time has elapsed
        counter = 0;
        clearInterval(id);
        id = setInterval(() => counter++, 1);
        return callback();
      }
    }
  }
  return closureFn;
}
```

Debouncing and throttling are important concepts in modern web development (this functionality is provided by many libraries). Here we're implementing a simple debounce using closure and callbacks. We need a counter and a flag to indicate whether the function has ran before before, these variables need to reside in our _memory_, so in the outer scope. We then increment the counter using setInterval, and in subsequent runs we check to see if enough time has passed or not (based on interval). If enough time has not passed, we need to reset the counter and return undefined. If enough time has passed, we again reset the counter but this time execute and return the callback.

This brings our asynchronicity chapter to a close. Next we'll take a closer look at classed and the prototype.
