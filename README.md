# Object iterator methods

A TC39 proposal to add iterator-like methods to objects

## Status

Authors: Benjamin Koltes

Champions: Benjamin Koltes

This proposal is at Stage 0 of [The TC39 Process](https://tc39.es/process-document/).

## Motivation

Currently in ECMAScript, you can use `for..of` with `Array`s, `Set`s, `Map`s, `Iterator`s, but not `Object`s.
Also, `Array`s (and soon `Iterator`s, see [proposal](https://github.com/tc39/proposal-iterator-helpers)) have `.forEach()`, `.map()`, `.filter()` methods.

Iterating over & manipulating `Object`s is also a bit tedious at the moment in ECMAScript. We also need to use `Object.keys(obj)`, `Object.values(obj)`, or `Object.entries(obj)`.
And if we want to create a new object from an previous one with the same keys, only a modification of its values, we need to write:
```js
Object.fromEntries(Object.entries(object).map(([key, value]) => [key, transformation(value)])
```

Those could be made simpler.

## Proposal

This proposal comes in 2 new additions:
- making objects iterable,
- adding new methods to the object prototype:
  - `.forEach()`
  - `.filter()`
  - `.map()`
  - ... TO FILL
 
> [!Note]
> Those 2 could be split in 2 different proposals

### Iterable

To follow `Map`’s iterator (see [MDN](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Map/@@iterator)), `Object`s could behave in a similar manner:

```js
for (const entry of object) {
  const key = entry[0];
  const value = entry[1];
}
```

With:
- `entry` behind the same thing as what `Object.entries(object)` returns: an array of 2 elements with the key & the value,
- the order of iteration respecting the existing order in object,
- just like `Object.entries(…)`, `for .. of` would wollow the own keys of objects and not follow the prototype

### `.forEach()`

To mimic iterators & arrays, `Object.prototype.forEach()` can be added:

```js
forEach(callbackFn)
forEach(callbackFn, thisArg)
```

#### Signature

**Parameters:**
- `callbackFn`\
  A function to execute for each element in the array. Its return value is discarded. The function is called with the following arguments:
  - `value`\
    The current value of the entry being processed in the object.
  - `key`\
    The key of the current entry being processed in the object.
  - `object`\
     The object `forEach()` was called upon.
- `thisArg` (Optional)\
  A value to use as `this` when executing `callbackFn`

**Return value:** None (`undefined`).

#### Example

```js
const object = { foo: 1, bar: '2' };

object.forEach((value, key, reference) => {
  console.log(value, key, reference);
})
// Would log:
// 1   'foo' { foo: 1, bar: '2' }
// '2' 'bar' { foo: 1, bar: '2' }
```

### `.map()`

To mimic iterators & arrays, `Object.prototype.map()` can be added:

```js
map(callbackFn)
map(callbackFn, thisArg)
```

#### Signature

**Parameters:**
- `callbackFn`\
  A function to execute for each element in the object. Its return value is used as the value of a new single entry in the new object. The function is called with the following arguments:
  - `value`\
    The current value of the entry being processed in the object.
  - `key`\
    The key of the current entry being processed in the object.
  - `object`\
     The object `forEach()` was called upon.
- `thisArg` (Optional)\
  A value to use as `this` when executing `callbackFn`

**Return value:** A new object with each entry being the result of the callback function assigned to their original keys.

#### Example

```js
const object = { foo: 1, bar: 2 };

const newObject = object.map((value, key, reference) => {
  console.log(value, key, reference);
  return value * 3
})
// Would log:
// 1 'foo' { foo: 1, bar: '2' }
// 2 'bar' { foo: 1, bar: '2' }

console.log(newObject)
// { foo: 3, bar: 6 }
```

### `.filter()`

To mimic iterators & arrays, `Object.prototype.filter()` can be added:

```js
filter(callbackFn)
filter(callbackFn, thisArg)
```

#### Signature

**Parameters:**
- `callbackFn`\
  A function to execute for each element in the object. It should return a [truthy](https://developer.mozilla.org/en-US/docs/Glossary/Truthy) value to keep the element in the resulting object, and a [falsy](https://developer.mozilla.org/en-US/docs/Glossary/Falsy) value otherwise. The function is called with the following arguments:
  - `value`\
    The current value of the entry being processed in the object.
  - `key`\
    The key of the current entry being processed in the object.
  - `object`\
     The object `forEach()` was called upon.
- `thisArg` (Optional)\
  A value to use as `this` when executing `callbackFn`

**Return value:** A [shallow copy](https://developer.mozilla.org/en-US/docs/Glossary/Shallow_copy) of the given object containing just the entries that pass the test. If no elements pass the test, an empty object is returned.

#### Example

```js
const object = { foo: 1, bar: '2' };

const newObject = object.filter((value, key, reference) => {
  console.log(value, key, reference);
  return typeof value === 'number'
})
// Would log:
// 1   'foo' { foo: 1, bar: '2' }
// '2' 'bar' { foo: 1, bar: '2' }

console.log(newObject)
// { foo: 1 }
```

## Open questions

### Can objects be considered as iterables without having an effect on the spread operator?

We still want `[...object]` to throw, and `{...object}` to create a shallow copy without prototype of `object`

### Why this signature for `.forEach()` (and other)?

When adding `Object.prototype.forEach`, it could behave in 2 ways:
- either as a shorthand for `Object.entries(object).forEach()` with `object.forEach((entry, index, objectReference) => {  })`,
- or similarly to `Array.prototype.forEach((value, key, objectReference) => {  })`

I’d advise to follow the `Array` behavior for 3 reasons:
1. a bit more practical as most of the time, people want to use the `value`, and you can still use the `key`
2. why would anyone need the `index`? Also `index` doesn’t make sense for objects
3. if it were a shorthand for `Object.entries(object).forEach()`, the last argument should be the output of `Object.entries(object)`, so an array of entries, and not the `objectReference` itself. So even in the best scenario, it wouldn’t be a drop-in replacement for `Object.entries(object).forEach()`

### Should `.map()`, `.filter()`, and others re-use the original prototype or strat from 0?

In [`Array.prototype.filter()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/filter), it is mentioned that the returned array is a shallow copy. And in arrays again, new arrays are already created. And the same happens when using `Object.fromEntries(Object.entries(object))`

So I think those return new objects that don’t extend from the original prototype.

### What about collisions with other libraries?

In ECMAScript, except for the spread operator mentioned earlier, those shouldn’t collide with anything in the current specification. But in the past, APIs were picked to avoid collisions with older libraries mutating the global polyfills. For example `Array.prototype.flatten()` was renamed `.flat()` to avoid collisions with MooTools, see https://developer.chrome.com/blog/smooshgate.
