# @xethya/dice

[![npm version](https://badge.fury.io/js/%40xethya%2Fdice.svg)](https://badge.fury.io/js/%40xethya%2Fdice)

> _**"Not only does God play dice, but... he sometimes throws them where they cannot be seen."**   
> — Stephen Hawking_

## Description

This package controls everything related to rolling dice. It provides a base class called `Die` which implements a [pRNG](https://en.wikipedia.org/wiki/Pseudorandom_number_generator) to return numbers in a given range \(according to the amount of faces of the die\).

## Installation

```bash
npm install @xethya/dice
```

## Usage

### Via instantiation

```typescript
import { Die } from '@xethya/dice';

const die = new Die(6);
die.roll(); // returns a number between 1 and 6
```

### Via static invocation

```typescript
import { Die } from '@xethya/dice';

Die.rollD(6); // returns a number between 1 and 6
```

### Listening to events

Every time the `roll()` function is called, two events are triggered:

| Event name | Parameters | Description |
| :--- | :--- | :--- |
| `before:roll` | — | Event triggered before generating a pseudo-random number \(a.k.a. actually rolling the die\). |
| `roll` | `rolled: number` | Event triggered after rolling the die. |

You can use the following methods to subscribe to these events:

| Method | Parameters |
| :--- | :--- |
| `onBeforeRoll` | A callback with no parameters. |
| `onRoll` | A callback receiving a `rolled: number` argument. |
| `onNextRoll` | Same as `onRoll`, but only triggered for the next immediate invocation of `roll`. |

For example:

```typescript
import { Die } from `@xethya/dice`;

const die = new Die(6);
die.onNextRoll((rolled: number) => {
  if (rolled === 6) {
    // Do something with other system component.
    // Events are useful to composing functionality
    // across different game modules.
  }
});
const roll = die.roll();
// Do something else local to this module.
```

### Presets

The component includes a series of predefined dice, which you can use instead of creating an instance of `Die` with a given number of faces:

| Preset | Range |
| :--- | :--- |
| CoinFlip | 1 - 2 |
| D4 | 1 - 4 |
| D6 | 1 - 6 |
| D8 | 1 - 8 |
| D10 | 1 - 10 |
| D12 | 1 - 12 |
| D20 | 1 - 20 |
| D100 | 1 - 100 |

Any preset can be used importing the proper class from the package root or from the Presets namespace:

```typescript
import { D12 } from '@xethya/dice';
import { presetDice } from '@xethya/dice';

const d12 = new D12();
// or...
const d12 = new presetDice.D12();
```

### Customizing the randomizer

Dice rolls are calculated by default using an implementation of the [Blum Blum Shub](https://en.wikipedia.org/wiki/Blum_Blum_Shub) generator. You can implement any other algorithm of your choice, by passing an instance of it into the settings:

```typescript
import MyPRNG from './my-prng';
import Die from '@xethya/dice';

const customDie = new Die(10, { randomizer: new MyPRNG() });
customDie.roll();
```

Any randomizer must implement the `IPseudoRandomNumberGenerator` interface, implementing a `generateRandom()` function. In order to use this interface in TypeScript, you'll need to install the `@xethya/random-core` package:

```typescript
import { IPseudoRandomNumberGenerator } from '@xethya/random-core';

class MyPRNG implements IPseudoRandomNumberGenerator {
  generateRandom(): number {
    // Your magic here!
  }
}
```

### Rolling multiple dice at once

Although you can instantiate multiple dice in separate variables, it can be somewhat impractical to execute the `roll()` method on each instance. To simplify this use case, there is a `DiceThrow` class available.

```typescript
import { DiceThrow } from '@xethya/dice';

const diceThrow = new DiceThrow({
  diceCount: 6,   // How many dice to roll?
  faces: 6,       // What type of dice are we rolling?
});

// Unlike Die#roll(), instead of a single number, you'll get
// an instance of DiceThrowResult, containing an array of numbers; 
// each number represents a die roll.
diceThrow.roll();   // { rolled: [1, 6, 3, 5, 1, 2] }

// You can also get a sum of all the numbers
// using the getRollSum() function.
const totalPoints = diceThrow.roll().getRollSum(); // 18
```

You can also pass a `randomizer` property into the settings with an instance of `IPseudoRandomNumberGenerator` to replace the default Blum Blum Shub algorithm.

### Calculating success with dice

There's a special kind of dice throw called a _chance throw_ \(represented by the `ChanceThrow` class\). This throw uses a D100 and attaches how successful the roll was. A typical use case for this kind of throw is to determine, for example, if a stab or a punch produce critical damage to an opponent.

```typescript
import { ChanceThrow, DiceThrowTypes } from '@xethya/dice';

const chanceThrow = new ChanceThrow();

// The roll() function returns a single rolled number between
// 1 and 100 and a property "throwType" to determine if the
// throw was a failure, a success or a critical success.
const result = chanceThrow.roll();

if (result.throwType === DiceThrowTypes.FAILURE) {
  console.log('Boo-hoo, you missed!');
} else {
  console.log('Nicely done!');
}
```

The property `throwType` uses an enumerable called `DiceThrowType` . By default, the property's value is determined with these value ranges:

| Range | Resulting throw type | Consequence example |
| :--- | :--- | :--- |
| **1 - 20** | DiceThrowType.FAILURE | A character tried to open a lock with a key, but acted clumsily and broke the key. |
| **21 - 90** | DiceThrowType.SUCCESS | _A characted tried to shoot down a bird with and arrow, and succedeed._ |
| **91 - 100** | DiceThrowType.CRITICAL\_SUCCESS | _A character tried to persuade an enemy to not sound the alarm upon being seen,  and not only achieved that, but turned the enemy into a friend._ |

The value ranges can be customized by supplying a `chanceRanges` property in the configuration object that is passed to the constructor. You'll need to use the `Range` class provided in the `@xethya/utils` package to set the values:

```typescript
import { ChanceThrow } from '@xethya/dice';
import { Range } from '@xethya/utils';

const chanceThrow = new ChanceThrow({
  chanceRanges: {
    failureRange: new Range(1, 80),
    successRange: new Range(81, 95),
    critialSuccessRange: new Range(96, 100),
  }
});
```

