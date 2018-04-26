# Concurrency and asynchrony

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
