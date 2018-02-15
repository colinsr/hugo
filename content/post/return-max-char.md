---
title: "Return Max Char"
date: 2018-02-10T07:33:48-05:00
tags: ["javascript", "exercises"]
categories: ["javascript"]
---

Today we're going to be writing a function that will return the most frequent character that appears in a string.
Given the string: "abcdd" we would get back "d" because that character appears twice whereas the other characters only appear once.

This function would work the same when working with numbers, which will be coming in as a string anyways.

We can create our new directory inside of our exercises directory as well as our two files, `index.js` and `test.js` by running the following command from our `exercises/` dir.

```bash
mkdir maxchar && cd maxchar/ && touch index.js test.js
```

We'll dump the following into our test file:
```javascript
const maxChar = require('./index');

test('maxChar function exists', () => {
  expect(typeof maxChar).toEqual('function');
});

test('Finds the most frequently used character', () => {
  expect(maxChar('c')).toEqual('c');
  expect(maxChar('abcdddefghiddd')).toEqual('d');
});

test('Works with numbers in the string', () => {
  expect(maxChar('testing11is11not11optional111')).toEqual('1');
});
```

Once we get our `index.js` file stubbed out we should be up and running:
```javascript
function maxChar(str) {
}

module.exports = maxChar;
```

Now we can run our test suite (in watch mode again) by running `jest test.js --watch` from inside our `machar/` directory.  We should see our first test pass since we do have a function defined, but our remaining two test fail as we have yet to implement them.

After thinking about this for a moment, I came up with the following approach:
```javascript
function maxChar(str){
    //turn string into array
    //create a json object to use as a character map
    //{ a: 1, b: 2 ... }

    //return the key of the object with the highest count
}
```

We know we can call `str.split('')` to get an array object from our string, but in order to create our character map object we're going to use javascript's "Falsy" in incrementing our map.

If we create an object, `let foo = { a: 1}` and then `foo['a'] = foo['a'] + 1`, now `foo['a']` is 2.
On the other hand, if we need to create a key on `foo` for letter b, it would a predicate checking if b exists and incrementing if it does or creating b with a value of 1 if it doesn't.
But since incrementing `foo['b']` is currently undefined, incrementing undefined will return NaN.  Both of which are Falsy values.

Here is what I came up with to handle the conversion to an array of chars as well as creating our char map object.
```javascript
const charMap = str
    .split('')
    .reduce((charMap, c) => {
        charMap[c] = charMap[c] + 1 || 1;
        return charMap;
    }, {});
```
You can notice the interaction with falsiness in the `charMap[c] = fasly || truthy` bit.
That line right there will default to the value 1 if the increment returns NaN.

Now the only thing left is to return the character that has the highest count in the char map object.
It sounds to me like a separate function, so we'll update our module.exports to include both functions then we'll update our tests file to import the module and call through to the functions.

`index.js`:
```javascript
const findMaxChar = (charMap) => {
    //todo: return most frequent char
}

const maxChar = (str) => {
    const charMap = str
        .split('')
        .reduce((charMap, c) => {
            charMap[c] = charMap[c] + 1 || 1;
            return charMap;
        }, {});

    return findMaxChar(charMap)
}

module.exports = {
    maxChar,
    findMaxChar
};
```

`test.js`
```javascript
const maxCharChecker = require('./index');

test('maxChar function exists', () => {
    expect(typeof maxCharChecker.maxChar).toEqual('function');
});

test('Finds the most frequently used char', () => {
    expect(maxCharChecker.maxChar('a')).toEqual('a');
    expect(maxCharChecker.maxChar('abcdefghijklmnaaaaa')).toEqual('a');
});

test('Works with numbers in the string', () => {
    expect(maxCharChecker.maxChar('ab1c1d1e1f1g1')).toEqual('1');
});

test('findMaxChar function exists', () => {
    expect(typeof maxCharChecker.findMaxChar).toEqual('function');
});

test('Works with numbers in the string', () => {
    expect(maxCharChecker.findMaxChar({ a: 1, b: 2, '1': 5 })).toEqual('1');
});
```

Now we just need to iterate through the keys on the object and return the key that has the highest count.
```javascript
const findMaxChar = (charMap) => {
    return Object.keys(charMap)
        .reduce((result, key) => {
            if (charMap[key] > result.count) {
                result.char = key;
                result.count = charMap[key];
            }
            return result;
        }, { count: 0, char: '' }).char;
}
```

I've been trying to use the `array.reduce()` function for things like this, and in order for this to work as expected we need to pass in a result accumulator object to store our biggest key/value set.
Then once we've run through all the keys we'll just return the character with the highest count.

With that code in place we should be smooth sailing with all 5 of our tests passing. 
:thumbsup: