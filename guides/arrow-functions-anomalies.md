# Anomalies in JavaScript Arrow Functions

Personally, I think arrow functions are one of the most awesome syntax additions to the JavaScript language introduced in the ES6 specification — _my opinion by the way_. I get to use them almost everyday since I knew about them and I guess that goes for most JavaScript developers.

Arrow functions can be used in so many ways as regular JavaScript functions. They, however, are commonly used wherever an anonymous function expression is required, for example as callback functions.

The following example shows how an arrow function can be used as a callback function especially with array methods like: `.map()`, `.filter()`, `.reduce()`, `.sort()`, etc.

```js
const scores = [ /* ...some scores here... */ ];
const maxScore = Math.max(...scores);

// Arrow Function as .map() callback
scores.map(score => +(score / maxScore).toFixed(2))
```

At first glance, it may seem like arrow functions can be used or defined in every way a regular JavaScript function can, but that is not true. Arrow functions, for very good reasons, are not meant to behave exactly the same way as regular JavaScript functions. Perhaps arrow functions can be considered **_JavaScript functions with anomalies_**.

Although arrow functions have a pretty simple syntax, that will not be the focus of this article. This article aims at exposing the major ways in which arrow functions behave differently from regular functions and how that knowledge can be used to the developer's advantage.

**Please Note:** Throughout this article, I use the term _regular function_ or _regular JavaScript function_ to refer to a traditional JavaScript function statement or expression defined using the `function` keyword.

## TL;DR

- Arrow functions can never have duplicate named parameters, whether in strict or non-strict mode.

- Arrow functions do not have an `arguments` binding. However, they have access to the `arguments` object of the closest non-arrow parent function. Named and rest parameters are heavily relied upon to capture the arguments passed to arrow functions.

- Arrow functions can never be used as constructor functions. Hence, they can never be invoked with the `new` keyword. As such, a `prototype` property does not exist for an arrow function.

- The value of `this` inside an arrow function remains the same throughout the lifecycle of the function, and is always bound to the value of `this` in the closest non-arrow parent function.

## Named Function Parameters

Functions in JavaScript are usually defined with named parameters. Named parameters are used to map arguments to local variables within the function scope based on positioning.

Let's consider the following JavaScript function.

```js
function logParams (first, second, third) {
  console.log(first, second, third);
}

// first => 'Hello'
// second => 'World'
// third => '!!!'
logParams('Hello', 'World', '!!!'); // "Hello"  "World"  "!!!"

// first => { o: 3 }
// second => [ 1, 2, 3 ]
// third => undefined
logParams({ o: 3 }, [ 1, 2, 3 ]); // {o: 3}  [1, 2, 3]
```

The `logParams()` function is defined with 3 named parameters namely: `first`, `second` and `third`. The named parameters are mapped to the arguments the function was called with based on the position. If there are more named parameters than the arguments passed to the function, the excess parameters are set to `undefined`.

Regular JavaScript functions exhibit a strange behavior in _non-strict_ mode with regards to named parameters. In _non-strict_ mode, regular JavaScript functions allow duplicate named parameters. The following code snippet shows the consequence of that behavior.

```js
function logParams (first, second, first) {
  console.log(first, second);
}

// first => 'Hello'
// second => 'World'
// first => '!!!'
logParams('Hello', 'World', '!!!'); // "!!!"  "World"

// first => { o: 3 }
// second => [ 1, 2, 3 ]
// first => undefined
logParams({ o: 3 }, [ 1, 2, 3 ]); // undefined  [1, 2, 3]
```

As you can see, the `first` parameter is a duplicate, hence it is mapped to the value of the third argument passed to the function call, completely overriding the first argument passed. This is not a desirable behavior.

The good news is that this behavior is not allowed in _strict_ mode. Defining a function with duplicate parameters in _strict_ mode will throw a `Syntax Error` indicating that duplicate parameters are not allowed.

```js
// Throws an error because of duplicate parameters (Strict mode)
function logParams (first, second, first) {
  "use strict";
  console.log(first, second);
}
```

**How do arrow functions treat duplicate parameters in their definition?** Now here is something about arrow functions:

> Unlike regular functions, arrow functions do not allow duplicate parameters whether in strict or non-strict mode. Duplicate parameters will cause a `Syntax Error` to be thrown.

```js
// Always throws a syntax error
const logParams = (first, second, first) => {
  console.log(first, second);
}
```

## Function Overloading

**Function overloading** is the ability to define a function such that it can be invoked with different _call signatures_ (shapes or number of arguments). Good a thing, the `arguments` binding for JavaScript functions makes this possible.

For a start, consider this very simple overloaded function that calculates the average of any number of arguments passed to it.

```js
function average() {
  // the number of arguments passed
  const length = arguments.length;

  if (length == 0) return 0;

  // convert the arguments to a proper array of numbers
  const numbers = Array.prototype.slice.call(arguments);

  // a reducer function to sum up array items
  const sumReduceFn = function (a, b) { return a + Number(b) };

  // return the sum of array items divided by yhe number of items
  return numbers.reduce(sumReduceFn, 0) / length;
}
```

I have tried to make the function definition as verbose as possible so that the behavior of the function can be clearly understood. The function can be called with any number of arguments from zero to the max number of arguments that a function can take — that should be 255.

Here are some results from calls to the `average()` function.

```js
average(); // 0
average('3o', 4, 5); // NaN
average('1', 2, '3', 4, '5', 6, 7, 8, 9, 10); // 5.5
average(1.75, 2.25, 3.5, 4.125, 5.875); // 3.5
```

Now try to replicate the `average()` function using the arrow function syntax. I mean, how difficult can that be? First guess — all you have to do is this:

```js
const average = () => {
  const length = arguments.length;

  if (length == 0) return 0;

  const numbers = Array.prototype.slice.call(arguments);
  const sumReduceFn = function (a, b) { return a + Number(b) };

  return numbers.reduce(sumReduceFn, 0) / length;
}
```

When you test this function now, you realize that it throws a `Reference Error`, and guess what — of all the possible causes, it is complaining that `arguments` is not defined.

**What are you getting wrong?**
Now here is something about arrow functions:

> Unlike regular functions, the arguments binding does not exist for arrow functions. However, they can have access to the arguments of a non-arrow parent function.

Based on this understanding, you can modify the `average()` function to be a regular function that will return the result of an immediately-invoked nested arrow function which should have access to the `arguments` of the parent function. This will look like this:

```js
function average() {
  return (() => {
    const length = arguments.length;

    if (length == 0) return 0;

    const numbers = Array.prototype.slice.call(arguments);
    const sumReduceFn = function (a, b) { return a + Number(b) };

    return numbers.reduce(sumReduceFn, 0) / length;
  })();
}
```

Obviously, that solved the problem you had with the `arguments` object not being defined. However, you had to use a nested arrow function inside a regular function, which seems rather too unnecessary for such a simple function as this.

**Can you do this differently?**
Since accessing the `arguments` object is obviously the problem here, is there an alternative object you can use? The answer is **YES**. Say hello to ES6 **_rest parameters_**.

With ES6 rest parameters, you can get an array that gives you access to all or part of the arguments that were passed to a function. This works for all function flavors, whether regular functions or arrow functions. Here is what it looks like.

```js
const average = (...args) => {
  if (args.length == 0) return 0;

  const sumReduceFn = function (a, b) { return a + Number(b) };

  return args.reduce(sumReduceFn, 0) / args.length;
}
```

Wow!!! Rest parameters to the rescue — you finally arrived at an elegant solution for implementing the `average()` function as an arrow function.

However, there are some caveats against relying on rest parameters for accessing function arguments.

- A rest parameter is not the same as the internal `arguments` object of the function. The rest parameter is an actual function parameter while the `arguments` object is an internal object bound to the scope of the function.

- A function can only have one rest parameter and it must always be the last parameter. This means a function can have a combination of named parameters and a rest parameter.

- The rest parameter when present, may not capture all the function's arguments especially when it is used together with named parameters. However, when it is the only function parameter, it captures the whole function arguments. On the other hand, the `arguments` object of the function always captures all the function's arguments.

- The rest parameter points to an array object containing all the captured function arguments, whereas the `arguments` object points to an _array-like_ object containing all the function's arguments.

Before you proceed, consider another very simple overloaded function that converts a number from one number base to another. The function can be called with 1 to 3 arguments. However, when it is called with two arguments or less, it swaps the second and third function parameters in its implementation.

Here is what it looks like with a regular function.

```js
function baseConvert (num, fromRadix = 10, toRadix = 10) {
  if (arguments.length < 3) {
    // swap variables using array destructuring
    [toRadix, fromRadix] = [fromRadix, toRadix];
  }

  return parseInt(num, fromRadix).toString(toRadix);
}
```

Here are some calls to the `baseConvert()` function.

```js
// num => 123, fromRadix => 10, toRadix => 10
console.log(baseConvert(123)); // "123"

// num => 255, fromRadix => 10, toRadix => 2
console.log(baseConvert(255, 2)); // "11111111"

// num => 'ff', fromRadix => 16, toRadix => 8
console.log(baseConvert('ff', 16, 8)); // "377"
```

Based on what you know about arrow functions not having `arguments` binding of their own, you can rewrite the `baseConvert()` function using the arrow function syntax as follows:

```js
const baseConvert = (num, ...args) => {
  // destructure the `args` array and
  // set the `fromRadix` and `toRadix` local variables
  let [fromRadix = 10, toRadix = 10] = args;

  if (args.length < 2) {
    // swap variables using array destructuring
    [toRadix, fromRadix] = [fromRadix, toRadix];
  }

  return parseInt(num, fromRadix).toString(toRadix);
}
```

Notice in the previous code snippets that I have used the ES6 array destructuring syntax to set local variables from array items and also to swap variables. You can learn more about destructuring by reading this guide — [**ES6 Destructuring: The Complete Guide**](https://codeburst.io/es6-destructuring-the-complete-guide-7f842d08b98f).

## Constructor Functions

A regular JavaScript function can be called with the `new` keyword, for which the function behaves as a class constructor for creating new instance objects.

Here is a simple example of a function being used as a constructor.

```js
function Square (length = 10) {
  this.length = parseInt(length) || 10;

  this.getArea = function() {
    return Math.pow(this.length, 2);
  }

  this.getPerimeter = function() {
    return 4 * this.length;
  }
}

const square = new Square();

console.log(square.length); // 10
console.log(square.getArea()); // 100
console.log(square.getPerimeter()); // 40

console.log(typeof square); // "object"
console.log(square instanceof Square); // true
```

When a regular JavaScript function is invoked with the `new` keyword, the function's internal `[[Construct]]` function is called to create a new instance object and allocate memory. After that, the function body is executed normally, mapping `this` to the newly created instance object. Finally, the function implicitly returns `this` (the newly created instance object), except a different return value has been specified in the function definition.

Also, all regular JavaScript functions have a `prototype` property. The `prototype` property of a function is an object that contains properties and methods that are shared amongst all instance objects created by the function when used as a constructor.

Initially, the `prototype` property is an empty object with a `constructor` property that points to the function. However, it can be augumented with properties and methods to add more functionality to objects created using the function as a constructor.

Here is a slight modification of the previous `Square` function that defines the methods on the function's `prototype` instead of the constructor itself.

```js
function Square (length = 10) {
  this.length = parseInt(length) || 10;
}

Square.prototype.getArea = function() {
  return Math.pow(this.length, 2);
}

Square.prototype.getPerimeter = function() {
  return 4 * this.length;
}

const square = new Square();

console.log(square.length); // 10
console.log(square.getArea()); // 100
console.log(square.getPerimeter()); // 40

console.log(typeof square); // "object"
console.log(square instanceof Square); // true
```

As you can tell, everything still works as expected. In fact here is a little secret, ES6 classes do something similar to the above code snippet on the background — they are simply _syntactic sugar_.

So what about arrow functions? Do they also share this behavior with regular JavaScript functions? The answer is **NO**. Now here is something about arrow functions:

> Unlike regular functions, arrow functions can never be called with the `new` keyword because they do not have the `[[Construct]]` method. As such, the `prototype` property also does not exist for arrow functions.

Sadly, that is very true. Arrow functions cannot be used as constructors. They cannot be called with the `new` keyword. Doing that throws an error indicating thet the function is not a constructor. As a result, bindings such as `new.target` that exist inside functions that can be called as constructors do not exist for arrow functions, instead they use the `new.target` value of the closest non-arrow parent function.

Also, because arrow functions cannot be called with the `new` keyword, there is really no need for them to have a prototype. Hence, the `prototype` property does not exist for arrow functions. Since the `prototype` of an arrow function is `undefined`, attempting to augument it with properties and methods or access a property on it will throw an error.

```js
const Square = (length = 10) => {
  this.length = parseInt(length) || 10;
}

// throws an error
const square = new Square(5);

// throws an error
Square.prototype.getArea = function() {
  return Math.pow(this.length, 2);
}

console.log(Square.prototype); // undefined
```

## What is "this"?

If you have been writing JavaScript programs for some time now, you would have noticed that every invocation of a JavaScript function is associated with an invocation context depending on how or where the function was invoked.

The value of `this` inside a function is heavily dependent on the invocation context of the function at call time, which usually puts JavaScript developers in a situation where they have to ask themselves the famous question: **What is the value of `this`?**

Here is a summary of what the value of `this` points to for different kinds of function invocations:

- **Invoked with the `new` keyword** — `this` points to the new instance object created by the internal `[[Construct]]` method of the function. `this` (the newly created instance object) is usually returned by default, except a different return value was explicitly specified in the function definition.

- **Invoked directly without the `new` keyword** — In _non-strict_ mode, `this` points to the global object of the JavaScript host environment (in a web browser, this is usually the `window` object). However, in _strict_ mode, the value of `this` is `undefined`, hence, trying to access or set a property on `this` will throw an error.

- **Invoked indirectly with a bound object** — The `Function.prototype` object provides three methods that make it possible for functions to be bound to an arbitrary object when they are called, namely: `.call()`, `.apply()` and `.bind()`. When the function is called using any of these methods, `this` points to the specified bound object.

- **Invoked as an object method** — `this` points to the object on which the function (method) was invoked regardless of whether the method is defined as an own property of the object or resolved from the object's prototype chain.

- **Invoked as an event handler** — For regular JavaScript functions that are used as DOM event listeners, `this` points to the target object, DOM element, `document` or `window` on which the event was fired.

For a start, consider this very simple JavaScript function that will be used as a click event listener for say a form submit button.

```js
function processFormData (evt) {
  evt.preventDefault();

  // get the parent form of the submit button
  const form = this.closest('form');

  // extract the form data, action and method
  const data = new FormData(form);
  const { action: url, method } = form;

  // send the form data to the server via some AJAX request
  // you can use Fetch API or jQuery Ajax or native XHR
}

button.addEventListener('click', processFormData, false);
```

If you try this code, you will see that everything works correctly. The value `this` inside the event listener function, like you saw earlier, is the DOM element on which the click event was fired, which in this case is `button`. Hence, it is possible to point to the parent form of the submit button using:

```js
this.closest('form');
```

At the moment, you are using a regular JavaScript function as the event listener. What happens if you change the function definition to use the all-new beautiful arrow function syntax?

```js
const processFormData = (evt) => {
  evt.preventDefault();

  const form = this.closest('form');
  const data = new FormData(form);
  const { action: url, method } = form;

  // send the form data to the server via some AJAX request
  // you can use Fetch API or jQuery Ajax or native XHR
}

button.addEventListener('click', processFormData, false);
```

If you try this now, you will notice that you are getting an error. From the look of things, it seems the value of `this` isn't what you were expecting. For some reason, `this` no longer points to the `button` element, instead it points to the global `window` object.

What can you do to fix the `this` binding? Do you remember `Function.prototype.bind`? You can use that to force the value of `this` to be bound to the `button` element when you are setting up the event listener for the submit button. Here it is:

```js
// Bind the event listener function (`processFormData`) to the `button` element
button.addEventListener('click', processFormData.bind(button), false);
```

Ooops!!! It seems that was not the fix you were looking for. `this` still points to the global `window` object. Is this a problem peculiar to arrow functions? Does that mean arrow functions cannot be used for event handlers that rely on `this`?

**What are you getting wrong?**
Now here is something about arrow functions:

> Unlike regular functions, arrow functions do not have a `this` binding of their own. The value of `this` is resolved to that of the closest non-arrow parent function or the global object otherwise.

This explains why the value of `this` in the event listener arrow function points to the `window` object (global object). Since it was not nested within a parent function, it derives is `this` value from the closest scope which is the global scope.

This, however, does not explain why you cannot bind the event listener arrow function to the `button` element using the `bind()`. Here comes an explanation for that.

> Unlike regular functions, the value of `this` inside arrow functions remains the same and cannot change throughout their lifecycle, irrespective of the invocation context.

This behavior of arrow functions makes it possible for JavaScript engines to optimize them, since the function bindings can be determined beforehand.

Consider a slightly different scenario where the event handler is defined using a regular function inside an object's method and also depends on another method of the same object.

```js
({
  _sortByFileSize: function (filelist) {
    const files = Array.from(filelist).sort(function (a, b) {
      return a.size - b.size;
    });

    return files.map(function (file) {
      return file.name;
    });
  },

  init: function (input) {
    input.addEventListener('change', function (evt) {
      const files = evt.target.files;
      console.log(this._sortByFileSize(files));
    }, false);
  }

}).init(document.getElementById('file-input'));
```

Here is a one-off object literal with a `sortByFileSize()` method and an `init()` method which is invoked immediately. The `init()` method takes a file `input` element and sets up a change event handler for the input element that sorts the uploaded files by file size and logs them on the browser's console.

If you test this code you will realize that when you select files to upload, the file list doesn't get sorted and logged to the console, instead an error is thrown on the console. The problem comes from this line:

```js
console.log(this._sortByFileSize(files));
```

Inside the event listener function, `this` points to the DOM element on which the event was fired, which in this case is the `input` element, hence `this._sortByFileSize` is `undefined`.

However, outside the event listener function in the `init()` method, `this` points to the correct object that owns the method, hence `this._sortByFileSize` is defined.

To solve this problem, you need to bind `this` inside the event listener to the outer object containing the methods so that you can be able to call `this._sortByFileSize()`. Here, you can use `bind()` as follows:

```js
init: function (input) {
  input.addEventListener('change', (function (evt) {
    const files = evt.target.files;
    console.log(this._sortByFileSize(files));
  }).bind(this), false);
}
```

Now everything works as expected. Instead of using `bind()` here, you could simply replace the event listener regular function with an arrow function. The arrow function will use the `this` value from the parent `init()` method which will be the required object.

```js
init: function (input) {
  input.addEventListener('change', evt => {
    const files = evt.target.files;
    console.log(this._sortByFileSize(files));
  }, false);
}
```

Before you proceed, consider one more scenario. Let's say you have a simple timer function that can be invoked as a constructor to create countdown timers in seconds. It will use `setInterval()` to keep conting down until the duration elapses or until the interval is cleared. Here it is:

```js
function Timer (seconds = 60) {
  this.seconds = parseInt(seconds) || 60;
  console.log(this.seconds);

  this.interval = setInterval(function () {
    console.log(--this.seconds);

    if (this.seconds == 0) {
      this.interval && clearInterval(this.interval);
    }
  }, 1000);
}

const timer = new Timer(30);
```

If you run this code, you will see that the countdown timer seems to be broken. It keeps logging `NaN` on the console infinitely. The problem here is that inside the callback function passed to `setInterval()`, `this` points to the global `window` object instead of the newly created instance object within the scope of the `Timer()` function. Hence, both `this.seconds` and `this.interval` are undefined.

As before, to fix this, you can use `bind()` to bind the value of `this` inside the `setInterval()` callback function to the newly created innstance object as follows:

```js
function Timer (seconds = 60) {
  this.seconds = parseInt(seconds) || 60;
  console.log(this.seconds);

  this.interval = setInterval((function () {
    console.log(--this.seconds);

    if (this.seconds == 0) {
      this.interval && clearInterval(this.interval);
    }
  }).bind(this), 1000);
}
```

Or, better still, you can replace the `setInterval()` callback regular function with an arrow function so that it can use the value of `this` from the closest non-arrow parent function, which is `Timer()` in this case.

```js
function Timer (seconds = 60) {
  this.seconds = parseInt(seconds) || 60;
  console.log(this.seconds);

  this.interval = setInterval(() => {
    console.log(--this.seconds);

    if (this.seconds == 0) {
      this.interval && clearInterval(this.interval);
    }
  }, 1000);
}
```

Now that you completely understand how arrow functions handle the `this` keyword, it is important to note that an arrow function will not be ideal for cases where you need the value of `this` to be preserved — for example, when defining object methods that need a reference to the object or augumenting a function's prototype with methods that need a reference to the target object.

## Non-Existent Bindings

Throughout this article, you have seen several bindings that are available inside regular JavaScript functions but don't exist for arrow functions. Instead arrow functions derive the values of such bindings from their closest non-arrow parent function.

In summary, here is a list of the non-existent bindings in arrow functions:

- `arguments` — List of arguments passed to the function when it is called.
- `new.target` — A reference to the function being called as a constructor with the `new` keyword.
- `super` — A reference to the prototype of the object to which the function belongs, provided it is defined as a concise object method.
- `this` — A reference to the invocation context object of the function.
