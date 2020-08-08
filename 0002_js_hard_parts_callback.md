---
series: JavaScript The Hard Parts
description: An introduction to JavaScript callbacks by completing 24 exercises
---

# Mastering Hard Parts of JavaScript - Part 1: Callbacks

I'm currently undertaking [JavaScript: The Hard Parts v2](https://frontendmasters.com/courses/javascript-hard-parts-v2) course at Frontend Masters. It is a brilliant course taught by the amazing Will Sentance. The course goes over the following key concepts:

1. Callbacks & Higher order functions
2. Closure (scope and execution context)
3. Asynchronous JavaScript & the event loop
4. Classes & Prototypes (OOP)

In this tutorial series, I will go over the exercises given in each section, provide my own solution and provide a commentary as to how I came to that solution. This first part deals with Callbacks.

Callbacks are an inherently fundamental concept in JS, as most everything from closure to asynchronous JavaScript is built upon them. Prior to my introduction to JS, I had never encountered higher ordered functions (a function that can take another function as input, or return a function) so I initially found the concept very confusing. Thankfully, with lots of practice, I was able to get a good handle on callbacks. I'd encourage you to implement your own solutions first before looking at mine and then compare and contrast. There are certainly many different ways of solving these exercises and mine are definitely not necessarily the best. My solutions are all available [on github](https://github.com/RyanAmeri/FrontEnd-Masters/tree/master/js_the_hard_parts_v2) and you are very welcome to fork the repo to work on your own or, if you have found a better way of solving these, send a PR.

If you are new to JS or have a hard time getting your head wrapped around callbacks, I think going through these exercises will help you master the concept. For more information, Will's slides for the course can be found [here](https://static.frontendmasters.com/resources/2019-09-18-javascript-hard-parts-v2/javascript-hard-parts-v2.pdf)(pdf).

## Exercise 1

> Create a function addTwo that accepts one input and adds 2 to it.

`console.log(addTwo(3))` should output `5`
and
`console.log(addTwo(10))`
should output `12`

### Solution 1

```javascript
function addTwo(num) {
  return num + 2;
}
```

The most simple exercise. It gives us a nice comforting feeling knowing that we know how to use functions. Don't worry, things will get interesting soon!

## Exercise 2

> Create a function addS that accepts one input and adds an "s" to it.

`console.log(addS("pizza"));` should output `pizzas` and `console.log(addS("bagel"));` should output `bagels`

### Solution 2

```javascript
function addS(word) {
  return word + "s";
}
```

Another easy function. Good reminder that `+` is an overloaded operator in JS that can work with strings _*and*_ numbers.

## Exercise 3

> Create a function called map that takes two inputs:
> an array of numbers (a list of numbers)
> a 'callback' function - a function that is applied to each element of the array (inside of the function 'map')
> Have map return a new array filled with numbers that are the result of using the 'callback' function on each element of the input array.

`console.log(map([1, 2, 3], addTwo));` should output `[ 3, 4, 5 ]`

### Solution 3

```js
function map(array, callback) {
  const newArr = [];
  for (let i = 0; i < array.length; i++) {
    newArr.push(callback(array[i]));
  }
  return newArr;
}
```

Now this is more interesting! We are basically re-implementing a simple version of the native Array.prototype.map() function here. I decided to use a basic for loop here as most people should be familiar with it. I think this is probably the most important exercise in the series, if you can get head around this, you've basically gotten callbacks!

## Exercise 4

> The function forEach takes an array and a callback, and runs the callback on each element of the array. forEach does not return anything.

```js
let alphabet = "";
const letters = ["a", "b", "c", "d"];
forEach(letters, function (char) {
  alphabet += char;
});
console.log(alphabet);
```

should output `abcd`

### Solution 4

```js
function forEach(array, callback) {
  for (let i = 0; i < array.length; i++) {
    callback(array[i]);
  }
}
```

Another reimplementation of a native Array method. Notice the difference with map, map returns an array, forEach doesn't return anything so whatever needs to happen needs to take place in the body of the callback function.

## Exercise 5

> Rebuild your map function, this time instead of using a for loop, use your own forEach function that you just defined. Call this new function mapWith.

`console.log(mapWith([1, 2, 3], addTwo));` should output `[ 3, 4, 5 ]`

### Solution 5

```js
function mapWith(array, callback) {
  const newArr = [];
  forEach(array, (item) => {
    newArr.push(callback(item));
  });
  return newArr;
}
```

Using your own previously defined function in this manner is very powerful. It allows you to get to grips with how functions exactly work. Now when you use a library such as lodash or underscore, you can imagine how the underlying function is implemented.

## Exercise 6

> The function reduce takes an array and reduces the elements to a single value. For example it can sum all the numbers, multiply them, or any operation that you can put into a function.

`console.log(reduce(nums, add, 0))` shout output `8`.

### Solution 6

```js
function reduce(array, callback, initialValue) {
  initialValue = initialValue ?? array[0];
  forEach(array, (item) => {
    initialValue = callback(initialValue, item);
  });
  return initialValue;
}
const nums = [4, 1, 3];
const add = function (a, b) {
  return a + b;
};
```

Ah reduce! One of the most misunderstood yet powerful functions in JS (and more broadly in functional programming). The basic concept is this: You have an initial value, you run the callback function on every item in an array, and assign the result to this initial value. At the end, you return this value.

Notice how we used our own forEach function, we could have of course also used the native array.forEach(), with the same behaviour. Also notice the use of the Nullish Coalescing Operator. This operator is new in ES2020 and it's my favourite as it makes the code a lot more readable (and doesn't suffer issues of the `||` operator when the initial value is set to `0`). For more information on this operator refer to [MDN](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Nullish_coalescing_operator).

## Exercise 7

> Construct a function intersection that compares input arrays and returns a new array with elements found in all of the inputs. BONUS: Use reduce!

```js
console.log(
  intersection([5, 10, 15, 20], [15, 88, 1, 5, 7], [1, 10, 15, 5, 20])
);
```

Should output `[5, 15]`

### Solution 7

```js
function intersection(...arrays) {
  return arrays.reduce((acc, array) => {
    return array.filter((item) => acc.includes(item));
  });
}
```

Combining reduce and filter results in a powerful function. Here, if `acc` is not provided as a param, it is set to the first array, and we are not providing it as an argument. So in subsequent calls we just filter the arrays to return items that were also included in the `acc`` array.

Notice the use of `...arrays`, here we are using the rest parameters because we don't know how many arguments will be supplied to the function.

## Exercise 8

> Construct a function union that compares input arrays and returns a new array that contains all elements. If there are duplicate elements, only add it once to the new array. Preserve the order of the elements starting from the first element of the first input array. BONUS: Use reduce!

`console.log(union([5, 10, 15], [15, 88, 1, 5, 7], [100, 15, 10, 1, 5]));` should output `[5, 10, 15, 88, 1, 7, 100]`.

### Solution 8

```js
function union(...arrays) {
  return arrays.reduce((acc, array) => {
    const newItem = array.filter((item) => !acc.includes(item));
    return acc.concat(newItem);
  });
}
```

Here again we are using reduce and filter, but the logic is flipped inside the filter method. The `acc` array is again set to the first item, but then we are checking every item in the subsequent arrays, and if that item is not included in our `acc` array, we are adding it and finally returning the accumulator.

## Exercise 9

> Construct a function objOfMatches that accepts two arrays and a callback. objOfMatches will build an object and return it. To build the object, objOfMatches will test each element of the first array using the callback to see if the output matches the corresponding element (by index) of the second array. If there is a match, the element from the first array becomes a key in an object, and the element from the second array becomes the corresponding value.

```js
console.log(
  objOfMatches(
    ["hi", "howdy", "bye", "later", "hello"],
    ["HI", "Howdy", "BYE", "LATER", "hello"],
    function (str) {
      return str.toUpperCase();
    }
  )
);
```

Should log `{ hi: 'HI', bye: 'BYE', later: 'LATER' }`

### Solution 9

```js
function objOfMatches(array1, array2, callback) {
  return array2.reduce((res, value, index) => {
    if (value === callback(array1[index])) {
      res[array1[index]] = value;
    }
    return res;
  }, Object.create(null));
}
```

The _trick_ here is to note that the accumulator that goes into reduce doesn't need to be just a primitive type, it can also be an array or an object. So here we set the accumulator `res` to an empty object, and then we check to see if calling the callback on array1 results in the same value as the item in array 2. If they are equal, we add it to our accumulator and finally return our accumulator. The power of reduce should be apparent now, but yet it might take you a bit of time and practice to wrap your head around this. That's ok! We're going to be using reduce a lot in the following exercises 😛.

## Exercise 10

> Construct a function multiMap that will accept two arrays: an array of values and an array of callbacks. multiMap will return an object whose keys match the elements in the array of values. The corresponding values that are assigned to the keys will be arrays consisting of outputs from the array of callbacks, where the input to each callback is the key.

```js
console.log(
  multiMap(
    ["catfood", "glue", "beer"],
    [
      function (str) {
        return str.toUpperCase();
      },
      function (str) {
        return str[0].toUpperCase() + str.slice(1).toLowerCase();
      },
      function (str) {
        return str + str;
      },
    ]
  )
);
```

should output `{ catfood: ['CATFOOD', 'Catfood', 'catfoodcatfood'], glue: ['GLUE', 'Glue', 'glueglue'], beer: ['BEER', 'Beer', 'beerbeer'] }`

### Solution 10

```js
function multiMap(arrVals, arrCallbacks) {
  return arrVals.reduce((accum, item) => {
    accum[item] = arrCallbacks.map((fn) => fn(item));
    return accum;
  }, Object.create(null));
}
```

Reading the exercise, it can look a little bit challenging but looking at the expected output should make things a bit more clear. Our function accepts two parameters, an array of values and an array of functions. Then we need to construct an object in some fashion. So constructing an object from an array, immediately should spring to mind _reduce_.

The next difficulty is figuring out what the value of each prop inside the object is. Based on the example output, we can see that the value should be an array, an array whereby the callback function have been called on the item one by one. So we are providing an array as input and want a different array as output, this should spring to mind _map_.

This really is the gist of callbacks in functional programming, and this example using reduce and map shows us how much can be achieved using a little declarative code.

## Exercise 11

> Construct a function objectFilter that accepts an object as the first parameter and a callback function as the second parameter. objectFilter will return a new object. The new object will contain only the properties from the input object such that the property's value is equal to the property's key passed into the callback.

```js
const cities = {
  London: "LONDON",
  LA: "Los Angeles",
  Paris: "PARIS",
};
console.log(objectFilter(cities, (city) => city.toUpperCase()));
```

Should output `{ London: 'LONDON', Paris: 'PARIS'}`

### Solution 11

```js
function objectFilter(obj, callback) {
  const newObj = Object.create(null);
  for (let [key, value] of Object.entries(obj)) {
    if (
      Object.prototype.hasOwnProperty.call(obj, key) &&
      callback(key) === value
    )
      newObj[key] = value;
  }
  return newObj;
}
```

The only trick here is to iterate properly in an array. In the old days this used to be difficult with a `for...in``` loop which could cause some unintended sideffects. Thankfully nowadays we have Object.entries() which gives us a nice array of key and values in the object, which we can safely iterate through.

In the conditional if statement, I would have normally used `if (obj.hasOwnProperty(key))` but ESLint yelled at me and said that that's not safe, and I should call the prototype method insead in this fashion to make the code safer. Technically this check is unnessary for the given example but I just wanted to demonstrate how to safely check if an object has a property in modern JS.

## Exercise 12

> Create a function majority that accepts an array and a callback. The callback will return either true or false. majority will iterate through the array and perform the callback on each element until it can be determined if the majority of the return values from the callback are true. If the number of true returns is equal to the number of false returns, majority should return false.

```js
const isOdd = function (num) {
  return num % 2 === 1;
};
console.log(majority([1, 2, 3, 4, 5, 7, 9, 11], isOdd));
```

should log `true`

```js
console.log(majority([2, 3, 4, 5], isOdd));
```

should log `false`.

### Solution 12

```js
function majority(array, callback) {
  let trueCount = 0;
  let falseCount = 0;
  array.forEach((item) => {
    callback(item) ? trueCount++ : falseCount++;
  });
  return trueCount > falseCount ? true : false;
}
```

I found this exercice to be on the easy side, as long as you use two variables and initialise them to zero. Also demonstrating the use of the terneray operator, which I find helps in readablity of simple `if...else...` blocks.

## Exercise 13

> Create a function prioritize that accepts an array and a callback. The callback will return either true or false. prioritize will iterate through the array and perform the callback on each element, and return a new array, where all the elements that yielded a return value of true come first in the array, and the rest of the elements come second.

```js
const startsWithS = function (str) {
  return str[0] === "s" || str[0] === "S";
};
console.log(
  prioritize(
    ["curb", "rickandmorty", "seinfeld", "sunny", "friends"],
    startsWithS
  )
);
```

Should log `['sunny', 'seinfeld', 'curb', 'rickandmorty', 'friends']`

### Solution 13

```js
function prioritize(array, callback) {
  return array.reduce((accum, item) => {
    callback(item) ? accum.unshift(item) : accum.push(item);
    return accum;
  }, []);
}
```

This is actually very similar to the previous exercise, except now instead of having two variables, we have two arrays, a truthy array and a falsy array. Since the example expects a single array to be returned, we need to concat these two arrays. I decided to save a bit of code and memory and just use a single array, and use two different array methods, unshift() and push() to put truthy and falsy values at the two ends of the array. Also notice that we are again using reduce, but providing an empty array as the accumulator of our reduce.

## Exercice 14

> Create a function countBy that accepts an array and a callback, and returns an object. countBy will iterate through the array and perform the callback on each element. Each return value from the callback will be saved as a key on the object. The value associated with each key will be the number of times that particular return value was returned.

```js
console.log(
  countBy([1, 2, 3, 4, 5], function (num) {
    if (num % 2 === 0) return "even";
    else return "odd";
  })
);
```

should log `{ odd: 3, even: 2 }`

### Solution 14

```js
function countBy(array, callback) {
  return array.reduce((obj, item) => {
    let result = callback(item);
    obj[result] ? (obj[result] = obj[result] + 1) : (obj[result] = 1);
    return obj;
  }, Object.create(null));
}
```

By now we should be familiar with this pattern. We are taking in an array and returning a single object, so we're looking for reduce! We provide the new object as the accumulator to reduce, run the callback on the items in the array, and based on its return value, set the value in the object accordingly.

The power of reduce should be apparent now.

## Exercise 15

> Create a function groupBy that accepts an array and a callback, and returns an object. groupBy will iterate through the array and perform the callback on each element. Each return value from the callback will be saved as a key on the object. The value associated with each key will be an array consisting of all the elements that resulted in that return value when passed into the callback.

```js
const decimals = [1.3, 2.1, 2.4];
const floored = function (num) {
  return Math.floor(num);
};
console.log(groupBy(decimals, floored));
```

should log `{ 1: [1.3], 2: [2.1, 2.4] }`

### Solution 15

```js
function groupBy(array, callback) {
  return array.reduce((obj, item, index, arr) => {
    let res = callback(item);
    obj[res] = arr.filter((element) => parseInt(element) === parseInt(res));
    return obj;
  }, Object.create(null));
}
```

The solution here requires knowing that reduce can take more than a callback and the item, it can also (optionally) take the index of the array and the whole array as parameters as well.

If this was being done by traditional for loops, you'd need two nested loops here. Using these Array methods, the first loop is replaced with array.reduce() and the second loop is replaced by arr.filter(). filter() takes a callback function and returns all the elements for which the callback returns true. Here filter returns an array, and we just assign that to be the value inside our newly created (accumulator) object.

It took me a while to get comfortable with this style of declarative programming and using Array methods. Once you do get comfortable with them though, you don't want to go back to for loops, with all the potential off-by-one errors they introduce. But sometimes a loop is just easier to read and implement, as we'll see in the next exercise.

## Exercise 16

> Create a function goodKeys that accepts an object and a callback. The callback will return either true or false. goodKeys will iterate through the object and perform the callback on each value. goodKeys will then return an array consisting only the keys whose associated values yielded a true return value from the callback.

```js
const sunny = {
  mac: "priest",
  dennis: "calculating",
  charlie: "birdlaw",
  dee: "bird",
  frank: "warthog",
};
const startsWithBird = function (str) {
  return str.slice(0, 4).toLowerCase() === "bird";
};
console.log(goodKeys(sunny, startsWithBird));
```

should log `['charlie', 'dee']`

### Solution 16

```js
function goodKeys(obj, callback) {
  const arr = [];
  for (let [key, value] of Object.entries(obj)) {
    if (callback(value)) arr.push(key);
  }
  return arr;
}
```

In this exercise, I really struggled to use Array methods instead. The callback function ended up looking rather ugly, and it really felt like it wasn't the right tool for the job. There is no point being dogmatic, if a loop is easier to read and does the job well enough, we don't need to avoid it! Especially now that we can easily turn objects into iterables with Object.entries() and then use destructuring and the `for...of` loop to easily iterate through it.

If you have a neat way of using an Array method instead of a loop to solve this, please let me know!

## Exercise 17

> Create a function commutative that accepts two callbacks and a value. commutative will return a boolean indicating if the passing the value into the first function, and then passing the resulting output into the second function, yields the same output as the same operation with the order of the functions reversed (passing the value into the second function, and then passing the output into the first function).

```js
const multBy3 = (n) => n * 3;
const divBy4 = (n) => n / 4;
const subtract5 = (n) => n - 5;
console.log(commutative(multBy3, divBy4, 11));
```

should log `true`

```js
console.log(commutative(multBy3, subtract5, 10));
```

should log `false`

```js
console.log(commutative(divBy4, subtract5, 48));
```

should log `false`

### Solution 17

```js
function commutative(func1, func2, value) {
  return func1(func2(value)) === func2(func1(value)) ? true : false;
}
```

The explanation might look daunting, but once you delve into it, you'll realise that it's actually a rather simple exercise, it just requires good understanding of callbacks (which is the point of all of these really!) The question is, if we pass the value to the first function, is the result equal to passing it to the second functrion?

Using the ternary operator for simple statements like this can make the code both concise and readable. Just note that you have to put the return statemtn before the first operand.

## Exercise 18

> Create a function objFilter that accepts an object and a callback. objFilter should make a new object, and then iterate through the passed-in object, using each key as input for the callback. If the output from the callback is equal to the corresponding value, then that key-value pair is copied into the new object. objFilter will return this new object.

```js
const startingObj = {};
startingObj[6] = 3;
startingObj[2] = 1;
startingObj[12] = 4;
const half = (n) => n / 2;
console.log(objFilter(startingObj, half));
```

should log `{ 2: 1, 6: 3 }`

### Solution 18

```js
function objFilter(obj, callback) {
  const newObj = Object.create(null);
  for (let [key, value] of Object.entries(obj)) {
    if (value === callback(parseInt(key))) newObj[key] = value;
  }
  return newObj;
}
```

Once again, because an object is being passed to the function, I find using a for loop easier than an Array method, though the latter is also definitely possible.

The key thing to keep in mind here is that object properties are stored as strings, even if they are just numbers. So when making the comparison, we need to make sure we cast it to the correct type (using parseInt()) to make sure that strict equality passes.

## Exercise 19

> Create a function rating that accepts an array (of functions) and a value. All the functions in the array will return true or false. rating should return the percentage of functions from the array that return true when the value is used as input.

```js
const isEven = (n) => n % 2 === 0;
const greaterThanFour = (n) => n > 4;
const isSquare = (n) => Math.sqrt(n) % 1 === 0;
const hasSix = (n) => n.toString().includes("6");
const checks = [isEven, greaterThanFour, isSquare, hasSix];
console.log(rating(checks, 64));
```

should log `100`

```js
console.log(rating(checks, 66));
```

should log `75`

### Solution 19

```js
function rating(arrOfFuncs, value) {
  let trueCnt = arrOfFuncs.reduce((accum, fn) => {
    if (fn(value)) accum++;
    return accum;
  }, 0);
  return (trueCnt / arrOfFuncs.length) * 100;
}
```

An array of functions can look a bit intimidating at first, but it's just an array! So we are taking in an array, and we want a single value to be computed from the array (the number of times its functions returned true) so we are looking at reduce again! Yay!

Here `accum` is initially set to `0`, and everytime the function inside the array returnes true, we increment it. Finally we do a quick calculation based on the size of the array to turn this count into a percentage and return it.

## Exercise 20

> Create a function pipe that accepts an array (of functions) and a value. pipe should input the value into the first function in the array, and then use the output from that function as input for the second function, and then use the output from that function as input for the third function, and so forth, until we have an output from the last function in the array. pipe should return the final output.

```js
const capitalize = (str) => str.toUpperCase();
const addLowerCase = (str) => str + str.toLowerCase();
const repeat = (str) => str + str;
const capAddlowRepeat = [capitalize, addLowerCase, repeat];
console.log(pipe(capAddlowRepeat, "cat"));
```

should log `'CATcatCATcat'`

### Solution 20

```js
function pipe(arrOfFuncs, value) {
  return arrOfFuncs.reduce((accum, fn) => {
    return fn(accum) || fn(value);
  }, "");
}
```

I admit that I found this exercise difficult at first. The trick is in the callback function inside the reduce. The line `return fn(accum) || fn(value);` means `if fn(accum) return fn(accum) else return fn(value)` But I've seen this condensed format using the `||` operator used a lot in open source projects so I decided to use it here even if to a newbie's eyes, the more descriptive format is more readbale.

The initial `accum` in the reduce is an empty string. If fn(accum) returns empty, then assign it to fn(value). In consecutive calls, fn(accum) will return true, so its new value is returned.

## Exercise 21

> Create a function highestFunc that accepts an object (which will contain functions) and a subject (which is any value). highestFunc should return the key of the object whose associated value (which will be a function) returns the largest number, when the subject is given as input.

```js
const groupOfFuncs = {};
groupOfFuncs.double = (n) => n * 2;
groupOfFuncs.addTen = (n) => n + 10;
groupOfFuncs.inverse = (n) => n * -1;
console.log(highestFunc(groupOfFuncs, 5));
// should log: 'addTen'
console.log(highestFunc(groupOfFuncs, 11));
// should log: 'double'
console.log(highestFunc(groupOfFuncs, -20));
// should log: 'inverse'
```

### Solution 21

```js
function highestFunc(objOfFuncs, subject) {
  let largest = Number.NEGATIVE_INFINITY;
  let rightKey = undefined;
  for (let [key, fn] of Object.entries(objOfFuncs)) {
    if (fn(subject) > largest) {
      largest = fn(subject);
      rightKey = key;
    }
  }
  return rightKey;
}
```

The important thing here is to note that we need to keep hold of two values, what is the largest number returned from the functions, and what is its key. So we define these two variables and initialize them to temporary values. We then loop through the object using our tried and true Object.entries() method, call the function on the subjct, and check to see if its return value is larger than what we currently have stored. If it is, we store that key, and finally once we have looped over the object, we return that key.

## Exercise 22

> Create a function, combineOperations, that takes two parameters: a starting value and an array of functions. combineOperations should pass the starting value into the first function in the array. combineOperations should pass the value returned by the first function into the second function, and so on until every function in the array has been called. combineOperations should return the final value returned by the last function in the array.

```js
function add100(num) {
  return num + 100;
}

function divByFive(num) {
  return num / 5;
}

function multiplyByThree(num) {
  return num * 3;
}

function multiplyFive(num) {
  return num * 5;
}

function addTen(num) {
  return num + 10;
}

console.log(combineOperations(0, [add100, divByFive, multiplyByThree]));
// Should output 60 -->
console.log(combineOperations(0, [divByFive, multiplyFive, addTen]));
// Should output 10
```

### Solution 22

```js
function combineOperations(startVal, arrOfFuncs) {
  return arrOfFuncs.reduce((accum, fn) => {
    return fn(accum);
  }, startVal);
}
```

Again, we are being given an array, and we want a single value computed from that array, so we are looking at reduce. This is very similar to exercise 20. The only thing to note here is that we can set the accum of reduce to startVal when we create our reduce.

## Exercise 23

> Define a function myFunc that takes an array and a callback. myFunc should pass each element from the array (in order) into the callback. If the callback returns true, myFunc should return the index of the current element. If the callback never returns true, myFunc should return -1;

```js
const numbers = [2, 3, 6, 64, 10, 8, 12];
const evens = [2, 4, 6, 8, 10, 12, 64];

function isOddAgain(num) {
  return num % 2 !== 0;
}

console.log(myFunc(numbers, isOddAgain));
// Output should be 1
console.log(myFunc(evens, isOddAgain));
// Output should be -1
```

### Solution 23

```js
function myFunc(array, callback) {
  return array.findIndex(callback);
}
```

At first I was going to implement the functionality manually using reduce (I think I'm overusing reduce at this point!) but then I looked at the definition again: return the first index if found, return `-1` if not found. I realised that this was the definition of the findIndex() Array method, so all we need to do is run findIndex in the input array using the callback. Simple!

## Exercise 24

> Write a function myForEach that accepts an array and a callback function. Your function should pass each element of the array (in order) into the callback function. The behavior of this function should mirror the functionality of the native .forEach() JavaScript array method as closely as possible.

```js
let sum = 0;

function addToSum(num) {
  sum += num;
}

const nums2 = [1, 2, 3];
myForEach(nums2, addToSum);
console.log(sum);
// Should output 6
```

### Solution 24

```js
function myForEach(array, callback) {
  for (let item of array) {
    callback(item);
  }
}
```

A bit of a throwback to the earlier exercises, implementing forEach manually again. The only difference is that we're manipulating the variable `sum` in the global scope here. I decided that using array.forEach() to create my own forEach was cheating 😉so used a `for... of` loop instead.

If you found this last exercise very easy, you should see how far you have come since the first exercise in using callbacks and just being comfortable with them.
