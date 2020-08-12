---
series: Mastering Hard Parts of JavaScript
description: An introduction to JavaScript Prototype & Class using 15 exercises
---

# Mastering Hard Parts of JavaScript: Prototype & Class

## How come JavaScript is not Object Oriented but everything in JavaScript is an Object?

You've probably heard that in JavaScript, "everything is an object". This is actually incorrect (primitive types such as String or undefined are not objects), but **a lot** of things, i.e., everything other than primitive types _is_ an object, for example functions, arrays or data structures such as Set or Map. JavaScript is a strongly Object Oriented language, yet many people coming to it from other languages such as Python or Java find JavaScript's OO design baffling. Why is that? It's because JavaScript was traditionally a prototypal, classless Object Oriented language.

Prototypal and class-based object oriented languages both implement OOP principles but in strikingly different ways. Each have their own pros and cons and neither is inherently superior to the other. However if you come from a background in Java or Python or C#, spending some time to coming to grips with JavaScript's prototypal structure will pay huge dividends.

In recent times (ES6), a `class` keyword was added to JavaScript that closely mimics the class-based OOP of other languages. But `class` is simply syntactic sugar in JS, and under the hood it still implements OOP using prototypal concepts. Even if you prefer using the class structure, understanding JS's prototypes is required to be able to properly debug your code or understand how it works.

In the first part of this section, we'll solve our exercises using object literals, which is the original/old-fashioned way of implementing OOP in JS. Next we'll use the Object.create() method that was added to ES5. In the third section we'll see how the use of the `new` keyword simplifies object creation (but doesn't fundamentally change anything). Finally we'll see the `class` structure that was introduced in ES6.

Just to clarify, the heading of this section was written in jest. JavaScript is an Object Oriented language, and not everything in JavaScript is an object.

## Using Object Literals

### Exercise 1

> Create a function that accepts two inputs (name and age) and returns an object. Let's call this function makePerson. This function will:
>
> - create an empty object
> - add a name property to the newly created object with its value being the 'name' argument passed into the function
> - add an age property to the newly created object with its value being the 'age' argument passed into the function
>   return the object.

```js
function makePerson(name, age) {
  //add code here
}

const vicky = makePerson("Vicky", 24);
console.log(vicky.name);
// -> Logs 'Vicky'
console.log(vicky.age);
// -> Logs 24
```

### Solution 1

```js
function makePerson(name, age) {
  const person = {};
  person.name = name;
  person.age = age;
  return person;
}
```

Creating an object the "old fashioned" way using a function. No one uses this pattern anymore, but everything that comes later (such as `Object.create()` or `new`) is still doing this in the background, so it's important to understand how this works.

## Using Object.create()

### Exercise 2

> Inside personStore object, create a property greet where the value is a function that logs "hello".

```js
const personStore = {
  // add code here
};

personStore.greet(); // -> Logs 'hello'
```

### Solution 2

```js
const personStore = {
  greet: function someName() {
    console.log("hello");
  },
};
```

Here greet is just a property on an object. It's value happens to be a function (that does something). We can access that property like accessing any other property of an object, using the dot notation `personStore.greet`. And since it's a function, we can run the function by adding the parentheses so `personStore.greet()` works and runs the function that we have defined.

It turns out the name of this function `someName` is not important since it's assigned to the property `greet` and so `someName` is never directly used. So we can turn it into an anonymous function:

```js
const personStore = {
  greet: function () {
    console.log("hello");
  },
};
```

And later on in ES6, they introduced a new more concise syntax, which allows us to write this example as:

```js
const personStore = {
  greet() {
    console.log("hello");
  },
};
```

All three solution shown here do exactly the same thing.

### Exercise 3

> Create a function personFromPersonStore that takes as input a name and an age. When called, the function will create person objects using the Object.create method on the personStore object.

```js
function personFromPersonStore(name, age) {
  // add code here
}

const sandra = personFromPersonStore("Sandra", 26);
console.log(sandra.name);
// -> Logs 'Sandra'
console.log(sandra.age);
//-> Logs 26
sandra.greet();
//-> Logs 'hello'
```

### Solution 3

```js
function personFromPersonStore(name, age) {
  const person = Object.create(personStore);
  person.name = name;
  person.age = age;
  return person;
}
```

The important thing to note here is that Object.create(), irrespective of the argument passed to it, always returns an empty object. So initially, `person` is an empty object with no properties.

Every object has a hidden [[prototype]] property (bad name!). Simply put, when you call a property on an object, the JS engine first checks to see if the object has that property. If it can't find such a property, it will look at its [[prototype]] property to see which Object it descends from. `const person = Object.create(personStore)` tells the JS engine to create a new empty object and return it and call it `person`, but if we call a property of `person` and `person` doesn't have that property, look up to `personStore` and see if it has that property.

So when `sandra.name` is called, the JS engine looks at the `sandra` object to see if it has a `name` property. It does, so its value is returned. Next when `sandra.age` is called, the JS engine looks at the `sandra` object to see if it has an `age` property. It does, so its value is returned. Next, `sandra.greet()` is called. The JS engine looks at the object `sandra` to see if it has a `greet` property. It does not. Instead of failing, the JS engine then looks at `sandra`'s hidden [[prototype]] property which points to personStore, so it then goes to see if `personStore` has a `greet` property. It does, so that function is invoked.

This basically achieves what class-based object oriented languages call inheritance, without using any classes. This is a gross simplification of JavaScript's prototypal architecture, for more information I've found [this chapter](https://javascript.info/prototype-inheritance) of the [javascript.info](https://javascript.info) very helpful.

### Exercise 4

> Without editing the code you've already written, add an introduce method to the personStore object that logs "Hi, my name is [name]".

```js
// add code here
sandra.introduce();
// -> Logs 'Hi, my name is Sandra'
```

### Solution 4

```js
personStore.introduce = function () {
  console.log(`Hi, my name is ${this.name}`);
};
```

We can add methods to our personStore at a later time. And the dynamic nature of JS means that even though sandra was created before we created this method, it still gets to use it. Here we also see our first introduction to the `this` keyword. In this context, `this` is always the value of the object to the left of the the function being invoked. So here when `introduce()` is called, the object to its left is `sandra`, so `this` gets assigned to `sandra`. Now when in the body of the method `this.name` is equal to `sandra.name` so its value is returned.

Do not use arrow functions when declaring methods on an object. In arrow functions `this` is lexically scoped, so in this example, if we had declared

```js
personStore.introduce = () => {
  console.log(`Hi, my name is ${this.name}`);
};
```

Here `this` would not refer to `sandra`. It would look to see if the function introduce has a `this`, it doesn't. So it would look at its outside scope to see if it has a `this`. Here the outside scope is the global memory or `window`, which does have an object called `this` so it checks to see if that object has a `name` property. It doesn't, so `undefined` is returned.

Understanding the prototype chain, what JavaScript's lexical scoping means, and how arrow functions change the behaviour of `this` are perhaps the most challenging parts of JavaScript for someone new to the language. It requires practice, but it _will_ make sense in the end.

## Using the `new` keyword

### Exercise 5

> Create a function PersonConstructor that uses the this keyword to save a single property onto its scope called greet. greet should be a function that logs the string 'hello'.

```js
function PersonConstructor() {
  // add code here
}

const simon = new PersonConstructor();
simon.greet();
// -> Logs 'hello'
```

### Solution 5

```js
function PersonConstructor() {
  this.greet = function () {
    console.log("hello");
  };
}
```

Wait, what's happening here? Actually nothing we have not seen before. The `new` keyword is a modifier, it makes some changes to the function that follows it (in this case the `PersonConstructor` function). What changes you ask? Nothing we haven't seen in previous exercises. It basically creates a new empty object, sets its [[prototype]] correctly, assigns it to the value `this` and returns this object. We can of course add new properties to this `this` object, which will be methods. And we're adding a greet method here to our object.

`new` doesn't do anything magical. It just automates what we've been doing manually until this point, create an empty object, set its [[prototype]] and return it. That's it!

### Exercise 6

> Create a function personFromConstructor that takes as input a name and an age. When called, the function will create person objects using the new keyword instead of the Object.create method.

```js
function personFromConstructor(name, age) {
  // add code here
}

const mike = personFromConstructor("Mike", 30);
console.log(mike.name); // -> Logs 'Mike'
console.log(mike.age); //-> Logs 30
mike.greet(); //-> Logs 'hello'
```

### Solution 6

```js
function personFromConstructor(name, age) {
  const person = new PersonConstructor();
  person.name = name;
  person.age = age;
  return person;
}
```

A combination of using new to create a new object, but then giving it extra properties (which are passed as parameters) and returning this object. Nothing terribly new happening here, just a combination of the two patterns we've seen before.

### Exercise 7

> Without editing the code you've already written, add an introduce method to the PersonConstructor function that logs "Hi, my name is [name]".

```js
mike.introduce();
// -> Logs 'Hi, my name is Mike'
```

### Solution 7

```js
PersonConstructor.prototype.introduce = function () {
  console.log(`Hi, my name is ${this.name}`);
};
```

Notice that this is very similar to Exercise 4, we are adding a method to an object. But which object? In Exercise 4, we had manually defined the name of our object as `personStore` so we simply added introduce as a property of that object. Here we don't have the name of the object that the JS engine automatically creates for us using the `new` keyword, so where can we store our functions? The answer is the (confusingly named) property `prototype`, which itself is an object.

Remember that all functions are objects in JavaScript? So PersonConstructor is a function which we can execute when we use (). But it's also an object, and we can access its properties like any other object using the dot notation. So in order to add any methods to our personConstructor function, we access its `prototype` property and add our methods there.

Spend some time to compare and contrast exercises 2, 3 and 4 with exercises 5, 6 and 7. You'll see that they are fundamentally doing the same thing. This comparison should make clear what the keyword `new` does and what the property `prototype` refers to.

## Using ES6 Classes

### Exercise 8

> Create a class PersonClass. PersonClass should have a constructor that is passed an input of name and saves it to a property by the same name. PersonClass should also have a method called greet that logs the string 'hello'.

```js
class PersonClass {
  constructor() {
    // add code here
  }

  // add code here
}
const george = new PersonClass();
george.greet();
// -> Logs 'hello'
```

### Solution 8

```js
class PersonClass {
  constructor(name) {
    this.name = name;
  }

  greet() {
    console.log("hello");
  }
}
```

An exact recreation of Exercise 2 and 5, here using the `class` pattern. More readable for people who come from other languages, though many JS diehards complain that this pattern just doesn't look _native_ to JS. In effect the JS engine is doing exactly what it did in Exercise 5.

### Exercise 9

> Create a class DeveloperClass that creates objects by extending the PersonClass class. In addition to having a name property and greet method, DeveloperClass should have an introduce method. When called, introduce should log the string 'Hello World, my name is [name]'.

```js
const thai = new DeveloperClass("Thai", 32);
console.log(thai.name);
// -> Logs 'Thai'
thai.introduce();
//-> Logs 'Hello World, my name is Thai'
```

### Solution 9

```js
class DeveloperClass {
  constructor(name, age) {
    this.name = name;
    this.age = age;
  }
  introduce() {
    console.log(`Hello World, my name is ${this.name}`);
  }
}
```

This of course is exactly like exercise 7, but uses the class syntax instead of adding a method to the prototype property directly. Again, behind the scenes, the JS engine performs exactly the same operation.

## Subclassing & Mixins

### Exercise 10

> Create an object adminFunctionStore that has access to all methods in the userFunctionStore object, without copying them over individually.

```js
const userFunctionStore = {
  sayType: function () {
    console.log("I am a " + this.type);
  },
};
```

### Solution 10

```js
const adminFunctionStore = Object.create(userFunctionStore);
```

### Exercise 11

> Create an adminFactory function that creates an object with all the same data fields (and default values) as objects of the userFactory class, but without copying each data field individually.

```js
function userFactory(name, score) {
  let user = Object.create(userFunctionStore);
  user.type = "User";
  user.name = name;
  user.score = score;
  return user;
}
```

### Solution 11

```js
function adminFactory(name, score) {
  const admin = Object.create(adminFunctionStore);
  admin.name = name;
  admin.score = score;
  return admin;
}
```

### Exercise 12

> Then make sure the value of the 'type' field for adminFactory objects is 'Admin' instead of 'User'.

### Solution 12

```js
adminFunctionStore.type = "Admin";
```

### Exercise 13

> Make sure that adminFactory objects have access to adminFunctionStore methods, without copying them over.

```js
function adminFactory(name, score) {
  // Put code here
}
```

### Solution 13

```js
function adminFactory(name, score) {
  const admin = Object.create(adminFunctionStore);
  admin.name = name;
  admin.score = score;
  return admin;
}
```

### Exercise 14

> Created a method called sharePublicMessage that logs 'Welcome users!' and will be available to adminFactory objects, but not userFactory objects. Do not add this method directly in the adminFactory function.

```js
const adminFromFactory = adminFactory("Eva", 5);
adminFromFactory.sayType();
// -> Logs "I am a Admin"
adminFromFactory.sharePublicMessage();
// -> Logs "Welcome users!"
```

### Solution 14

```js
userFunctionStore.sharePublicMessage = function () {
  console.log("Welcome users!");
};
```

### Exercise 15

> Mixins are a tool in object-oriented programming that allows objects to be given methods and properties beyond those provided by their inheritance. For this challenge, complete the missing code, giving all of the robotMixin properties to robotFido. Do this in a single line of code, without adding the properties individually.

```js
class Dog {
  constructor() {
    this.legs = 4;
  }
  speak() {
    console.log("Woof!");
  }
}

const robotMixin = {
  skin: "metal",
  speak: function () {
    console.log(`I have ${this.legs} legs and am made of ${this.skin}`);
  },
};

let robotFido = new Dog();

robotFido = /* Put code here to give Fido robot skills */;
robotFido.speak()
// -> Logs "I am made of metal"
```

### Solution 15

```js
Object.assign(robotFido, robotMixin);
```

In JavaScript, each object's [[prototype]] can only refer to one other object (in traditional OOP speak, each class can only extend from one class). How do we give an object extra methods declared elsewhere? `Object.assign` allows us to to that, the first argument is an object, and the second argument is also an object which has a bunch of methods. It adds those methods to the first object.

This brings to an end our Mastering Hard Parts of JavaScript tutorial series. If you have followed each section and implemented your own solutions, take a moment to reflect just how much you have learnt and how far you have come in your understanding of the hard parts of JavaScript!

I am sure my tutorial series is not without its faults. If you find any errors or a better way of solving any of these exercises, please do leave a comment, or send a PR to the github repo. Thanks!
