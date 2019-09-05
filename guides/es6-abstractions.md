# JavaScript ES6: 5 Abstractions for a Better Code

## 1. Template Literals

In ES6, template literals were introduced for dealing with a few challenges associated with formatting and representing strings. With template literals it has become possible to deal with multiline strings with ease. It also makes it possible to perform enhanced string substitutions and proper formatting of seemingly dangerous strings such as strings to be embedded into HTML.

Prior to ES6, strings are delimited by either a pair of _single quotes_(`'string'`) or a pair of _double quotes_(`"string"`). In ES6, strings can also be delimited by a pair of _backticks_(`` `string` ``). Such strings are called **template literals**.

Just as with single and double quotes delimeters, backticks can also be escaped in template literals if the string contains a backtick character. To escape a backtick character in a template literal, a backward slash(`\`) must be placed before the backtick character. Note however that single and double quotes don't need to be escaped in template literals.

Here is a simple example:

```js
const greeting = `Good morning!`;
const shortcut = `\`cmd\` + \`shift\` + \`G\``;

console.log(greeting); // "Good morning!"
console.log(shortcut); // "`cmd` + `shift` + `G`"
```

Using template literals this way isn't any much different from using regular JavaScript strings delimited by quotes. The begin to get real advantages when dealing with _multiline strings_, _string substitutions_ and _tagged templates_.

### Multiline Strings

Prior to ES6, strings in JavaScript were limited to a single line. However, ending a line with a backward slash(`\`) before beginning a newline made it possible to create seeming multiline strings even though the newlines are not output in the string. Here is an example:

```js
const message = "Hello Glad, \
Your meeting is scheduled for noon today.";

console.log(message);
// Hello Glad, Your meeting is scheduled for noon today.
```

If you want to output a newline in the string, you will need to use the newline escape sequence(`\n`) before the newline. Here is an example:

```js
const message = "Hello Glad,\n\
Your meeting is scheduled for noon today.";

console.log(message);
// Hello Glad,
// Your meeting is scheduled for noon today.
```

With ES6 template literals, the string is output with the formatting intact. All newlines and whitespaces in the string are preserved, making multiline strings easy to create without any additional syntax. However since whitespaces are preserved, care should be taken when indenting the string.

Consider this example:

```js
const html = (`
<html>
  <body>
    <div>Template literals are super cool.</div>
  </body>
</html>
`).trim();

console.log(html);
// <html>
//   <body>
//     <div>Template literals are super cool.</div>
//   </body>
// </html>
```

Notice that the newlines and indentations are preserved in the string. The `trim()` method is also used to remove any newlines and whitespaces at the start and end of the html string.

### String Substitution

Template literals also make string substitutions fun. Prior to ES6, _string concatenation_ was heavily relied on for creating dynamic strings. Here is a simple example:

```js
const price = 24.99;

console.log("The item costs $" + price + " on the online store.");
// The item costs $24.99 on the online store.
```

Using ES6 template literals, the substitution can be done as follows:

```js
const price = 24.99;

console.log(`The item costs $${price} on the online store.`);
// The item costs $24.99 on the online store.
```

A string substitution is delimited by an opening `${` and a closing `}` and can contain any valid JavaScript expression in between.

In the previous example, we substituted the value of a simple variable in the template literal. Let's say we want to add a 10% discount to the price of all items in the store. Here is what it will look like:

```js
const price = 24.99;
const discount = 10;

console.log(`The item costs $${(price * (100 - discount) / 100).toFixed(2)} on the online store.`);
// The item costs $22.49 on the online store.
```

Template literals are JavaScript expressions themselves and as such can be nested inside of other template literals.

### Template Tags

With tagged templates, you even have more control over the substitutions and transformation of the template literal. A _template tag_ is simply a function that defines how a template literal should be transformed.

A template tag function can accept multiple arguments. The first argument is an array containing all the literal strings in the template literal. The remaining arguments correspond with the substitutions in the template literal. Hence the second argument corresponds with the first substitution, the third argument corresponds with the second substitution and so on.

Here is a simple illustration. Given the following template literal:

```js
`The price of ${quantity} units of the item on the online store is $${quantity * price}.`
```

The first argument passed to a template tag for the above template literal will be the array of literal strings which is as follows:

```js
[
  'The price of ',
  ' units of the item on the online store is $',
  '.'
]
```

The second argument will be the value of `quantity` and the third argument will be the value of `(quantity * price)`.

Let's go ahead and create a template tag named `pricing` which we can use to transform pricing summary. It will ensure that price values are rounded to 2 decimal places. It will also ensure that the `$` currency symbol before any price is converted to `USD`. Here is the function:

```js
function pricing(literals, ...replacements) {
  // Initialize the final string
  let finalString = '';

  for (let i = 0; i < replacements.length; i++) {
    // Get the current literal and replacement
    const literal = literals[i];
    const replacement = replacements[i];

    // Trim trailing whitespaces from the current literal
    const trimmed = literal.trimRight();
    const length = trimmed.length;

    // Check if current replacement is a number
    const isNumber = typeof replacement === 'number';

    // Check if current literal string ends with $
    const isPrice = /\$$/.test(trimmed);

    // Check if number is followed by literal that ends with $
    // and use the desired formatting
    finalString += (isNumber && isPrice)
      ? `${trimmed.substr(0, length - 1).trimRight()} USD ${replacement.toFixed(2)}`
      : `${literal}${replacement}`;
  }

  // Attach the last literal to the final string
  return finalString + literals[literals.length - 1];
}
```

You would notice in this code snippet that we used a rest parameter named `replacements` to capture all the substitutions in the template literal.

Now that we have created a template tag, using it is the easy part. To use a template tag, simply attach the name of the template tag just before the first back-tick(`` ` ``) delimiter of the template literal.

Here is an example using the `pricing` template tag we just created:

```js
const price = 24.99;
const discount = 10;
const quantity = 4;

const totalPrice = quantity * price * (100 - discount) / 100;

// WITHOUT TEMPLATE TAG
console.log(`The price of ${quantity} units of the item on the online store is $${totalPrice}.`);

// The price of 4 units of the item on the online store is $89.964.


// WITH TEMPLATE TAG (pricing)
console.log(pricing`The price of ${quantity} units of the item on the online store is $${totalPrice}.`);

// The price of 4 units of the item on the online store is USD 89.96.
```

## 2. Default and Rest Parameters

Functions in JavaScript are very important objects. It is very possible that you have come across the statement: "Functions are first-class citizens". This is true about JavaScript functions because you can pass them around in your program like you would with any other regular value.

JavaScript functions have not had any considerable syntax improvements until ES6. With ES6, we have some syntax improvements like _default parameters_, _rest parameters_, _arrow functions_, etc.

### Default Parameters

Prior to ES6, there was basically no syntax for setting default values for function parameters. However, there were some hacks for setting fallback values for function parameters when they are not passed values on invocation time. Here is a simple example:

```js
// METHOD 1: Short-circuiting
// Using the logical OR (||) operator
function convertToBase(number, base) {
  number = parseInt(number) || 0;
  base = parseInt(base) || 10;

  return number.toString(base);
}

// METHOD 2: Ternary (?:) operator
// With additional type check (safer option)
function convertToBase(number, base) {
  number = (typeof number !== "undefined") ? parseInt(number) : 0;
  base = (typeof base !== "undefined") ? parseInt(base) : 10;

  return number.toString(base);
}
```

In this snippet, we've been able to set default values for the function parameters. Hence these parameters behave as though they are optional, since fallback values are used when the parameters are not passed.

In ES6, you can initialize the function parameter with a default value that will be used when the parameter is not passed. Here is how we can rewrite our previous `convertToBase()` function:

```js
function convertToBase(number = 0, base = 10) {
  return parseInt(number).toString(parseInt(base));
}
```

> Named function parameters in ES6(including default parameters) have the same behavior as `let` declarations.

Default values in ES6 are not limited to only literal values. Any JavaScript expression can also be used as default values for function parameters. Here is an example:

```js
function getDefaultNumberBase() {
  return 10;
}

function convertToBase(number = 0, base = getDefaultNumberBase()) {
  return parseInt(number).toString(parseInt(base));
}
```

Here, we are using the return value from `getDefaultNumberBase()` as the default value for the `base` parameter. You can even use the value of a previous parameter when setting the default value for another parameter. Here is an example:

```js
function cropImage(width, height = width) {
  // ...implementation
}
```

In this snippet, the `height` parameter will be set to the value of the `width` parameter whenever it is not passed.

Although you can use previous parameter values when setting default values, you cannot use variables declared within the function body. This is because default parameters have their own scope that is separated from the scope function body.

### Rest Parameters

Prior to ES6, relying on the `arguments` object was the ultimate means of capturing all the arguments passed to a function on invocation. This made it possible to create overloaded functions that could accept varying number of arguments. However, the `arguments` object, though being _array-like_, needs to be converted to an actual array before certain array operations can be carried out on it. Here is a simple example:

```js
function sum() {
  // Convert arguments to array
  var args = Array.prototype.slice.call(arguments);

  // Compute sum using array reduce()
  return args.reduce(function(a, b) { return a + Number(b) }, 0);
}

sum(); // 0
sum(1); // 1
sum(1, 2, 3, 4, 5); // 15
```

This function computes the sum of any number of arguments passed to it. If the argument is not a `number`, it tries to convert it to a number using the `Number()` global function. It returns `0` if no argument is passed. Notice that the `arguments` object was first converted to an array and assigned to the `args` variable in order to use the `reduce()` method.

In ES6, _rest parameters_ have been introduced. A rest parameter is simply a named function parameter preceeded by three dots(`...`). The rest parameter is assigned an array that contains the remaining arguments passed to a function. Here is how we can rewrite our previous `sum()` function using a rest parameter:

```js
function sum(...args) {
  // Compute sum using array reduce()
  return args.reduce(function(a, b) { return a + Number(b) }, 0);
}
```

There are a few things that are worth noting as regards rest parameters.

1. You can only have one rest parameter for a function.
2. The rest parameter, when present, must be the last parameter.
3. A rest parameter is not the same as the `arguments` object. It only captures the remaining arguments after the other named parameters while the `arguments` object captures all the arguments passed to the function regardless.
4. A rest parameter cannot be used in an object literal setter.

### Spread Operator

Let's say we have an array containing the scores of students in a class and we want to compute the average score of the students. Basically, we will first compute the sum of the scores and then divide the sum by the number of scores.

We can use the `sum()` function we created in the previous section to compute the sum of the scores. However, the issue is that we have an array of scores and sum expects numbers as arguments.

Prior to ES6, the `Function.prototype.apply()` method can be used to handle cases like this. This method takes an array as its second argument which represents the arguments the function should be invoked with. Here is an example:

```js
const scores = [42, 68, 49, 83, 72, 65, 77, 74, 86, 51, 69, 47, 53, 58, 51];
const totalScore = sum.apply(null, scores);
const averageScore = totalScore / scores.length;

console.log(totalScore); // 945
console.log(averageScore); // 63
```

In ES6, a new operator known as the _spread operator_(`...`) has been introduced. It is closely related to rest parameters and is very useful for dealing with arrays and other iterables. With the spread operator we can compute the `totalScore` as follows:

```js
const totalScore = sum(...scores);
```

Hence for most of the use cases, the spread operator is a good replacement for the `Function.prototype.apply()` method.

## 3. Arrow Functions

Another very important syntax improvement in ES6 is the introduction of _arrow functions_. Arrow functions make use of a completely new syntax and offer a couple of great advantages when used in ways they are best suited for. The syntax for arrow functions omits the `function` keyword. Also the function parameters are separated from the function body using an _arrow_ (`=>`), hence the name _arrow functions_.

Although arrow functions are more compact and shorter than regular functions, they are significantly different from regular functions in some ways that define how they can be used:

1. Arrow functions cannot be used as constructors and they have no prototype. Hence, using the `new` keyword with an arrow function will usually result in an error.

2. Array function does not have `arguments` object, hence named parameters and rest parameters must be used for function arguments. Duplicate named parameters are also not allowed.

3. The `this` binding inside an arrow function cannot be modified, and it always points up to the closest non-arrow parent function.

Arrow functions may look slightly differently depending on what you want to achieve. Let's take a look at some forms:

**Without Parameters**

If there are no parameters for the arrow function, then an empty pair of parentheses(`()`) is used before the arrow(`=>`) as shown in the following snippet.

```js
// USING REGULAR FUNCTION
const getTimestamp = function() {
  return +new Date;
}

// USING ARROW FUNCTION
const getTimestamp = () => {
  return +new Date;
}
```

For very simple arrow functions like this that just return the value of a JavaScript expression, the `return` keyword and the pair of curly braces(`{}`) surrounding the function body can be omitted. Hence, the arrow function can be rewritten like this:

```js
const getTimestamp = () => +new Date;
```

However, if an object literal is returned from the arrow function, then it needs to be wrapped with a pair of parentheses(`()`), otherwise the JavaScript engine sees the curly braces(`{}`) of the object literal as a block which will result in syntax error. Here is an example:

```js
// Returned object literal wrapped in parenthesis
const getProfile = () => ({
  name: 'Glad Chinda',
  gender: 'Male',
  birthday: 'August 15'
});
```

**With Parameters**

For arrow functions that take just one named parameter, the enclosing pair of parentheses surrounding the parameters list can be omitted as shown in the following snippet:

```js
// Pair of parentheses is omitted
const computeSquare = num => num * num;
```

However, there are situations where the enclosing parenthesis surrounding the parameters list cannot be omitted. Here are some of such situations:

1. When there are more than one named parameters.

```js
// Pair of parentheses cannot be omitted
const addNumbers = (numA, numB) => numA + numB;
```

2. When there is a default parameter, even if it is the only parameter.

```js
// Pair of parentheses cannot be omitted
const factorial = (n = 1) => (n <= 1) ? 1 : n * factorial(n - 1);
```

3. When there is a rest parameter, even if it is the only parameter.

```js
// Pair of parentheses cannot be omitted
const range = (...numbers) => Math.max(...numbers) - Math.min(...numbers);
```

**Traditional Function Body**

As shown earlier for very simple arrow functions that just return the value of a JavaScript expression, the `return` keyword and the pair of curly braces(`{}`) surrounding the function body can be omitted. However, you can still use the traditional function body if you want, especially when the function has multiple statements.

```js
const snakeCase = value => {
  const regex = /[A-Z][^A-Z]+/g;
  const withoutSpaces = value.trim().replace(/\s+/g, '_');

  const caps = withoutSpaces.match(regex);
  const splits = withoutSpaces.split(regex);

  let finalString = splits.shift();

  for (let i = 0; i < splits.length; i++) {
    finalString += `${caps[i]}_${splits[i]}_`;
  }

  return finalString
    .toLowerCase()
    .replace(/_+/g, '_')
    .replace(/^_?(.+?)_?$/, '$1');
}
```

The above function tries to mimic the `snakeCase()` function of the _Lodash_ JavaScript library. Here, we compulsorily have to use the traditional function body wrapped in curly braces(`{}`) since we have so many JavaScript statements within the function body.

Unlike with regular functions, the `arguments` object does not exist for array functions. However, they can have access to the `arguments` object of a non-arrow parent function.

```js
function fetchLastScore() {
  return () => {
    console.log(arguments[arguments.length - 1]);
  }
}

fetchLastScore(42, 68, 49, 83, 72)(); // 72
```

### Immediately Invoked Function Expressions (IIFEs)

One useful application of functions in JavaScript is observed in _Immediately Invoked Function Expressions (IIFEs)_ which are functions that are defined and called immediately without saving a reference to the function. This is usually seen in one-off initialization functions, JavaScript libraries like _jQuery_. Using regular JavaScript functions, IIFEs usually take one of these forms:

```js
// FIRST SHAPE:
// Wrap the function expression in parentheses
// The invocation expression comes afterwards

(function(a, b) {
  // ...function body here
})(arg1, arg2);


// SECOND SHAPE:
// Wrap the function expression together with
// the invocation expression in parentheses

(function(a, b) {
  // ...function body here
}(arg1, arg2));
```

The arrow function syntax can also be used with IIFEs provided that the arrow function is wrapped in parentheses.

```js
// IIFE: With Arrow Function
// The arrow function is called immediately with a list of arguments
// and the return value is assigned to the `compute` variable

const compute = ((...numbers) => {

  // Private members
  const length = numbers.length;
  const min = Math.min(...numbers);
  const max = Math.max(...numbers);

  const sum = numbers.reduce((a, b) => a + Number(b), 0);

  // Expose an inteface of public methods
  return {
    sum: () => sum,
    avg: () => sum / length,
    range: () => max - min
  };

})(42, 68, 49, 83, 72, 65, 77, 74, 86, 51, 69, 47, 53, 58, 51);


// Access the exposed public methods

console.log(compute.sum()); // 945
console.log(compute.avg()); // 63
console.log(compute.range()); // 44
```

### Callback Functions

Callback functions are heavily used in asynchronous programs and also in higher-order array methods like `map()`, `filter()`, `forEach()`, `reduce()`, `sort()`, etc. Arrow functions are perfect for use as callback functions.

In a previous code snippet, we saw how an arrow function was used with `reduce()` to compute the sum of an array of numbers. Using the arrow function is more compact and neater. Again, here is the comparison:

```js
// WITHOUT ARROW FUNCTION
const sum = numbers.reduce(function(a, b) {
  return a + Number(b);
}, 0);

// WITH ARROW FUNCTION
const sum = numbers.reduce((a, b) => a + Number(b), 0);
```

Let's do something a little more involved to demonstrate how using arrow functions as array callbacks can help us achieve more with less code. We will mimic the `flattenDeep()` method of the _Lodash_ JavaScript library. This method recursively flattens an array. However, in our implementation, we will recursively flatten the array of arguments passed to the function.

Here is the code snippet for the `flattenDeep()` function:

```js
const flattenDeep = (...args) => args.reduce(
  (a, b) => [].concat(a, Array.isArray(b) ? flattenDeep(...b) : b)
);
```

This is how cool array functions can be when used as callback functions, especially when working with array iteration methods that take callback functions.

### this and Arrow Functions

One major source of confusion and errors in a lot of JavaScript programs is the value resolution of `this`. `this` resolves to different values depending on the scope and context of a function invocation.

For example, when a function is invoked with the `new` keyword, `this` points to the instance created by the constructor, however, when the same function is called without the `new` keyword, `this` points to the global object which in the browser environment is the `window` object in _non-strict mode_.

Here is a simple illustration. In the following code snippet, calling `Person()` without the `new` keyword will accidentally create a global variable called `name` because the function is not in _strict mode_.

```js
function Person(name) {
  this.name = name;
}

var person = Person('Glad Chinda');

console.log(person); // undefined
console.log(name); // "Glad Chinda"
console.log(window.name); // "Glad Chinda"
```

Another common source confusion with `this` is in DOM element event listeners. In event listeners, `this` points to the DOM element the event is targeted at. Consider the following code snippet:

```js
function ScrollController(offset) {
  this.offsets = { offsetY: offset };
}

ScrollController.prototype.registerScrollHandler = function() {
  window.addEventListener('scroll', function(event) {
    if (window.scrollY === this.offsets.offsetY) {
      console.log(`${this.offsets.offsetY}px`);
    }
  }, false);
}

var controller = new ScrollController(100);
controller.registerScrollHandler();
```

Everything looks good with this code snippet. However, when you begin scrolling the browser window vertically, you see that an error is logged on the console. The reason for the error is that `this.offsets` is `undefined` and we are trying to access the `offsetY` property of `undefined`.

The question is: How is it possible that `this.offsets` is `undefined`?

It's because the value of `this` inside the event listener is different from the value of `this` inside the enclosing prototype function. `this` inside the event listener points to `window` which is the event target and `offsets` does not exist as a property on `window`. Hence, `this.offsets` inside the event listener is `undefined`.

Prior to ES6, `Function.prototype.bind()` can be used to explicitly set the `this` binding for a function. Here is how the error can be fixed by explicitly setting the `this` binding using `Function.prototype.bind()`:

```js
// Using .bind() on event listener to resolve the value of `this`

ScrollController.prototype.registerScrollHandler = function() {
  this.element.addEventListener('scroll', (function(event) {
    if (window.scrollY === this.offsets.offsetY) {
      console.log(`${this.offsets.offsetY}px`);
    }
  }).bind(this), false);
}
```

Here, we wrapped the event listener with parentheses and called the `bind()` method passing the value of `this` from the enclosing prototype function. Calling `bind()` actually returns a new function with the specified `this` binding. Everything works perfectly now without any errors.

With ES6 arrow functions, there is no `this` binding. Hence, arrow functions use the value of `this` from their closest non-arrow function ancestor. In a case like ours, instead of using `bind()` which actually returns a new function, we can use an arrow function instead - since the `this` binding from the enclosing prototype function is retained. Here it is:

```js
// Using arrow function for event listener

ScrollController.prototype.registerScrollHandler = function() {
  this.element.addEventListener('scroll', event => {
    if (window.scrollY === this.offsets.offsetY) {
      console.log(`${this.offsets.offsetY}px`);
    }
  }, false);
}
```

## 4. Destructuring

Destructuring is another very important improvement to the JavaScript syntax. Destructuring makes it possible to access and assign values from within complex structures like arrays and objects to local variables, no matter how deeply nested those values are in the parent array or object. There are two forms of destructuring: **Object destructuring** and **Array destructuring**.

### Object Destructuring

To illustrate object destructuring, let's say we have a `country` object that looks like the following:

```js
const country = {
  name: 'Nigeria',
  region: 'Africa',
  codes: {
    cca2: 'NG',
    dialcode: '+234'
  },
  cities: [
    'Lagos',
    'Abuja',
    'Port Harcourt',
    'Benin',
    'Ibadan',
    'Calabar',
    'Warri'
  ]
}
```

We want to display some information about this country to our visitors. The following code snippet shows a very basic `countryInfo()` function that does just that:

```js
function countryInfo(country) {
  const name = country.name;
  const region = country.region || 'the world';
  const code2 = country.codes.cca2;
  const dialcode = country.codes.dialcode;
  const cities = country.cities;

  return (
`
COUNTRY TIPS:

${name}(${code2}) is one of the largest countries in ${region}.
There are so many important cities you can visit in ${name}
and here are some of them:

${cities.slice(0, 3).join(', ')} and ${cities.slice(3).length} others.

Phone numbers in ${name} usually begin with ${dialcode}.
`
  );
}

console.log(countryInfo(country));

// COUNTRY TIPS:
//
// Nigeria(NG) is one of the largest countries in Africa.
// There are so many important cities you can visit in Nigeria
// and here are some of them:
//
// Lagos, Abuja, Port Harcourt and 4 others.
//
// Phone numbers in Nigeria usually begin with +234.
```

In this snippet, we have been able to extract some values from the country object and assign them to local variables in the `countryInfo()` function - which worked out very well.

With ES6 destructuring, we can extract these values and assign them to variables with a more elegant, cleaner and shorter syntax. Here is a comparison between the old snippet and ES6 destructuring:

```js
// OLD METHOD
const name = country.name;
const region = country.region || 'the world';
const code2 = country.codes.cca2;
const dialcode = country.codes.dialcode;
const cities = country.cities;

// ES6 DESTRUCTURING
const {
  name,
  region = 'the world',
  codes: { cca2: code2, dialcode },
  cities
} = country;
```

This form of destructuring in the above code snippet is known as `object destructuring` - because we are extracting values from an object and assigning them to local variables.

> For object destructuring, an object literal is used on the left-hand-side of an assignment expression.

You can even use destructuring with function parameters as shown in the following snippet:

```js
const person = {
  name: 'Glad Chinda',
  birthday: 'August 15'
}

// FUNCTION WITHOUT DESTRUCTURED PARAMETERS
function aboutPerson(person = {}) {
  const { name, birthday, age = 'just a few' } = person;

  console.log(`My name is ${name} and I'm ${age} years old. I celebrate my birthday on ${birthday} every year.`);
}

// FUNCTION WITH DESTRUCTURED PARAMETERS
function aboutPerson({ name, birthday, age = 'just a few' } = {}) {
  console.log(`My name is ${name} and I'm ${age} years old. I celebrate my birthday on ${birthday} every year.`);
}

aboutPerson(person);

// My name is Glad Chinda and I'm just a few years old. I celebrate my birthday on August 15 every year.
```

### Array Destructuring

Array destructuring is used for extracting values from arrays and assigning them to local variables. Let's say we have the RGB(Red-Green-Blue) values of a color represented as an array as follows:

```js
const color = [240, 80, 124];
```

We want to display the RGB values for the given color. Here is how it can be done with array destructuring.

```js
// Array Destructuring
const [red, green, blue] = color;

console.log(`R: ${red}, G: ${green}, B: ${blue}`);
// R: 240, G: 80, B: 124
```

> For array destructuring, an array literal is used on the left-hand-side of an assignment expression.

With array destructuring, it is possible to skip assigning values that you don't need. Let's say we want only the blue value of the color. Here is how we can skip the red and green values without assigning them to local variables.

```js
const [,, blue] = color;

console.log(`B: ${blue}`);
// B: 124
```

Array destructuring can also be used with function parameters in a much similar fashion as object destructuring. However, there are some other ways in which array destructuring can be used for solving common problems.

A very important use case is in swapping variables. Let's say we want to search a database for records stored between two dates. We could write a simple function that accepts two `Date` objects: `fromDate` and `toDate` as follows:

```js
function fetchDatabaseRecords(fromDate, toDate) {
  // ...execute database query
}
```

We want to ensure that `fromDate` is always before `toDate` - hence we want to simply swap the dates in cases where `fromDate` is after `toDate`. Here is how we can swap the dates using array destructuring:

```js
function fetchDatabaseRecords(fromDate, toDate) {
  if (fromDate > toDate) {
    // swap the dates
    [fromDate, toDate] = [toDate, fromDate];
  }

  // ...execute database query
}
```

> For a more detailed guide on destructuring, you can have a look at [ES6 Destructuring: The Complete Guide](https://codeburst.io/es6-destructuring-the-complete-guide-7f842d08b98f).

## 5. Classes

Classes are one feature that some JavaScript developers have always wanted for a long while, especially those that had prior experience with other object-oriented programming languages. JavaScript ES6 syntax enhancements finally included classes.

Although classes are now a part of JavaScript, they don't behave the same way as in other classical programming languages. They are more like syntactic sugar for the previous methods of simulating class-based behavior. They still work based on JavaScript's prototypal inheritance.

Prior to ES6, classes were simulated using _function constructors_ and instance methods were basically created by enhancing the constructor function's prototype. Hence, when the function constructor is called with the `new` keyword, it returns an instance of the constructor that has access to all of the methods in its prototype. The value of `this` also points to the constructor instance.

Here is an example:

```js
// The Rectangle constructor
function Rectangle(length, breadth) {
  this.length = length || 10;
  this.breadth = breadth || 10;
}

// An instance method
Rectangle.prototype.computeArea = function() {
  return this.length * this.breadth;
}

// Create an instance using the new keyword
var rectangle = new Rectangle(50, 20);

console.log(rectangle.computeArea()); // 1000

// rectangle is also an instance of Object
// Due to JavaScript's prototypal inheritance
console.log(rectangle instanceof Rectangle); // true
console.log(rectangle instanceof Object); // true
```

### Class Syntax

Classes are similar to functions in so many ways. Just as with functions, classes can be defined using _class declarations_ and _class expressions_ using the `class` keyword. Also as with functions, _classes are first-hand citizens_ and can be passed around as values around your program. However, there are a couple of significant differences between classes and functions.

1. Class declarations are not hoisted and behave like `let` declarations.
2. Class constructors must always be called with `new` while the class methods cannot be called with `new`.
3. Class definition code is always in _strict mode_.
4. All class methods are non-enumerable.
5. A class name cannot be modified from within the class.

Here is our previous `Rectangle` type rewritten using the class syntax:

```js
class Rectangle {
  // The class constructor
  constructor(length, breadth) {
    this.length = length || 10;
    this.breadth = breadth || 10;
  }

  // An instance method
  computeArea() {
    return this.length * this.breadth;
  }
}

// Create an instance using the new keyword
const rectangle = new Rectangle(50, 20);

console.log(rectangle.computeArea()); // 1000

// rectangle is also an instance of Object
// Due to JavaScript's prototypal inheritance
console.log(rectangle instanceof Rectangle); // true
console.log(rectangle instanceof Object); // true

console.log(typeof Rectangle); // function
console.log(typeof Rectangle.prototype.computeArea); // function
```

Here, we use a special `constructor()` method to define the class constructor logic and also set all the instance properties. In fact, when we use the `typeof` operator on a class, it returns `"function"` - whether you explicitly define a constructor for the class or not.

Also notice that the `computeArea()` instance method is actually added to the prototype object of the underlying class constructor function. That is the reason why using the `typeof` operator on `Rectangle.prototype.computeArea` returns `"function"` as well. Hence, looking at this similarities, you can conclude that the class syntax is mostly syntactic sugar on top of the previous methods of creating custom types.

Let's see another example that is slightly more involved to demonstrate using class expressions and passing classes as function arguments.

```js
// An anonymous class expression
// assigned to a variable

const Rectangle = class {
  // The class constructor
  constructor(length, breadth) {
    this.length = length || 10;
    this.breadth = breadth || 10;
  }

  // An instance method
  computeArea() {
    return this.length * this.breadth;
  }
}

// A class passed as argument to a function
// Notice how the class is instantiated with new

const computeArea = (Shape, ...dimensions) => {
  return (new Shape(...dimensions)).computeArea();
}

console.log(computeArea(Rectangle, 50, 20)); // 1000
```

Here, we first created an anonymous class expression and assigned it to the `Rectangle` variable. Next, we created a function that accepts a `Shape` class as first argument and the dimensions for instantiating the Shape as the remaining arguments. The code snippet assumes that any Shape class it receives implements the `computeArea()` method.

### Extending Classes

Just like with other object-oriented programming languages, JavaScript classes have functionalities for extending classes. Hence it is possible to create _derived_ or _child_ classes with modified functionality from a _parent_ class.

Let's say we have a `Rectangle` class for creating rectangles and we want to create a `Square` class for creating rectangles with equal length and breadth. Here is how we can do it:

```js
class Rectangle {
  constructor(length, breadth) {
    this.length = length || 10;
    this.breadth = breadth || 10;
  }

  computeArea() {
    return this.length * this.breadth;
  }
}

// The Square class extends the Rectangle class
class Square extends Rectangle {
  constructor(length) {
    // super() calls the constructor of the parent class
    super(length, length);
  }
}

const square = new Square;

// Square inherits the methods and properties of Rectangle
console.log(square.length); // 10
console.log(square.breadth); // 10
console.log(square.computeArea()); // 100

// square is also an instance of Rectangle
console.log(square instanceof Square); // true
console.log(square instanceof Rectangle); // true
```

First, notice the use of the `extends` keyword, which indicates that we want to create a derived class from a parent class. The derived class inherits all the properties and methods in the prototype of the parent class including the `constructor`.

Also notice that we use a `super` reference to invoke the constructor of the parent class from within the constructor of the derived class. This is very useful when you want to enhance the functionality of an inherited method in the derived class. For example, a call to `super.computeArea()` from within the `Square` class will call the `computeArea()` method implemented in the `Rectangle` class.

A call to `super()` must be made in the constructor of every derived class and it must come before any reference is made to `this`. This is because calling `super()` sets the value of `this`. However, `super()` should never be used in a class that is not a derived class as it is considered a syntax error.

Creating derived classes is not limited to extending classes alone. Derived classes are generally created by extending any JavaScript expression that can used as a constructor and has a prototype - such as JavaScript functions. Hence the following is possible:

```js
function Person(name) {
  this.name = name || 'Glad Chinda';
}

Person.prototype.getGender = function() {
  return this.gender;
}

class Male extends Person {
  constructor(name) {
    super(name);
    this.gender = 'MALE';
  }
}

const me = new Male;

// Male inherits the methods and properties of Person
console.log(me.getGender()); // "MALE"

// me is also an instance of Person
console.log(me instanceof Male); // true
console.log(me instanceof Person); // true
```

### Static Class Members

So far we have been looking at instance methods and properties. There are times when you require static methods or properties that apply directly to the class and don't change from one instance to another. Prior to ES6, static members can be added as follows:

```js
function Lion() {
  // constructor function
}

// Static property
Lion.category = 'ANIMAL';

// Static method
Lion.animalType = function() {
  return 'CAT';
}

console.log(Lion.category); // "ANIMAL"
console.log(Lion.animalType()); // "CAT"
```

With ES6 classes, the `static` keyword is placed before a method name to indicate that the method is a _static method_. However, static properties cannot be created from within the class. Here is how we can create static members:

```js
class Lion {

  // Static method
  static animalType() {
    return 'CAT';
  }

}

// Static property
Lion.category = 'ANIMAL';

console.log(Lion.category); // "ANIMAL"
console.log(Lion.animalType()); // "CAT"
```

Static class members are also inherited by derived classes. They can also be overridden by the derived class in much the same way as instance methods and properties. Here is a simple example:

```js
class Lion {

  // Static method
  static animalType() {
    return 'CAT';
  }

}

// Static property
Lion.category = 'ANIMAL';

// Derived Lioness class
class Lioness extends Lion {

  // Override static method
  static animalType() {
    return `${super.animalType()}::LION`;
  }

}

console.log(Lioness.category); // "ANIMAL"
console.log(Lioness.animalType()); // "CAT::LION"
```

### More Features

There are a couple more class features worth considering, one of which is _accessor properties_. They can be very useful in cases where you need to have properties on the class prototype. Here is a simple example:

```js
class Person {
  constructor(firstname, lastname) {
    this.firstname = firstname || 'Glad';
    this.lastname = lastname || 'Chinda';
  }

  get fullname() {
    return `${this.firstname} ${this.lastname}`;
  }

  set fullname(value) {
    const [ firstname, lastname ] = value.split(' ');

    if (firstname) this.firstname = firstname;
    if (lastname) this.lastname = lastname;
  }
}

const me = new Person;
console.log(me.fullname); // "Glad Chinda"

me.fullname = "Jamie";
console.log(me.fullname); // "Jamie Chinda"

me.fullname = "John Doe (Junior)";
console.log(me.fullname); // "John Doe"
```

Another nice feature of classes which is very similar to object literals is the ability to use _computed names_ for class members. These computed names can also be used for accessor properties. Computed names are usually wrapped between a pair of _square brackets_(`[]`).

Here is a simple example:

```js
const prefix = 'compute';

class Square {
  constructor(length) {
    this.length = length || 10;
  }

  // An computed class method
  [`${prefix}${Square.prototype.constructor.name}Area`]() {
    return this.length * this.length;
  }
}

const square = new Square;
console.log(square.computeSquareArea()); // 100
```
