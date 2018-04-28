- [index](https://github.com/KiraDiShira/ConcurrencyAndAsynchrony#concurrency-and-asynchrony)

# Tasks

- [Introduction](#introduction)

## Introduction

A thread is a low-level tool for creating concurrency, and as such it has limitations. In particular

- While it’s easy to pass data into a thread that you start, there’s no easy way to get a “return value” back from a thread that you Join. You have to set up some
kind of shared field. And if the operation throws an exception, catching and propagating that exception is equally painful.
- You can’t tell a thread to start something else when it’s finished; instead you
must Join it (blocking your own thread in the process).

```c#
Task.Run (() => Console.WriteLine ("Foo"));
```

Calling **Wait** on a task blocks until it completes and is the equivalent of calling Join on a thread:

```c#
Task task = Task.Run(() =>
{
    Thread.Sleep(2000);
    Console.WriteLine("Foo");
});
Console.WriteLine(task.IsCompleted); // False
task.Wait(); // Blocks until task is complete
```

Wait lets you optionally specify a timeout and a cancellation token to end the wait early.

By default, the CLR runs tasks on pooled threads, which is ideal for short-running compute-bound work. For longer-running and blocking operations (such as our preceding example), you can prevent use of a pooled thread as follows:

```c#
Task task = Task.Factory.StartNew (() => ..., TaskCreationOptions.LongRunning);
```

Running one long-running task on a pooled thread won’t cause trouble; it’s when you run multiple long-running tasks in parallel (particularly ones that block) that performance can suffer. And in that case, there are usually better solutions than TaskCreationOptions.LongRunning:

• If the tasks are I/O-bound, **TaskCompletionSource** and asynchronous functions let you implement concurrency with callbacks (continuations) instead of threads.
• If the tasks are compute-bound, a **producer/consumer queue** lets you throttle the concurrency for those tasks, avoiding starvation for other threads and processes (see “Writing a Producer/Consumer Queue” on page 950 in Chapter 23).

## Returning values

```c#
Task<int> task = Task.Run (() => { Console.WriteLine ("Foo"); return 3; });
```

```c#
int result = task.Result; // Blocks if not already finished
Console.WriteLine (result); // 3
```

## Exceptions

Unlike with threads, tasks conveniently propagate exceptions. 

```c#
// Start a Task that throws a NullReferenceException:
Task task = Task.Run(() => { throw null; });
try
{
    task.Wait();
}
catch (AggregateException aex)
{
    if (aex.InnerException is NullReferenceException)
        Console.WriteLine("Null!");
    else
        throw;
}
```

## Continuations
