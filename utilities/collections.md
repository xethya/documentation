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

## Getting elements

The `get()` and `getAll()` functions are used to obtain elements from a collection. 

When looking for a single element, its unique identifier (given by the value of the property set in `indexName`) must be provided.

When retrieving all elements, `getAll()` will create a copy of the collection's elements.

```typescript
const alice = characterCollection.get('alice');
const charactersArray = characterCollection.getAll();
```

At any time, you can count how many elements are in the collection:

```typescript
if (characterCollection.count > 0) {
  // Collection is not empty!
}
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

## Looking for an element

The `where()` function receives a `QueryFunction<T>` callback that allows to filter the collection and returns those items that match the criteria specified by the callback. Works as `Array#filter`.

```typescript
// A function that returns a list of characters
// according to the team they belong to.
const getMembersByTeam = (team: string) => characterCollection.where(
  character => character.team === team
);
```

## Listening to events

The `Collection` object triggers events when adding and removing elements, exposing the following event listener registration mechanisms:

```typescript
characterCollection.onBeforeAdd((collection: Collection<Character>, ...items: Character[]) => { /* ... */ });
characterCollection.onAdd((collection: Collection<Character>, ...items: Character[]) => { /* ... */ });
characterCollection.onBeforeRemove((collection: Collection<Character>, ...items: Character[]) => { /* ... */ });
characterCollection.onRemove((collection: Collection<Character>, ...items: Character[]) => { /* ... */ });
characterCollection.onBeforeRemoveAll((collection: Collection<Character>) => { /* ... */ });
characterCollection.onRemoveAll((collection: Collection<Character>) => { /* ... */ });
```

As you can see, all events (except for `*removeAll`) allow access to the items being manipulated at the time the event was triggered, as well as to the affected collection.