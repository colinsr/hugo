---
title: "Anagram Check"
date: 2018-02-20T19:41:08-05:00
draft: true
---

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
  expect(
    anagrams('A tree, a life, a bench', 'A tree, a fence, a yard')
  ).toBeFalsy();
});
```
```javascript
function anagrams(stringA, stringB) {
  return cleanString(stringA) === cleanString(stringB);
}

function cleanString(str) {
  return str
    .replace(/[^\w]/g, '')
    .toLowerCase()
    .split('')
    .sort()
    .join('');
}

module.exports = anagrams;

// function anagrams(stringA, stringB) {
//   const aCharMap = buildCharMap(stringA);
//   const bCharMap = buildCharMap(stringB);
//
//   if (Object.keys(aCharMap).length !== Object.keys(bCharMap).length) {
//     return false;
//   }
//
//   for (let char in aCharMap) {
//     if (aCharMap[char] !== bCharMap[char]) {
//       return false;
//     }
//   }
//
//   return true;
// }
//
// function buildCharMap(str) {
//   const charMap = {};
//
//   for (let char of str.replace(/[^\w]/g, '').toLowerCase()) {
//     charMap[char] = charMap[char] + 1 || 1;
//   }
//
//   return charMap;
// }
```