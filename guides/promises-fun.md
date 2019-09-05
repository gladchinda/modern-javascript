# More Fun with JavaScript Promises

Asynchronous programs in JavaScript aren't something new. Infact if you've been writing asynchronous codes for sometime you should already know about using callbacks. One known issue with using callbacks is the possibility of running into a _callback hell_.

In ES6, JavaScript Promises were added to the language spec, bringing about an entirely new shift in how asynchronous code is written, and also mitigating against the issue of running into _callback hells_. If you are already using ES6 syntax in your codes you may already be familiar with Promises.

In this guide, you will see some fun and practical ways of using JavaScript Promises in your programs. This guide is not in anyway an introduction to JavaScript Promises. Hence, prior knowledge of Promises is required to understand and follow it through.

## Creating Promises

Just to serve as a little reminder, a JavaScript can be created using the `Promise` constructor. The constructor function takes an `executor` function as its argument, which in turn takes the `resolve` and `reject` functions as arguments for resolving or rejecting the Promise respectively.

Here is a very simple example of creating a JavaScript Promise that is settled after loading an image:

```js
const loadPhoto = photo_url => new Promise((resolve, reject) => {
  // Create a new Image object
  const img = new Image();

  // Set the image source
  img.src = photo_url;

  // If the image is loads,
  // fulfil the promise with the Image object
  img.onload = () => resolve(img);

  // If there was an error,
  // reject the promise with the Error object
  img.onerror = evt => reject(new Error('Image could not be loaded.'));
});
```

The returned promise can then be handled by chaining callbacks to the `then()`, `catch()` or `finally()` methods as follows:

```js
loadPhoto('https://picsum.photos/g/800/600/?image=0&gravity=north')
  .then(console.log)
  .catch(console.error);

// Alternatively, the error can be handled in the same .then() call
loadPhoto('https://picsum.photos/g/800/600/?image=0&gravity=north')
  .then(console.log, console.error);
```

Often times, you just want to create a promise that is already _settled_ (either _fulfilled_ with a value or _rejected_ with a reason). For cases like this, you can use the `Promise.resolve()` and `Promise.reject()` methods. Here is a simple example:

```js
// This promise is already fulfilled with a number (100)
const fulfilledPromise = Promise.resolve(100);

// This promise is already rejected with an error
const rejectedPromise = Promise.reject(new Error('Operation failed.'));
```

There are also times when you are not sure if a value is a Promise or not. For cases like this, you can use `Promise.resolve()` to create a fulfilled promise with the value and then work with the returned promise.

```js
// User object
const USER = {
  name: 'Glad Chinda',
  country: 'Nigeria',
  job: 'Fullstack Engineer'
};

// Create a fulfilled promise using Promise.resolve()
Promise.resolve(USER)
  .then(user => console.log(user.name));
```

## Timing with Promises

### Delaying Execution

Promises can be very useful for timing applications. Some programming languages like PHP have a `sleep()` function that can be used to delay execution of an operation until after the sleep time.

```php
<?php
// Sleep for 5 seconds
sleep(5);

// Then execute this operation
App::start();
```

While a `sleep()` function does not exist as part of the JavaScript spec, the global `setTimeout()` and `setInterval()` functions are commonly used for delayed or time-based executions.

Here is how the `sleep()` function can be simulated using Promises in JavaScript. However, in this version of the `sleep()` function, the halt time will be in _milliseconds_ instead of _seconds_.

```js
const sleep = ms => new Promise(resolve => setTimeout(resolve, ms));
```

Here is what using the `sleep()` function could look like:

```js
// Sleep for 5 seconds
// Then execute the operation

sleep(5000).then(executeOperation);


// Delay function
// Using async/await with sleep()

const delay = async (seconds, callback) => {
  // Sleep for the specified seconds
  await sleep(seconds * 1000);

  // Then execute the operation
  callback();
}

// Execution delayed by 5 seconds
delay(5, executeOperation);
```

### Execution Time

What if you are interested in knowing how long it took for an asynchronous operation to be completed. This is usually the case when benchmarking the performance of some functionality or form of implementation.

Here is a simple implementation that leverages on JavaScript Promise to compute the execution time for an aynchronous operation.

```js
const timing = callback => {
  // Get the start time using performance.now()
  const start = performance.now();

  // Perform the asynchronous operation
  // Finally, log the time difference
  return Promise.resolve(callback())
    .finally(() => console.log(`Timing: ${performance.now() - start}`));
};
```

In this implementation, `performance.now()` is used instead of `Date.now()` for getting the time with a higher resolution. However, for non-browser environments where the `performance` object does not exist, you can fallback to using `Date.now()` or other host implementations.

```js
// Async operation that takes between 1 - 5 seconds
const asyncOperation = () => new Promise(resolve => {
  setTimeout(() => resolve('DONE'), Math.ceil(Math.random() * 5) * 1000);
});


// Compute execution time in ms
// And log it to the console

timing(asyncOperation); // Timing: 4003.4000000014203
```

## Sequential Execution with Promises

With JavaScript Promises, you can execute asynchronous operations in sequence. This is usually the case when a later asynchronous operation depends on the execution of a former aynchronous operation, or when the result of a former asynchronous operation is required for a later operation.

Executing asynchronous operations in sequence usually involves chaining `.then()` and `.catch()` handlers to a promise. When a promise is rejected in the chain, it is handled by the rejection handler defined in the next `.then()` handler in the chain and execution continues down the chain.

However, if no rejection handler has been defined in the next `.then()` handler in the chain, the promise rejection is cascaded down the chain until it reaches the first `.catch()` handler.

### A Collection of Photos

<!-- Let's say you receive a large collection of records as an array from an external source at some interval and you want to build a simple application that processes the collection to remove duplicate records, filters the collection to remove the records already existing in your database, and finally store the fresh records in your database. -->
Let's say you are building a photo gallery application and you want to be able to fetch photos and filter photos by format, aspect-ratio, dimension ranges, etc. Here are some possible functions you could have in your application.

<!-- The application will look like the following: -->

```js
/**
 * Fetches photos from the Picsum API
 * @returns {array} An array of photos
 */
const fetchPhotos = () =>
  fetch('https://picsum.photos/list')
    .then(response => response.json());

/**
 * Filters photos and returns only JPEG photos
 * @param {array} photos
 * @returns {array} An array of JPEG photos
 */
const jpegOnly = photos =>
  photos.filter(({ format }) => format.toLowerCase() === 'jpeg')

/**
 * Filters photos and returns only square photos
 * @param {array} photos
 * @returns {array} An array of square photos
 */
const squareOnly = photos =>
  photos.filter(({ width, height }) => height == width)

/**
 * Returns a function for filtering photos by size based on `px`
 * @param {number} px The maximum allowed photo dimension in pixels
 * @returns {function} Function that filters photos and returns an array of photos smaller than `px`
 */
const smallerThan = px => photos =>
  photos.filter(({ width, height }) => Math.max(width, height) < px)

/**
 * Return an object containing the photos count and URLs.
 * @param {array} photos
 * @returns {object} An object containing the photos count and URLs
 */
const listPhotos = photos => ({
  count: photos.length,
  photos: photos.map(({ post_url }) => post_url)
})
```

The previous code snippet contains 5 functions you could possibly have in your application. The `fetchPhotos()` function fetches a collection of photos from the Picsum API using `fetch()` and returns a Promise that is fulfilled with the collection of photos.

Here is what the collection returned from the Picsum API looks like:

```json
```

The filter functions accept a collection of photos as argument and filter the collection in some manner. For example: `jpegOnly()` filters the original collection and returns a collection of only JPEG images, while `squareOnly()` filters the original collection and returns a collection of only photos with square aspect-ratio.

The `smallerThan()` function is a higher-order function that takes a dimension and returns a photos filtering function to get photos with dimensions below the specified dimension threshold.

The following code snippet shows how you can chain the operations in an execution sequence.

```js
// Execute asynchronous operations in sequence
fetchPhotos()
  .then(jpegOnly)
  .then(squareOnly)
  .then(smallerThan(2500))
  .then(listPhotos)
  .then(console.log)
  .catch(console.error);
```

Here is the sequence:

1. fetch the photos collection
2. filter the collection leaving only JPEG photos
3. filter the collection leaving only photos with square aspect-ratio
4. filter the collection leaving only photos smaller than 2500px
5. extract the photos count and URLs from the collection
6. log the final output on the console
7. log error to the console if an error occurred at any point in the sequence

The resulting output that is logged to the console looks like this:

### Dumb Then Handlers

If you look closely at the previous code snippet for the photos collection filtering app, you will notice that all the functions are synchronous except the `fetchPhotos()` function, which is asynchronous and actually returns a pending promise.

**_How then is it possible to chain `.then()` calls with synchronous handler functions?_**

One important thing to understand about the `.then()` promise method is that it always returns a promise. Here is a breakdown of how `.then()` returns a promise based on what is returned from the handler function is passed to it:

| handlerFn() | .then() |
| - | - |
| `value` | promise that is resolved with `value` |
| nothing (`undefined`) | promise that is resolved with `undefined` |
| throws an error | promise that is rejected with the error |
| resolved promise | promise that is resolved to the value of the returned handler promise |
| rejected promise | promise that is rejected with the value of the returned handler promise |
| pending promise | promise whose resolution/rejection is dependent on the resolution/rejection of the returned handler promise |

With this breakdown, it is easy to understand how the `photos` array returned from the handler functions gets wrapped into promises with each `.then()` call.

**_What then happens when the argument passed to the `.then()` call is not a function?_**

The `.then()` method can take up to two handler functions as its arguments: _fulfilment handler_ and _rejection handler_ respectively. However, if any of these two arguments is not a function, `.then()` replaces that argument with a function and continues with the normal execution flow.

It becomes important to know what kind of function the argument is replaced with. Here is what it is:

- If the _fulfilment handler_ argument is not a function, it is replaced with an **Identity Function**. An _identity function_ is a function that simply returns the argument it receives.

- If the _rejection handler_ argument is not a function, it is replaced with a **Thrower Function**. A _thrower function_ is a function that simply throws the error or value it receives as its argument.

**_Dumb Handlers_**

If you observe correctly, then you will notice that either the _identity function_ or the _thrower function_ does not alter the normal execution flow of the promise sequence.

They simply have the same effect to the promise chain as omitting the particular `.then()` call. For this reason, I will be referring to these handler arguments as **dumb handlers**.

## Batch Execution With Promises

With JavaScript Promises, you can execute multiple independent asynchronous operations in batch or parallel using the `Promise.all()` method.

`Promise.all()` accepts an iterable of promises as its argument and returns a promise that is fulfilled when all the promises in the iterable are fulfilled or is rejected when one of the promises in the iterable is rejected.

If the returned promise fulfills, it is fulfilled with an array of all the values from the fulfilled promises in the iterable in the same order. However, if it rejects, it is rejected with the reason of the first promise in the iterable that rejected.

Continuing with the photo gallery app, let's say you want to display the fetched photos on the app. Usually, you would loop through the each photo and load it into the view. Let's see how that goes:

```js
/**
 * Loads a photo from source.
 * @param {string} src The source URL
 * @returns {Promise} A promise that resolves with an Image object
 */
const loadPhoto = src => new Promise((resolve, reject) => {
  const img = new Image();
  img.src = src;
  img.onload = () => resolve(img);
  img.onerror = evt => reject(new Error('Image could not be loaded.'));
});

/**
 * Returns a function for getting photo URLs at a given resample size.
 * @param {number} px The resample size in pixels
 * @returns {function} Function that returns an array of photo URLs at the given resample size
 */
const getPhotoUrls = px => photos =>
  photos.map(({ id }) => `https://picsum.photos/${px}/?image=${id}`)

/**
 * Returns a function for displaying a photo on a grid.
 * @param {Node} $grid The DOM Node for the photo grid
 * @returns {function} Function for displaying an Image object on a photo grid
 */
const displayPhoto = $grid => photo => {
  photo.classList = 'photos-grid__img';
  $grid.appendChild(photo);
}

/**
 * Returns a function for rendering photos on a grid.
 * @param {Node} $grid The DOM Node for the photo grid
 * @returns {function} Function for rendering Image objects on a photo grid
 */
const displayPhotos = $grid => photos => {
  while ($grid.firstChild) {
    $grid.removeChild($grid.firstChild);
  }

  const div = document.createElement('div');
  div.classList = 'photos-grid__header';
  div.innerHTML = `${photos.length} Photos`;

  $grid.appendChild(div);

  photos.forEach(photo => {
    loadPhoto(photo)
      .then(displayPhoto($grid))
  });

  return photos;
}
```

A couple more functions have been added to aid rendering the photos on a grid in the view.

- `loadPhoto()` - loads an image and returns a promise that resolves to the `Image` object.
- `getPhotoUrls()` - higher-order function for getting Picsum photo urls by photo ID.
- `displayPhoto()` - higher-order function for displaying a single `Image` on a photo grid.
- `displayPhotos()` - higher-order function for displaying an array of `Image` objects on a photo grid.

```js
fetchPhotos()
  .then(jpegOnly)
  .then(squareOnly)
  .then(smallerThan(2500))
  .then(getPhotoUrls(2000))
  .then(displayPhotos($photosGrid))
  .catch(console.error);
```

## Racing with Promises

