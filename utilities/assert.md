# Assert

This is a very simple function that throws an `AssertionError` typed error with a custom message when a condition is not met.

Since it throws an error, usage of `assert` **will** halt execution of the current script.

## Usage

This is the syntax for using `assert`:

```typescript
import { assert } from '@xethya/utils';

assert(condition, 'message');
```

## Example

Let's assume you're creating a custom version of the [`Die`](../components/dice.md) class, adding a `color` property,
and you want to make sure the provided color is a valid hexadecimal number.

Use the `assert` function to test the parameter's value against a regular expression, and provide a readable error message when the condition isn't met.

```typescript
import { assert } from '@xethya/utils';
import { Die } from '@xethya/dice';

const hexColorPattern = /^#?[0-9a-fA-F]{3}[0-9a-fA-F]{3}?$/;

class ColoredDie extends Die {
  public readonly color: string;

  constructor(faces: number, color: string) {
    super(faces);
    assert(color.match(hexColorPattern), 'You must provide a valid hexadecimal color');
  }
}
```