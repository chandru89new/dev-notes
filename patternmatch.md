# pattern matching in javascript

Pattern matching is a very powerful concept found in functional programming languages like Haskell, Ocaml etc.

It works like a brute-force on function arguments for some values of the arguments.

Example: a factorial function.

In a language like Haskell or Purescript, we can write a factorial function like so:

```purs
fact :: Int -> Int
fact 0 = 1 -- <-- pattern matching in action
fact 1 = 1 -- <-- pattern matching in action
fact n = n * fact (n - 1) -- <-- the general case
```

`fact` is the name of the function. It takes one Int as argument and returns another Int as result. (that's what is denoted by `Int -> Int`)

We describe what happens when `fact` is given `0` and then `1` and then write the general case for `n` which is any number other than 0 and 1.

That's pattern matching in action.

In normal JS, we'd write a factorial function like this:

```js
const fact = n => {
  if (n === 0) {
    return 1
  }
  if (n === 1) {
    return 1
  }
  return n * fact(n - 1)
}
```

Or

```js
const fact = n => {
  if (n === 0 || n === 1) {
    return 1
  }
  return n * fact(n - 1)
}
```

That is, pattern matching is done by regular `if` statements in JS.

But this gets very unwieldy after a point, when you have a lot of patterns to match.

We can write our own pattern matching helper that can help us write code that reads better.

Here's one way of how I'd like to write a factorial function:
(although this doesnt work)

```js
const fact = n => {
  return patternMatch(
    [n === 0, 1],
    [n === 1, 1],
    [n, n * fact(n - 1)]
  )
}
```

Basically, we got all `if`s removed and replaced with pairs of statements inside arrays. The first statement in every pair is the condition, the second statement is the result.

There are two problems here:

1. The statements are immediately evaluated. This can cause trouble in cases where the condition has to be evaluated in a complex manner.
2. Because of the immediate evaluation, in recursive function definitions (like the one above), patternMatch will blow the stack right away. Our program won't even compile.

We can prevent this by making everything wrapped in a function. Like so:

```js
const fact = n => {
  return patternMatch(
    [() => n === 0, () => 1],
    [() => n === 1, () => 1],
    [() => n, () => n * fact(n - 1)]
  )
}
```

So now, all `patternMatch` has to do is:

- take the first argument from the given list of arguments
- evaluate the first part of the pair and check if it's true
- if it's true, evaluate the second part of the pair and return that value
- if not, move to the next argument and repeat the process

If it runs through all pairs and it still cannot find a "true" evaluation, that means the user has not supplied all cases/patterns.

Okay, so given this information, let's write our pattern matching helper. Code comments describe what we're doing:

```js
const patternMatch = (...args) => {
  // patternMatch gets comma-separated arguments. We convert all of that into an array by using `...args` so `args` is an array of arguments.

  // we pick the first pair
  const head = args[0] || null

  // if there is no head, patternMatch was called without arguments/patterns. This is an error
  if (!head) { throw new Error('You have not written all cases for the pattern matcher') }

  // first part of the pair. ie, the condition to check/test
  const fst = head[0]

  // second part of the pair. ie, the function to run if condition is true
  const snd = head[1]

  // we run fst and if it's true, we run snd() and return its value
  if (fst()) return snd()
  
  // otherwise, remove the first/head of the arguments, and pass the rest to patternMatch again.
  else return patternMatch(...args.slice(1))
}
```

Without comments:
```js
const patternMatch = (...args) => {
  const head = args[0] || null
  if (!head) { throw new Error('You have not written all cases for the pattern matcher') }
  const fst = head[0]
  const snd = head[1]
  if (fst()) return snd()
  else return patternMatch(...args.slice(1))
}
```
