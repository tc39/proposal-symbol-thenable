# Symbol.thenable

Gus Caplan, Stage -1

Champion Needed

### Why

The behaviour of `Promise.resolve` includes checking for `then` functions
on whatever you pass it. In most cases this is probably fine but just talk
with any avid user of JS and they won't have only nice things to say about
it.

This is especially true with dynamic import, which will become part of the
language spec very soon. There has been a lot of discussion around how an
import declaration and a dynamic import, because of "thenables," may have
different things passed to a user which won't even be actual namespaces.
__This is absolutely crazy.__

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

This came from trying to think of the most generalised solution to this
problem. Alternatives could be explicitly blacklisting a Module Namespace
Object in `Promise.resolve`, however there was resistance to this, as it might
not be entirely intuitive why this object was being treated differently.
`Symbol.thenable` provides clear explanation for that.

### Alternatives and Options

- [First-Class Protocols Proposal][]

- Mount symbol on `Promise` namespace (`Promise.thenable`)


- Whitelist specific objects we don't want to treat as "thenables":\

  ```
  1. If _resolution_ is a Module Namespace Object, then
    1. Return FulfullPromise(_promise_, _resolution_).
  ```

[First-Class Protocols Proposal]: https://github.com/michaelficarra/proposal-first-class-protocols
