# a pipe of our own

A pipe function is something that runs functions one after the other.

Here's a very contrived example:

Let's say we want to do these things:
- take a number, double it
- then add 1 to it and square the result
- then get one third of the result
- and finally find out if that result is divisible by 2


We define the functions:

```js
const double = num => num * 2 // doubles the given number
const add1square = num => (num+1) * (num+1) // squares a function
const onethird = num => num * (1/3) // one third of the number
const mod2 = num => num % 2 
```

Now, to do the steps, we'll typically do this:

```js
const x = 3
const result = mod2(onethird(add1square(double(x))))
console.log(result)
```

That looks ugly. Anything better is verbose. Like this:

```js
const x = 3
const doublex = double(x)
const add1squarex = add1square(doublex)
const onethirdx = onethird(add1squarex)
const mod2x = mod2(onethirdx)
console.log(mod2x)
```

But we could do something that's better:

```js
const x = 3
const pipedFunction = pipe(
  double,
  add1square,
  onethird
  mod2
)
console.log(pipedFunc(3))
```

Or the shorter:

```js
console.log(
  pipe(
    double,
    add1square,
    onethird,
    mod2
  )(3)
)
```

JS doesn't have a `pipe` function. Libraries like `lodash` and `ramda` have them. But let's write our own. A simple one but it works.

```js
const pipe = (...fns) => arg => {
  return fns.reduce((acc, currFn) => currFn(acc), arg)
}
```

What's happening?

- we take an array of functions using the [spread operator][spread-operator] `(...fns)`
- then we take an argument `arg` (which we use as the first argument we pass to the actual workflow in the pipe)
- then, because functions is an array, we just use the [reducer][array-reduce] to take a value, apply a function on that value, get an output and pass that output to the next function in the array

In its current form, our `pipe` returns a function that can take only one argument `arg`. We can make this take as many args as the user gives. Here's how:

```js
const pipe = (...fns) => (...args) => {
  return fns.reduce((arg, currFn) => {
    if (arg.length > 1) return currFn(...arg)
    else return currFn(arg)
  }, args)
}
````

Here's what's happening in that code:

```js
// we make args an array using the spread-operator (same as we did with functions)
const pipe = (...fns) => (...args) => { 
  return fns.reduce((arg, currFn) => {
    // we check if the current argument is an array. if yes, we spread it before passing to the current function
    if (Array.isArray(arg)) return currFn(...arg) 
    // otherwise just send it
    else return currFn(arg) 
  }, args) // we pass in the arguments as an array (so if you sent fn(2,3), the args will be [2,3])
}
````

Here's some interesting things (limitations) that you should notice about pipe functions:
(and these sort of give us an insight into functional programming)

- you'll notice that the next function in the pipe expects a single argument only. That means the preceding function must return a value - and a value that this current function can accept and work on.
- the pipe is what's called a higher-order function : that is, it's a function that returns another function which we can (and did) use
- also notice how the pipe (and the functions inside the pipe) produce a new value. They don't mutate or change any of the actual inputs you gave: they simply use them and produce a new value from them. This is called pure functions. 


[spread-operator]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Spread_syntax
[array-reduce]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/Reduce