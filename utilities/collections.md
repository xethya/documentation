# Collections

A Collection is a hybrid object that lies somewhere between an array and a set, which also happens to emit events as its state changes.

## Defining a collection

A Collection takes an `indexName` parameter to define which object property will act as its unique identifier or primary key. Assuming we continue with our characters list, we can define a collection like this:

```typescript
import { Collection } from '@xethya/utils';

const characterCollection = new Collection<Character>('name');
```

## Converting an array into a collection

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

## Adding elements

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

## Removing elements

The `remove()` method allows you to delete an element from the collection. It uses a value that can be found in `item[indexName]` as its argument. In our characters collection example, if you wanted to remove _Bob_ from the list, you could simply do:

```typescript
characterCollection.remove('Bob'); // the collection now excludes Bob
```

You can also empty the collection using the `removeAll()` method:

```typescript
characterCollection.removeAll(); // the collection is now empty
```

## Checking if an element is part of the collection

The `contains()` function receives a value that can be found in `item[indexName]` and returns `true` if the element exists, or `false` otherwise. This function is also used to check for uniqueness before adding an element:

```typescript
if (characterCollection.contains('Alice')) {
  // ...
}
```

