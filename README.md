# Concurrency and asynchrony

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

```
yyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyThread t has ended!
```
