<h3 align="center">
  <img align="center" src="assets/listopard.png" alt="List logo" width=400 />
</h3>

<p align="center">
A fast immutable list with a functional API.
</p>

<p align="center">
  <a href="https://gitter.im/funkia/General"><img src="https://img.shields.io/gitter/room/funkia/General.svg" alt="Gitter" height="20"></a>
  <a href="https://travis-ci.org/funkia/list"><img src="https://travis-ci.org/funkia/list.svg?branch=master" alt="Build Status" height="20"></a>
  <a href="https://codecov.io/gh/funkia/list"><img src="https://codecov.io/gh/funkia/list/branch/master/graph/badge.svg" alt="codecov" height="20"></a>
  <a href="https://badge.fury.io/js/list"><img src="https://badge.fury.io/js/list.svg" alt="npm version" height="20"></a>
  <a href="https://david-dm.org/funkia/list"><img src="https://david-dm.org/funkia/list/status.svg" alt="npm version" height="20"></a>
  <a href="https://david-dm.org/funkia/list?type=dev"><img src="https://david-dm.org/funkia/list/dev-status.svg" alt="npm version" height="20"></a>
  <a href="https://greenkeeper.io/"><img src="https://badges.greenkeeper.io/funkia/list.svg" alt="Greenkeeper badge"></a>
</p>

# List

List is a purely functional alternative to arrays. It is an
implementation of a fast persistent sequence data structure. Compared
to JavaScript's `Array` List has two major benefits.

* **Safety**. List is immutable. This makes it safer and better suited
  for functional programming. It doesn't tempt you with an imperative
  API and accidental mutations won't be a source of bugs.
* **Performance**. Since List doesn't allow mutations it can be
  heavily optimized for pure operations. This makes List much faster
  for functional programming than arrays. [See the
  benchmarks](https://funkia.github.io/list/benchmarks/).

## Features

* **Familiar functional API**. List follows the naming conventions
  common in functional programming and has arguments ordered for
  currying/partial application.
* **Extensive API**. List has all the functions you know from `Array`
  and a lot of other functions that'll save the day once you need
  them.
* **Extremely fast**. List is a carefully optimized implementation of
  the highly efficient data-structure _relaxed radix balanced trees_.
  Performance has been a central focus from the beginning and we have
  an [extensive benchmark
  suite](https://funkia.github.io/list/benchmarks/) to ensure optimal
  performance.
* **Does one thing well**. Instead of offering a wealth of data
  structures List has a tight focus on being the best immutable list
  possible. It doesn't do everything but is designed to work well with
  the libraries you're already using.
* **Seamless Ramda integration**. If you know Ramda you already know
  how to use List. List includes a [Ramda
  specific](#seamless-ramda-integration) module where functions are
  curried using Ramda's `R.curry` and where equality comparisons are
  done using `R.equals`.
* **Type safe**. List is written in TypeScript and comes with accurate
  types that cover the entire library.
* **Fully compatible with tree-shaking**. List ships with tree-shaking
  compatible ECMAScript modules. `import * as L from "list"` in itself
  adds zero bytes to your bundle when using Webpack. Using a function
  adds only that function and the very small (<1KB) core of the
  library. You only pay in size for the functions that you actually
  use.
* **Iterable**. Implements the JavaScript iterable protocol. This
  means that lists can be use in `for..of` loops, works with
  destructuring, and can be passed to any function expecting an
  iterable. [See more](#iterable).
* **Fantasy Land support**. List
  [implements](#fantasy-land-static-land) both the Fantasy Land and
  the Static Land specification.

## Getting started

This section explains how to get started using List. First you'll have
to install the library.

```
npm install list
```

Then you can import it.

```js
// As an ES module
import * as L from "list";
// Or with require
const L = require("list");
```

Then you can begin using List instead of arrays and enjoy immutability
the performance benefits.

As a replacement for array literals List offers the function `list`
for constructing lists:

```js
// An array literal
const myArray = [0, 1, 2, 3];
// A list "literal"
const myList = L.list(0, 1, 2, 3);
```

List has all the common functions that you know from native arrays and
other libraries.

```js
const myList = L.list(0, 1, 2, 3, 4, 5);
myList.length; //=> 6
L.filter(isEven, myList); //=> list(0, 2, 4)
L.map(n => n * n, myList); //=> list(0, 1, 4, 9, 16, 25)
L.reduce((sum, n) => sum + n, 0, myList); //=> 15
L.slice(2, 5, myList); //=> list(2, 3, 4)
L.concat(myList, L.list(6, 7, 8)); //=> list(0, 1, 2, 3, 4, 5, 6, 7, 8);
```

You'll probably also end up needing to convert between arrays and
List. You can do that with the functions `fromArray` and `toArray`.

```js
L.toArray(L.list("foo", "bar")); //=> ["foo", "bar"];
L.fromArray(["foo", "bar"]); //=> L.list("foo", "bar");
```

List offers a wealth of other useful and high-performing functions.
You can see them all in the [API documentation](#api-documentation)

## Iterable

List implements the JavaScript iterable protocol. This means that
lists can be used with array destructuring just like normal arrays.

```js
const myList = L.list("first", "second", "third", "fourth");
const [first, second] = myList;
first; //=> "first"
second; //=> "second"
```

Lists can also be used in `for..of` loops.

```js
for (const element of myList) {
  console.log(element);
}
// logs: first, second, third, fourth
```

And they can be passed to any function that takes an iterable as its
argument. As an example a list can be converted into a native
[`Set`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Set).

```js
const mySet = new Set(myList);
mySet.has("third"); //=> true
```

This works because the `Set` constructor accepts any iterable as
argument.

Lists also work with [spread syntax](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Spread_syntax). For instannce, you can call a function like this.

```js
console.log(...list("hello", "there", "i'm", "logging", "elements"));
```

And each element of the list will be passed as an argument to `console.log`.

The iterable protocol allows for some very convenient patterns and
means that lists can integrate nicely with JavaScript syntax. But,
here are two anti-patterns that you should be aware of.

1. Don't overuse `for..of` loops. Functions like [`map`](#map) and
   [`foldl`](#foldl) are often a better choice. If you want to perform
   a side-effect for each element in a list you should probably use
   [`forEach`](#forEach).
2. Don't use the spread syntax in destructuring
   ```js
   const [a, b, ...cs] = myList; // Don't do this
   ```
   The syntax converts the rest of the iterable (in this case a list)
   into an array by iterating through the entire iterable. This is
   slow and it turns our list into an array. This alternative avoids
   both problems.
   ```js
   const [[a, b], cs] = splitAt(2, myList); // Do this
   ```
   This uses the [`splitAt`](#splitAt) function which splits and
   creates the list `cs` very efficiently in `O(log(n))` time.

## Seamless Ramda integration

List is designed to work seamlessly together with Ramda. Ramda offers
a large number of useful functions for working with arrays. List
implements the same methods on its immutable data structure. This
means that Ramda users can keep using the API they're familiar with.

Additionally, List offers an entry point where all functions are
curried using Ramda's `R.curry` and where all equality comparisons are
done using `R.equals`.

```js
import * as L from "list/ramda";
const indexOfFoo1 = indexOf({ foo: 1 });
indexOfFoo1(list({ foo: 0 }, { foo: 1 }, { foo: 2 })); //=> 1
```

In the example above `indexOf` is curried and it uses `R.equals` to
find an element equivalent to `{ foo: 1 }`.

Since List implements Ramda's array API it is very easy to convert
code from using arrays to using immutable lists. As an example,
consider the code below.

```js
import * as R from "ramda";

R.pipe(R.filter(n => n % 2 === 0), R.map(R.multiply(3)), R.reduce(R.add, 0))(
  array
);
```

It can be converted to code using List as follows.

```js
import * as R from "ramda";
import * as L from "list";

R.pipe(L.filter(n => n % 2 === 0), L.map(R.multiply(3)), L.reduce(R.add, 0))(
  list
);
```

For each function operating on arrays, the `R` is simply changed to an
`L`. This works because List exports functions that have the same name
and behavior as Ramdas functions.

### Implemented Ramda functions

The goal is to implement the entirety of Ramda's array functions for
List. The list below keeps track of how many of Ramda functions that
are missing and of how many that are already implemented. Currently 43
out of 75 functions have been implemented.

Implemented: `adjust`, `all`, `any`, `append`, `chain`, `concat`,
`contains`, `drop`, `dropLast`, `dropWhile`, `filter`, `find`,
`findIndex`, `head`, `flatten`, `indexOf`, `init`, `insert`,
`insertAll`, `last`, `length`, `join`, `map`, `none`, `nth`, `pair`,
`partition`, `pluck`, `prepend`, `range`, `reduce`, `reduceRight`,
`reject`, `remove`, `reverse`, `repeat`, `slice`, `splitAt`, `take`,
`takeWhile`, `tail`, `takeLast`, `times`, `update`.

Not implemented: `aperture`, `dropLastWhile`, `dropRepeats`,
`dropRepeatsWith`, `endsWith`, `findLast`, `findLastIndex`,
`groupWith`, `indexBy`, `intersperse`, `lastIndexOf`, `mapAccum`,
`mapAccumRight`, `reduceWhile`, `scan`, `sequence`, `sort`,
`splitEvery`, `splitWhen`, `startsWith`, `takeLastWhile`, `transpose`,
`traverse`, `unfold`, `uniq`, `uniqBy`, `uniqWith`,
`unnest` `without`, `xprod`, `zip`, `zipWith`.

## Fantasy Land & Static Land

<img align="right" width="82" height="82" alt="Fantasy Land" src="https://raw.github.com/fantasyland/fantasy-land/master/logo.png">
<img align="right" width="131" height="82" src="https://raw.githubusercontent.com/rpominov/static-land/master/logo/logo.png" />

List currently implements the following Fantasy Land and Static Land
specifications: Setoid, semigroup, monoid, foldable, functor, apply,
applicative, chain, monad.

The following specifications have not been implemented yet:
Traversable.

Since methods hinder tree-shaking the Fantasy Land methods are not
included by default. In order to get them you must import it likes
this:

```js
import "list/fantasy-land";
```

## API documentation

The API is organized into three parts.

1. [Creating lists](#creating-lists) — Functions that _create_ lists.
2. [Updating lists](#updating-lists) — Functions that _transform_ lists.
   That is, functions that take one or more lists as arguments and
   returns a new list.
3. [Folds](#folds) — Functions that _extracts_ values based on lists.
   They take one or more lists as arguments and returns something that
   is not a list.

### Creating lists

### `list`

Creates a list based on the arguments given.

**Complexity**: `O(n)`

**Example**

```js
const l = list(1, 2, 3, 4); // creates a list of four elements
const l2 = list("foo"); // creates a singleton
```

### `empty`

Returns an empty list.

**Complexity**: `O(1)`

**Example**

```js
const emptyList = empty(); //=> list()
```

### `of`

Takes a single arguments and returns a singleton list that contains it.

**Complexity**: `O(1)`

**Example**

```js
of("foo"); //=> list("foo")
```

### `pair`

Takes two arguments and returns a list that contains them.

**Complexity**: `O(1)`

**Example**

```js
pair("foo", "bar"); //=> list("foo", "bar")
```

### `fromArray`

Converts an array into a list.

**Complexity**: `O(n)`

**Example**

```js
fromArray([0, 1, 2, 3, 4]); //=> list(0, 1, 2, 3, 4)
```

### `range`

Returns a list of numbers between an inclusive lower bound and an
exclusive upper bound.

**Complexity**: `O(n)`

**Example**

```js
range(3, 8); //=> list(3, 4, 5, 6, 7)
```

### `repeat`

Returns a list of a given length that contains the specified value in
all positions.

**Complexity**: `O(n)`

**Example**

```js
repeat(1, 7); //=> list(1, 1, 1, 1, 1, 1, 1)
repeat("foo", 3); //=> list("foo", "foo", "foo")
```

### `times`

Returns a list of given length that contains the value of the given function called with current index.

**Complexity**: `O(n)`

**Example**

```js
const twoFirsOdds = times(i => i * 2 + 1, 2);
const dots = times(() => {
  const x = Math.random() * width;
  const y = Math.random() * height;
  return { x, y };
}, 50);
```

### Updating lists

### `concat`

Concatenates two lists.

**Complexity**: `O(log(n))`

**Example**

```js
concat(list(0, 1, 2), list(3, 4)); //=> list(0, 1, 2, 3, 4)
```

### `flatten`

Flattens a list of lists into a list. Note that this function does
_not_ flatten recursively. It removes one level of nesting only.

**Complexity**: `O(n * log(m))` where `n` is the length of the outer
list and `m` the length of the inner lists.

**Example**

```js
const nested = list(list(0, 1, 2, 3), list(4), empty(), list(5, 6));
flatten(nested); //=> list(0, 1, 2, 3, 4, 5, 6)
```

### `prepend`

Prepends an element to the front of a list and returns the new list.

**Complexity**: `O(log(n))`, practically constant

**Example**

```js
const newList = prepend(0, list(1, 2, 3)); //=> list(0, 1, 2, 3)
```

### `append`

Appends an element to the end of a list and returns the new list.

**Complexity**: `O(log(n))`, practically constant

**Example**

```js
const newList = append(3, list(0, 1, 2)); //=> list(0, 1, 2, 3)
```

### `map`

Applies a function to each element in the given list and returns a new
list of the values that the function return.

**Complexity**: `O(n)`

**Example**

```js
map(n => n * n, list(0, 1, 2, 3, 4)); //=> list(0, 1, 4, 9, 16)
```

### `pluck`

Extracts the specified property from each object in the list.

**Example**

```js
const l = list(
  { foo: 0, bar: "a" },
  { foo: 1, bar: "b" },
  { foo: 2, bar: "c" }
);
pluck("foo", l); //=> list(0, 1, 2)
```

### `update`

Returns a list that has the entry specified by the index replaced with
the given value.

**Complexity**: `O(log(n))`

**Example**

```js
update(2, "X", list("a", "b", "c", "d", "e")); //=> list("a", "b", "X", "d", "e")
```

### `adjust`

Returns a list that has the entry specified by the index replaced with
the value returned by applying the function to the value.

**Complexity**: `O(log(n))`

**Example**

```js
adjust(2, inc, list(0, 1, 2, 3, 4, 5)); //=> list(0, 1, 3, 3, 4, 5)
```

### `slice`

Returns a slice of a list. Elements are removed from the beginning and
end. Both the indices can be negative in which case they will count
from the right end of the list.

**Complexity**: `O(log(n))`

**Example**

```js
const l = list(0, 1, 2, 3, 4, 5);
slice(1, 4, l); //=> list(1, 2, 3)
slice(2, -2, l); //=> list(2, 3)
```

### `take`

Takes the first `n` elements from a list and returns them in a new list.

**Complexity**: `O(log(n))`

**Example**

```js
take(3, list(0, 1, 2, 3, 4, 5)); //=> list(0, 1, 2)
```

### `takeWhile`

Takes the first elements in the list for which the predicate returns
`true`.

**Complexity**: `O(k + log(n))` where `k` is the number of elements
satisfying the predicate.

**Example**

```js
takeWhile(n => n < 4, list(0, 1, 2, 3, 4, 5, 6)); //=> list(0, 1, 2, 3)
```

### `takeLast`

Takes the last `n` elements from a list and returns them in a new
list.

**Complexity**: `O(log(n))`

**Example**

```js
takeLast(3, list(0, 1, 2, 3, 4, 5)); //=> list(3, 4, 5)
```

### `splitAt`

Splits a list at the given index and return the two sides in a pair.
The left side will contain all elements before but not including the
element at the given index. The right side contains the element at the
index and all elements after it.

**Complexity**: `O(log(n))`

**Example**

```js
const l = list(0, 1, 2, 3, 4, 5, 6, 7, 8);
splitAt(4, l); //=> [list(0, 1, 2, 3), list(4, 5, 6, 7, 8)]
```

### `remove`

Takes an index, a number of elements to remove and a list. Returns a
new list with the given amount of elements removed from the specified
index.

**Complexity**: `O(log(n))`

**Example**

```js
const l = list(0, 1, 2, 3, 4, 5, 6, 7, 8);
remove(4, 3, l); //=> list(0, 1, 2, 3, 7, 8)
remove(2, 5, l); //=> list(0, 1, 7, 8)
```

### `drop`

Returns a new list without the first `n` elements.

**Complexity**: `O(log(n))`

**Example**

```js
drop(2, list(0, 1, 2, 3, 4, 5)); //=> list(2, 3, 4, 5)
```

### `dropWhile`

Removes the first elements in the list for which the predicate returns
`true`.

**Complexity**: `O(k + log(n))` where `k` is the number of elements
satisfying the predicate.

**Example**

```js
dropWhile(n => n < 4, list(0, 1, 2, 3, 4, 5, 6)); //=> list(4, 5, 6)
```

### `dropLast`

Returns a new list without the last `n` elements.

**Complexity**: `O(log(n))`

**Example**

```js
dropLast(2, list(0, 1, 2, 3, 4, 5)); //=> list(0, 1, 2, 3)
```

### `tail`

Returns a new list with the first element removed.

**Complexity**: `O(1)`

**Example**

```js
tail(list(0, 1, 2, 3)); //=> list(1, 2, 3)
```

### `pop`

Returns a new list with the last element removed.

**Aliases**: `init`

**Complexity**: `O(1)`

**Example**

```js
pop(list(0, 1, 2, 3)); //=> list(0, 1, 2)
```

### `filter`

Returns a new list that only contains the elements of the original
list for which the predicate returns `true`.

**Complexity**: `O(n)`

**Example**

```js
filter(isEven, list(0, 1, 2, 3, 4, 5, 6)); //=> list(0, 2, 4, 6)
```

### `reject`

Returns a new list that only contains the elements of the original
list for which the predicate returns `false`.

**Complexity**: `O(n)`

**Example**

```js
reject(isEven, list(0, 1, 2, 3, 4, 5, 6)); //=> list(1, 3, 5)
```

### `reverse`

Reverses a list.

**Complexity**: `O(n)`

**Example**

```js
reverse(list(0, 1, 2, 3, 4, 5)); //=> list(5, 4, 3, 2, 1, 0)
```

### `ap`

Applies a list of functions to a list of values.

**Example**

```js
ap(list((n: number) => n + 2, n => 2 * n, n => n * n), list(1, 2, 3)); //=> list(3, 4, 5, 2, 4, 6, 1, 4, 9)
```

### `chain`

Maps a function over a list and concatenates all the resulting lists
together.

Also known as `flatMap`.

**Example**

```js
chain(n => list(n, 2 * n, n * n), list(1, 2, 3)); //=> list(1, 2, 1, 2, 4, 4, 3, 6, 9)
```

### `partition`

Splits the list into two lists. One list that contains all the values
for which the predicate returns `true` and one containing the values for
which it returns `false`.

**Complexity**: `O(n)`

**Example**

```js
partition(isEven, list(0, 1, 2, 3, 4, 5)); //=> list(list(0, 2, 4), list(1, 3, 5))
```

### `insert`

Inserts the given element at the given index in the list.

**Complexity**: `O(log(n))`

**Example**

```js
insert(2, "c", list("a", "b", "d", "e")); //=> list("a", "b", "c", "d", "e")
```

### `insertAll`

Inserts the given list of elements at the given index in the list.

**Complexity**: `O(log(n))`

**Example**

```js
insertAll(2, list("c", "d"), list("a", "b", "e", "f")); //=> list("a", "b", "c", "d", "e", "f")
```

### Folds

### `isList`

Returns `true` if the given argument is a list.

**Complexity**: `O(1)`

**Example**

```js
isList([0, 1, 2]); //=> false
isList("string"); //=> false
isList({ foo: 0, bar: 1 }); //=> false
isList(list(0, 1, 2)); //=> true
```

### `equals`

Returns true if the two lists are equivalent.

**Complexity**: `O(n)`

**Example**

```js
equals(list(0, 1, 2, 3), list(0, 1, 2, 3)); //=> true
equals(list("a", "b", "c"), list("a", "z", "c")); //=> false
```

### `toArray`

Converts a list into an array.

**Complexity**: `O(n)`

**Example**

```js
toArray(list(0, 1, 2, 3, 4)); //=> [0, 1, 2, 3, 4]
```

### `nth`

Gets the `n`th element of the list.

**Complexity**: `O(log(n))`, practically constant

**Example**

```js
const l = list(0, 1, 2, 3, 4);
nth(2, l); //=> 2
```

### `length`

Returns the length of a list. I.e. the number of elements that it
contains.

**Complexity**: `O(1)`

**Example**

```js
length(list(0, 1, 2, 3)); //=> 4
```

### `first`

Returns the first element of the list. If the list is empty the
function returns `undefined`.

**Aliases**: `head`

**Complexity**: `O(1)`

**Example**

```js
first(list(0, 1, 2, 3)); //=> 0
first(list()); //=> undefined
```

### `last`

Returns the last element of the list. If the list is empty the
function returns `undefined`.

**Complexity**: `O(1)`

**Example**

```js
last(list(0, 1, 2, 3)); //=> 3
last(list()); //=> undefined
```

### `foldl`

Folds a function over a list. Left-associative.

**Aliases**: `reduce`

**Complexity**: `O(n)`

**Example**

```js
foldl((n, m) => n - m, 1, list(2, 3, 4, 5));
1 - 2 - 3 - 4 - 5; //=> -13
```

### `foldr`

Folds a function over a list. Right-associative.

**Aliases**: `reduceRight`

**Complexity**: `O(n)`

**Example**

```js
foldr((n, m) => n - m, 5, list(1, 2, 3, 4));
1 - (2 - (3 - (4 - 5))); //=> 3
```

### `forEach`

Invokes a given callback for each element in the list from left to
right. Returns `undefined`.

This function is very similar to `map`. It should be used instead of
`map` when the mapping function has side-effects. Whereas `map`
constructs a new list `forEach` merely returns `undefined`. This makes
`forEach` faster when the new list is unneeded.

**Complexity**: `O(n)`

**Example**

```js
const l = list(0, 1, 2);
forEach(element => console.log(element));
//=> 0
//=> 1
//=> 2
```

### `every`

Returns `true` if and only if the predicate function returns `true`
for all elements in the given list.

**Aliases**: `all`

**Complexity**: `O(n)`

**Example**

```js
const isEven = n => n % 2 === 0;
every(isEven, empty()); //=> true
every(isEven, list(2, 4, 6, 8)); //=> true
every(isEven, list(2, 3, 4, 6, 7, 8)); //=> false
every(isEven, list(1, 3, 5, 7)); //=> false
```

### `some`

Returns `true` if and only if there exists an element in the list for
which the predicate returns `true`.

**Aliases**: `any`

**Complexity**: `O(n)`

**Example**

```js
const isEven = n => n % 2 === 0;
some(isEven, empty()); //=> false
some(isEven, list(2, 4, 6, 8)); //=> true
some(isEven, list(2, 3, 4, 6, 7, 8)); //=> true
some(isEven, list(1, 3, 5, 7)); //=> false
```

### `indexOf`

Returns the index of the first element in the list that is equal to
the given element. If no such element is found the function returns
`-1`.

**Complexity**: `O(n)`

**Example**

```js
const l = list(12, 4, 2, 89, 6, 18, 7);
indexOf(12, l); //=> 0
indexOf(89, l); //=> 3
indexOf(10, l); //=> -1
```

### `find`

Returns the first element for which the predicate returns `true`. If
no such element is found the function returns `undefined`.

**Complexity**: `O(n)`

**Example**

```js
find(isEven, list(1, 3, 5, 6, 7, 9, 10)); //=> 6
find(isEven, list(1, 3, 5, 7, 9)); //=> undefined
```

### `findIndex`

Returns the index of the first element for which the predicate returns
`true`. If no such element is found the function returns `-1`.

**Complexity**: `O(n)`

**Example**

```js
findIndex(isEven, list(1, 3, 5, 6, 7, 9, 10)); //=> 3
findIndex(isEven, list(1, 3, 5, 7, 9)); //=> -1
```

### `none`

Returns `true` if and only if the predicate function returns `false`
for all elements in the given list.

**Complexity**: `O(n)`

**Example**

```js
const isEven = n => n % 2 === 0;
none(isEven, empty()); //=> true
none(isEven, list(2, 4, 6, 8)); //=> false
none(isEven, list(2, 3, 4, 6, 7, 8)); //=> false
none(isEven, list(1, 3, 5, 7)); //=> true
```

### `includes`

Returns `true` if the list contains the specified element. Otherwise
it returns `false`.

**Aliases**: `contains`

**Complexity**: `O(n)`

**Example**

```js
includes(3, list(0, 1, 2, 3, 4, 5)); //=> true
includes(3, list(0, 1, 2, 4, 5)); //=> false
```

### `join`

Concats the strings in a list separated by a specified separator.

**Complexity**: `O(n)`

**Example**

```js
join(", ", list("one", "two", "three")); //=> "one, two, three"
```

## Benchmarks

The benchmarks are located in the [`bench` directory](/test/bench).
