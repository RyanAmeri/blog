---
series: series: Mastering Hard Parts of JavaScript
description: An introduction to JavaScript closure using 19 exercises
---

# Mastering JavaScript: Closure

Closure is both a deceptively simple yet incredibly powerful part of JavaScript to grasp. The reason why callback functions are so powerful, the reason why asynchronous JS and everything it encompasses (Promises etc) are possible, is closure.

But what is Closure? Dan Abramov [describes it best](https://whatthefuck.is/closure):

> You have a closure when a function accesses variables defined outside of it.

> For example, this code snippet contains a closure:

```js
let users = ["Alice", "Dan", "Jessica"];
let query = "A";
let user = users.filter((user) => user.startsWith(query));
```

> Notice how `user => user.startsWith(query)` is itself a function. It uses the query variable. But the query variable is defined outside of that function. That’s a closure.

How is this possible? It's because when you return a function from inside another function, you are not only return the function but also its _"Variable Environment"_. This variable environment includes any variable or object that was declared in the outer function. The returned function retains a link to this outside variable environment. This link is more formally called _Closed over ‘Variable Environment’ (C.O.V.E.)_
or _Persistent Lexical Scope Referenced Data (P.L.S.R.D.)_.

The concept can be a bit confusing, but we'll master it by going through these exercises together. As always, I'd recommend you solve the problems yourself first before looking at my solution, and then compare and contrast them.

## Exercise 1

> Create a function createFunction that creates and returns a function. When that created function is called, it should print "hello". When you think you completed createFunction, un-comment out those lines in the code and run it to see if it works.

```js
function createFunction() {}
const function1 = createFunction();
function1();
// => should console.log('hello');
```

### Solution 1

```js
function createFunction() {
  function printHello() {
    console.log("hello");
  }
  return printHello;
}
```

A nice and easy start, yet this is the perfect demonstration of closure. We first call createFunction() and assign its value to `function1`. `function1` is now effectively the printHello() function as that is what was returned. We can now call function1() and it would execute the body of the printHello() function.

## Exercise 2

> Create a function createFunctionPrinter that accepts one input and returns a function. When that created function is called, it should print out the input that was used when the function was created.

```js
function createFunctionPrinter() {}
const printSample = createFunctionPrinter("sample");
printSample();
// => should console.log('sample');
const printHello = createFunctionPrinter("hello");
printHello();
// => should console.log('hello');
```

### Solution 2

```js
function createFunctionPrinter(input) {
  function printInput() {
    console.log(input);
  }
  return printInput;
}
```

Very similar to the previous exercise, except here we are also demonstrating the concept of COVE or P.L.S.R.D. The inner function `printInput()` gets access to the variables that were present in the outside function, in this instance the parameter `input`.

## Exercise 3

> Examine the code for the outer function. Notice that we are returning a function and that function is using variables that are outside of its scope.
> Try to deduce the output before executing.

```js
function outer() {
  let counter = 0;
  // this variable is outside incrementCounter's scope
  function incrementCounter() {
    counter++;
    console.log("counter", counter);
  }
  return incrementCounter;
}

const willCounter = outer();
const jasCounter = outer();
willCounter();
willCounter();
willCounter();

jasCounter();
willCounter();
```

> Now create a function addByX that returns a function that will add an input by x.

```js
function addByX() {}
const addByTwo = addByX(2);
console.log(addByTwo(1));
// => should return 3
console.log(addByTwo(2));
// => should return 4
console.log(addByTwo(3));
// => should return 5

const addByThree = addByX(3);
console.log(addByThree(1));
// => should return 4
console.log(addByThree(2));
// => should return 5

const addByFour = addByX(4);
console.log(addByFour(4));
// => should return 8
console.log(addByFour(5));
// => should return 9
```

### Solution 3

```js
function addByX(x) {
  function addByNum(num) {
    return num + x;
  }
  return addByNum;
}
```

We should be getting the hang of these type of functions. The first time addByX is called, it receives an argument and returns a function. This inner function will itself receive a parameter, but it will access to both, its own parameter and the addByX parameter, so it's able to whatever calculation is required on them both.

## Exercise 4

> Write a function once that accepts a callback as input and returns a function. When the returned function is called the first time, it should call the callback and return that output. If it is called any additional times, instead of calling the callback again it will simply return the output value from the first time it was called.

```js
function once() {}

// /*** Uncomment these to check your work! ***/
const onceFunc = once(addByTwo);
console.log(onceFunc(4)); // => should log 6
console.log(onceFunc(10)); // => should log 6
console.log(onceFunc(9001)); // => should log 6
```

### Solution 4

```js
function once(func) {
  let counter = 0;
  let res = undefined;
  function runOnce(num) {
    if (counter === 0) {
      res = func(num);
      counter++;
    }

    return res;
  }
  return runOnce;
}
```

This is the first example where we can see how to use closure to give our functions a _memory_. By simply declaring a counter variable in the outside scope and then mutating it in the inner function, we can see how many times our function has been called and then have different behaviour based on the number of times the inner function has been called. This gives our functions _a lot_ more flexibility and power, which we'll further explore in the following exercises.

## Exercise 5

> Write a function after that takes the number of times the callback needs to be called before being executed as the first parameter and the callback as the second parameter.

```js
function after() {}
const called = function () {
  console.log("hello");
};
const afterCalled = after(3, called);
afterCalled(); // => nothing is printed
afterCalled(); // => nothing is printed
afterCalled(); // => 'hello' is printed
```

### Solution 5

```js
function after(count, func) {
  let counter = 0;
  function runAfter() {
    counter++;
    if (count === counter) {
      func();
    }
  }
  return runAfter;
}
```

A similar example to the previous exercise, here we are just demonstrating a different behaviour. Again we can see that we can set a counter in the outside scope, using which we can determine how many times our function has been called. And based on that, we can implement different logic for our function.

## Exercise 6

> Write function delay that accepts a callback as the first parameter and the wait in milliseconds before allowing the callback to be invoked as the second parameter. Any additional arguments after wait are provided to func when it is invoked. HINT: research setTimeout();

### Solution 6

```js
function delay(func, wait, ...rest) {
  function delayRun() {
    func(...rest);
  }
  setTimeout(delayRun, wait);
}
```

A couple of things should be noted here. First of all, using rest parameters to make sure that all the following parameters are passed on to the inner function.

Secondly, notice that the function return technically doesn't return anything. It simply uses the setTimeout() which is a API provides by the browser/node.js. It is setTimeout that invokes the `delayRun` function, with a delay of `wait` milliseconds. And yet thanks to closure, inside delayRun we still have access to all the parameters that were passed to `delay`.

## Exercise 7

> Write a function rollCall that accepts an array of names and returns a function. The first time the returned function is invoked, it should log the first name to the console. The second time it is invoked, it should log the second name to the console, and so on, until all names have been called. Once all names have been called, it should log 'Everyone accounted for'.

```js
function rollCall() {}

const rollCaller = rollCall(["Victoria", "Juan", "Ruth"]);
rollCaller(); // => should log 'Victoria'
rollCaller(); // => should log 'Juan'
rollCaller(); // => should log 'Ruth'
rollCaller(); // => should log 'Everyone accounted for'
```

### Solution 7

```js
function rollCall(names) {
  let counter = 0;
  function runCall() {
    if (counter < names.length) {
      console.log(names[counter]);
      counter++;
    } else {
      console.log("Everyone accounted for");
    }
  }
  return runCall;
}
```

This is similar to exercise 5, in that we need to output different things based on how many times the function has been called. So immediately you should think, we need a counter, and this counter needs to be in the outside scope. After that it's pretty simple, our function receives an array and we just need to console.log a different element of that array based on how many times our function has been called. Simple, yet so beautiful!

## Exercise 8

> Create a function saveOutput that accepts a function (that will accept one argument), and a string (that will act as a password). saveOutput will then return a function that behaves exactly like the passed-in function, except for when the password string is passed in as an argument. When this happens, the returned function will return an object with all previously passed-in arguments as keys, and the corresponding outputs as values.

```js
function saveOutput() {}
const multiplyBy2 = function (num) {
  return num * 2;
};
const multBy2AndLog = saveOutput(multiplyBy2, "boo");
console.log(multBy2AndLog(2));
// => should log 4
console.log(multBy2AndLog(9));
// => should log 18
console.log(multBy2AndLog("boo"));
// => should log { 2: 4, 9: 18 }
```

### Solution 8

```js
function saveOutput(func, magicWord) {
  const log = {};
  function funcAndLog(num) {
    if (num !== magicWord) {
      log[num] = func(num);
      return log[num];
    } else {
      return log;
    }
  }
  return funcAndLog;
}
```

Now we are expanding our function's memory to more than just a counter. Instead of simply counting how many times the function has been called, we have to keep track of all the parameters that our function receives and the output values that our function returns.

So we need an empty object, and this object needs to reside in the outside scope so that it's _persistent_. Beyond that, it's rather simple. In our closure function we check if the magic password has been given. If it not, we record the parameter and its value and return that value. If the magic password has been given, we return our whole _log_ function which contains all the parameters and the returned values previously stored.

## Exercise 9

> Create a function cycleIterator that accepts an array, and returns a function. The returned function will accept zero arguments. When first invoked, the returned function will return the first element of the array. When invoked a second time, the returned function will return the second element of the array, and so forth. After returning the last element of the array, the next invocation will return the first element of the array again, and continue on with the second after that, and so forth.

```js
function cycleIterator() {}
const threeDayWeekend = ["Fri", "Sat", "Sun"];
const getDay = cycleIterator(threeDayWeekend);
console.log(getDay()); // => should log 'Fri'
console.log(getDay()); // => should log 'Sat'
console.log(getDay()); // => should log 'Sun'
console.log(getDay()); // => should log 'Fri'
```

### Solution 9

```js
function cycleIterator(array) {
  let counter = 0;
  function cyclingItems() {
    counter++;
    return array[(counter - 1) % array.length];
  }
  return cyclingItems;
}
```

This is similar to exercise 7, in that we have to keep count of how many times the function has been called and return an item from the original parameter array accordingly. The only difference here is that when we run out of the array items, we need to go back to the beginning of the array. So basically we need to use the mod operator to continuously cycle through the array.

## Exercise 10

> Create a function defineFirstArg that accepts a function and an argument. Also, the function being passed in will accept at least one argument. defineFirstArg will return a new function that invokes the passed-in function with the passed-in argument as the passed-in function's first argument. Additional arguments needed by the passed-in function will need to be passed into the returned function.

```js
function defineFirstArg() {}
const subtract = function (big, small) {
  return big - small;
};
const subFrom20 = defineFirstArg(subtract, 20);
console.log(subFrom20(5)); // => should log 15
```

### Solution 10

```js
function defineFirstArg(func, arg) {
  function insideFn(second) {
    return func(arg, second);
  }
  return insideFn;
}
```

Reading the description of the exercise made my head spin a little bit! But thankfully looking at the expected output cleared it up a bit. Basically our function needs to return an inside function, and this function needs to run a function that was originally given as a parameter to the outside function.

I believe this is basically a very gentle introduction to the concept of currying.

## Exercise 11

> Create a function dateStamp that accepts a function and returns a function. The returned function will accept however many arguments the passed-in function accepts, and return an object with a date key that contains a timestamp with the time of invocation, and an output key that contains the result from invoking the passed-in function. HINT: You may need to research how to access information on Date objects.

```js
function dateStamp() {}
const stampedMultBy2 = dateStamp((n) => n * 2);
console.log(stampedMultBy2(4));
// => should log { date: (today's date), output: 8 }
console.log(stampedMultBy2(6));
// => should log { date: (today's date), output: 12 }
```

### Solution 11

```js
function dateStamp(func) {
  const logTime = {};
  function stamping(input) {
    logTime.date = new Date();
    logTime.output = func(input);
    return logTime;
  }
  return stamping;
}
```

Another way of giving our function _memory_, except here instead of counting how many times our function has been called, we are keeping track of _when_ it was called. Since our function needs to have a memory, it needs to have a persistent object in its outside scope, i.e., closure. This object then gets a date attribute that is set to the when the function is invoked, and an output attribute that is set to the return value of the original parameter with the second function's parameter as its argument.

We should be feeling pretty confident about giving our functions _memory_ now, and that basically is the gist of closure.

## Exercise 12

> Create a function censor that accepts no arguments. censor will return a function that will accept either two strings, or one string. When two strings are given, the returned function will hold onto the two strings as a pair, for future use. When one string is given, the returned function will return the same string, except all instances of first strings (of saved pairs) will be replaced with their corresponding second strings (of those saved pairs).

```js
function censor() {}
const changeScene = censor();
changeScene("dogs", "cats");
changeScene("quick", "slow");
console.log(changeScene("The quick, brown fox jumps over the lazy dogs."));
// => should log 'The slow, brown fox jumps over the lazy cats.'
```

### Solution 12

```js
function censor() {
  const phrases = new Map();
  function actualFn(...args) {
    if (args.length === 2) {
      phrases.set(args[0], args[1]);
    } else {
      let input = args[0];
      for (let [key, value] of phrases) {
        let regex = new RegExp(key, "g");
        input = input.replace(regex, value);
      }
      return input;
    }
  }
  return actualFn;
}
```

Now our function is getting a little bit more interesting, but when you break this exercise down, it is still very much doing the same things we've been practicing in the previous exercises, i.e., we need to have _memory_ of some sort, and our function needs to have different behaviour based on the number of arguments passed.

For this exercise, I decided to use a Map() for the _memory_ part but an object could also be used. I use rest parameters to capture all the arguments passed to the inside function and then check the array's size to see how many arguments there were. If two arguments were passed, we store them in our phrases map and we're done. If only one argument was passed, we use string.prototype.replace() and replace everything in our string that matches the previously stored values in our phrases map.

I wanted to use the new [String.prototype.replaceAll()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String/replaceAll) but at the time of this writing that's not still widely supported (for example it's not supported in the version of node.js 14 that I'm using to run my exercises). Once support for replaceAll() becomes more widespread, we can use that and we wouldn't need to construct a regex.

## Exercise 13

> There's no such thing as private properties on a JavaScript object! But, maybe there are? Implement a function createSecretHolder(secret) which accepts any value as secret and returns an object with ONLY two methods. getSecret() which returns the secret setSecret() which sets the secret

```js
function createSecretHolder() {}
const obj = createSecretHolder(5);
console.log(obj.getSecret());
// => returns 5
obj.setSecret(2);
console.log(obj.getSecret());
// => returns 2
```

### Solution 13

```js
function createSecretHolder(secret) {
  let num = secret;
  const obj = {
    getSecret() {
      return num;
    },
    setSecret(n) {
      num = n;
    },
  };
  return obj;
}
```

Ha! An interesting method of implementing getter and setters! We'll cover these in more detail in Chapter 4, Classes and the Prototype, but here we are seeing how they such getters and setters can be implemented _behind the scenes_ because classes in JS are (mostly) syntatic sugar.

I also believe that you now _can_ (kind of) set private properties on an object in JS in the form of [private class fields](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Classes/Private_class_fields) which was added in ES2019 (if these two paragraphs don't make any sense to you, don't worry, you should still be able to solve the exercise!)

So how did I solve this? Instead of returning a function, here I returned an object. However our object has two methods one is the getter which doesn't receive any parameters and instead returns the value of the num variable stored in its outside scope. The other is a setter which which just changes that value. Because the object is persistent and retains its values, this acts similar to how a normal getter and setter act in OOP languages such as Java.

## Exercise 14

> Write a function, callTimes, that returns a new function. The new function should return the number of times it’s been called.

```js
function callTimes() {}
let myNewFunc1 = callTimes();
let myNewFunc2 = callTimes();
console.log(myNewFunc1()); // => 1
console.log(myNewFunc1()); // => 2
console.log(myNewFunc2()); // => 1
console.log(myNewFunc2()); // => 2
```

### Solution 14

```js
function callTimes() {
  let counter = 0;
  function insideFn() {
    counter++;
    return counter;
  }
  return insideFn;
}
```

Compared to some of our recent exercises, this is rather simple again, but it's good practice to remember how to count the number of times a function has been called. Good demonstration that we have access to the COVE (the outside variables) and can both retrieve them or change them.

## Exercise 15

> Create a function russianRoulette that accepts a number (let us call it n), and returns a function. The returned function will take no arguments, and will return the string 'click' the first n - 1 number of times it is invoked. On the very next invocation (the nth invocation), the returned function will return the string 'bang'. On every invocation after that, the returned function returns the string 'reload to play again'.

```js
function russianRoulette() {}
const play = russianRoulette(3);
console.log(play());
// => should log 'click'
console.log(play());
// => should log 'click'
console.log(play());
// => should log 'bang'
console.log(play());
// => should log 'reload to play again'
console.log(play());
// => should log 'reload to play again'
```

### Solution 15

```js
function russianRoulette(num) {
  let count = 0;
  function closureFn() {
    count++;
    if (count < num) return "click";
    else if (count === num) return "bang";
    else return "reload to play again";
  }
  return closureFn;
}
```

Russian Roulette sounds scary, but this really is a variation of the same problem we've been solving in these past few exercises: count the number of times a function has been called, and perform different task based on that. Here what we do also depends on the parameter that was originally passed to the function (the num).

The flexibility and power of closure should be quite apparent here. To implement this functionality using a traditional OOP language such as Java would require quite a bit more lines of code.

## Exercise 16

> Create a function average that accepts no arguments, and returns a function (that will accept either a number as its lone argument, or no arguments at all). When the returned function is invoked with a number, the output should be average of all the numbers have ever been passed into that function (duplicate numbers count just like any other number). When the returned function is invoked with no arguments, the current average is outputted. If the returned function is invoked with no arguments before any numbers are passed in, then it should return 0.

```js
function average() {}

const avgSoFar = average();
console.log(avgSoFar()); // => should log 0
console.log(avgSoFar(4)); // => should log 4
console.log(avgSoFar(8)); // => should log 6
console.log(avgSoFar()); // => should log 6
console.log(avgSoFar(12)); // => should log 8
console.log(avgSoFar()); // => should log 8
```

### Solution 16

```js
function average() {
  let counter = 0;
  let total = 0;
  function closureFn(num) {
    if (num === undefined) {
      return counter === 0 ? 0 : total / counter;
    }
    counter++;
    total += num;
    return total / counter;
  }
  return closureFn;
}
```

Again the example output should make the required functionality clear. We are creating averages, so we need two variables in our outside scope, a counter to keep count and a variable to keep track of the total of arguments that have been passed `total`. Our inside function then exhibits different functionality based on whether it receives an argument or not.

## Exercise 17

> Create a function makeFuncTester that accepts an array (of two-element sub-arrays), and returns a function (that will accept a callback). The returned function should return true if the first elements (of each sub-array) being passed into the callback all yield the corresponding second elements (of the same sub-array). Otherwise, the returned function should return false.

```js
function makeFuncTester() {}
const capLastTestCases = [];
capLastTestCases.push(["hello", "hellO"]);
capLastTestCases.push(["goodbye", "goodbyE"]);
capLastTestCases.push(["howdy", "howdY"]);
const shouldCapitalizeLast = makeFuncTester(capLastTestCases);
const capLastAttempt1 = (str) => str.toUpperCase();
const capLastAttempt2 = (str) => str.slice(0, -1) + str.slice(-1).toUpperCase();
console.log(shouldCapitalizeLast(capLastAttempt1));
// => should log false
console.log(shouldCapitalizeLast(capLastAttempt2));
// => should log true
```

### Solution 17

```js
function makeFuncTester(arrOfTests) {
  function closureFn(callback) {
    return arrOfTests.every((couple) => callback(couple[0]) === couple[1]);
  }
  return closureFn;
}
```

We are mixing closure and callbacks, so it can be a _little_ confusing here, but basically passing an array (of arrays) to our outer function, and then when we provide a callback as an argument to the inside function, we want to make sure that the result of the callback is correctly stored as the second element in our original array.

Notice the use of Array.prototype.every() method here, a very useful Array method that returns true only if the callback returns true for every element of the array. It simplifies our code quite a bit.

## Exercise 18

> Create a function makeHistory that accepts a number (which will serve as a limit), and returns a function (that will accept a string). The returned function will save a history of the most recent "limit" number of strings passed into the returned function (one per invocation only). Every time a string is passed into the function, the function should return that same string with the word 'done' after it (separated by a space). However, if the string 'undo' is passed into the function, then the function should delete the last action saved in the history, and return that deleted string with the word 'undone' after (separated by a space). If 'undo' is passed into the function and the function's history is empty, then the function should return the string 'nothing to undo'.

```js
function makeHistory() {}

const myActions = makeHistory(2);
console.log(myActions("jump"));
// => should log 'jump done'
console.log(myActions("undo"));
// => should log 'jump undone'
console.log(myActions("walk"));
// => should log 'walk done'
console.log(myActions("code"));
// => should log 'code done'
console.log(myActions("pose"));
// => should log 'pose done'
console.log(myActions("undo"));
// => should log 'pose undone'
console.log(myActions("undo"));
// => should log 'code undone'
console.log(myActions("undo"));
// => should log 'nothing to undo'
```

### Solution 18

```js
function makeHistory(limit) {
  const memory = [];
  function closureFn(input) {
    if (input !== "undo") {
      if (memory.length >= limit) memory.shift();
      memory.push(input);
      return input + " done";
    } else {
      if (memory.length === 0) return "nothing to do";
      let remove = memory.pop();
      return remove + " undone";
    }
  }
  return closureFn;
}
```

Implementing "undo" was an interesting challenge. Turns out we basically need our usual _memory_ in the outer scope (this time in the form of an array) but our memory should only stretch `limit` items. So we need to keep count of how many items is in our memory array, and if we input more elements in it, implement a sliding window as in a FIFO way to only keep the correct number of items in it.

## Exercise 19

> Inspect the commented out test cases carefully if you need help to understand these instructions.

> Create a function blackjack that accepts an array (which will contain numbers ranging from 1 through 11), and returns a DEALER function. The DEALER function will take two arguments (both numbers), and then return yet ANOTHER function, which we will call the PLAYER function.
> On the FIRST invocation of the PLAYER function, it will return the sum of the two numbers passed into the DEALER function.

> On the SECOND invocation of the PLAYER function, it will return either:

> the first number in the array that was passed into blackjack PLUS the sum of the two numbers passed in as arguments into the DEALER function, IF that sum is 21 or below, OR
> the string 'bust' if that sum is over 21.

> If it is 'bust', then every invocation of the PLAYER function AFTER THAT will return the string 'you are done!' (but unlike 'bust', the 'you are done!' output will NOT use a number in the array). If it is NOT 'bust', then the next invocation of the PLAYER function will return either:

> the most recent sum plus the next number in the array (a new sum) if that new sum is 21 or less, OR
> the string 'bust' if the new sum is over 21.

> And again, if it is 'bust', then every subsequent invocation of the PLAYER function will return the string 'you are done!'. Otherwise, it can continue on to give the next sum with the next number in the array, and so forth.
> You may assume that the given array is long enough to give a 'bust' before running out of numbers.

> BONUS: Implement blackjack so the DEALER function can return more PLAYER functions that will each continue to take the next number in the array after the previous PLAYER function left off. You will just need to make sure the array has enough numbers for all the PLAYER functions.

```js
function blackjack() {}
// /*** DEALER ***/
const deal = blackjack([
  2,
  6,
  1,
  7,
  11,
  4,
  6,
  3,
  9,
  8,
  9,
  3,
  10,
  4,
  5,
  3,
  7,
  4,
  9,
  6,
  10,
  11,
]);

// /*** PLAYER 1 ***/
const i_like_to_live_dangerously = deal(4, 5);
console.log(i_like_to_live_dangerously());
// => should log 9
console.log(i_like_to_live_dangerously());
// => should log 11
console.log(i_like_to_live_dangerously());
// => should log 17
console.log(i_like_to_live_dangerously());
// => should log 18
console.log(i_like_to_live_dangerously());
// => should log 'bust'
console.log(i_like_to_live_dangerously());
// => should log 'you are done!'
console.log(i_like_to_live_dangerously());
// => should log 'you are done!'

// /*** BELOW LINES ARE FOR THE BONUS ***/

// /*** PLAYER 2 ***/
const i_TOO_like_to_live_dangerously = deal(2, 2);
console.log(i_TOO_like_to_live_dangerously());
// => should log 4
console.log(i_TOO_like_to_live_dangerously());
// => should log 15
console.log(i_TOO_like_to_live_dangerously());
// => should log 19
console.log(i_TOO_like_to_live_dangerously());
// => should log 'bust'
console.log(i_TOO_like_to_live_dangerously());
// => should log 'you are done!
console.log(i_TOO_like_to_live_dangerously());
// => should log 'you are done!

// /*** PLAYER 3 ***/
const i_ALSO_like_to_live_dangerously = deal(3, 7);
console.log(i_ALSO_like_to_live_dangerously());
// => should log 10
console.log(i_ALSO_like_to_live_dangerously());
// => should log 13
console.log(i_ALSO_like_to_live_dangerously());
// => should log 'bust'
console.log(i_ALSO_like_to_live_dangerously());
// => should log 'you are done!
console.log(i_ALSO_like_to_live_dangerously());
// => should log 'you are done!
```

### Solution 19

```js
function blackjack(array) {
  let dealerCount = 0;
  function dealer(a, b) {
    let playerCount = 0;
    let total = a + b;
    function player() {
      if (total === "bust") return "you are done!";
      dealerCount++;
      playerCount++;
      if (playerCount === 1) return total;
      total += array[dealerCount - 2];
      if (total > 21) {
        total = "bust";
        dealerCount--;
      }
      return total;
    }
    return player;
  }
  return dealer;
}
```

By this point, the code should be pretty self-explanatory so I won't explain it line by line. The most important concept here is that we have two closures here, one inside the other. The outer function can be thought of as the deck of cards, the function inside that can be thought of as the dealer, and the one inside that can be thought of as the players. Thinking logically about blackjack, a dealer can deal many players, and a single deck of cards can be used in many dealings. Thinking like this should clarify where each variable that acts as _memory_ should reside.

Implementing the bonus part just required realising that we needed two different counters, one for the dealer and one for the players, and then to modify the logic very slightly to count correctly.

I know I've harped on this time and time, but I have implemented blackjack exercises quite a few times in different languages, generally using OOP paradigms. It has always required a lot more code than this. Using closure and realising the power that having _memory_ gives functions is quite amazing.

We're done with closure exercises. Next up: Asynchronous JavaScript!
