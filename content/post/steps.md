---
title: "Steps"
date: 2018-02-25T17:35:40-05:00
tags: ["javascript", "exercises"]
categories: ["javascript"]
---

Ok, next up on our list of exercises is going to be to create a `steps(n)` function.

This function will print out to the console a number of log statements like so:
```javascript
steps(3) => 
'#  '
'## '
'###'
```

Meaning the base will print N "#" signs, and any steps above will have a single space to the right for each step up when given a positive number.  

Following the same approach, we'll navigate to our `exercises/` directory and create a new directory called `steps/`, then we'll jump into the directory and create our `index.js` and our `test.js` files.

After that we can get our tests running in watch mode, here is the test suite we'll be working with:
``` javascript
const steps = require('./index');

beforeEach(() => {
  jest.spyOn(console, 'log');
});

afterEach(() => {
  console.log.mockRestore();
});

test('steps function exists', () => {
  expect(typeof steps).toEqual('function');
});

test('steps called with n = 1', () => {
  steps(1);
  expect(console.log.mock.calls[0][0]).toEqual('#');
  expect(console.log.mock.calls.length).toEqual(1);
});

test('steps called with n = 2', () => {
  steps(2);
  expect(console.log.mock.calls[0][0]).toEqual('# ');
  expect(console.log.mock.calls[1][0]).toEqual('##');
  expect(console.log.mock.calls.length).toEqual(2);
});

test('steps called with n = 5', () => {
  steps(4);
  expect(console.log.mock.calls[0][0]).toEqual('#    ');
  expect(console.log.mock.calls[1][0]).toEqual('##   ');
  expect(console.log.mock.calls[2][0]).toEqual('###  ');
  expect(console.log.mock.calls[3][0]).toEqual('#### ');
  expect(console.log.mock.calls[4][0]).toEqual('#####');
  expect(console.log.mock.calls.length).toEqual(5);
});
```
Making sure to execute `jest test.js --watch` to get the tests rolling.

Then we can add our function declaration to our `index.js` and export it to get our first green test.
```javascript
const steps = n => {

}

module.exports = steps;
```

I know that most people will look at this function and think about the return values as a sort of matrix.
In this view each line that gets logged out to the console would be N char long, and we would have N rows to print out.
What I came up with were two alternatives to that approach - a map function and a recursive function.

I'll cover the map version first since the recursive function takes a bit more effort to explain clearly.

We have this number, N, that's getting passed into our `steps(n)` function.  The way I wanted to solve this exercise was to build up an array from 1 to N, and then call map on the array - printing out to the console during each execution within the map.

```javascript
const steps = n => {
  //build array from 1..N
  //map through the array and print out to the console, doing some math to calculate # vs spaces
}
```

Sounds pretty easy and straightforward in my opinion, so let's get started.

We've already gone through a couple handy ways to create arrays from a number.
```javascript 
[...Array(n).keys()]  // return array from 0 to N-1
//and
Array.from(Array(n).keys(), x => x+1)  // return an array from 1 to N
```

I'm going to go with option B. here as I'd rather be working with the actual number instead of having to add one to each element of the array before calculating my '#' and spaces.

That's brings our `index.js` to look roughly like:
```javascript
const steps = n => {
  //build array from 1..N
  Array.from(Array(n).keys(), x => x + 1)  // we have an array of 1..N
  //use map to print out to the console, doing some math to calculate # vs spaces
}
```

Now we just need to come up with the logic of how many hashtags/spaces to print out in each `console.log()` statement.
The way I came up with this is that if we wanted to print 10 steps, our top step would consist of 1 hashtag followed by 9 spaces.
Second step would be 2 hashtags and 8 spaces.  The math starts staring you in the face if you work through a few of these steps before just hacking away.

```javascript
const steps = n => {
  
  Array.from(Array(n).keys(), x => x + 1)  // we have an array of 1..N
    //execute this console.log on each element of the array, doing some math to calculate #'s vs spaces
    .map(num => console.log(`${'#'.repeat(num)}${' '.repeat(n - num)}`));
}
```

Bam, check our jest output and we get... WTH.. we still have a test failing.  See if you can track down the issue.

Once you've had a chance to correct the issue with my bad test we can move on to the recursive approach.

With a recursive function the first thing we should do is identify the "done" condition.  In this case we want to complete the function when we've printed N lines, so first we'll add our escape hatch.  In order to know when it's safe to bail out we'll need to keep track of how many lines we've printed out already, so we'll add a second parameter to our function definition with a default of 0.  Here is what that'll look like:
```javascript
const steps = (n, linesPrinted = 0) => {
  if (linesPrinted === n) return; // this will fire off once we've printed N lines
} 
```

Alright, so now we're ready to start actually implementing this recursive function.
In order to do so we're going to need a way to pass the state from one call to the next - this is known as an accumulator.
To get this working we're going to have to add _another_ parameter to our function...
```javascript
const steps = (n, linesPrinted = 0, currentLine = "") => {
  if (linesPrinted === n) return; // this will fire off once we've printed N lines
} 
```
We'll initialize `currentLine` to an empty string and then we'll use that to build up our output for the current iteration.
We can also use the length of the `currentLine` variable to see when it's time for us to log out to the console and jump to our next iteration.
```javascript
const steps = (n, linesPrinted = 0, currentLine = "") => {
  if (linesPrinted === n) return; // this will fire off once we've printed N lines

  if (currentLine.length === n) { // we've added our N characters to the currentLine
    console.log(currentLine);     // take care of business here
    return steps(n, linesPrinted + 1);   // increment linesPrinted and call steps() again!
  }
} 
```

Score, we now have a way to print out the content of `currentLine`, all that's left to do is populate it.
We are basically adding one char per execution of our function, so we'll use `linesPrinted` along with the length of `currentLine` to figure out which character to add to `currentLine`.
```javascript
const steps = (n, linesPrinted = 0, currentLine = "") => {
  if (linesPrinted === n) return; // this will fire off once we've printed N lines

  if (currentLine.length === n) { // we've added our N characters to the currentLine
    console.log(currentLine);     // take care of business here
    return steps(n, linesPrinted + 1);   // increment linesPrinted and call steps() again!
  }
  const currentChar = currentLine.length <= linesPrinted ? '#' : ' ';
  // if currentLine.length is 3 and we've printed 4 lines use a '#'
  // if currentLine.length is 3 and we've printed 2 lines use a ' ' 
  return steps(n, linesPrinted, currentLine += currentChar);
  // make our recursive call, passing in the original N value
  // the same number of lines printed - we didn't do a console.log if we made it here
  // and add the current character to our currentLine before passing it through
} 
```

You may want to step through this with the debugger if it doesn't make sense.  Recursion is hard.
Getting in the recursive mindset is hard.

I think in this particular case, I choose option A.  The array.map function once again proves to be easier to read in my opinion.

Anyhow, if you made it through all the way - great job!  Keep it up!!
:thumbsup:
 