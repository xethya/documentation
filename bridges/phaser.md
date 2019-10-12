# Integrating with Phaser.js

Install the package with `npm` or `yarn`:

```bash
npm install @xethya/bridge-phaser
```

```bash
yarn add @xethya/bridge-phaser
```

Add it to your game's configuration like this:

```javascript
  plugins: {
    global: [
      {
        key: "XethyaPlugin",
        plugin: XethyaPlugin,
        start: true,
      },
    ],
  },
```

When on a `Scene` (or any object that extends from `GameObjectFactory`), you'll have these functions available:

- Dice: `add.die(faces)`, `add.coinFlip()`, `add.d4()`, `add.d6()`, `add.d8()`, `add.d10()`, `add.d12()`, `add.d20()`, `add.d100()`.
- Inventory: `add.inventory(options)`.