# Optional keys in objects

## Synopsis

Adding new keys to objects is a common operation in ECMA Script. With special syntax ECMA Script became more expressive and more readable.

```ts
{
  foo: 'bar',
  baz?: possibleUndefinedValue,
  bar?,
}
```

## Motivation

### Expressiveness

Say, you have a function, which could add some keys to target object:

```ts
function enrich(target, optionalData) {
  return {
    ...target,
    additionalKey: optionalData
  }
}
```

If optionalData is undefined, `enrich({}).additionalKey` will be undefined as well, but `'additionalKey' in enrich({})` will be `true`, and returned object will be `{ additionalKey: undefined }`

This could lead to bugs when developer try to check optionalData in the following cases:

```ts
if ('additionalKey' in enrich({})) {
    // The key is here, but the data is not
}

if (Object.hasOwnProperty(enrich({}), 'additionalKey')) {
    // Same as above
}

if (Object.keys(enrich({})).length === 0) {
    // The result will be ['additionalKey']
}
```

To prevent this type of problems, we have to modify `enrich` function as:

```ts
function enrich(target, optionalData) {
  const output = { ...target }

  if (optionalData !== undefined) {
    output.additionalKey = optionalData
  }

  return output
}
```

So, now `enrich` function is 9 lines of code instead of 6. Lets consider more complex case with two optional keys:

```ts
function enrich(target) {
  // we don't want to call calcDerived twice on same value
  const derivedOnTarget = calcDerived(target)
  const derivedOnSome = target.some ? calcDerived(target.some) : void 0

  const out = {
    length: Object.keys(target).length
  }

  if (derivedOnTarget !== undefined) {
    out.derived = derivedOnTarget
  }

  if (derivedOnSome !== undefined) {
    out.prop = derivedOnSome
  }

  return out
}
```

With new syntax it is possible to shrink the code from 20 to 8 lines:

```ts
function enrich(target) {
  return {
    length: Object.keys(target).length,
    derived?: calcDerived(target),
    prop?: target.some ? calcDerived(target.some) : void 0,
  }
}
```

## Syntax and semantics

The `?` character right after key in object notation leads to the following: key will be added to object only if the value is not `undefined`.
It may appear in three positions:

```js
{ foo?: 5 } // optional object literal property definition
{ [expr]?: value } // optional object computed property definition
{ foo? } // optional object property value shorthand defintion
```

### Examples

```ts
const a = { foo?: undefined } // {}

const b = { foo?: 5 } // { foo: 5 }

const c = { foo?: condition ? 'bar' : undefined } // { foo: 'bar' } or {}, depends on condition

const d = { [`foo`]?: undefined } // {}

const e = { foo? } // {} if foo === undefined
```


### Evaluation order

Evaluation order of computed property name and property value pair in object definition should be same as without optional key `?` operator.

```js
{ [exprName]?: exprValue, } // exprName evaluated first, exprValue second
```

This corresponds to the [well-established order](https://www.ecma-international.org/ecma-262/5.1/#sec-11.1.5) of execution of property names and values.

So, in the following code

```js
let i = 0
const a = {
  [i++]?: undefined,
  [i++]?: 'a',
}
```

variable `a` should be `{ '1': 'a' }` and not `{ '0': 'a' }`.

Another example that illustrates well the importance of consistent behavior. Having the following expression `{[++i]: ++i}` we will get `{ '1': 2 }` as a result of its execution. But if we are going to calculate the value first using `?` operator we will get `{ '2': 1 }`, which does not meet our expectations.

This way adding optional key `?` operator to object definition should not change result of its evaluation, except removing keys with undefined values.
