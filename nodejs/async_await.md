# Async/Await

[Modern Concurrency: async and await](https://www.youtube.com/watch?v=NsQ2QIrQShU)

**Concurrency** - doing multiple tasks in a period of time
(generally order-independent or partially-ordered units of work)

- Some things can be done in parallel and others synchronously
- This is hard to do in computing

  - Hard to reason about
  - Leads to race conditions

- Concurrency is important when waiting on input/output such as network requests, reading/writing from disk, or user input

## 2 patterns/ways for programs to wait for IO

- Blocking (synchronous)
  - Easy to write
  - uses multi-threading (pre-requisite for this to work)
  - memory and context-switching overhead
    - Every thread requires memory
    - CPU switching between threads
- Non-blocking/event-loop (asynchronous)

  - Single-threaded (thread cannot be blocked)
  - High-concurrency with low-memory consumption (the win)
  - Great for UI (why the browser uses the event-loop pattern)
  - Great for IO-bound services (reading from network, database, disk) rather than CPU-bound (this blocks other things from happening)

- All modern JS engines use the non-blocking/event-loop approach

## Blocking in JS

Certain things halts execution:

- alert/prompt/confirm (halts everything in the browser)
- synchronous XMLHttpRequest (rare)
- fs.readFileSync and friends in Node

## How to write concurrent software without blocking

### Callbacks

- Just pass a function that will be called when the task is complete
- eg: onClick event, network request

Pros:

- Great low-level abstraction
- Performant with low overhead (no context switching)
- Can do almost any async task with callbacks

Cons:

- Doing things in sequence is hard. Doing things in parallel is harder!
  - Chaining tasks gets messy
- Give up constructs such as for/while and try/catch (what people from other languages are used to)
  - Instead we can do workarounds like setting a conditional to ensure that all elements have been iterated through (if doing parallel calls)
- Error handling is difficult (needs to handle this error parameter)
  - We spend a lot of effort checking if an async task failed (we lose try/catch)
- Code readability suffers and systems become hard to maintain
  - When code readability is bad, were more likely to let errors sneak in

### Promises

- Promises do a little better
- A promise is an object that represents what the value will be when the operation finishes
- A promise is a thin but powerful abstraction on top of callbacks

- Solves several problems:
  - Easy chaining (reads better and in a sequential order)
  - Easy sequential/parallel tasks (helpers to do things in parallel)
    - Flow control - we can easily combine sequential and parallel tasks to create advanced flows
  - Error handling - attach a single catch at the end
    - Exceptions will bubble up similar to how it works in synchronous code
  - Composable - can pass around a representation of a future value

```js
// Promise style
readFile('config.json')
  .then(...)
  .catch(...)
```

```js
sleep(1000)
  .then(() => {
    console.log('one');
    return sleep(1000);
  })
  .then(() => {
    console.log('three');
  });
```

- Inside each callback (`then`-handler) we're returning another promise that resolves in the future
  - this is what allows us to call another `then` handler
  - In the last `then` handler we're not returning anything, so it finishes

```js
// This would be very difficult with callbacks
// The first two things are done in sequence, other things in parallel
// Promises gives us this power in chaining and control flow

fetchJSON('/user-profile').
  then((user) => {
    return fetchJSON(`/users/$user.id}/friends`);
  });
  .then((friendsIDs) => {
    let promises = friendIDs.map((id) => {
      return fetchJSON(`/users/${id};`);
    });
    return Promise.all(promise);
  })
  .then((friends) => console.log(friends));
```

- Problem: We're still putting callbacks inside `.then()`. Can we do better?
- NB: JS is fundamentally single-threaded, so we can't block

## Generator functions

- generator functions are special functions that can be paused
- They're not about asynchronous or event loops or concurrency, just about pausing things with `yield`

```js
function* generatorFuc() {
  let result = fetch('/users');
  // Pause execution by yielding.
  yield result;
  // Later something caused us to resume.
  console.log(`we're back!`);
}
```

- Generators are complex, and the only thing that allows this to work
- They just pause themselves (their function), nothing else

**Promises + Generators = Awesome!**

## async/await

- is basically a thin layer of syntax over Promises and Generator

```js
aync function getUsers() {
  // Here's the magic. It pauses while letting Event Loop do other stuff
  let result = await fetchJSON('/users');
  console.log(result)
}
```

- Can only use `await` inside an `async` function
- It has to work with promises

#### total win

We get back most of our traditional constructs:

- for/while
- try/catch
- readable, sequential program flow
- powerful inter-op with promises

```js
async function readConfig() {
  try {
    let content = await readFile('config.json');
    let obj = JSON.parse(content.toString());
    console.log(obj);
  } catch (error) {
    console.error('An error occurred', error);
  }
}
```

```js
async function animate(element) {
  for (let i = 0; i < 100; i++) {
    element.style.left = i + 'px';
    await sleep(16);
  }
}
```

It's just promises

- _an async function always returns a promise_
- when we await a promise, our function pauses until the promise is ready (resolved)
- We can still use all our favorite promise helpers such as Promise.all()

```js
// the whole async function itself returns a promise, so it can be awaited as well
async function getUserFriends() {
  let user = await fetchJSON('/users/me').
  let friendIDs = await fetchJSON(`/friends/${user.id}`);
  let promises = friendIDs.map((id) => {
    return fetchJSON(`/users/${id}`);
  });
  let friends = await Promise.all(promises); // as long as everything to the right of `await` is a promise(s), it works
  console.log(friends);
}

let promise = getUserFriends();
```

## Pro Tips

- Don't forget to await!
- Be careful about doing too much sequentially when you can actually do it in parallel
  - eg: putting an await inside every iteration of a `for` loop will pause it for every iteration
- Using await in map/filter won't do what you might expect!
  - they return promises, not booleans, so don't use with `filter`
  - ok with `map`, just know you will get an array of promises
- even though it looks synchronous, remember your code has been paused and resumed later
  - some assumptions about state might have changed since the code was paused
