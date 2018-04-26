- [index](https://github.com/KiraDiShira/ConcurrencyAndAsynchrony#concurrency-and-asynchrony)

# Threading

- [Introduction](#introduction)
- [Join and Sleep](#join-and-sleep)
- [Blocking](#blocking)
- [Local Versus Shared State](#local-versus-shared-state)
- [Locking and Thread Safety](#locking-and-thread-safety)
- [Exception Handling](#exception-handling)
- [Foreground Versus Background Threads](#foreground-versus-background-threads)
- [Thread Priority](#thread-priority)
- [Threading in Rich-Client Applications](#threading-in-rich-client-applications)
- [Synchronization Contexts](#synchronization-contexts)
- [The Thread Pool](#the-thread-pool)

## Introduction

Applications need to deal with more than one thing happening at a time (**concurrency**).

The mechanism by which a program can simultaneously execute code is called **multithreading**.

A **thread** is an execution path that can proceed independently of others. Each thread runs within an operating system **process**, which provides an isolated environment in which a program runs. 

With a single-threaded program, just one thread runs in the process’s isolated environment and so that thread has exclusive access to it. With a multithreaded program, multiple threads run in a single process, sharing the same execution environment (memory, in particular). This, in part, is why multithreading is useful: one thread can fetch data in the background, for instance, while another thread displays the data as it arrives. This data is referred to as **shared state**.

A client program (Console, WPF, UWP, or Windows Forms) starts in a single thread that’s created automatically by the operating system (the **main thread**). Here it lives out its life as a single-threaded application, unless you do otherwise, by creating more threads (directly or indirectly - The CLR creates other threads behind the scenes for garbage collection and finalization).

You can create and start a new thread:

```c#

static void Main(string[] args)
{
    Thread thread = new Thread(WriteY); // Kick off a new thread
    thread.Start(); // running WriteY()
    // Simultaneously, do something on the main thread.
    for (int i = 0; i < 1000; i++) Console.Write("x");

    Console.ReadLine();
}

static void WriteY()
{
    for (int i = 0; i < 1000; i++) Console.Write("y");
}

```

```
xxxxxxxxxxxxxxxxyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyy
xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxyyyyyyyyyyyyy
yyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyxxxxxxxxxxxxxxxxxxxxxx
xxxxxxxxxxxxxxxxxxxxxxyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyy
yyyyyyyyyyyyyxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
```

On a single-core computer, the operating system must allocate **slices** of time to each thread (typically 20 ms in Windows) to simulate
concurrency, resulting in repeated blocks of x and y. On a multicore or multiprocessor machine, the two threads can genuinely execute in parallel, although you still get repeated blocks of x and y in this example because of subtleties in the mechanism by which Console handles concurrent requests.

Once started, a thread’s **IsAlive** property returns true, until the point where the thread ends. A thread ends when the delegate passed to the Thread’s constructor finishes executing. Once ended, a thread cannot restart.

Each thread has a **Name** property that you can set for the benefit of debugging.

## Join and Sleep

```c#
static void Main(string[] args)
{
    Thread t = new Thread(Go);
    t.Start();
    t.Join();
    Console.WriteLine("Thread t has ended!");
    Console.ReadLine();
}

static void Go() { for (int i = 0; i < 1000; i++) Console.Write("y"); }
```

`
yyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyThread t has ended!
`

**Thread.Sleep** pauses the current thread for a specified period:

```c#
Thread.Sleep (TimeSpan.FromHours (1)); // Sleep for 1 hour
Thread.Sleep (500); // Sleep for 500 milliseconds
```

**Thread.Sleep(0)** relinquishes the thread’s current time slice immediately, voluntarily handing over the CPU to other threads. Thread. **Yield()** does the same thing —except that it relinquishes only to threads running on the same processor.

While waiting on a Sleep or Join, a thread is **blocked**.

## Blocking

A thread is deemed blocked when its execution is paused for some reason, such as when Sleeping or waiting for another to end via Join. A blocked thread immediately yields its processor time slice, and from then on consumes no processor time until its blocking condition is satisfied.

When a thread blocks or unblocks, the operating system performs a **context switch**. This incurs a small overhead, typically one or two microseconds.

## Local Versus Shared State

The CLR assigns each thread its own memory stack so that local variables are kept separate.

Threads share data if they have a common reference to the same object instance:

```c#

class ThreadTest
{
    bool _done;
    static void Main()
    {
        ThreadTest tt = new ThreadTest(); // Create a common instance
        new Thread(tt.Go).Start();
        tt.Go();
    }
    void Go() // Note that this is an instance method
    {
        if (!_done) { _done = true; Console.WriteLine("Done"); }
    }
}

```

Local variables captured by a lambda expression or anonymous delegate are converted by the compiler into fields, and so can also be shared:

```c#
class ThreadTest
{
    static void Main()
    {
        bool done = false;
        ThreadStart action = () =>
        {
            if (!done) { done = true; Console.WriteLine("Done"); }
        };
        new Thread(action).Start();
        action();
    }
}
```

Static fields offer another way to share data between threads:

```c#

class ThreadTest
{
    static bool _done; // Static fields are shared between all threads
    // in the same application domain.
    static void Main()
    {
        new Thread(Go).Start();
        Go();
    }
    static void Go()
    {
        if (!_done) { _done = true; Console.WriteLine("Done"); }
    }
}

```

it’s better to avoid shared state altogether where possible. We’ll see later how asynchronous programming patterns help with this.

## Locking and Thread Safety

We can fix the previous example by obtaining an exclusive lock while reading and writing to the shared field.

```c#

class ThreadSafe
{
    static bool _done;
    static readonly object _locker = new object();
    static void Main()
    {
        new Thread(Go).Start();
        Go();
    }
    static void Go()
    {
        lock (_locker)
        {
            if (!_done) { Console.WriteLine("Done"); _done = true; }
        }
    }
}

```

When two threads simultaneously contend a **lock** (which can be upon any reference-type object, in this case, _locker), one thread waits, or blocks, until the lock becomes available. In this case, it ensures only one thread can enter its code block at a time, and “Done” will be printed just once. Code that’s protected in such a manner—from indeterminacy in a multithreaded context—is called **thread-safe**.

Locking is not a silver bullet for thread safety—it’s easy to forget to lock around accessing a field, and locking can create problems of its own (such as **deadlocking**).

## Exception Handling

Any try/catch/finally blocks in effect when a thread is created are of no relevance to the thread when it starts executing. The try/catch statement in this example is ineffective

```c#

public static void Main()
{
    try
    {
        new Thread(Go).Start();
    }
    catch (Exception ex)
    {
        // We'll never get here!
        Console.WriteLine("Exception!");
    }
}
static void Go() { throw null; } // Throws a NullReferenceException

```

## Foreground Versus Background Threads

By default, threads you create explicitly are **foreground threads**. Foreground threads keep the application alive for as long as any one of them is running, whereas **background threads** do not.

## Thread Priority

A thread’s Priority property determines how much execution time it gets relative to other active threads in the operating system, on the following scale:

```c#

enum ThreadPriority { Lowest, BelowNormal, Normal, AboveNormal, Highest }

```

## Threading in Rich-Client Applications

In WPF, UWP, and Windows Forms applications, executing long-running operations on the main thread makes the application unresponsive, because the main thread also processes the message loop that performs rendering and handles keyboard and mouse events. A popular approach is to start up “worker” threads for time-consuming operations. The code on a worker thread runs a time-consuming operation and then updates the UI when complete. However, all rich-client applications have a threading model whereby UI elements and controls can be accessed only from the thread that created them (typically the main UI thread). Violating this causes either unpredictable behavior, or an exception to be thrown. Hence when you want to update the UI from a worker thread, you must forward the request to the UI thread (the technical term is **marshal**).

## Synchronization Contexts

In the System.ComponentModel namespace, there’s an abstract class called **SynchronizationContext** that enables the generalization of thread marshaling. The rich-client APIs for mobile and desktop (UWP, WPF, and Windows Forms) each define and instantiate SynchronizationContext subclasses, which you can obtain via the static property SynchronizationContext.Current (while running on a UI thread). Capturing this property lets you later “post” to UI controls from a worker thread:

```c#
partial class MyWindow : Window
{
    SynchronizationContext _uiSyncContext;
    public MyWindow()
    {
        InitializeComponent();
        // Capture the synchronization context for the current UI thread:
        _uiSyncContext = SynchronizationContext.Current;
        new Thread(Work).Start();
    }
    void Work()
    {
        Thread.Sleep(5000); // Simulate time-consuming task
        UpdateMessage("The answer");
    }
    void UpdateMessage(string message)
    {
        // Marshal the delegate to the UI thread:
        _uiSyncContext.Post(_ => txtMessage.Text = message, null);
    }
}
```

## The Thread Pool

Whenever you start a thread, a few hundred microseconds are spent organizing such things as a fresh local variable stack. The **thread pool** cuts this overhead by having a pool of pre-created recyclable threads. Thread pooling is essential for efficient parallel programming and fine-grained concurrency; it allows short operations to run without being overwhelmed with the overhead of thread startup.

There are a few things to be wary of when using pooled threads:
• You cannot set the Name of a pooled thread, making debugging more difficult (although you can attach a description when debugging in Visual Studio’s Threads window).
• Pooled threads are always background threads.
• Blocking pooled threads can degrade performance

The easiest way to explicitly run something on a pooled thread is to use *Task.Run* (we’ll cover this in more detail in the following section):

```c#
// Task is in System.Threading.Tasks
Task.Run (() => Console.WriteLine ("Hello from the thread pool"));
```

The thread pool serves another function, which is to ensure that a temporary excess of compute-bound work does not cause CPU **oversubscription**. Oversubscription is the condition of there being more active threads than CPU cores, with the operating system having to time-slice threads. Oversubscription hurts performance because time-slicing requires expensive context switches and can invalidate the CPU caches that have become essential in delivering performance to modern processors.

The CLR avoids oversubscription in the thread pool by queuing tasks and throttling their startup. It begins by running as many concurrent tasks as there are hardware cores, and then tunes the level of concurrency via a hill-climbing algorithm, continually adjusting the workload in a particular direction. If throughput improves, it continues in the same direction (otherwise it reverses). This ensures that it always tracks the optimal performance curve—even in the face of competing process activity on the computer.The CLR’s strategy works best if two conditions are met:

- Work items are mostly short-running (<250ms, or ideally <100ms), so that the CLR has plenty of opportunities to measure and adjust.
- Jobs that spend most of their time blocked do not dominate the pool.

Blocking is troublesome because it gives the CLR the false idea that it’s loading up the CPU. The CLR is smart enough to detect and compensate (by injecting more threads into the pool), although this can make the pool vulnerable to subsequent oversubscription. It also may introduce latency, as the CLR throttles the rate at which it injects new threads, particularly early in an application’s life (more so on client operating systems where it favors lower resource consumption). Maintaining good hygiene in the thread pool is particularly relevant when you want to fully utilize the CPU (e.g., via the parallel programming APIs in Chapter 23).
