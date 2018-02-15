---
title: "Fizzbuzz"
date: 2018-02-12T08:47:19-05:00
tags: ["javascript", "exercises"]
categories: ["javascript"]
---

Our next exercise is going to be Fizzbuzz.  This is a pretty common exercise that's pretty simple.
Let's get started!

Same like we always do, we'll create a new directory from inside our exercises directory to hold our tests/code and create a couple files - `index.js test.js`.
```bash
mkdir fizzbuzz && cd fizzbuzz/
touch index.js test.js
```

Using your mechanism of choice we'll get the following content into your `test.js` file:
```javascript
const fizzBuzz = require('./index');

test('fizzBuzz function is defined', () => {
  expect(fizzBuzz).toBeDefined();
});

test('Calling fizzbuzz with `3` prints out three statements', () => {
  fizzBuzz(3);

  expect(console.log.mock.calls.length).toEqual(3);
});

test('Calling fizzbuzz with 20 prints out the correct values', () => {
  fizzBuzz(20);

  expect(console.log.mock.calls[0][0]).toEqual(1);
  expect(console.log.mock.calls[1][0]).toEqual(2);
  expect(console.log.mock.calls[2][0]).toEqual('fizz');
  expect(console.log.mock.calls[3][0]).toEqual(4);
  expect(console.log.mock.calls[4][0]).toEqual('buzz');
  expect(console.log.mock.calls[5][0]).toEqual('fizz');
  expect(console.log.mock.calls[6][0]).toEqual(7);
  expect(console.log.mock.calls[7][0]).toEqual(8);
  expect(console.log.mock.calls[8][0]).toEqual('fizz');
  expect(console.log.mock.calls[9][0]).toEqual('buzz');
  expect(console.log.mock.calls[10][0]).toEqual(11);
  expect(console.log.mock.calls[11][0]).toEqual('fizz');
  expect(console.log.mock.calls[12][0]).toEqual(13);
  expect(console.log.mock.calls[13][0]).toEqual(14);
  expect(console.log.mock.calls[14][0]).toEqual('fizzbuzz');
  expect(console.log.mock.calls[15][0]).toEqual(16);
  expect(console.log.mock.calls[16][0]).toEqual(17);
  expect(console.log.mock.calls[17][0]).toEqual('fizz');
  expect(console.log.mock.calls[18][0]).toEqual(19);
  expect(console.log.mock.calls[19][0]).toEqual('buzz');
});

beforeEach(() => {
  jest.spyOn(console, 'log');
});

afterEach(() => {
  console.log.mockRestore();
});
```

Now a little bit of explanation around some new constructs in our test file...

The `beforeEach()`/`afterEach()` functions are used for test setup and teardown. 
Like the name implies, these setup functions are called before/after each test run.
Alternatively, if you only need to run setup/teardown once and not in between each test you should use the `beforeAll() && afterAll()` functions.

The `jest.spyOn()` is used to create a mock that we can use to ensure a function was called N times or that it returns precisely what we expect it to.  
The version of `spyOn()` that we're using in our `beforeEach()` takes two arguments, the object we're spying on as well as the method.
Then in our `afterEach()`, between test runs, we're removing the mock.
We are using our mock to ensure that our calls to `console.log()` occur the expected amount of times, and return the expected content.

Since the tests are setup for our exercise we can start getting the function stubbed out:
```javascript
function fizzBuzz(max) {
}

module.exports = fizzBuzz;
```

Next up we're ready to run our jest tests in watch mode (`jest test.js --watch`) and start getting this thing done.

Here's the game plan:
```javascript
function fizzBuzz(max) {
  //loop through numbers up through max

    //number is divisible by 15 should log fizzbuzz
    //number is divisible by 5 should log buzz
    //number is divisible by 3 should log fizz
    //number doesn't match any of the above criteria should log the number
}
```

Let's take the looping approach first here:
```javascript
function fizzBuzz(max) {      //loop through numbers up through max
  for (let i = 1; i <= max; i++) {
    if (i % 15 === 0)         //number is divisible by 15 should log fizzbuzz
      console.log('fizzbuzz');
    else if (i % 5 === 0)     //number is divisible by 5 should log buzz
      console.log('buzz');
    else if (i % 3 === 0)     //number is divisible by 3 should log fizz
      console.log('fizz');
    else 
      console.log(i);        //number doesn't match any of the above criteria should log the number
  }
}
```

Extremely readable and at this point we should have 3 passing tests.

But we all know that I've been trying to embrace some of the more advanced/modern functions in javascript in these solutions, so let's see if this thing is still easily readable when shoving an `array.map()` into the mix.

```javascript
function fizzBuzz(max) {
  Array.from([...Array(n).keys()], x => x + 1)  
    .map(num => {
      let current = '';
      current = (num % 3 === 0) ? 'fizz' : current;
      current = (num % 5 === 0) ? current += 'buzz' : current;
      current = (current === '') ? num : current;
      console.log(current);
    });
}
```

I'm not a fan of all those if/if else/else blocks in the first solution, so in our second run I just set an empty string up to hold our result for each number and log it out at the end of the "loop", which is happening inside of our `.map()` function.

The line where I actually create the array from `[1...max]` may need some explanation though.  Taking advantage of some of the newer ES language features we're first using `Array.from()` which creates a new array from an array like or iterable object.
For example, if you had an array that looks like this: `[10,20]` and you were to call `Array.from([10,20], x => x*x))`, you would get a new array with the contents of `[100,400]`.  The second argument is the map function that gets applied to each element in the original array.  In our case, we're taking the number and adding 1 to it in our map function, since if we didn't our array would begin at zero, and fizzBuzz pity's the fool who starts at zero.  

But where is the "array like object" that we're passing into the `Array.from()` command come from?
Well, we're using two more new-ish features of ES - the spread operator, and the `array.keys()` function.
We're using the spread operator to create an array (`[...Array(3).keys()]`), that will give us an array of `[0,1,2]`, now we have many ways to take that array in increment each element by 1 to use in our fizzBuzz loop.  The `.keys()` bit is returning the keys for each index in the array. Meaning for our 3 element array we have `myArray[0]`, `myArray[1]`, `myArray[2]` - which is what brings us to the need to increment each element in our newly created array.
My two ideas were that we could either just call `[...Array(3).keys()].map(x => x+1)`, or we could use the `Array.from()` syntax from above.

Either way we go it's doing the same thing so pick your own path...

After checking the output of our continually running jest test suite we should see 3 passing tests and we are golden.

I'm not sure which is the "better" solution of those two, it's a fine line trying to find the most readable code that doesn't make you feel dirty and I feel pretty good about the second option. 

:thumbsup:
