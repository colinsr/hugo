---
title: "Javascript More String Exercises"
date: 2018-02-07T08:58:34-05:00
draft: true
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

Now last time when we setup jest, we did so at the level of our `reversestring/` directory, I don't really feel like configuring this inside of each subdirectory of our `exercises/` directory so we'll add a global jest configuration inside `exercises/package.json`.
From inside our palindrome directory we can do this with the following command:
```bash
cat <<EOF>> ../package.json
{
  "name": "exercises",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "author": "",
  "license": "ISC"
}
EOF
```

Then we'll verify that our tests are running as expected with the command `jest test.js` from inside our palindrome directory.
At this point we should see that 1 test passed, and with that we're ready to start getting this function doing some work.

We know we need to add some more tests before we get down to business.
This time around, let's tell jest to watch for changes to our file.
We can do that by running `jest test.js --watch`.
