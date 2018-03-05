---
title: "Vowels"
date: 2018-03-02T16:53:52-05:00
tags: ["javascript", "exercises"]
categories: ["javascript"]
---


This exercise will be a walk in the park.
We need to do a case insensitive count of vowel characters in a given string.

In this exercise we'll try to use <span style="color:blue;">**`map()`</span>, <span style="color:blue;">`filter()`**, and **<span style="color:blue;">`reduce()`**</span>.

This is going to be a fun one.  Starting like we always do, go to our `exercises/` directory and create our new directory, `vowels/` and create two files `index.js` and `test.js`.

We'll add some prebaked tests into the `test.js` file:
```javascript
const vowels = require('./index');

test('Vowels exists', () => {
    expect(typeof vowels).toEqual('function');
});

test('returns five when given five lowercase vowels', () => {
    expect(vowels('uioae')).toEqual(5);
});

test('returns five when they are capitalized', () => {
    expect(vowels('EOIAU')).toEqual(5);
});

test('returns five when given the entire alphabet', () => {
    expect(vowels('abcdefghijklmnopqrstuvwxyz')).toEqual(5);
});

test('returns zero when given no vowels', () => {
    expect(vowels('bcdfghjkl')).toEqual(0);
});

test('returns seven when given "Foot Doctor Ombordok', () => {
    expect(vowels('Foot Doctor Ombordok')).toEqual(7);
});
```

This will get our test scaffold in place, now we just need to start it up by running the following from inside your `exercises/vowels/` directory:<br/>
`jest test.js --watch`

Then we'll get our first test to pass by declaring the function in our `index.js` file.
```javascript
const vowels = str => {}

module.exports = vowels;
```

Let's start with the **<span style="color:blue">map()</span>** approach.<br/>
We take in a string, get it in lower case, turn it into an array.<br/>
Then we call a map function that adds 1 to a variable if the letter is a vowel.<br/>
Finally, we'll return the count. We'll get a const string setup holding our five static vowels.
```javascript
const VOWELS = 'aeiou';

const vowels = str => {
    let count = 0;
    
    str.toLowerCase()
      .split('')
      .map(char => VOWELS.indexOf(char) >= 0 ? count++ : 0);
    return count;
}

module.exports = vowels;
```
Tests all coming back green!

Now we can move on to **<span style="color:blue">reduce()</span>**.<br/>
We're going to do the same steps in the beginning to get our casing consistent and massage our string into an array, but that's all the two approaches share.<br/>
Once we're looking at the array, we going to pass in an accumulator which will get incremented in the reduce function if the char is a vowel.<br/>
Making **sure** to pass 0 in for the accumulator's starting value, as well as return it when we're done.
```javascript
const VOWELS = 'aeiou';

const vowels = str => {
    return str
        .toLowerCase()
        .split('')
        .reduce((acc, x) => {
            if (VOWELS.indexOf(x) >= 0) ++acc;
            return acc;
        }, 0);
}
```

I like #2 better than #1 in this case, but I prefer the following option #3 to the others.<br/>
Next up we have the **<span style="color:blue">filter()</span>** function.<br/>

We'll do the same song and dance to get our string to lower case for ease of comparison, and into an array so that we can use the filter function where we can get rid of any non vowel characters.  Lastly we'll get the length of our remaining string and send that count back up.
```javascript
const VOWELS = 'aeiou';

const vowels = str => {
    return str.toLowerCase()
            .split('')
            .filter(x => VOWELS.indexOf(x) >= 0)
            .length;
}
```

Filter was definitely the best suited tool for this job.  Hope you had as much fun with these as I did.

:thumbsup: