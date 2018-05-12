# Symbol.thenable

Champions: Myles Borins, Jordan Harband

Author: Gus Caplan

Stage 0

### Why

The behaviour of `Promise.resolve` includes checking for `then` functions
on whatever you pass it. This allows interop with legacy third-party
Promise implementation protocols, and in most cases does not result in any
issues, but there remain scenarios where it is unwanted.

This is especially true with dynamic import of module namespace objects,
which will become part of the language spec very soon. A namespace might
be interpreted as a "thenable" resulting in a non-namespace return from
`import()` - which may be very unexpected. There has been a lot of
previous discussion around these scenarios, including
https://github.com/tc39/proposal-dynamic-import/issues/47 and
https://github.com/tc39/proposal-dynamic-import/issues/48.

```js
import * as static from 'X'

import('X').then((dynamic) => {
  assert(static === dynamic); // might be false, /hopefully/ the consumer of X
                              // and the author of X are both aware of this
                              // behaviour (and if they are, no one exploits
                              // it to do confusing things)
});
```

### Proposal

The natural conclusion of this is to add some sort of modifier to an object
such that `Promise.resolve` is aware that it should not perform "thenable"
behaviour.

Enter: `Symbol.thenable`

```js
Promise.resolve({
  [Symbol.thenable]: false,
  then() { return 'a time that isn\'t now'; },
}).then((o) => {
  o.then() === 'a time that isn\'t now';
});
```

In addition this symbol would be set by default on Module Namespace objects.
While users can already `import *` a namespace object and return it through a
promise resolution, this can still be considered an incredibly rare case today.

This came from trying to think of the most generalised solution to this
problem. Alternatives could include explicitly blacklisting a Module Namespace
Object in `Promise.resolve`, however there was resistance to this, as it might
not be entirely intuitive why this object was being treated differently.
`Symbol.thenable` provides clear explanation for that.

In the future, this symbol can also be used by a protocol (see the
[First-Class Protocols Proposal][]) along the lines of `Promise.Thenable` as a
sort of "disabled" symbol.

### Alternatives and Options

- Mount symbol on `Promise` namespace (`Promise.thenable`)

- Whitelist specific objects we don't want to treat as "thenables":

  ```
  1. If _resolution_ is a Module Namespace Object, then
    1. Return FulfullPromise(_promise_, _resolution_).
  ```

[First-Class Protocols Proposal]: https://github.com/michaelficarra/proposal-first-class-protocols
