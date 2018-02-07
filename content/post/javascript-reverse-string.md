---
title: "Javascript Reverse String"
date: 2018-02-03T10:03:12-05:00
---

In an effort to learn the in's and out's of ES2017 I'm going to be working through some exercises.

The first exercise I'd like to work through is a pretty simple one, reversing a string with javascript.

Basically, given string `'foobar'` I'd like to return `'raboof'` - again, this is pretty simple and I think it would be a great place to start.

To start with, let's go ahead and create a new directory in our terminal with: 
```bash
cd ~/
mkdir exercises
cd exercises/
mkdir reverseString
cd reverseString/
```

Now we'd like to create two files in this new `reverseString` directory, one for our code and one for our tests.  Yes... tests are required if you consider yourself a professional developer so go ahead and get over it.

Next we're going to create an `index.js` file as well as a `test.js` file.
And we're going to use the Jest test framework.  We'll get it installed globally by running:
```bash
npm install -g jest
```

To get Jest setup we just need to add one more file to our `reversestring/` directory, the `package.json`.
The contents of the file will be:
```json
{
  "scripts": {
    "test": "jest"
  }
}
```

We can now go ahead and stub out our reverse function inside our `index.js` file:
```javascript
function reverse(string) {

}

module.exports = reverse;
```

And we can stub out our test class (`test.js`) with the following:
```javascript
const reverse = require('./index');

test('Reverse function exists', () => {
  expect(reverse).toBeDefined();
});
```

This will pull in our `index.js` file and validate that the `reverse` function is exported in the module.

Back in our terminal we can execute the tests by running `jest test.js`, and we'll see that our initial test passes.

Now we can go about tackling the implementation of our test script.  We'll be exploring a few implementation options - starting with a simple solution, cranking it up a bit, and, finally, ending on a more advanced approach.
Our final solution will embrace and demonstrate some of the built in functions of javascript arrays.

We can start the implementation by writing a couple tests to ensure our code actually works.
Appending our `test.js` file with the following two tests should do the trick:
```javascript
test('Reverse reverses a string', () => {
  expect(reverse('foobar')).toEqual('raboof');
});

test('Reverse reverses a string with spaces', () => {
  expect(reverse(' reverse this string! ')).toEqual(' !gnirts siht esrever ');
});
```

It always makes sense to next run our new tests and see them fail before we proceed to make them pass. We can run the tests once by executing `jest test.js` at our terminal.
If everything is configured correctly we should see 2 failed tests and 1 passing test.

Finally we can get down to business.
Since we already know javascript, we know that there is an Array.prototype.reverse() method that will take an array and reverse the order of the elements in the array.

Let's add some comments to our reverse function so that we know what we need to do:
```javascript
function reverse(string){
  //convert string to array

  //reverse the array

  //turn the reversed array into a string

  //return the reversed string
}
```

Our first order of business will be getting the string converted to a javascript array.
We can get this sorted out by calling `string.split('')`.
After we've got our string stored in a variable as an array, we can simply call the reverse method, `stringArray.reverse()`.
Then we need to convert the reversed char array to a string - `reversedStrArray.join('');`.
And finally return the reversed string, `return reversed`.

```javascript
function reverse(string){
  //convert string to array
  let stringArray = string.split('');
  //reverse the array
  let reversedStrArray = stringArray.reverse();
  //turn the reversed array into a string
  let reversed = reversedStrArray.join('');
  //return the reversed string
  return reversed;
}
```

Rerun the tests with the `jest test.js` command at the terminal, and we should now have three passing tests.

This code is far from great, which means next we'll enter into our refactoring step.  We're storing all these temporary variables, and in my opinion, it is a total waste of characters.
We'll go ahead and just chain these functions together to accomplish the same results with less code that's easier to understand.

What that looks like is this:
```javascript
function reverse(string) {
     return string
       .split('')
       .reverse()
       .join('');
}
```

Rerun our tests to ensure we haven't introduced any regressions and we're good to go.

With that out of the way we can implement this without using a built in function.
Here, we're going to be iterating over the chars in the string and prepending each char to the existing reversed string.
We'll start with an empty string for our reversed string, and loop through each letter in the string prepending it to our reversed string one letetr at a time.
We could use our go to `for(var i = 0;i < string.length; i++){ }` syntax as our loop construct, but if we did we'd be missing out on some of the newer ES2015 features available to use in the `for...of` loop.
I like the syntax of this looping construct better, it exposes us to less typos/errors in our loop construction.
What if we were to use a greater than instead of a less than?  Or if we used the wrong variable to grab the length?
There will still be some cases where a trusty old for loop will be necessary, but in this instance we can definitely use the newer approach.

We'll nuke the existing implementation (or comment it out so we can compare our three approaches for later) and again add some comments to plan our attack.
```javascript
function reverse(string){
  //declare an emptry string to hold our reversed string

  //setup our for...of loop
  //inside our loop, we'll shove each character to the front of reversed string variable

  //return the reversed string
}
```

What this ends up looking like is the following function:
```javascript
function reverse(string) {
  let reversed = "";
  for(let char of string){
    reversed = char + reversed;
  }
  return reversed;
}
```

Run our tests once again to make sure we're still doing what we set out to do and then we can move on to a more advanced implementation.

Here we're going to take advantage of the `array.reduce()` method.  We've already gone over how to turn a string into an array using the `string.split('')` function, so we'll use the same approach here to get our string into the correct format to take advantage of reduce.

You may be thinking to yourself, "wait, what the heck is this reduce thing all about?" and I'm hoping I can help illuminate that for your right now.

In layman's terms, reduce takes an array and "reduces" it into a single value.  
You pass an array in, and get a single value out.
Reduce is one of the cornerstones of functional programming and can be used in a wide variety of situations, but for illustrative purposes what works for me is thinking about taking an array of values and summing them up in one fell swoop.
Given the following array `const transactions = [1,2,3,4,5,6,7,8,9,10]` what is the simplest way to sum them up?
A loop?  Hells no.  We are looking at reduce right here.
How does one use reduce to return the sum of these values, you may be asking.
Like so: 
```javascript
const total = transactions.reduce((total, current) => total + current);
```

Now, just becuase reduce returns a single value does not mean that you can't return an array from reduce, you could just as easily write a reduce function that will return all the transactions plus 10% interest as an array by writing:
```javascript
const totalWithTax = transactions.reduce((total, current) => {
  total.push(current * 1.1));
  return total;
}, []);
```

We're ready to implement our own reduce function to return the reverse of a given string:
```javascript
function reverse(string) {
  return string
    .split('')
    .reduce((rvs, char) =>  char + rvs, "");
}
```

The logic should look pretty familiar, with the exception of the call to Array.prototype.reduce().
We just need to run our tests once again to make sure we're still working as expected and once we confirm, we're good 2 go.

It's a little more terse and little more functional than our previous attempts.  I hope you've enjoyed this quick run through reversing a string in javascript. :thumbsup: