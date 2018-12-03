# @xethya/utils

[![npm version](https://badge.fury.io/js/%40xethya%2Futils.svg)](https://badge.fury.io/js/%40xethya%2Futils)

## Description

This package contains lots of objects and functions built to simplify working with primitives on the framework. It is comprised of tools for:

* working with arrays,
* creating collections of a given type,
* performing some specific numeric and string operations,
* dealing with intervals,
* managing time,
* executing assertions,
* etc.

## Installation

```bash
npm install @xethya/utils
```

## Usage

```typescript
// Bring the entire library.
import * as utils from '@xethya/utils';

// Bring specific subpackages.
import { array } from '@xethya/utils';
import { assert } from '@xethya/utils'; 
import { Collection } from '@xethya/utils'; 
import { numeric } from '@xethya/utils'; 
import { string } from '@xethya/utils'; 
import { Range } from '@xethya/utils'; 
import { Time } from '@xethya/utils'; 
```

## Array utilities

### Shuffling

The `shuffle()` function allows to randomly rearrange an array. It uses `Math.random()` to assign a new index to the elements and then sort them accordingly.

```typescript
import { array } from '@xethya/utils';

const numbers = [1, 2, 3, 4, 5];
const shuffledNumbers = array.shuffle(numbers); // [5, 3, 1, 2, 4]

// It works with objects too!
const characters = [
  { name: 'Alice', hp: 100 },
  { name: 'Bob', hp: 100 },
  { name: 'Candice', hp: 100 },
  { name: 'Danielle', hp: 100 },
];
const shuffledCharacters = array.shuffle(characters); // [Candice, Alice, ...]
```

### Grouping

Grouping is a useful feature to arrange elements by a given criteria. Suppose you have a list of characters, each one belonging to a different team. If you want to get a list of characters by team, you can use the `group()` function to do just that. Let's assume such list of characters:

```typescript
import { array } from '@xethya/utils';

const characters = [
  { name: 'Alice', hp: 100, team: 'red' },
  { name: 'Bob', hp: 100, team: 'green' },
  { name: 'Candice', hp: 100, team: 'green' },
  { name: 'Danielle', hp: 100, team: 'red' },
];
```

In order to define the grouping criteria, you need to pass on a `GroupMatchingFunction<T>` , which is basically a callback with the signature `(element: T) => string`. For our example list, such function could be defined as:

```typescript
const byTeam: GroupMatchingFunction<Character> = 
  (character: Character) => character.team;
```

With that function defined, we can group our characters by team:

```typescript
const charactersByTeam = array.group(characters, byTeam);
```

If we inspect the value of `charactersByTeam`, we'll get something like this:

```javascript
{
  red: [
    { id: 'Alice', hp: 100, team: 'red' },
    { id: 'Danielle', hp: 100, team: 'red' }
  ],
  green: [
    { id: 'Candice', hp: 100, team: 'red' },
    { id: 'Bob', hp: 100, team: 'red' }
  ]
}
```

Suppose we don't want the entire Character object, and we want to have a group of characters by teams with just their names. We can use the `groupAndMap` function, which receives an additional `TransformFunction<T, R>` as a parameter, with the signature `(element: T) => R`.

Such transform function would be defined as:

```typescript
const withName: TransformFunction<Character, string> =
  (character: Character) => character.id;
```

And then, instead of using `group()`, we invoke the `groupAndMap` function:

```typescript
const characterNamesByTeam = array.groupAndMap(characters, byTeam, withName);
```

The result will be something like this:

```javascript
{
  red: ['Alice', 'Danielle'],
  green: ['Candice', 'Bob']
}
```

### Mappers and reducers

Xethya works a lot with arrays and numbers. Most operations can be represented, for example, as the result of a sum of multiple numbers.

In plain JavaScript, you'd sum the values of an array using something like this:

```javascript
const sum = [1, 2, 3, 4].reduce((a, b) => a + b, 0);
```

To prevent repeating the reducer function over and over again, there are some reusable preset functions of this kind.

Using this package, you have available two utility functions:

* `mappers.byKey` will transform a list of objects into a list of values for a given key in each object.
* `reducers.bySum` will reduce a list of numbers into a single number by adition.

For example, the previous `reduce` example could be implemented as:

```typescript
import { array } from '@xethya/utils';
const { bySum } = array.reducers;

const sum = [1, 2, 3, 4].reduce(bySum, 0);
```

Also, the `groupAndMap()` function described in the previous section makes use of the `byKey` mapper:

```typescript
import { array } from '@xethya/utils';
const { byKey } = array.mappers;

const characters = [
  { name: 'Alice', hp: 100, team: 'red' },
  { name: 'Bob', hp: 100, team: 'green' },
  { name: 'Candice', hp: 100, team: 'green' },
  { name: 'Danielle', hp: 100, team: 'red' },
];

const characterNames = characters.map(byKey('name')); // ['Alice', 'Bob', ...]
```

## Collections

A Collection is a hybrid object that lies somewhere between an array and a set, which also happens to emit events as its state changes.

### Defining a collection

A Collection takes an `indexName` parameter to define which object property will act as its unique identifier or primary key. Assuming we continue with our characters list, we can define a collection like this:

```typescript
import { Collection } from '@xethya/utils';

const characterCollection = new Collection<Character>('name');
```

### Converting an array into a collection

We can also create a Collection instance from a pre-existing array, using the static function `Collection#fromArray()`:

```typescript
const characters = [
  { name: 'Alice', hp: 100, team: 'red' },
  { name: 'Bob', hp: 100, team: 'green' },
  { name: 'Candice', hp: 100, team: 'green' },
  { name: 'Danielle', hp: 100, team: 'red' },
];

const characterCollection = Collection.fromArray<Character>(characters, 'name');
```

### Adding elements

The `add()` method allows to add new elements to the collection. 

```typescript
characterCollection.add({ name: 'Edward', hp: 100, team: 'red' });
```

You can also add multiple elements at once:

```typescript
characterCollection.add(
  { name: 'Edward', hp: 100, team: 'red' },
  { name: 'Faith', hp: 100, team: 'blue' },
);
```

Or even spread an array:

```typescript
const newCharacters = [
  { name: 'Gwen', hp: 100, team: 'orange' },
  { name: 'Henry', hp: 100, team: 'purple' },
];

characterCollection.add(...newCharacters);
```

Every time you add an element, the value in `item[indexName]` will be checked for uniqueness. This means, in our characters collection example, that if you've already added _Alice_, you can't add her data again:

```typescript
// This will throw an AssertionError reporting that Alice already exists
// in the collection.
characterCollection.add({ name: 'Alice', hp: 50, team: 'yellow' });
```

### Removing elements

The `remove()` method allows you to delete an element from the collection. It uses a value that can be found in `item[indexName]` as its argument. In our characters collection example, if you wanted to remove _Bob_ from the list, you could simply do:

```typescript
characterCollection.remove('Bob'); // the collection now excludes Bob
```

You can also empty the collection using the `removeAll()` method:

```typescript
characterCollection.removeAll(); // the collection is now empty
```

### Checking if an element is part of the collection

The `contains()` function receives a value that can be found in `item[indexName]` and returns `true` if the element exists, or `false` otherwise. This function is also used to check for uniqueness before adding an element:

```typescript
if (characterCollection.contains('Alice')) {
  // ...
}
```

