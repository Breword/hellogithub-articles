```yaml
> * 原文地址：https://hackernoon.com/9-functional-programming-concepts-everyone-should-know-uy503u21
> * 原文作者：Victor Cordova
> * 译者：陈元
> * 校对：HelloGitHub-小鱼干
```

# 9 Functional Programming Concepts Everyone Should Know

![](https://static.breword.com/9-functional-programing-cover.jpeg)

This article will introduce functional programming concepts that every programmer should know. Let's begin by defining what functional programming is (FP from now on). FP is a programming paradigm where software is written by applying and composing functions. A **paradigm** is a "Philosophical or theoretical framework of any kind." In other words, FP is a way for us to think of problems as a matter of interconnecting functions.

**Here, I will give a basic understanding of fundamental concepts in FP and some of the problems it helps solve.**

Note: For practicality, I'll omit specific mathematical properties that define these concepts. This is not necessary for you to use these concepts and apply them in your programs.

## 1. Immutability

A mutation is a modification of the value or structure of an object. Immutability means that something cannot be modified. Consider the following example:

```javascript
const cartProducts = [
  {
    "name": "Nintendo Switch",
    "price": 320.0,
    "currency": "EUR"
  },
  {
    "name": "Play station 4",
    "price": 350.0,
    "currency": "USD"
  }
]

// Let's format the price field so it includes the currency e.g. 320 €
cartProducts.forEach((product) => {
  const currencySign = product.currency === 'EUR' ? '€' : '$'
  // Alert! We're mutating the original object
  product.price = `${product.price} ${currencyName}`
})

// Calculate total
let total = 0
cartProducts.forEach((product) => {
  total += product.price
})

// Now let's print the total
console.log(total) // Prints '0320 €350 $' 😟
```

What happened? Since we're mutating the  `cartProducts`  object, we **lose the original value** of  `price`.

**Mutation can be problematic because it makes tracing the state changes in our application hard or even impossible.** You don't want to call a function in a third party library and not know if it will modify the object you're passing.

Let's look at a better option:

```javascript
const cartProducts = [...]

const productsWithCurrencySign = cartProducts.map((product) => {
  const currencyName = product.currency === 'EUR' ? 'euros' : 'dollars'
  // Copy the original data and then add priceWithCurrency
  return {
    ...product,
    priceWithCurrency: `${product.price} ${currencyName}`
  }
})

let total = 0
cartProducts.forEach((product) => {
  total += product.price
})

console.log(total) // Prints 670 as expected 😎
```

Now, instead of modifying the original object, we clone the data in the original `cartProducts` by using the spread operator

```javascript
return {
  ...product,
  priceWithCurrency: `${product.price} ${currencyName}`
}
```

With this second option, we avoid mutating the original object by creating a new one that has the `priceWithCurrency` property.

Immutability can actually be mandated by the language. JavaScript has the `Object.freeze` utility, but there are also mature libraries such as `Immutable.js` you can use instead. Nevertheless, before enforcing immutability everywhere, evaluate the tradeoff of adding a new library + the extra syntax; maybe you'd be better off creating an agreement in your team not to mutate objects if possible.

## 2. Function Composition

It's the application of a function to the output of another function. Here's a small example:

```javascript
const deductTaxes = (grossSalary) => grossSalary * 0.8
const addBonus = (grossSalary) => grossSalary + 500

const netSalary = addBonus(deductTaxes(2000))
```

In practice, this means we can split out algorithms into smaller pieces, reuse them throughout our application, and test each part separately.

## 3. Deterministic Functions

A function is deterministic if, given the same input, it returns the same output. For example:

```javascript
const joinWithComma = (names) => names.join(', ')

console.log(joinWithComma(["Shrek", "Donkey"])) // Prints Shrek, Donkey
console.log(joinWithComma(["Shrek", "Donkey"])) // Prints Shrek, Donkey again!
```

A common non-deterministic function is `Math.random`:

```javascript
console.log(Math.random()) // Maybe we get 0.6924493472043922
console.log(Math.random()) // Maybe we get 0.4146573369082662
```

Deterministic functions help your software's behavior be more predictable and make the chance of bugs lower.

It's worth noting that we don't always want deterministic functions. For example, when we want to generate a new ID for a database row or get the current date in milliseconds, we need a new value to be returned on every call.

## 4. Pure functions

A pure function is a function that is **deterministi**c and **has no side effects**. We already saw what deterministic means. A side effect is a modification of state outside the local environment of a function.

Let's look at a function with a nasty side-effect:

```javascript
let sessionState = 'ACTIVE'

const sessionIsActive = (lastLogin, expirationDate) => {
  if (lastLogin > expirationDate) {
    // Modify state outside of this function 😟
    sessionState = 'EXPIRED'
    return false
  }
  return true
}

const expirationDate = new Date(2020, 10, 01)
const currentDate = new Date()
const isActive = sessionIsActive(currentDate, expirationDate)

// This condition will always evaluate to false 🐛
if (!isActive && sessionState === 'ACTIVE') {
  logout()
}
```

As you can see, `sessionIsActive` modifies a variable outside its scope, which causes problems for the function caller.

Now here's an alternative without side effects:

```javascript
let sessionState = 'ACTIVE'

function sessionIsActive(lastLogin, expirationDate) {
  if (lastLogin > expirationDate) {
    return false
  }
  return true
}

function getSessionState(currentState, isActive) {
  if (currentState === 'ACTIVE' && !isActive) {
    return 'EXPIRED'
  }
  return currentState
}

const expirationDate = new Date(2020, 10, 01)
const currentDate = new Date()
const isActive = sessionIsActive(currentDate, expirationDate)
const newState = getSessionState(sessionState, isActive)

// Now, this function will only logout when necessary 😎
if (!isActive && sessionState === 'ACTIVE') {
  logout()
}
```

It's important to understand we don't want to eliminate all side effects since all programs need to do some sort of side-effect such as calling APIs or printing to some stdout. What we want is to minimize side effects, so our program's behavior is easier to predict and test.

## 5. High-order Functions

Despite the intimidating name, high-order functions are just functions that either: take one or more functions as arguments, or returns a function as its output.

Here's an example that takes a function as a parameter and also returns a function:

```javascript
const simpleProfile = (longRunningTask) => {
  return () => {
    console.log(`Started running at: ${new Date().getTime()}`)
    longRunningTask()
    console.log(`Finished running at: ${new Date().getTime()}`)
  }
}

const calculateBigSum = () => {
  let total = 0
  for (let counter = 0; counter < 100000000; counter += 1) {
    total += counter
  }
  return total
}


const runCalculationWithProfile = simpleProfile(calculateBigSum)

runCalculationWithProfile()
```

As you can see, we can do cool stuff, such as adding functionality around the execution of the original function. We'll see other uses of higher-order in curried functions.

## 6. Arity

Arity is the number of arguments that a function takes.

```javascript
// This function has an arity of 1. Also called unary
const stringify = x => `Current number is ${x}`

// This function has an arity of 2. Also called binary
const sum => (x, y) => x + y
```

That's why in programming, you sometimes hear unary operators such as ++ or !

## 7. Curried Functions

Curried functions are functions that take multiple parameters, only that one at a time (have an arity of one). They can be created in JavaScript via high-order functions.

Here's a curried function with the ES6 arrow function syntax:

```javascript
const generateGreeting = (ocassion) => (relationship) => (name) => {
  console.log(`My dear ${relationship} ${name}. Hope you have a great ${ocassion}`)
}

const greeter = generateGreeting('birthday')

// Specialized greeter for cousin birthday
const greeterCousin = greeter('cousin')
const cousins = ['Jamie', 'Tyrion', 'Cersei']

cousins.forEach((cousin) => {
  greeterCousin(cousin)
})
/* Prints:
  My dear cousin Jamie. Hope you have a great birthday
  My dear cousin Tyrion. Hope you have a great birthday
  My dear cousin Cersei. Hope you have a great birthday
*/

// Specialized greeter for friends birthday
const greeterFriend = greeter('friend')
const friends = ['Ned', 'John', 'Rob']
friends.forEach((friend) => {
  greeterFriend(friend)
})
/* Prints:
  My dear friend Ned. Hope you have a great birthday
  My dear friend John. Hope you have a great birthday
  My dear friend Rob. Hope you have a great birthday
*/
```

Great right? We were able to customize the functionality of our function by passing one argument at a time.

More generally, curried functions are great for giving functions polymorphic behavior and to simplify their composition.

## 8. Functors

Don't get intimidated by the name. Functors are just abstractions that wrap a value into a context and allows mapping over this value. Mapping means applying a function to a value to get another value. Here's how a very simple Functor looks like:

```javascript
const Identity = value => ({
  map: fn => Identity(fn(value)),
  valueOf: () => value
})
```

Why would you go over the trouble of creating a Functor instead of just applying a function? To facilitate function composition. Functors are agnostic of the type inside of them so you can apply transformation functions sequentially. Let's see an example:

```javascript
const double = (x) => {
  return x * 2
}

const plusTen = (x) => {
  return x + 10
}

const num = 10
const doubledPlus10 = Identity(num)
  .map(double)
  .map(plusTen)

console.log(doubledPlus10.valueOf()) // Prints 30
```

This technique is very powerful because you can decompose your programs into smaller reusable pieces and test each one separately without a problem. In case you were wondering, JavaScript's `Array` object is also a Functor.

## 9. Monads

A Monad is a Functor that also provides a `flatMap` operation. This structure helps to compose type lifting functions. We'll now explain each part of this definition step by step and why we might want to use it.

**What are type lifting functions?**

Type lifting functions are functions that wrap a value inside some context. Let's look at some examples:

```javascript
// Here we lift x into an Array data structure and also repeat the value twice.
const repeatTwice = x => [x, x]

// Here we lift x into a Set data structure and also square it.
const setWithSquared = x => new Set(x ** 2)
```

Type lifting functions can be quite common, so it makes sense we would want to compose them.

**What is a flat function?**

The `flat` function (also called join) is a function that extracts the value from some context. You can easily understand this operation with the help of JavaScript's `Array.prototype.flat` function.

```javascript
// Notice the [2, 3] inside the following array. 2 and 3 are inside the context of an Array
const favouriteNumbers = [1, [2, 3], 4]

// JavaScript's Array.prototype.flat method will go over each of its element, and if the value is itself an array, its values will be extracted and concatenated with the outermost array.
console.log(favouriteNumbers.flat()) // Will print [1, 2, 3, 4]
```

**What is a flatMap function?**

It's a function that first applies a mapping function (map), then removes the context around it (flat). Yeah, I know it's confusing that the operations are not applied in the same order as the method name implies.

**How are monads useful?**

Imagine we want to compose two type lifting functions that square and divide by two inside a context. Let's first try to use map and a very simple functor called Identity.

```javascript
const Identity = value => ({
  // flatMap: f => f(value),
  map: f => Identity.of(f(value)),
  valueOf: () => value
})

// The `of` method is a common type lifting functions to create a Monad object.
Identity.of = value => Identity(value)

const squareIdentity = x => Identity.of(x ** 2)
const divideByTwoIdentity = x => Identity.of(x / 2)

const result = Identity(3)
  .map(squareIdentity)
  .map(divideByTwoIdentity) // 💣 This will fail because will receive an Identity.of(9) which cannot be divided by 2
  .valueOf()
```

We cannot just use the map function and need to first extract the values inside of the Identity. Here's where the flatMap function comes into place.

```javascript
const Identity = value => ({
  flatMap: f => f(value),
  valueOf: () => value
})

...

const result = Identity(3)
  .flatMap(squareIdentity)
  .flatMap(divideByTwoIdentity)
  .valueOf()

console.log(result); // Logs out 4.5
```

We are finally able to compose type lifting functions, thanks to monads.

## Conclusion

I hope this article gives you a basic understanding of some fundamental concepts in functional programming and encourages you to dig deeper into this paradigm so you can write more reusable, maintainable, and easy-to-test software.

