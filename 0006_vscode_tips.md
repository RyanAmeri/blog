# Getting the most out of VS Code as a Web Developer

Lessons from taking the Visual Studio Code course on [Frontend Masters](https://frontendmasters.com/courses/visual-studio-code/) by [Mike North](https://mike.works)

Overview

1. Creating better Markdown Documents
2. Becoming proficient in Emmet
3. Refactoring code with VS Code
4. Type-Checking JavaScript!
5. Debugging

## Creating Better Markdown Documents

You can use a lot of HTML elements inside your markdown to create better markdown documents.

### Images

To insert an image and have full control over its size attributes, you can use the HTML IMG tag such as
`<img src="assets/vscode.png" height=50 align=right vspace=20>` <img src="assets/vscode.png" height=50 align=right vspace=20>
You cannot use complex things such as srcset but for basic formatting this should do.

### Align

The `align` attribute works on most elements including `<p>` tags.

```html
<p align="right">right</p>
<p align="center">center</p>
<p align="left">left</p>
```

<p align=right>right</p>
<p align=center>center</p>
<p align=left>left</p>

### Lists

#### Unordered Lists

Unordered lists can be nested using indentation

```md
- one
  - two
    - three
      - four
```

- one
  - two
    - three
      - four

#### Ordered Lists

Ordered lists cannot be nested in most markdown rendering engines

```md
1. first
1. second
1. third
```

1. first
1. second
1. third

But we can use HTML tags instead

```html
<ol type="A">
  <li>first</li>
  <li>second</li>
  <li>third</li>
  <ol type="a">
    <li>fourth</li>
    <ol type="i">
      <li>fifth</li>
    </ol>
  </ol>
</ol>
```

<ol type='A' >
  <li> first </li>
  <li> second </li>
  <li> third</li>
  <ol type='a' >
    <li> fourth </li>
    <ol type='i' >
      <li> fifth </li>
    </ol>
  </ol>
</ol>

#### Definition Lists

We can also use the HTML `<dt> <dl>` tags to create definitions:

```html
<dl>
  <dt>Images</dt>
  <dd>.jpg, .gif, .png</dd>
  <dt>Styles</dt>
  <dd>.css</dd>
  <dt>Scripts</dt>
  <dd>.js</dd>
  <dt>Documents</dt>
  <dd>.html</dd>
</dl>
```

 <dl>
   <dt>Images</dt>
   <dd>.jpg, .gif, .png</dd>
   <dt>Styles</dt>
   <dd>.css</dd>
   <dt>Scripts</dt>   <dd>.js</dd>
   <dt>Documents</dt>
   <dd>.html</dd>
 </dl>

### Details

We can use the HTML `<details>` and `<summary>` tags to hide certain details with an arrow in our markdown

```html
<details>
  <summary>Click me!</summary>

  Some amazing things were hidden here!
</details>
```

<details>
  <summary>Click me!</summary>

Some amazing things were hidden here!

</details>
<br>

This is especially useful for example for when sending a error message in a PR in GitHub, you can hide the details so people don't have to scroll through long error messages.

### Tables

We can make tables easily, by writing down ASCII art:

```md
| Item      | Price  | Qty |
| --------- | ------ | --- |
| 🍇 Grapes | \$2.99 | 3   |
| 🍐 Pears  | \$4.15 | 1   |
| 🍋 Lemons | \$0.99 | 2   |
```

| Item      | Price  | Qty |
| --------- | ------ | --- |
| 🍇 Grapes | \$2.99 | 3   |
| 🍐 Pears  | \$4.15 | 1   |
| 🍋 Lemons | \$0.99 | 2   |

## Becoming proficient in Emmet

Emmet speeds up repetitive tasks with tab key. Most useful for creating HTML and CSS files. Think of it the way CSS selectors work for DOM generation. Emmet is built into VS Code so we don't need to install a plugin to use it in VS Code.

You don't need to care about emmet readability because it's disposable. You hit tab and it's gone.

### Using Emmet in HTML

By default it generates a div element but it has some idea of the surrounding elements so if you type `span>span` it will create `<span><span></span></span>` or `ul>li` will create `<ul><li></li></ul>` but it only looks at the string and once the tab is hit and the HTML is generated, it doesn't look at the outside context anymore.

Similar to CSS, there is a direct descendant selector `>` and sibling selector `+` so
`ul>li.special-item` will create:

```html
<ul>
  <li class="special-item"></li>
</ul>
```

whereas `.navbar+.content+.footer` will create

```html
<div class="navar"></div>
<div class="content"></div>
<div class="footer"></div>
```

As siblings of each other. There is also a climb up operator `^` but it's easier to just use parentheses:

`div+div>(p>(span+em))+bq`

```html
<div></div>
<div>
  <p><span></span><em></em></p>
  <blockquote></blockquote>
</div>
```

But if you have to put too much mental effort into emmet, then it's probably not worth it. It's easier to just use simple emmet commands and generate the html.

Multiplication is also useful, so `ul.col>li.col-item*5`

```html
<ul class="col">
  <li class="co-item"></li>
  <li class="co-item"></li>
  <li class="co-item"></li>
  <li class="co-item"></li>
  <li class="co-item"></li>
</ul>
```

And you can add text to the body of the item by using `{}` such that `ul.col>li.col-item*5{some text}*5` becomes:

```html
<ul class="col">
  <li class="co-item">some text</li>
  <li class="co-item">some text</li>
  <li class="co-item">some text</li>
  <li class="co-item">some text</li>
  <li class="co-item">some text</li>
</ul>
```

You can use the `$` inside the `{}` so that the text has consecutively counting numbers in it, such that `ul.col>li.col-item*5{some text $}*5` becomes:

```html
<ul class="col">
  <li class="col-item">some text 1</li>
  <li class="col-item">some text 2</li>
  <li class="col-item">some text 3</li>
  <li class="col-item">some text 4</li>
  <li class="col-item">some text 5</li>
</ul>
```

### Using Emmet in CSS

Inside an element, we can use emmet as well, so `p10` becomes:

```css
padding: 10px;
```

and `c#0` becomes:

```css
color: #000000;
```

and `gtc1fr` becomes:

```css
grid-template-columns: 1fr;
```

For multi-name css properties in general, emmet uses the initial letter of each word.

The value aliases for units are: `p` for percent, `e` for `em`, `r` for `rem` and `px` for pixels. So `lh1.5r` becomes:

```css
line-height: 1.5rem;
```

Any new line or space finishes the emmet, so basically it stops it from working so you need to generate your html or css before hitting space or return. There is no reason to construct complex convoluted emmet commands, just go two steps deep and construct small snippets and build from there.

## Refactoring with VS Code

### Navigating through code with VS Code

The `Cmd + P` keyboard shortcut will bring up a list of recently opened files that we can quickly go to.

When the mouse is hovering on a JS function, if we hold the `Cmd` key it turns the function into a link, and if we click on it it will take us to the definition of that function.

If you are working on a typescript project, even if the file you are currently on is pure JS, as long as the types are defined somewhere, you can right click on a method and go to type definition and vsc will take you to the definition of the method.

Opening the command palette and then entering the `#` character, you can search through 'symbols'. Symbols are high level names that related to your data structures such as classes, functions, constants, or even headings in markdown files etc.

If we begin with `#` vsc searches through all our current workspace. If we want to search through the symbols only the current file, we can put the `@` character at the beginning of the command palette.

Putting the `:` character at the beginning of the command palette will allow you go jump to a line number.

### Peek Editing

`Shift + F12` Will allow you to peek at the definition, and also see everywhere else that that definition has been used. This is most useful for when you have your types defined, either as JS or TS.

### Renaming

Three methods to rename things:

1. Find/Replace using `Cmd + F` the old fashioned way.
2. Using `Cmd + D` or `Cmd + Shift + L` and using multi cursor to rename
3. Pressing `F2` allows us to rename something in all files in our workspace. Notice that after this renaming, all files where the modifications have been made need to be saved (or we can use Save All).

You can also select a few lines of code. On the left you'll see a yellow lightbulb, if you click on that you'll see the option to "Extract to method..." which allows you to refactor your code into a new function. You can give it a name right there and it automatically creates the required arguments and parameters in the method.

## Type Checking

### Why use types

- Allows us to catch bugs at compile time instead of runtime.
- Provides better developer experience especially combined with VS Code.
- Our code makes our intention much clearer
- We can get away with writing less test, as our tests need to cover less areas.
- In modern JS runtimes (such as V8) code is by default interpreted. However if if the shape of our code is found to be reliably functioning in the same way, meaning it has consistent properties and the types of values are always the same, V8 puts this code into a different mode and compiles it to assembly. This is much faster than interpreted. If we change the type once, the code is put back into interpreted mode. So writing code with types results in more optimised code.

### How to add types to JavaScript

A lot of VS Code's power comes from type checking and type inferencing. VS Code itself is mostly written in TypeScript and is heavily tied to it, in order to get the most out of VS Code, we either have to use TS or we can add types to our JavaScript.

TypeScript is a typed superset of JavaScript. It relies on type inference: the automatic deduction of data types of an expression. TypeScript is an opt-in, you can rename your .js file to .ts and it will (mostly) be valid TypeScript. Obviously we'll get lots of errors about missing types, but it should work.

But we can also apply type checking to JavaScript using JSDoc comments (which is also what TS uses in the background).

### Activating Type Checking in VS Code

- If our project is already using typescript, all we need to do is add `"checkJS": ture` to our `tsconfig.json` file.
- If we are working on a purely JS project, we can add the comment `// @ts-check` to the top of our JS file (first line)

### Explicitly adding type information with JSDoc

```ts
/** @type {number} */
let value = 21;
value = "hello"; //VS Code will warn us of an error;
```

### Defining our own types

We can use `@typedef` to create a new named type of our own. For example:

```js
/**
 * @typedef {Object} InventoryItem
 * @prop {string} name
 * @prop {number} price
 */

/**
 * @typedef {{qty: number, item: InventoryItem}} InvoiceItem //first {} says that I'm defining my type here and the second {} is saying that this is an object
 */

/** @type {InvoiceItem[]} */
let invoice = [];
invoice.push({
  qty: 2,
  item: {
    name: "Apple",
    price: 1.32,
  },
}); // This is fine

invoice.push({ bad: "thing" }); // This is not ok and will show error
```

**In mathematical terms: _The signature of our function is the hypothesis and the implementation is the proof._**

## Debugging in VS Code

// We have left the course at this point. Come back and finish this later.

## Miscellaneous

### Useful keybindings

All these keybindings showcase the default key bindings but they can of course be changed to mimic any other editor or changed to match your own preferences. The shortcuts presented here are on Mac OS. They will be subtly different on other OSes.

#### Selection

- `Cmd + D` selects the word where the cursor is located. If pressed again, it will also select the subsequent occurrences of the word (with multiple cursors).
- Select all occurrences: `Cmd + Shift + L`
- To select a random section of text, hold down `Option + Shift` and then drag the mouse over the selection area. This is basically a _box selection_ and is useful for example for copy/pasting HTML from somewhere.
- `Option + Click` gives you multi cursor. If you click on the wrong place once, you can use `Cmd + U` to undo the last selection.

#### Line Manipulation

- `Option + Up` or `Option + Down` moves the line up or down.
- `Shift + Option + Up` or `Shift + Option + Down` copies the current line up or down.

* `` Ctrl + ` `` will open the terminal

### VS Code and git

VS Code has a full git UI built in. To bring it up, press `Ctrl + Shit + g`. You can write your commit message and press `Command + Enter` to commit it, and if the changes are not staged yet it will ask you and can stage them first before committing.

VSC also provides a diff so you can see everything that has been changed since the last commit. The diff can be viewed either side by side (which is the default). clicking on the number next to the filename brings up the diff but this can also be toggled with inline view to show the diff in a more traditional way using colour highlighting.

Many common git commands from push and pull to creating new branches can be performed inside VS Code, but probably the most useful is to commit the file you are currently working on and view diffs. You can also view the file as it currently exists in HEAD.

The number of files with unsaved changes, and the number of files with pending commits are also highlighted helpfully in the side bar. If your side bar is not in view you can toggle it using `Cmd + B`.
