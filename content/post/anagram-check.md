---
title: "Anagram Check"
date: 2018-02-20T19:41:08-05:00
tags: ["javascript", "exercises"]
categories: ["javascript"]
---


It's time to work through another (easy) exercise in javascript.
This time we'll be working through an anagram checker function.

Just so that we're all on the same page I've taken the liberty of pulling up an official definition of "anagram":
> a word, phrase, or sentence formed from another by rearranging its letters: “Angel” is an anagram of “glean.”.

Pretty much what we are about to set out to do is see if the letters contained within the two string parameters match.
We don't care about punctuation, spacing, or casing.

We'll setup our folder structure by navigating to our `exercises/` directory and running: 

`mkdir anagram && cd anagram && touch test.js index.js`


Then we'll get our test suite set up and running by adding the following to our `test.js` file:
```javascript
const anagrams = require('./index.js');

test('anagrams function exists', () => {
  expect(typeof anagrams).toEqual('function');
});

test('"hello" is an anagram of "llohe"', () => {
  expect(anagrams('hello', 'llohe')).toBeTruthy();
});

test('"Whoa! Hi!" is an anagram of "Hi! Whoa!"', () => {
  expect(anagrams('Whoa! Hi!', 'Hi! Whoa!')).toBeTruthy();
});

test('"One One" is not an anagram of "Two two two"', () => {
  expect(anagrams('One One', 'Two two two')).toBeFalsy();
});

test('"One one" is not an anagram of "One one c"', () => {
  expect(anagrams('One one', 'One one c')).toBeFalsy();
});

test('"A tree, a life, a bench" is not an anagram of "A tree, a fence, a yard"', () => {
  expect(anagrams('A tree, a life, a bench', 'A tree, a fence, a yard')).toBeFalsy();
});
```

We'll run `jest test.js --watch` to get our tests running in watch mode.

Then we can get our first test passing by adding our function to our module.exports:
```javascript
const anagrams = (stringA, stringB) => {

}

module.exports = anagrams;
```

Now we have a couple tests passing - what gives?
Well, our tests are checking for truthy/falsy-ness and our tests are inadvertantly passing tests are actually checking for falsiness so bear with us as we work through this function.

First approach will be to create a character map object that has a letter and the count for each letter in a given string.
`"bab" => {a: 1, b:2 }`

We can do this by creating a `buildCharMap()` helper.
```javascript
const buildCharMap = (str) => {
  const charMap = {};

  for (let char of str.replace(/[^\w][1-9]+/g, '').toLowerCase()) {
    charMap[char] = charMap[char] + 1 || 1;
  }

  return charMap;
}
```

Looping through each alphabetic character, after stripping out any special characters, punctuation, or numbers, and if the character already exists in the map incrementing it otherwise adding it with the value of 1.

Then of course returning the `charMap` object to the caller.

Once we have our character maps to work with, the first thing we can do is make sure that the `charMap`'s have the same number of keys.
I'm a big believer in the "fail fast" mentality.
```javascript
const anagrams = (stringA, stringB) => {
  const aCharMap = buildCharMap(stringA);
  const bCharMap = buildCharMap(stringB);

  if (Object.keys(aCharMap).length !== Object.keys(bCharMap).length) {
      return false;
  }
  return true;
}
```

This gets us close to having all passing tests.  Now we only have to check to make sure tha values foreach key are the same.
In other words, the count for each letter should match.

```javascript
const anagrams = (stringA, stringB) => {
  const aCharMap = buildCharMap(stringA);
  const bCharMap = buildCharMap(stringB);

  if (Object.keys(aCharMap).length !== Object.keys(bCharMap).length) {
      return false;
  }

  for (let char in aCharMap) {
      if (aCharMap[char] !== bCharMap[char]) {
          return false;
      }
  }
  return true;
}
```

The for loop using the keyword `in` is looping through the keys in our `aCharMap`, then we can check the count for each character and if they're not equal => return false.

Booya.  All of our tests pass, and we have a working solution.

Next up is a proper implementation.

Step 1, I decided to reuse the same regex to filter out any non alphabet characters.
```javascript
const anagrams = (stringA, stringB) => {

}

const cleanString = (str) => {
  return str.replace(/[^\w][1-9]+/g, '');
}
```

Not many people I've run into are regex fans, but this is fairly straight forward, and simply replaces any non-word character or number with an empty string.

This is a slight hack, that prevents us from having to loop through `stringA`, `stringB`, and then the keys in our character map.
```javascript
const cleanString = (str) => {
  return str.replace(/[^\w][1-9]+/g, '')
    .toLowerCase()
    .split('')
    .sort()
    .join('');
}
```
Here we are turning our string into an array, and then using the array prototype's sort method to get our characters in order before turning the array back to a string and returning it.

Finally, we just have to call `cleanString` on each of our string inputs and see if they match.
Viola:
```javascript
const anagrams = (stringA, stringB) => {
  return cleanString(stringA) === cleanString(stringB);
}

const cleanString = (str) => {
  return str
    .replace(/[^\w][1-9]+/g, '')
    .toLowerCase()
    .split('')
    .sort()
    .join('');
}

module.exports = anagrams;
```

As usual, I much prefer option 2.  I hope this was useful!

:thumbsup: