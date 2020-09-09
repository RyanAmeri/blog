---
title: Why you should think like JavaScript
description: An introduction to Think Like JavaScript series
tags: javascript, webdev
series: Think Like JavaScript
---

## JavaScript is peculiar

JavaScript is a _peculiar_ and _unique_ language. Peculiar because on the surface, it's a scripting language which largely resembles languages such as Java and C#. Deep down however, it's got more in common with programming languages such as [Scheme](<https://en.wikipedia.org/wiki/Scheme_(programming_language)>) and [Self](<https://en.wikipedia.org/wiki/Self_(programming_language)>); languages that are mostly unknown outside of computer science academic circles or niche specialties. Most of us approaching JavaScript with a background in Java or PHP are initially deceived by JavaScript's Java-like syntax into thinking we know how it works. Then we scratch the surface, come across prototypes or how to determine the scope of `this`, and our mental model breaks down. Suddenly JavaScript looks weird, and our code has inexplicable bugs.

## But JavaScript is universal

But JavaScript is the world's [most used programming language](https://insights.stackoverflow.com/survey/2019#technology-_-programming-scripting-and-markup-languages). It is also the lingua franca of the biggest platform ever created: the Web. In terms of reach, number of users or number of applications developed, the web is larger than iOS, Android and Windows combined; and JavaScript has been pretty much its only programming language since first introduced in 1995. Many have tried to dislodge it from its place, Sun with Java applets, Adobe with Flash and ActionScript, Microsoft with JScript and Active X, and later on again with .Net and Silverlight, Google with Chrome Native Client. All have failed. I am a big fan of [Wasm](https://en.wikipedia.org/wiki/WebAssembly) and very hopeful that it can succeed in bringing other programming languages to the web, but I have no doubts that twenty years from now, as was the case twenty years ago, the main language of the web will still be JavaScript.

Not that JavaScript's use is limited to frontend web development of course. Now that with node.js and Deno it runs on the backend, with Electron it runs desktop applications, and with React Native (among others) it can be used to create mobile apps. No matter which area we practice in, it is upon us as practitioners to learn our tool well. If we need to use JavaScript and want to learn how to write code with less bugs, we need to understand how to think like JavaScript.

[Kyle Simpson](https://github.com/getify) says:

<h3 align="center"> Whenever there's a divergence between what your brain thinks is happening, and what the computer does, that's where bugs enter code</h3><br>

## Who is this series for

In this series, we'll do a deep dive into the foundations of JavaScript. This series is geared towards the intrepid developer who has a fair understanding of using JavaScript and wants to delve a bit more into its internals to see how it _really_ works. If you have around a year of JavaScript programming under your belt, or are able to follow the exercises in my [Mastering Hard Parts of JavaScript](https://dev.to/ryanameri/mastering-hard-parts-of-javascript-callbacks-i-3aj0) series, you're good to go!

In particular, if you've ever wondered:

- What exactly is the difference between `==` and `===`? _(hint: if you think that `==` doesn't check the types, you are mistaken!)_
- Why were `let` and `const` introduced and what do they actually do that's different to `var`?
- What's the difference between a function declaration and function expression? What is this _"hoisting"_ you keep seeing?
- Where should you use arrow functions and where should you avoid them?
- Should you use `this`? Or should you architecture your code to avoid using it?
- Is there a place for factory functions in modern JavaScript with ES Modules?
- How are classes implemented with prototypes? And what's a prototype anyway?

This series will hopefully prove useful.

## A note on JavaScript's backwards compatibility

I won't repeat the rather unique history of JavaScript here, which has been [well covered](https://auth0.com/blog/a-brief-history-of-javascript/) elsewhere; but throughout the series we will come across _historical bugs_ (hello `typeof null === 'object'`!) or features that were _fixed_ in later years by adding more features, and it's important to understand why JavaScript is developed the way it is.

From its birth by Brendan Eich at Netscape, JavaScript has gone through periods of neglect (1999 to 2009) as well as rapid advancement (2015 til present). What has remained constant however has been the JavaScript designers' absolute commitment to backwards compatibility. Every line of code written by a developer in 1997 that conformed to the first standardised version of JavaScript (ES1) will run exactly as its author intended in the latest versions of Chrome and Firefox, even on devices that could not have been imagined in 1997.

Most other popular languages of the past twenty something years can't boast the same claim. Python programs written in say 2005 would have been written in Python 2, and they would need to be ported to Python 3 (which is not backwards compatible with Python 2) to continue working today. The PHP folks similarly went through huge pains going from PHP 5 to PHP 6 (which was abandoned) and now PHP 7. Perl 6 similarly diverged from Perl 5 so much that the people behind it decided to spin it off as a [different programming language](<https://en.wikipedia.org/wiki/Raku_(programming_language)>). This is not to disparage these programming communities. Breaking backwards compatibility has huge benefits: it allows the language designers to clear out the _bad parts_ of a language, or to re-architect it to move with the times.

JavaScript however has not had the luxury of breaking backwards compatibility due to its unique place as the language of the web. The designers of JavaScript have always been aware that changing JavaScript in a backwards incompatible way would mean that some old, neglected, but still useful website out there would break and never be fixed again. As such they've had to evolve JavaScript with this huge constraint around their neck. They even take this commitment so seriously that when introducing new features to the language, they try to make sure that whatever keyword they pick up breaks the least number of websites and libraries out there (this is how we end up with inelegant names such as [globalThis](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/globalThis)).

It's important to keep this unique history in mind as we learn to think like JavaScript, which involves being aware of its historical bugs and peculiarities. On the other hand, as a JavaScript developer, you are blessed knowing that whatever code you write will probably run as you intend it, twenty years from now.

## Structure of the series and credits

The "Think Like JavaScript" series will cover the following pillars of JavaScript:

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
   - Class
   - Prototypes

Each section will end with a practice exercise to solidify the topics covered in the post. In the latter sections, we will develop a library checkout system and implement the same program using various different structures: using functions, modules, classes and prototypes. This will enable us to clearly see the pros and cons of each, and where each structure might be preferred.

Though this blog series' content and presentation are my own and original, I owe a heavy debt to [Kyle Simpson](https://twitter.com/getify?lang=en) for his excellent structure of material in his [Deep JavaScript Foundations v3](https://frontendmasters.com/courses/deep-javascript-v3/introduction/) course on [Frontend Masters](https://frontendmasters.com). I'd highly encourage those who want to dive more in-depth to take the course and to read his [You Don't Know JS](https://github.com/getify/You-Dont-Know-JS) book series. This does **not** imply that the content here is in any shape or form endorsed or approved by Kyle Simpson.
