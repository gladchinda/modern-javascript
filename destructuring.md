# Modern JavaScript: Destructuring

JavaScript is currently one of the most used programming languages in the world, being used across different platforms ranging from web browsers, mobile devices, VR, web servers, etc. Although the language has not received a lot changes through the years when compared to other programming languages, there are some recent changes that are worth taking note of, because of the expressive power they add to the language - some of which are: _template literals_, _destructuring_, _spread operator_, _arrow functions_, _classes_, etc.

In this tutorial, our major focus will be on learning how to **harness the power of destructuring** to simplify our JavaScript programs. Before we dive into the _what_ and _how_ of destructuring I think the _why_ should be considered.

For this tutorial, it is assumed you already have some prior experience writing JavaScript code. There is a chance that some of the code snippets may not work on your browser depending on the version of the browser, since all the ES6 and ES7 features are not yet fully supported in all browsers. You may opt to test the code snippets on **Codepen** or your favorite online JavaScript live editor. Another alternative would be using the **Babel transpiler** to get it to work on your browser.

## Why Destructuring?

To explain the why of destructuring, we will consider a scenario which most of us might be familiar with or might have come across at one time or the other when coding in JavaScript. Imagine we have the data of a student including scores in three subjects(_Maths_, _Elementary Science_, _English_) represented in an object and we need to display some information based on this data. We could end up with something that looks like this:

```js

const student = {
    name: 'John Doe',
    age: 16,
    scores: {
        maths: 74,
        english: 63,
        science: 85
    }
};

function displaySummary(student) {
    console.log('Hello, ' + student.name);
    console.log('Your Maths score is ' + (student.scores.maths || 0));
    console.log('Your English score is ' + (student.scores.english || 0));
    console.log('Your Science score is ' + (student.scores.science || 0));
}

displaySummary(student);

// Hello, John Doe
// Your Maths score is 74
// Your English score is 63
// Your Science score is 85

```

With the above code snippet, we would achieve the desired result. However, there are a few caveats to writing code this way. One of which is - you can easily make a _typo_ and instead of writing `scores.english` for example, you write `score.english` which will result in an error. Again, if the `scores` object we are accessing is deeply nested in another object, the object access chain becomes longer which could mean more typing. These may not seem to be much an issue, but with destructuring we can do the same in a more expressive and compact syntax.

## What is Destructuring?

**Destructuring simply implies breaking down a complex structure into simpler parts.** In JavaScript, this complex structure is usually an object or an array. With the destructuring syntax, you can extract smaller fragments from arrays and objects. Destructuring syntax can be used for _variable declaration_ or _variable assignment_. You can also handle nested structures by using nested destructuring syntax.

Using destructuring, the function from our previous snippet will look like this:

```js

function displaySummary({ name, scores: { maths = 0, english = 0, science = 0 } }) {
    console.log('Hello, ' + name);
    console.log('Your Maths score is ' + maths);
    console.log('Your English score is ' + english);
    console.log('Your Science score is ' + science);
}

```

There is a chance that the code is the last snippet didn't go down well with you. If that's the case, then follow up with the tutorial till you get to the end - I assure you it will all make sense. Let's continue to learn how we can harness the power of destructuring.

## Object Destructuring

What we saw in that last snippet is a form of _object destructuring_ being used as an assignment to a function. Let's explore what we can do with object destructuring starting from the ground up. Basically, **you use an object literal on the left-hand-side of an assignment expression** for object destructuring. Here is a basic snippet:

```js

const student = {
    firstname: 'Glad',
    lastname: 'Chinda',
    country: 'Nigeria'
};

// Object Destructuring
const { firstname, lastname, country } = student;

console.log(firstname, lastname, country); // Glad Chinda Nigeria

```

Here we used object destructuring syntax to assign values to three variables: `firstname`, `lastname` and `country` using the values from their corresponding keys on the `student` object. This is the most basic form of object destructuring.

**The same syntax can be used in variable assignment** as follows:

```js

// Initialize local variables
let country = 'Canada';
let firstname = 'John';
let lastname = 'Doe';

const student = {
    firstname: 'Glad',
    lastname: 'Chinda',
    country: 'Nigeria'
};

// Reassign firstname and lastname using destructuring
// Enclose in a pair of parentheses, since this is an assignment expression
({ firstname, lastname } = student);

// country remains unchanged (Canada)
console.log(firstname, lastname, country); // Glad Chinda Canada

```

The above snippet shows how to use object destructuring to assign new values to local variables. Notice that we had to **use a pair of enclosing parentheses (`()`) in the assignment expression**. If omitted, the destructuring object literal will be taken to be a block statement, which will lead to an error because a block cannot appear at the left-hand-side of an assignment expression.

#### Default Values

Trying to assign a variable corresponding to a key that does not exist on the destructured object will cause the value `undefined` to be assigned instead. You can pass _default values_ that will be assigned to such variables instead of `undefined`. Here is a simple example.

```js

const person = {
    name: 'John Doe',
    country: 'Canada'
};

// Assign default value of 25 to age if undefined
const { name, country, age = 25 } = person;

// Here I am using ES6 template literals
console.log(`I am ${name} from ${country} and I am ${age} years old.`);

// I am John Doe from Canada and I am 25 years old.'

```

Here we assigned a default value of `25` to the `age` variable. Since the `age` key does not exist on the `person` object, `25` is assigned to the `age` variable instead of `undefined`.

#### Using Different Variable Names

So far, we have been assigning to variables that have the same name as the corresponding object key. You can assign to a different variable name using this syntax: `[object_key]:[variable_name]`. You can also pass default values using the syntax: `[object_key]:[variable_name] = [default_value]`. Here is a simple example.

```js

const person = {
    name: 'John Doe',
    country: 'Canada'
};

// Assign default value of 25 to years if age key is undefined
const { name: fullname, country: place, age: years = 25 } = person;

// Here I am using ES6 template literals
console.log(`I am ${fullname} from ${place} and I am ${years} years old.`);

// I am John Doe from Canada and I am 25 years old.'

```

Here we created three local variables namely: `fullname`, `place` and `years` that map to the `name`, `country` and `age` keys respectively of the `person` object. Notice that we specified a default value of `25` for the `years` variable in case the `age` key is missing on the `person` object.

#### Nested Object Destructuring

Referring back to our initial destructuring example, we recall that the `scores` object is nested in the `student` object. Let's say we want to assign the _Maths_ and _Science_ scores to local variables. The following code snippet shows how we can use nested object destructuring to do this.

```js

const student = {
    name: 'John Doe',
    age: 16,
    scores: {
        maths: 74,
        english: 63
    }
};

// We define 3 local variables: name, maths, science
const { name, scores: {maths, science = 50} } = student;
console.log(`${name} scored ${maths} in Maths and ${science} in Elementary Science.`);

// John Doe scored 74 in Maths and 50 in Elementary Science.

```

Here, we defined three local variables: `name`, `maths` and `science`. Also, we specified a default value of `50` for `science` in case it does not exist in the nested `scores` object. Notice that, `scores` is not defined as a variable. Instead, we use nested destructuring to extract the `maths` and `science` values from the `scores` object.

**When using nested object destructuring, be careful to avoid using an empty nested object literal.** Though it is valid syntax, it actually does no assignment. For example, the following destructuring does absolutely no assignment.

```js

// This does no assignment
// Because of the empty nested object literal ({})
const { scores: {} } = student;

```


## Array Destructuring

By now you are already feeling like a destructuring ninja, having gone through the rigours of understanding object destructuring. The good news is that array destructuring is very much similar and straight forward so let's dive in right away.

In array destructuring, **you use an array literal on the left-hand-side of an assignment expression**. Each variable name on the array literal maps to the corresponding item at the same index on the destructured array. Here is a quick example.

```js

const rgb = [255, 200, 0];

// Array Destructuring
const [red, green, blue] = rgb;

console.log(`R: ${red}, G: ${green}, B: ${blue}`); // R: 255, G: 200, B: 0

```

In this example, we have assigned the items in the `rgb` array to three local variables: `red`, `green` and `blue` using array destructuring. Notice that each variable is mapped to the corresponding item at the same index on the `rgb` array.

#### Default Values

If the number of items in the array is more than the number of local variables passed to the destructuring array literal, then the excess items are not mapped. But if the number of local variables passed to the destructuring array literal exceed the number of items in the array, then each excess local variable will be assigned a value of `undefined` except you specify a _default value_.

Just as with object destructuring, you can set default values for local variables using array destructuring. In the following example, we would set default values for some variables in case a corresponding item is not found.

```js

const rgb = [200];

// Assign default values of 255 to red and blue
const [red = 255, green, blue = 255] = rgb;

console.log(`R: ${red}, G: ${green}, B: ${blue}`); // R: 200, G: undefined, B: 255

```

You can also do an **array destructuring assignment**. Unlike with object destructuring, **you don't need to enclose the assignment expression in parentheses**. Here is an example.

```js

let red = 100;
let green = 200;
let blue = 50;

const rgb = [200, 255, 100];

// Destructuring assignment to red and green
[red, green] = rgb;

console.log(`R: ${red}, G: ${green}, B: ${blue}`); // R: 200, G: 255, B: 50

```

#### Skipping Items

It is possible to skip some items you don't want to assign to local variables and only assign the ones you are interested in. Here is an example of how we can assign only the blue value to a local variable.

```js

const rgb = [200, 255, 100];

// Skip the first two items
// Assign the only third item to the blue variable
const [,, blue] = rgb;

console.log(`Blue: ${blue}`); // Blue: 100

```

We used _comma separation_(`,`) to omit the first two items of the `rgb` array, since we only needed the third item.

#### Swapping Variables

One very nice application of array destructuring is in _swapping local variables_. Imagine you are building a photo manipulation app and you want to be able to swap the `height` and `width` dimensions of a photo when the orientation of the photo is switched between `landscape` and `portrait`. The old-fashioned way of doing this will look like the following.

```js

let width = 300;
let height = 400;
const landscape = true;

console.log(`${width} x ${height}`); // 300 x 400

if (landscape) {
    // An extra variable needed to copy initial width
    let initialWidth = width;

    // Swap the variables
    width = height;
    // height is assigned the copied width value
    height = initialWidth;

    console.log(`${width} x ${height}`); // 400 x 300
}

```

The above code snippet performs the task, although we had to use an additional variable to copy the value of one of the swapped variables. With array destructuring, we can perform the swap with a single assignment statement. The following snippet shows how array destructuring can be used to achieve this.

```js

let width = 300;
let height = 400;
const landscape = true;

console.log(`${width} x ${height}`); // 300 x 400

if (landscape) {
    // Swap the variables
    [width, height] = [height, width];
    console.log(`${width} x ${height}`); // 400 x 300
}

```

#### Nested Array Destructuring

Just as with objects, you can also do nested destructuring with arrays. The corresponding item must be an array in order to use a nested destructuring array literal to assign items in it to local variables. Here is a quick example to illustrate this.

```js

const color = ['#FF00FF', [255, 0, 255], 'rgb(255, 0, 255)'];

// Use nested destructuring to assign red, green and blue
const [hex, [red, green, blue]] = color;

console.log(hex, red, green, blue); // #FF00FF 255 0 255

```

In the code above, we assigned `hex` variable and used nested array destructuring to assign `red`, `green` and `blue` variables.

#### Rest Items

Sometimes you may want to assign some items to variables, while ensuring that the remaining items are _captured_(assigned to another local variable). The new **spread operator**(`...`) added in ES6 can be used with destructuring to achieve this. Here is a quick example.

```js

const rainbow = ['red', 'orange', 'yellow', 'green', 'blue', 'indigo', 'violet'];

// Assign the first and third items to red and yellow
// Assign the remaining items to otherColors variable using the spread operator(...)
const [red,, yellow, ...otherColors] = rainbow;

console.log(otherColors); // ['green', 'blue', 'indigo', 'violet']

```

Here you can see that after we assigned the first and third items of the `rainbow` array to `red` and `yellow` respectively, we used the _spread operator_(`...`) to capture and assign the remaining values to the `otherColors` variable. This is referred to as the _rest items_ variable. **Note however that the rest items variable, if used, must always appear as the last item** in the destructuring array literal otherwise an error will be thrown.

There are some nice applications of _rest items_ that are worth considering. One of which is cloning arrays. In JavaScript, **arrays are reference types and hence they are assigned by reference instead of being copied**. So in the following snippet, both the `rainbow` and the `rainbowClone` variables point to the same array reference in memory and hence any change made to `rainbow` is also applied to `rainbowClone` and vice-versa.

```js

const rainbow = ['red', 'orange', 'yellow', 'green', 'blue', 'indigo', 'violet'];
const rainbowClone = rainbow;

// Both rainbow and rainbowClone point to the same
// array reference in memory, hence it logs (true)
console.log(rainbow === rainbowClone); // true

// Keep only the first 3 items and discard the rest
rainbowClone.splice(3);

// The change on rainbowClone also affected rainbow
console.log(rainbow); // ['red', 'orange', 'yellow']
console.log(rainbowClone); // ['red', 'orange', 'yellow']

```

The following code snippet shows how we can clone an array in the old-fashioned(ES5) way - `Array.prototype.slice` and `Array.prototype.concat` to the rescue.

```js

const rainbow = ['red', 'orange', 'yellow', 'green', 'blue', 'indigo', 'violet'];

// Cloning with Array.prototype.slice
const rainbowClone1 = rainbow.slice();
console.log(rainbow === rainbowClone1); // false
console.log(rainbowClone1); // ['red', 'orange', 'yellow', 'green', 'blue', 'indigo', 'violet']

// Cloning with Array.prototype.concat
const rainbowClone2 = rainbow.concat();
console.log(rainbow === rainbowClone2); // false
console.log(rainbowClone2); // ['red', 'orange', 'yellow', 'green', 'blue', 'indigo', 'violet']

```

Here is how you can use array destructuring and the spread operator(`...`) to create an array clone.

```js

const rainbow = ['red', 'orange', 'yellow', 'green', 'blue', 'indigo', 'violet'];

// Cloning with array destructuring and spread operator
const [...rainbowClone] = rainbow;

console.log(rainbow === rainbowClone); // false
console.log(rainbowClone); // ['red', 'orange', 'yellow', 'green', 'blue', 'indigo', 'violet']

```


## Mixed Destructuring

There are cases when you are working with a pretty complex object/array structure and you need to assign some values from it to local variables. A good example would be an object with several deeply nested objects and arrays. In cases like this, you can use a combination of object destructuring and array destructuring to target certain parts of the complex structure as required. The following code shows a simple example.

```js

const person = {
    name: 'John Doe',
    age: 25,
    location: {
        country: 'Canada',
        city: 'Vancouver',
        coordinates: [49.2827, -123.1207]
    }
}

// Observe how mix of object and array destructuring is being used here
// We are assigning 5 variables: name, country, city, lat, lng
const {name, location: {country, city, coordinates: [lat, lng]}} = person;

console.log(`I am ${name} from ${city}, ${country}. Latitude(${lat}), Longitude(${lng})`);
// I am John Doe from Vancouver, Canada. Latitude(49.2827), Longitude(-123.1207)

```


## Destructured Function Parameters

Destructuring can also be applied on function parameters to extract values and assign them to local variables. Note however that **the destructured parameter cannot be omitted** (it is required) otherwise it throws an error. A good use case is the `displaySummary` function from our initial example that expects a `student` object as parameter. We can destructure the `student` object and assign the extracted values to local variables of the function. Here is the example again:

```js

const student = {
    name: 'John Doe',
    age: 16,
    scores: {
        maths: 74,
        english: 63,
        science: 85
    }
};

// Without Destructuring
function displaySummary(student) {
    console.log('Hello, ' + student.name);
    console.log('Your Maths score is ' + (student.scores.maths || 0));
    console.log('Your English score is ' + (student.scores.english || 0));
    console.log('Your Science score is ' + (student.scores.science || 0));
}

// With Destructuring
function displaySummary({name, scores: { maths = 0, english = 0, science = 0 }}) {
    console.log('Hello, ' + name);
    console.log('Your Maths score is ' + maths);
    console.log('Your English score is ' + english);
    console.log('Your Science score is ' + science);
}

displaySummary(student);

```

Here we extracted the values we need from the `student` object parameter and assigned them to local variables: `name`, `maths`, `english` and `science`. Notice that although we have specified default values for some of the variables, if you call the function with no arguments you will get an error because **destructured parameters are always required**. You can assign a fallback object literal as default value for the `student` object and the nested `scores` object in case they are not supplied to avoid the error as shown in the following snippet.

```js

function displaySummary({ name, scores: { maths = 0, english = 0, science = 0 } = {} } = {}) {
    console.log('Hello, ' + name);
    console.log('Your Maths score is ' + maths);
    console.log('Your English score is ' + english);
    console.log('Your Science score is ' + science);
}

displaySummary();

// Hello, undefined
// Your Maths score is 0
// Your English score is 0
// Your Science score is 0

```
