# Object iterator methods

A TC39 proposal to add iterator-like methods to objects

## Status

Authors: Benjamin Koltes

Champions: Benjamin Koltes

This proposal is at Stage 0 of [The TC39 Process](https://tc39.es/process-document/).

## Motivation

Currently in ECMAScript, iterating over & manipulating Objects is a bit tedious. We need to use `Object.keys(obj)`, `Object.values(obj)`, or `Object.entries(obj)` and chain them. For example, if we want to create a new object from an previous one with the same keys, and only transform its values, we need to write:
```js
Object.fromEntries(Object.entries(object).map(([key, value]) => [key, transformation(value)])
```
This is a bit verbose.

Manipulating objects could be made simpler.

## Proposal

This proposal comes in new static methods to the `Object` class:
- `Object.forEach()`
- `Object.filter()`
- `Object.map()`
- `Object.reduce()`
- `Object.some()`
- `Object.every()`

### `Object.forEach()`

To mimic iterators & arrays, `Object.forEach()` can be added:

```js
Object.forEach(obj, callbackFn)
Object.forEach(obj, callbackFn, thisArg)
```

#### Signature

**Parameters:**
- `obj`\
  An object.
- `callbackFn`\
  A function to execute for each entry in the object. Its return value is discarded. The function is called with the following arguments:
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
const obj = { foo: 1, bar: '2' };

Object.forEach(obj, (value, key, reference) => {
  console.log(value, key, reference);
})
// Would log:
// 1   'foo' { foo: 1, bar: '2' }
// '2' 'bar' { foo: 1, bar: '2' }
```

### `Object.map()`

To mimic iterators & arrays, `Object.map()` can be added:

```js
Object.map(obj, callbackFn)
Object.map(obj, callbackFn, thisArg)
```

#### Signature

**Parameters:**
- `obj`\
  An object.
- `callbackFn`\
  A function to execute for each element in the object. Its return value is used as the value of a new single entry in the new object. The function is called with the following arguments:
  - `value`\
    The current value of the entry being processed in the object.
  - `key`\
    The key of the current entry being processed in the object.
  - `object`\
     The object `map()` was called upon.
- `thisArg` (Optional)\
  A value to use as `this` when executing `callbackFn`

**Return value:** A new object with each entry being the result of the callback function assigned to their original keys.

#### Example

```js
const obj = { foo: 1, bar: 2 };

const newObject = Object.map(obj, (value, key, reference) => {
  console.log(value, key, reference);
  return value * 3
})
// Would log:
// 1 'foo' { foo: 1, bar: '2' }
// 2 'bar' { foo: 1, bar: '2' }

console.log(newObject)
// { foo: 3, bar: 6 }
```

### `Object.filter()`

To mimic iterators & arrays, `Object.filter()` can be added:

```js
Object.filter(obj, callbackFn)
Object.filter(obj, callbackFn, thisArg)
```

#### Signature

**Parameters:**
- `obj`\
  An object.
- `callbackFn`\
  A function to execute for each element in the object. It should return a [truthy](https://developer.mozilla.org/en-US/docs/Glossary/Truthy) value to keep the element in the resulting object, and a [falsy](https://developer.mozilla.org/en-US/docs/Glossary/Falsy) value otherwise. The function is called with the following arguments:
  - `value`\
    The current value of the entry being processed in the object.
  - `key`\
    The key of the current entry being processed in the object.
  - `object`\
     The object `filter()` was called upon.
- `thisArg` (Optional)\
  A value to use as `this` when executing `callbackFn`

**Return value:** A [shallow copy](https://developer.mozilla.org/en-US/docs/Glossary/Shallow_copy) of the given object containing just the entries that pass the test. If no elements pass the test, an empty object is returned.

#### Example

```js
const obj = { foo: 1, bar: '2' };

const newObject = Object.filter(obj, (value, key, reference) => {
  console.log(value, key, reference);
  return typeof value === 'number'
})
// Would log:
// 1   'foo' { foo: 1, bar: '2' }
// '2' 'bar' { foo: 1, bar: '2' }

console.log(newObject)
// { foo: 1 }
```

### `.reduce()`

To mimic iterators & arrays, `Object.reduce()` can be added:

```js
Object.reduce(obj, callbackFn)
Object.reduce(obj, callbackFn, initialValue)
```

#### Signature

**Parameters:**
- `obj`\
  An object.
- `callbackFn`\
  A function to execute for each entry in the object. Its return value becomes the value of the `accumulator` parameter on the next invocation of `callbackFn`. For the last invocation, the return value becomes the return value of `reduce()`. The function is called with the following arguments:
  - `accumulator`\
    The value resulting from the previous call to `callbackFn`. On the first call, its value is `initialValue` if the latter is specified; otherwise its value is `Object.values(object)[0]`.
  - `currentValue`\
    The current value of the entry being processed in the object.
  - `currentKey`\
    The key of the current entry being processed in the object.
  - `object`\
     The object `reduce()` was called upon.
- `initialValue` (Optional)\
  A value to which `accumulator` is initialized the first time the callback is called. If `initialValue` is specified, `callbackFn` starts executing with the first value in the object as `currentValue`. If `initialValue` is not specified, `accumulator` is initialized to the first value in the object, and `callbackFn` starts executing with the second value in the object as `currentValue`. In this case, if the object is empty (so that there’s no first value to return as `accumulator`), an error is thrown.

**Return value:** The value that results from running the "reducer" callback function to completion over the entire object.

#### Example

```js
const obj = { foo: 1, bar: 2 };

const max = Object.reduce(obj, (accumulator, currentValue, currentKey, reference) => {
  console.log({ accumulator, currentValue, currentKey });
  return Math.max(accumulator, currentValue);
})

// Would log:
// { accumulator: 1, currentValue: 2, currentKey: 'bar' }

console.log(max);
// 2



Object.reduce(obj, (accumulator, currentValue, currentKey, reference) => {
  console.log({ accumulator, currentValue });
  return Math.max(accumulator, currentValue);
}, -Infinity)

// Would log:
// { accumulator: -Infinity, currentValue: 1, currentKey: 'foo' }
// { accumulator: 1, currentValue: 2, currentKey: 'bar' }
```

### `Object.some()`

To mimic iterators & arrays, `Object.some()` can be added:

```js
Object.some(obj, callbackFn)
Object.some(obj, callbackFn, thisArg)
```

#### Signature

**Parameters:**
- `obj`\
  An object.
- `callbackFn`\
  A function to execute for each element in the object. It should return a [truthy](https://developer.mozilla.org/en-US/docs/Glossary/Truthy) to indicate the entry passes the test, and a [falsy](https://developer.mozilla.org/en-US/docs/Glossary/Falsy) value otherwise. The function is called with the following arguments:
  - `value`\
    The current value of the entry being processed in the object.
  - `key`\
    The key of the current entry being processed in the object.
  - `object`\
     The object `some()` was called upon.
- `thisArg` (Optional)\
  A value to use as `this` when executing `callbackFn`

**Return value:** `false` unless `callbackFn` returns a [truthy](https://developer.mozilla.org/en-US/docs/Glossary/Truthy) value for an object entry, in which case `true` is immediately returned.

#### Example

```js
const obj = { foo: 1, bar: '2' };

const doesContainNumber = Object.some(obj, (value, key, reference) => {
  console.log(value, key, reference);
  return typeof value === 'number';
})
// Would log:
// 1   'foo' { foo: 1, bar: '2' }
// the 2nd isn’t logged as it immediatly stops as the 1st matched

console.log(doesContainNumber);
// true

const doesContainArray = Object.some(obj, (value, key, reference) => {
  console.log(value, key, reference);
  return Array.isArray(value);
})
// Would log:
// 1   'foo' { foo: 1, bar: '2' }
// '2' 'bar' { foo: 1, bar: '2' }

console.log(doesContainArray);
// false
```

### `.every()`

To mimic iterators & arrays, `Object.every()` can be added:

```js
Object.every(obj, callbackFn)
Object.every(obj, callbackFn, thisArg)
```

#### Signature

**Parameters:**
- `obj`\
  An object.
- `callbackFn`\
  A function to execute for each element in the object. It should return a [truthy](https://developer.mozilla.org/en-US/docs/Glossary/Truthy) valueto indicate the entry passes the test, and a [falsy](https://developer.mozilla.org/en-US/docs/Glossary/Falsy) value otherwise. The function is called with the following arguments:
  - `value`\
    The current value of the entry being processed in the object.
  - `key`\
    The key of the current entry being processed in the object.
  - `object`\
     The object `every()` was called upon.
- `thisArg` (Optional)\
  A value to use as `this` when executing `callbackFn`

**Return value:** `true` unless `callbackFn` returns a [falsy](https://developer.mozilla.org/en-US/docs/Glossary/Falsy) value for an object entry, in which case `false` is immediately returned.

#### Example

```js
const obj = { foo: 1, bar: '2' };

const doesOnlyIncludeString = Object.every(obj, (value, key, reference) => {
  console.log(value, key, reference);
  return typeof value === 'string';
})
// Would log:
// 1   'foo' { foo: 1, bar: '2' }
// the 2nd isn’t logged as it immediatly stops as the 1st doesn’t match

console.log(doesOnlyIncludeString);
// false

const doesOnlyIncludeNumberLike = Object.every(obj, (value, key, reference) => {
  console.log(value, key, reference);
  return !Number.isNaN(Number(value));
})
// Would log:
// 1   'foo' { foo: 1, bar: '2' }
// '2' 'bar' { foo: 1, bar: '2' }

console.log(doesOnlyIncludeNumberLike);
// true
```

## QnA

### Why not trying to make objects iterables?

Modifying `Object`’s prototype to add a `[@@iterator]()` method would also make all classes that extend from `Object` iterable. This will have side effects, so it can be dangerous.
Also some classes like `WeakSet` do extend `Object` but we **don’t** want to make them iterable.

So we can **not** make them iterable.

### Why adding those as static methods and not as functions on the prototype?

For the same reason as making them iterable, if `Object.prototype.forEach()` was added, we could do `new WeakSet().forEach()` that:
- wouldn’t behave as expected (as `Object.keys(new WeakSet([{}]))` returns `[]`),
- should **not** work as `WeakSet` shouldn’t be iterable.

### Why this signature for `.forEach()` (and other)?

When adding `Object.forEach`, it could behave in 2 ways:
- either as a shorthand for `Object.entries(object).forEach(([key, value], index, objectReference) => {})`,
- or similarly to `Array.prototype.forEach((value, key, objectReference) => {  })`

I’d advise to follow the `Array` behavior for 3 reasons:
1. a bit more practical as most of the time, people want to use the `value`, and you can still use the `key`
2. why would anyone need the `index`? Also `index` doesn’t make sense for objects
3. if it were a shorthand for `Object.entries(object).forEach()`, the last argument should be the output of `Object.entries(object)`, so an array of entries, and not the `objectReference` itself. So even in the best scenario, it wouldn’t be a drop-in replacement for `Object.entries(object).forEach()`

### Should `Object.map()`, `Object.filter()`, and others re-use the original prototype or start from zero?

In [`Array.prototype.filter()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/filter), it is mentioned that the returned array is a shallow copy. And in arrays again, new arrays are already created. And the same happens when using `Object.fromEntries(Object.entries(object))`

So I think those return new objects that don’t extend from the original prototype.

### What about collisions with other libraries?

In ECMAScript, except for the spread operator mentioned earlier, those shouldn’t collide with anything in the current specification. But in the past, APIs were picked to avoid collisions with older libraries mutating the global polyfills. For example `Array.prototype.flatten()` was renamed `.flat()` to avoid collisions with MooTools, see https://developer.chrome.com/blog/smooshgate.

In this case, MooTools adds:
- `Object.prototype.erase`
- `Object.prototype.run`
- `Object.each` <- not the same name as `Object.forEach`
- `Object.merge`
- `Object.clone`
- `Object.append`
- `Object.subset`
- `Object.map`    <- conflicts
- `Object.filter` <- conflicts
- `Object.every`  <- conflicts
- `Object.some`   <- conflicts
- `Object.keys`
- `Object.values`
- `Object.getLength`
- `Object.keyOf`
- `Object.contains`
- `Object.toQueryString`

But when you check how they are implemented ([see doc](https://mootools.net/core/docs/1.6.0/Types/Object#Object:Object-map)):

<details><summary>Expand to see the extract from the docs</summary>

> **[Function: Object.map](https://mootools.net/core/docs/1.6.0/Types/Object#Object:Object-map)**
> 
> Creates a new map with the results of calling a provided function on every value in the map.
> 
> **Syntax:**
> 
> ```js
> var mappedObject = Object.map(object, fn[, bind]);
> ```
> 
> **Arguments:**
> 
> 1.  object - (_object_) The object.
> 2.  fn - (_function_) The function to produce an element of the Object from an element of the current one.
> 3.  bind - (_object_, optional) The object to use as 'this' in the function. For more information see [Function:bind](https://mootools.net/core/docs/1.6.0/Types/Function#Function:bind).
> 
> **Argument: fn**
> 
> **Syntax:**
> 
> ```js
> fn(value, key, object)
> ```
> 
> **Arguments:**
> 
> 1.  value - (_mixed_) The current value in the object.
> 2.  key - (_string_) The current value's key in the object.
> 3.  object - (_object_) The actual object.
> 
> **Returns:**
> 
> -   (_object_) The new mapped object.
> 
> **Examples:**
> 
> ```js
> var myObject = {a: 1, b: 2, c: 3};
> var timesTwo = Object.map(myObject, function(value, key){
>     return value * 2;
> }); // timesTwo now holds an object containing: {a: 2, b: 4, c: 6};
> ```

</details>

This is the exact same API & output. And same for the 3 other functions. So it shouldn’t break any MooTools website.

### Why this proposal? Everything is already doable without

Indeed, all of those can already be done without. But they’re a bit verbose at the moment, and they create a few temporary arrays (1 for the `Object.entries`, + 1 for each entry) which is not ideal for performances:

```js
// With the current ECMAScript spec:
Object.entries(obj).forEach(([key, value]) => callbackFn(value, key, obj));
Object.fromEntries(Object.entries(obj).map(([key, value]) => callbackFn(value, key, obj)));
Object.entries(obj).reduce((accumulator, [key, value]) => callbackFn(accumulator, value, key, obj), initialValue);
Object.entries(obj).some(([key, value]) => callbackFn(value, key, obj));
Object.entries(obj).every(([key, value]) => callbackFn(value, key, obj));

// With this proposal:
Object.forEach(obj, callbackFn);
Object.map(obj, callbackFn);
Object.reduce(callbackFn, initialValue);
Object.some(callbackFn);
Object.every(callbackFn);
```

The new code is simpler, and if the internals are optimized, it should also be more performant for the end user.

### Which order should be used for iterating over objects?

For a long time, objects in ECMAScript didn’t have a stable traversal order and it was up to implementations to decide what to do. But this has changed in recent years. To quote [MDN](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/for...in#description):

> The traversal order, as of modern ECMAScript specification, is well-defined and consistent across implementations. Within each component of the prototype chain, all non-negative integer keys (those that can be array indices) will be traversed first in ascending order by value, then other string keys in ascending chronological order of property creation.

Those rules are respected by `for..in`, `Object.keys()`, `Object.values()`, and `Object.entries()`. So the same logic can be applied to the new methods too.
