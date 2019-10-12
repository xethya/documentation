# Array

## Shuffling

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

## Grouping

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

## Mappers and reducers

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
