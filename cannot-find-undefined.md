# preventing "Cannot find x of undefined"

JS throws an error when you try to access a key in an object that does not exist.

Example:

```js
const obj = {
  userId: 1
}

console.log(obj.user.name) // throws an error
```

Let's say i want the value of `obj.user.name` to be just `null` instead of throwing an error.

Here's a snippet that gets the job done:

```js
const getValue = key => obj => {
  return obj
    ? obj.hasOwnProperty(key) ? obj[key] : null
    : null
}

// now
console.log(getValue('name')(obj.user)) // null
```

For extra safety:

```js
const user = getValue('user')(obj)
console.log(
  getValue('name')(user)
) // null

// or the shorter form:
console.log(
  getValue('name')(getValue('user')(obj))
)
````
