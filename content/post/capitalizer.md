---
title: "Capitalizer"
date: 2018-02-23T07:29:30-05:00
tags: ["javascript", "exercises"]
categories: ["javascript"]
draft: true
---

This particular exercise should be a breeze after some of the exercises that we've already gone through.

Given a string, capitalize the first letter of each word in that string.

Once again, we have some setup to take care of - creating our directory and files.

`mkdir capitalizer && cd capitalizer/ && touch index.js test.js`

Then get some tests into our `test.js` file:
```javascript
const capitalize = require('./index');

test('Capitalize exists', () => {
  expect(typeof capitalize).toEqual('function');
});

test('capitalizes the first letter of every word in a sentence', () => {
  expect(capitalize('hi colin, how is it going?')).toEqual(
    'Hi Colin, How Is It Going?'
  );
});

test('capitalizes the first letter', () => {
  expect(capitalize('double baco cheeseburger. it\'s for a cop.')).toEqual(
    'Double Baco Cheeseburger. It\'s For A Cop.'
  );
});
```

This content in your `index.js` will get the function defined and one test passing:
``` javascript
const capitalizer = (string) => {}

module.exports = capitalizer;
```

And now we're ready to take the first brute force-ish approach.

Let's get our `index.js` commented up with our plan.
```javascript
const capitalizer = (string) => {
  //add result variable to hold modified string

  //loop through chars

  //if index is greater than zero, check if the character to the left if a space
    //if char to left is a space, uppercase
    //else use original char
  //else use uppercase of string since it's the first letter in the string

  //make sure to return here
}
```

There's our game plan.  Here we go.  Setting up the loop is trivial:
```javascript
const capitalizer = (string) => {
  //add result variable to hold modified string
  let result = '';
  //loop through chars
  for (let i = 0;i < string.length;i++) {
  //if index is greater than zero, check if the character to the left if a space
    //if char to left is a space, uppercase
    //else use original char
  //else use uppercase of string since it's the first letter in the string
  
  }
  //make sure to return here
}
```

Next we'll check what our index, `i`, is:
```javascript
const capitalizer = (string) => {
  let result = '';

  for (let i = 0;i < string.length;i++) {
    if (i > 0) {
    //if char to left is a space, uppercase
      //else use original char
    }
    else {
      //else use uppercase of string since it's the first letter in the string
    }
  }
  //make sure to return here
}
```

Now we can append the result string with our next character.
```javascript
const capitalizer = (string) => {
  let result = '';

  for (let i = 0;i < string.length;i++) {
    if (i > 0) {
      if(string[i-1] === ' ') {
        result += string[i].toUpperCase();
      } else {
        result += string[i];
      }
    }
    else {
      //else use uppercase of string since it's the first letter in the string
    }
  }
  //make sure to return here
}
```

And finally, handle `string[0]`, which we'll always want to uppercase -and return the result.
```javascript
const capitalizer = (string) => {
  //add result variable to hold modified string
  let result = '';

  //loop through chars
  for (let i = 0;i < string.length;i++) {
    //if index is greater than zero, check if the character to the left if a space
    if (i > 0) {
    //if char to left is a space, uppercase
      if(string[i-1] === ' ') {
        result += string[i].toUpperCase();
      } else {
        //else use original char
        result += string[i].toUpperCase();
      }
    }
    else {
      result += string[i];
    }
  }
  //make sure to return here
  return result;
}
```

We should now see our tests are all green, and we can now circle back and clean this up.
If we know that we are always going to uppercase the first letter, let's just take care of that outside of the loop and then start our loop at index 1.
That way we can drop that entire if index zero crap.

Here's where that gets us:
```javascript
const capitalizer = (string) => {
  let result = string[0].toUpperCase();

  for (let i = 1;i < string.length;i++) {
      if(string[i-1] === ' ') {
        result += string[i].toUpperCase();
      } else {
        result += string[i].toUpperCase();
      }
    }
  }
  return result;
}
```

Next we can condense what we have left into this:
```javascript
const capitalizer = (string) => {
    let result = str[0].toUpperCase();

    for (let i = 1; i < str.length; i++)
        result += (str[i - 1] === ' ' ? str[i].toUpperCase() : str[i]);

    return result;
}
```

So there you have one way of solving this.  Now let's try a different way.
```javascript
const capitalizer = (string) => {
    return string.split(' ').map(w => {
        return w.replace(w[0], w[0].toUpperCase());
    }).join(' ');
}
```

Here we are turning the string into an array of words, and then just calling a function, inside our map, that will uppercase the first char of each word.  

Much better.

Much more readable.

And guess what.  We can actually tighten that up a little bit into this:
```javascript
const capitalizer = (string) => {
    return str.split(' ').map(w => w.replace(w[0], w[0].toUpperCase())).join(' ');
}
```

Same code, just a little less unnecessary noise of curly braces and a return keyword.

As usual, I hope you got a little out of this :thumbsup:.