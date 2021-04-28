# Event Loop

[NodeJS Event Loop: Not So Single Threaded](https://www.youtube.com/watch?v=zphcsoSJMvM)

## Evolution of processors

1. In the beginning there was only a single process

2. Then came _Cooperative Multitasking_ - where an app could call yield to give other programs/os a chance to run

- problem: It relies on the app calling yield + if an app crashed, there was no way for the system to recover
- DOS and MacOS until MacOS 9 relied on this system

3. Then came _Preemptive Multitasking_ - Now the OS itself has the ability to pause any app at any time (and save its state in memory) and run another app

- OS handles everything in this system
- Was there for a long time in UNIX and mainframe world, made it to personal computers later (Windows NT kernel, Windows 2000/XP, MacOS 10+)
- While they ran on single core CPUs, it made it look like they were multi-core

4. _Symmetric Multi Threading (SMT)_ - attempt to harness multi-core processors

- Intel was first with this technology, they called it _hyper-threading_
- OS takes advantage of new assembly instructions to give extra info to processes on how to run it in parallel (loading from separate cores)
- Speed is typically 0-20 % faster (not 2x speed with 2 processors)
- We can now run lots of code simultaneously

## Processes vs Threads

- _Tasks_ and _processes_ are used interchangeably
- Task is more generic and process more specific
- An _application_ is a process. It could technically use more than one process, but usually it's a one-to-one relationship

| Processes                                         | Threads                                              |
| ------------------------------------------------- | ---------------------------------------------------- |
| Top-level execution container                     | Runs inside a process                                |
| Separate memory space                             | Shared memory space (performant)                     |
| Communicate via Inter-Process Communication (IPC) | Plethora of communication options, runtime dependent |

- Viruses work by trying to break out of the process memory container
- If we have 2 or more process and want them to communicate with each other, we need IPC to do that, typically via a socket (usually UNIX, but could be a TCP socket)
- By default a process runs one thread inside of it
- We still need to perform synchronization when sharing data (eg. global variables) between threads in order to avoid race conditions (which make the program unpredictable)
  - Manual code synchronizes when different parts of code accesses data (making one part wait)
  - This is tricky and difficult to write correct, multi-threaded code that's bug-free
- NodeJS has a simple solution: They won't allow multi-threading to begin with!

## NodeJS is Single Threaded (except when it's not)

- It's true that NodeJS is single threaded but sometimes it's not
- **Main Thread** - All JS, V8 and event loop run in one thread, the main thread
- But there's about 1/3 C++ in NodeJS
  - C++ has access to threads
  - C++ backed _synchronous_ methods run in the main thread
  - C++ backed _asynchronous_ methods sometimes don't run in the main thread (depends on context of method call) but on separate threads
- It's recommended in NodeJS to always use asynchronous methods by default so that NodeJS can attempt to run methods in parallel -> performance benefits
- Example: With 4 asynchronous methods on a dual core CPU, it will run two threads on each process using pre-emptive multi-tasking (switching between them), making it look like they're running in parallel by finishing roughly at the same time
- **Thread Pool** - NodeJS uses a pre-allocated set of threads called the thread pool
  - The default is set to 4
  - It will constantly re-use these worker threads for all work
  - When the 5th request comes through, it will be placed in a queue until one of the workers becomes available
  - NB: C++ backed methods use C++ asynchronous primitives whenever possible (provided by OS itself, happening inside the kernel), which takes place in the main thread itself. An example is http requests using `http` module. This means we're not necessarily limited to only the number of threads in the thread pool.

## Event Loop

- Event loop is the central dispatch that routes requests to C++ and results back to JS
- Event loop is the director.
  - When we make a JS request, it will eventually cross from JS to C++ which is when the request goes to the event loop.
    ->
  - Event Loop then inspects whether the request is synchronous/asynchronous.
    - if Synchronous -> ships off to other C++ code that handles the request on the spot
    - If asynchronous
      - A) If it can be run using C++ asynchronous primitive -> ships to that C++ code to handle that inside the main thread
      - B) If it can't be run by C++ asynchronous primitive -> it ships to background thread (queued to be sent one of the threads in the thread pool).
      - When it's done (either through A or B), Event Loop notifies V8/JS-land that the operation is done and send the result.
  - In JS, NodeJS then calls all callbacks that are registered and waiting for that result
  - So basically we can think of it as a cirlle
  - NB: Event Loops also manages timers, when to shut down etc.

## APIs that use async mechanism

### Kernel Async

- TCP servers and clients
- pipes
- dns.resolve

### Thread Pool

- fs.\* (so limited, unfortunately no C++ async primitives for FileIO)
- dns.lookup
- pipes (edge cases, typically dealing with file system)
