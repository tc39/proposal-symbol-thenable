# Symbol.thenable

Gus Caplan, Stage 0

### Why

The behaviour of `Promise.resolve` includes checking for `then` functions
on whatever you pass it. In most cases this is probably fine but just talk
with any avid user of JS and they won't have only nice things to say about
it.

This is especially true with dynamic import, which will become part of the
language spec very soon. There has been a lot of discussion around how an
import declaration and a dynamic import, because of "thenables," may have
different namespaces passed to a user. __This is absolutely crazy.__

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
