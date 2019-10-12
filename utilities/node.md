# Node

A `Node` represents an element in a linked structure, such as a [Stack](./stack.md).

It contains a single, immutable property: `value: T`.

## Usage

```typescript
import { Node } from '@xethya/utils';

const numericNode = new Node<number>(42);
console.log(numericNode.value); // 42
```

## Connecting Nodes using a StackNode

A StackNode exposes a `next?: Node<T>` property that allows you to add a reference to another node, creating a singly-linked list.

```typescript
import { StackNode } from '@xethya/utils';

const nodeA = new StackNode<number>(1);
const nodeB = new StackNode<number>(2);
const nodeC = new StackNode<number>(3);

// Nodes are now linked!
nodeA.next = nodeB;
nodeB.next = nodeC;
```

With these nodes, you could traverse the nodes like this:

```typescript
let node = nodeA;
while (node.next) {
  console.log(node.value); // 1, 2, 3
  node = node.next;
}
```