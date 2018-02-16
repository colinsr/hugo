---
title: "Array Chunk"
date: 2018-02-15T08:47:31-05:00
tags: ["javascript", "exercises"]
categories: ["javascript"]
---

Today we'll be working through an exercise where the input is an array `[1,2,3,4,5]` and a size `2`.  
Based on that input our goal is to write a function that will take the original array and return a new multidimensional (nested) array with the elements each of the size param that was passed in.
It'll probably be easier to understand based on this example:

```javascript
chunk([1,2,3,4,5], 2) => [ [1,2], [3,4], [5]]
```

This problem should be easier to reason about now that we've seen a example.  
Since the `size` is even and the array length is odd, we have a leftover element that gets to sit alone in the last array.

Let's get this exercise setup with our test and index files.  First we'll make sure we're in our `exercises/` directory and run `mkdir chunk && cd chunk/ && touch index.js tests.js`.
Then we'll use our method of choice to get the following content into our `tests.js` file:
```javascript
const chunk = require('./index');

test('function chunk exists', () => {
  expect(typeof chunk).toEqual('function');
});

test('chunk divides an array of 8 elements with chunk size 2', () => {
  const arr = [1, 2, 3, 4, 5, 6, 7, 8];
  const chunked = chunk(arr, 2);

  expect(chunked).toEqual([[1, 2], [3, 4], [5, 6], [7, 8]);
});

test('chunk divides an array of 4 elements with chunk size 1', () => {
  const arr = [1, 2, 3, 4];
  const chunked = chunk(arr, 1);

  expect(chunked).toEqual([[1], [2], [3], [4]]);
});

test('chunk divides an array of 5 elements with chunk size 3', () => {
  const arr = [1, 2, 3, 4, 5, 6, 7, 8];
  const chunked = chunk(arr, 3);

  expect(chunked).toEqual([[1, 2, 3], [4, 5, 6], [7, 8]]);
});

test('chunk divides an array of 13 elements with chunk size 5', () => {
  const arr = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13];
  const chunked = chunk(arr, 5);

  expect(chunked).toEqual([[1, 2, 3, 4, 5], [6, 7, 8, 9, 10], [11, 12, 13]]);
});
```

Now we can start running our jest tests in watch mode with `jest tests.js --watch`, you can expect everything to fail at this point since we still need to stub out our chunk function.

Then we'll stub out our `index.js` like so:
```javascript
function chunk(array, size) {
}

module.exports = chunk;
```

With that out of the way after we save our file, jest will report that the first test is passing and we still have four failing tests at the moment.  Great, let's get started implementing this.

At first glance this is the approach I think we'll take to get this sorted out:
```javascript
function chunk(array, size) {
  //setup counter and new array objects

  //start looping through original array

  //shove size number of elements into a new array
  
  //return nested array
}
```

While working through this I came up with the following during my first pass:
```javascript
function chunk(array, size) {
  const nestedArray = [];//setup counter and new array objects
  let i = 0;
  //start looping through original array
  while(i < array.length) {
    //shove size number of elements into a new array
    let arrayChunk = array.slice(i, size);
    console.log(arrayChunk);
    i += size;
  }
  return nestedArray;//return nested array
}
```
This was whack because my `array.slice(begin, end)` "end" param wasn't being incremented at all.
Some quick debugging came up with one right solution:
```javascript
function chunk(array, size) {
  const nestedArray = [];
  let i = 0;
  
  while(i < array.length) {
    nestedArray.push(array.slice(i, i + size));
    i += size;
  }
  return nestedArray;
}
```

Alright!  Passing tests!

But we're not done.  I want something _more_, with no for/while loops.
It took me a while to get this working, and I don't think it's as readable as the while loop version above - but it's different, has no loops, and it works.

Here's the alternate solution, once you've had a chance to take it in we can talk about what is happening:
```javascript
function chunk(array, size) {
  const resultArrayLength = Math.ceil(array.length / size);
  return Array.from({ length: resultArrayLength }, (_, i) =>
      array.slice(i * size, i * size + size)
  );
}
```

OK, so first up we are finding out how many elements our nested array will contain.
For example, if you have the following: `Math.ceil([1,2,3,4,5,6,7,8,9].length / 2);` you'll get `5`.
`Math.ceil()` rounds up to the nearest integer.  We are passing in `9/2` which is 4.5 so you can see now why we get 5 back.

The second bit is a little bit less straight forward, we're using `Array.from()` again to create a new array.  The first argument is an "array-like" object with a length property, as you can see we're setting the length to the `resultArrayLength` to give us our outer array.
Next we're passing in a map function to call on the elements of the new array (the new array which has 5 elements).  The map function will add a slice from our original array into our new nested array.
To get the slice function working we have to do a bit more math, but it makes sense once you actually reason about it.

Element 1 `array.slice(0 * 2, 0 * 2 + 2) => [1,2]`.

Element 2 `array.slice(1 * 2, 1 * 2 + 2) => [3,4]`.

Element 3 `array.slice(2 * 2, 2 * 2 + 2) => [5,6]`.

Element 4 `array.slice(3 * 2, 3 * 2 + 2) => [7,8]`.

Element 5 `array.slice(4 * 2, 4 * 2 + 2) => [9]`.

We don't care about the element itself in our map function, so we just call it `_` to let people know it's a throwaway variable.

Anyways, there are two completely different ways to chunk an array in javascript.

:thumbsup:
