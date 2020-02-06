# Ergonomic brand checks for Private Fields

EcmaScript Proposal, specs, and reference implementation to provide brand checks without exceptions.

Spec drafted by [@ljharb](https://github.com/ljharb).

Built on top of https://github.com/tc39/ecma262/pull/1668/ pending Class Fields being stage 4 and merged into the larger spec.

This proposal is currently at stage 0 of the [process](https://tc39.github.io/process-document/).

## Rationale
Private fields have a built-in ”brand check”, in that if you try to access a private field on an object that does not have it installed, it throws an exception.

This is great! However, in order to keep the existence of a private field actually private, I often want to check _if_ an object has a private field, and if not, have some fallback behavior (which might even be throwing a custom exception, with a message that does not reveal that I’m using a private field as my mechanism).

Using `try`/`catch` for this does work, but is quite awkward:
```js
class C {
  #brand;

  static isC(obj) {
    try {
      obj.#brand;
      return true;
    } catch {
      return false;
    }
  }
}
```

What is desired is a simple solution to produce a boolean that does not require a `try`/`catch` or exceptions.

## Possible Solutions:

### `in`
The most obvious solution seems to be using the `in` keyword, like so:
```js
class C {
  #brand;

  static isC(obj) {
    return #brand in obj;
  }
}
```

This has no ASI hazards I am aware of; however, it does have one important tradeoff: it permanently kills any possibility for private field shorthand. Specifically, this is because if `#brand` (in the previous example) is a shorthand for `this.#brand`, then `(#brand) in obj` would have to mean the same as `this.#brand in obj`, which means "is the _value_ of `this.#brand` in `obj`?", not "does `obj` have the private field `#brand`?".

### `try` statement

Another alternative is:
```js
class C {
  #brand;

  static isC(obj) {
    return try obj.#brand;
  }
}
```

However, this strongly suggests that `try <expression>` would be a generic syntax for "if it throws an exception, produce `false`, otherwise produce `true`". That would be a much larger proposal, and would perhaps be a bit overkill to solve the specific problem around private fields.

This would require a lookahead restriction for a curly brace (which would mean that the try expression couldn't be an object literal, unless it was wrapped in parens).
