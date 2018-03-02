---
title: "Pyramid"
date: 2018-02-28T08:21:12-05:00
tags: ["javascript", "exercises"]
categories: ["javascript"]
---

Building on our last exercise this time we're going to be building a pyramid.
Given a positive number print that number of lines printing out a pyramid shape:
```javascript
pyramid(3) => 
'  #  '
' ### '
'#####'
```

You should be able to see the similarities in the output - but the program will have to do a little more work to figure out if it needs to print a space or hashtag.
First things first, we'll get our structure in place, navigate to the `exercises/` directory and `mkdir pyramid && cd pyramid && touch index.js test.js`.

Then we can get our tests up and running by adding the following to the `test.js` file:
```javascript
const pyramid = require('./index');

beforeEach(() => {
  jest.spyOn(console, 'log');
});

afterEach(() => {
  console.log.mockRestore();
});

test('pyramid function exists', () => {
  expect(typeof pyramid).toEqual('function');
});

test('prints a pryamid for n = 2', () => {
  pyramid(2);
  expect(console.log.mock.calls[0][0]).toEqual(' # ');
  expect(console.log.mock.calls[1][0]).toEqual('###');
  expect(console.log.mock.calls.length).toEqual(2);
});

test('prints a pryamid for n = 3', () => {
  pyramid(3);
  expect(console.log.mock.calls[0][0]).toEqual('  #  ');
  expect(console.log.mock.calls[1][0]).toEqual(' ### ');
  expect(console.log.mock.calls[2][0]).toEqual('#####');
  expect(console.log.mock.calls.length).toEqual(3);
});

test('prints a pryamid for n = 4', () => {
  pyramid(4);
  expect(console.log.mock.calls[0][0]).toEqual('   #   ');
  expect(console.log.mock.calls[1][0]).toEqual('  ###  ');
  expect(console.log.mock.calls[2][0]).toEqual(' ##### ');
  expect(console.log.mock.calls[3][0]).toEqual('#######');
  expect(console.log.mock.calls.length).toEqual(4);
});
```

Start up our tests by running `jest test.js --watch`, which will tell us we're failing everything right now.
Quick fix to get one of our tests is to wire up our `index.js` file:
```javascript
const pyramid = n => {

}

module.exports = pyramid;
```

Great, that gets a quarter of our tests green.  Now to work on the implementation of our algorithm. 
I suppose on the first pass we'll take the iterative approach using a couple of for loops.
```javascript
const pyramid = n => {
  //for loop 1 to n level of pyramid
    //let currentLine = ''
    //for loop to print 1 to how many chars we need per level
      //figure out which char to append to currentLine => '#' or ' '
    //print to console
}
```

The first todo is pretty easy:
```javascript
const pyramid = n => {
  //for loop 1 to n level of pyramid
  for (let curLine = 0; curLine < n; curLine++) {
    let currentLine = '';
    //for loop to print 1 to how many chars we need per level
      //figure out which char to append to currentLine => '#' or ' '
    //print to console
  }
}
```

Now we have a problem. We don't know exactly how many characters we need to print in each line.
In the steps exercise it was `n` rows of `n` characters, but here we'll need to calculate how many characters we need for a given number n.  Time to look at some examples and coax out the answer.
```javascript
pyramid(2) => 
' # '
'###'
pyramid(3) => 
'  #  '
' ### '
'#####'
pyramid(4) => 
'   #   '
'  ###  '
' ##### '
'#######'
```
We have `n` lines and in order to get that pyramid shape we're looking for an odd number -- every time.
The number of chars appears to be `2n-1`, meaning 2 => 3, 3 => 5, 4 => 7, which should be enough to get rolling on this next part.
```javascript
const pyramid = n => {
  const charsPerLine = (2*n)-1;
  //for loop 1 to n level of pyramid
  for (let curLine = 0; curLine < n; curLine++) {
    let currentLine = '';
    //for loop to print 1 to how many chars we need per level
    for (let curChar = 0; curChar < charsPerLine; curChar++) {}
      //figure out which char to append to currentLine => '#' or ' '
    }
    //print to console
  }
}
```

When we go to append the `currentLine` string with the character we're going to need to check two things.
But first things first, let's calculate the middle character for each line/layer in the pyramid.
Using our previous examples of `n = 2`, `n = 3`, `n = 4`, we learned that we double and subtract one to get the number of characters per line.  That leaves us with `3`, `5`, and `7` characters respectively.
To get the middle characters index then we simply need to divide by 2, and round down to the nearest whole number.
```javascript
const dividedByTwo = 5/2; //2.5
const middleIndex  = Math.floor(dividedByTwo); // 2 (remember arrays start at 0)
```
Alright, now we can start working on when to print a space versus a hashtag.

Walking through a 3 line scenario, we'll be looping through the chars to print inside of a single line.

Line 1, `curLine` will be 0.  We step into our inner loop, and `curChar` will be 0.
We know in line 1, we only want the middle character (of 5) to be a hashtag.  We're looking to limit printing a hashtag to index 2 (third char).<br/>
If the `midIndex - curLine <= curChar` => `2 - 0 <= 0` that will get us a false because 2 is not less than or equal to 0.<br/>
If the `midIndex + curLine >= curChar` => `2 + 0 >= 0` this is true, but we want to apply both of these rules to keep the '#' printing inside a range of indexes.<br/>
Moving on to `curChar = 1`, will get us `if (2 - 0 <= 1) && (2 + 0 >= 1)` false so we print a space.<br/>
When `curChar = 2`, will get us `if (2 - 0 <= 2) && (2 + 0 >= 2)` which gets us to true and we print a hashtag.<br/>
Next is `curChar = 3`, will get us `if (2 - 0 <= 3) && (2 + 0 >= 3)` back to false and we print a space.<br/>
The final character in our first line is `curChar = 4`, will get us `if (2 - 0 <= 4) && (2 + 0 >= 4)` false again print another space.

Line 2, `curLine` will be 1 and we're looking to print a leading and trailing space, with 3 hashtags sandwiched in the middle.<br/>
`curChar = 0` => `if (2 - 1 <= 0 && 2 + 1 >= 0) //false`<br/>
`curChar = 1` => `if (2 - 1 <= 1 && 2 + 1 >= 1) //true`<br/>
`curChar = 2` => `if (2 - 1 <= 2 && 2 + 1 >= 2) //true`<br/>
`curChar = 3` => `if (2 - 1 <= 3 && 2 + 1 >= 3) //true`<br/>
`curChar = 4` => `if (2 - 1 <= 4 && 2 + 1 >= 4) //false`<br/>

Line 3, `curLine` is now 2 and we want to print all hashtags.<br/>
`curChar = 0` => `if (2 - 2 <= 0 && 2 + 2 >= 0) //true`<br/> 
`curChar = 1` => `if (2 - 2 <= 1 && 2 + 2 >= 1) //true`<br/>
`curChar = 2` => `if (2 - 2 <= 2 && 2 + 2 >= 2) //true`<br/>
`curChar = 3` => `if (2 - 2 <= 3 && 2 + 2 >= 3) //true`<br/>
`curChar = 4` => `if (2 - 2 <= 4 && 2 + 2 >= 4) //true`

```javascript
const pyramid = n => {
  const charsPerLine = (2*n)-1,
        midIndex = Math.floor(charsPerLine / 2);

  //for loop 1 to n level of pyramid
  for (let curLine = 0; curLine < n; curLine++) {
    let currentLine = '';
    //for loop to print 1 to how many chars we need per level
    for (let curChar = 0; curChar < charsPerLine; curChar++) {
      //figure out which char to append to currentLine => '#' or ' '
      currentLine += (midIndex - curLine <= curChar 
                  &&  midIndex + curLine >= curChar)
                  ? '#' : ' ';
    }
    console.log(currentLine);
  }
}
```
So there you have a working solution... I cannot stand that logic around which character to print.
Let's come up with a different way of solving this.  We'll skip the recursive solution with this one, it'll end up being pretty close to the previous recursive solution we did to steps.

Let's take a stab at using a similar approach to our map solution.
```javascript
const pyramid = n => {
    Array.from(Array(n).keys(), x => x + 1)
        .map(x => console.log(x));
}
```
That will get us the array we need to work with (1 to `n`).  Now to handle the printing logic.
For starters, we'll stash our characters in some `const` variables.
```javascript
const hash  = '#',
      space = ' ';
```
We know that our middle character will always be a '#' so we can just hard code that into our `console.log()`.
```javascript
const pyramid = n => {
    Array.from(Array(n).keys(), x => x + 1)
        .map(x => console.log(`${TODO}#${TODO}`));
}
```
I'm thinking we create two helper functions.  One to handle printing for left of center, and one to handle right of center.
Let's come up with an example of how to handle the left of center.  We have `n = 3`, so we are working with 3 lines, and 5 characters per line.
We already have the middle character done, so we're essentially looking at what character to print for elements 0 and 1.

I'm swapping out spaces for underscores so that we have a better way to visualize this, as well as adding in a space between the characters:
```javascript
_ _ # _ _
_ # # # _
# # # # #
```
The numbers we have at our disposal are the number of lines to print and the current line number.
Our first round will be 3 lines, currently on line 1.

We'd like to print 2 spaces and 0 hashtags.

For our second pass we'll have 3 lines total, and we'll be on line 2.
We'd like to print 1 space and 1 hashtag.

For our final pass, 3 lines total, on line 3.

We want to print 2 hashtags and 0 spaces.

It's just the inverse for the trailing characters.

Okay, we can start coaxing out the printing logic now.
```javascript
const getLeadingChars  = (lines, lineNumber) => `${space.repeat(lines - lineNum)}${hash.repeat(lineNum - 1)}`;
const getTrailingChars = (lines, lineNumber) => `${hash.repeat(lineNum - 1)}${space.repeat(lines - lineNum)}`;
```
Plug these helpers into our map and let's check the results.
```javascript
const hash  = '#',
      space = ' ';

const getLeadingChars = (lineNum, lines) => `${space.repeat(lines - lineNum)}${hash.repeat(lineNum - 1)}`;
const getTrailingChars = (lineNum, lines) => `${hash.repeat(lineNum - 1)}${space.repeat(lines - lineNum)}`;

const pyramid = n => {
    Array.from(Array(n).keys(), x => x + 1)
        .map(x => console.log(`${getLeadingChars(x,n)}#${getTrailingChars(x,n)}`));
}
```

Nice!  Passing tests, and readable code.  I much prefer this option.

:thumbsup: