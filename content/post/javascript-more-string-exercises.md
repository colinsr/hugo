---
title: "Javascript More String Exercises"
date: 2018-02-07T08:58:34-05:00
---

Next up on our list of exercises is going to be two more simple functions.

First we'll walk through how to implement a palindrome checker, and then we'll cover reversing integers.

For the palindrome function, we'll want to return true/false depending on whether or not a given string is a palindrome.  For those of you who are not familiar with the term here is the definition:

`a word, phrase, or sequence that reads the same backward as forward`

We're going to want to return true when we pass in the string "abba", and false when we pass in "javascript".

Again, this is pretty straight forward.
First we'll walk through the process of getting this thing wired up with some tests.

Similar to our last setup, we'll create a new directory inside our exercises directory with `mkdir palindrome && cd palindrome`.
Then we can create two files, `touch index.js test.js`.

Let's use the terminal to stub out our `test.js`.  We'll use a Here Document to push our sanity check into our file:
```bash
cat <<EOF>> test.js
const palindrome = require('./index');

test('palindrome function exists', () => {
    expect(typeof palindrome).toEqual('function');
});
EOF
```

And we'll follow suit to wire up our palindrome function:
```bash
cat <<EOF>> index.js
function palindrome(str) {
  //TODO: implementation here
}

module.exports = palindrome;
EOF
```

Checking the contents of our files is easy with the cat command:
`cat index.js` and `cat test.js`.

Now last time when we setup jest, we did so at the level of our `reversestring/` directory, I don't really feel like configuring this inside of each subdirectory of our `exercises/` directory so we'll create a package.json file inside `exercises/package.json`.
We can accomplish this by running `npm init` and accepting all the defaults.

Then we'll verify that our tests are running as expected with the command `jest test.js` from inside our palindrome directory.
At this point we should see that 1 test passed, and with that we're ready to start getting this function doing some work.

We know we need to add some more tests before we get down to business.
This time around, let's tell jest to watch for changes to our file.
We can do that by running `jest test.js --watch`.

After running that command our tests will auto run anytime we update our files.  We currently have only the test to ensure our function is defined, so next up we'll add a few tests to verify our function is doing what it is supposed to do.

We can either add the additional tests via vim, your favorite text editor or just right here from the terminal:
```bash
echo "test('"mom" is a palindrome', () => {
    expect(palindrome('bba')).toBeFalsy();
    expect(palindrome('mom')).toBeTruthy();
});

test('"racecar" is a palindrome', () => {
    expect(palindrome('racecar')).toBeTruthy();
});

test('"bba" is not a palindrome', () => {
    expect(palindrome('bba')).toBeFalsy();
});

test('" dad" is not a palindrome', () => {
    expect(palindrome(' dad')).toBeFalsy();
});" >> test.js
```

This command will append our new tests to our `test.js` file.  Upon updating the test file, we should see some activity in the terminal where we ran the jest suite with the watch flag set.

3 passing and 2 failing tests.  Well, wait a minute.  How could I have 3 passing tests before I even write any code in my palindrome function?
Jest is checking the output of our function for "truthiness", if we get undefined back from our function jest is handling that as if it was a "falsy" response and therefore considering the tests to be green.

That being said, we can finally now start on our solution.
This is again a very simple exercise, so let's talk about what we need to do to get this thing fleshed out.

We have a string, "beer" and we'd like to see if the string is the same forwards as it is backwards.  With our "beer" example, our check would be to see if:
`"beer" === "reeb" //returns false in this case`

Now if we were talking about teen music sensations and screwed up the spelling, `"beeb" === "beeb" //returns true - is it too late to say I'm sorry now?` - 10 points if you're with me there.

Back to work now, we are talking about reversing a string and checking whether or not the two strings are the same.  We'll use our previous experience reverse strings and fly right through this one.
Adding the following to our `index.js` will get us where we need to be:
```javascript
function palindrome(str) {
    const reverse = str.split('').reduce((reverse, char) => char + reverse);
    return (str == reverse) ? true: false;
}
```

The jury is still out on how to handle upper/lower case characters, but it's as easy as calling `.toLowerCase()` on each of our strings inside our ternary operator to perform a case invariant check.  (If you _do_ want to add this case to your function, be sure to add an additional test case to cover it)

With that out of the way we can get started with our reverse int exercise.  From our current `exercises/palindrome/` directory we need to go up one level, then create a new directory called `reverseint/`.

We've got our jest tests configured at the exercises directory now so we're good to just add our `index.js` and `test.js` files inside our new `reverseint/` directory.

What we're looking for here is basically when given a number, 42 we reverse it and return 24, when given 80 return 8, and when given -15 return -51.

First we'll wire up our test file with the content:
```javascript
const reverseInt = require('./index');

test('ReverseInt function exists', () => {
  expect(reverseInt).toBeDefined();
});
```
And our index:
```javascript
function reverseInt(n) {
    //TODO: reverse the number
}

module.exports = reverseInt;
```

This exercise falls back into the easy bucket.  In order to reverse the number we need to turn it into a string, we can do that with the `.toString()` function.  Then we implement our reduce function to reverse it and we'll be good to go.  

Or will we?

We'll add some tests to find out.
```javascript
test('ReverseInt handles 0 as an input', () => {
  expect(reverseInt(0)).toEqual(0);
});

test('ReverseInt flips a positive number', () => {
  expect(reverseInt(7)).toEqual(7);
  expect(reverseInt(17)).toEqual(71);
  expect(reverseInt(70)).toEqual(7);
  expect(reverseInt(1783)).toEqual(3871);
});

test('ReverseInt flips a negative number', () => {
  expect(reverseInt(-7)).toEqual(-7);
  expect(reverseInt(-17)).toEqual(-71);
  expect(reverseInt(-70)).toEqual(-7);
  expect(reverseInt(-1783)).toEqual(-3871);
});
```

And with our tests defined we can run jest in watch mode `jest test.js --watch` from inside of our `reverseint/` directory which will give us 3 failing tests.

Next up is our implementation.
```javascript
function reverseInt(n) {
    //convert n to string
    const numberAsString = n.toString();
    //reverse the string
    const reversed = numberAsString
      .split('')
      .reduce((rvs, char) =>  char + rvs);
    //compare n to the reversed string
    return reversed;
}
```

Well, that didn't go quite according to plan.  We've got failing tests like crazy.  You'll notice from your test output that we are still returning the `reversed` as a string but we're expecting a number.

To handle this, javascript has a built in function called `parseInt()` - let's get that function into our code.

Instead of returning `reversed` we'll return `parseInt(reversed)`.  And... improvement, but not yet done.  We're passing on the positive and zero tests, but the negative case is still failing.

There are a few different ways to handle this, but the option I chose is pretty simple.
```javascript
function reverseInt(n) {
    const numberAsString = n.toString();
    const reversed = numberAsString
      .split('')
      .reduce((rvs, char) =>  char + rvs);
    const reversedInt = parseInt(reversed);
    return (n >= 0) ? reversedInt : -Math.abs(reversedInt);
}
```

Viola, we are passing on all cylinders.
Now we can circle back and cleanup some of this code.
```javascript
const reverse = parseInt(n.toString()
    .split('')
    .reduce((rvs, char) =>  char + rvs));
return (n >= 0) ? reverse : -Math.abs(reverse);
```

After verifying that our tests all still pass we can call this a wrap - :thumbsup: