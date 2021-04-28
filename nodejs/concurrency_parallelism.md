# Concurrency is not Parallelism by Rob Pike

[link: Concurrency is not Parallelism](https://www.youtube.com/watch?v=oV9rvDllKEg)

- [Concurrency is not Parallelism by Rob Pike](#concurrency-is-not-parallelism-by-rob-pike)
  - [Intro](#intro)
  - [Concurrency vs Parallelism](#concurrency-vs-parallelism)
    - [An analogy](#an-analogy)
  - [Concurrency plus communication](#concurrency-plus-communication)
  - [Performance observation](#performance-observation)
  - [Back to Computing](#back-to-computing)
  - [Lesson](#lesson)
  - [Conclusion](#conclusion)

## Intro

- the modern world is not object oriented, it's parallel
  - Multicore.
  - Networks.
  - Clouds of CPUs.
  - Loads of users.
- Go is a concurrent language

- Concurrency is better than parallelism

## Concurrency vs Parallelism

- **concurrency** is the composition of independently executing processes (general sense, not Linux processes)
  - about dealing with a lot of things at once
  - about structure
  - concurrency provides a way to structure a solution to solve a problem that may (but not necessarily) be parallelizable.
    - Concurrency's goal is a good structure
- **parallelism** is the _simultaneous_ execution of (possibly related) computations
  - about doing a lot of things at once
  - about execution

### An analogy

Concurrent: Mouse, keyboard, display, and disk drivers.

Parallel: Vector dot product.

## Concurrency plus communication

- _Concurrency_ is a way to structure a program by breaking it into pieces that can be executed independently.
  - parallelizable, but other models are also possible
  - Different concurrent designs enable different ways to parallelize.
- _Communication_ is the means to coordinate the independent executions.
- Concurrent composition of well-managed pieces makes programs run faster
- This is the Go model and (like Erlang and others) it's based on CSP:

C. A. R. Hoare: Communicating Sequential Processes (CACM 1978)

## Performance observation

- We improved performance by adding a concurrent procedure to the existing design (better concurrent expression of the problem).
  - More gophers doing more work; it runs better (red. example in slides)
- This is a deeper insight than mere parallelism
- We get optimization by thinking about how we break a problem down into independent components that can be separated, get right, and then compose together

## Back to Computing

In our book transport problem, substitute:

- book pile => web content
- gopher => CPU
- cart => marshaling, rendering, or networking
- incinerator => proxy, browser, or other consumer
  It becomes a concurrent design for a scalable web service.
  Gophers serving web content.

## Lesson

- A complex problem can be broken down into easy-to-understand components.
- The pieces can be composed concurrently.
- The result is easy to understand, efficient, scalable, and correct.
- Maybe even parallel

## Conclusion

- Concurrency is powerful.
- Concurrency is not parallelism.
- Concurrency enables parallelism.
- Concurrency makes parallelism (and scaling and everything else) easy.
