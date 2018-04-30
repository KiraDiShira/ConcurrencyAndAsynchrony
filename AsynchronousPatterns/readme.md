- [index](https://github.com/KiraDiShira/ConcurrencyAndAsynchrony#concurrency-and-asynchrony)

# Asynchronous Patterns

- [Cancellation](#cancellation)
- [Task Combinators](#task-combinators)

## Cancellation

It’s often important to be able to cancel a concurrent operation after it’s started, perhaps in response to a user request.

To get a cancellation token, we first instantiate a CancellationTokenSource:

```c#

var cancelSource = new CancellationTokenSource();
Task foo = Foo (cancelSource.Token);
...
... (some time later)
cancelSource.Cancel();

```

Example:

```c#
private CancellationTokenSource _cancelSource;

public MainWindow()
{
    InitializeComponent();
    _cancelSource = new CancellationTokenSource();
}

private void ButtonCount_OnClick(object sender, RoutedEventArgs e)
{
    Counter(_cancelSource.Token);
}

async Task Counter(CancellationToken cancellationToken)
{
    for (int i = 0; i < 10; i++)
    {
        Console.WriteLine(i);
        await Task.Delay(1000, cancellationToken);
    }
}

private void ButtonCancel_OnClick(object sender, RoutedEventArgs e)
{
    _cancelSource.Cancel();
}
```

Synchronous methods can support cancellation, too (such as Task’s Wait method). In such cases, the instruction to cancel will have to come asynchronously (e.g., from another task). For example:

```c#
var cancelSource = new CancellationTokenSource();
Task.Delay (5000).ContinueWith (ant => cancelSource.Cancel());
...
```

In fact, from Framework 4.5, you can specify a time interval when constructing CancellationTokenSource to initiate cancellation after a set period of time:

```c#
private CancellationTokenSource _cancelSource;

public MainWindow()
{
    InitializeComponent();
    _cancelSource = new CancellationTokenSource(5000);
}

private async void ButtonCount_OnClick(object sender, RoutedEventArgs e)
{
    try
    {
       await Counter(_cancelSource.Token);
    }
    catch (OperationCanceledException exception)
    {
        Console.WriteLine("Cancelled");
    }
}

async Task Counter(CancellationToken cancellationToken)
{
    for (int i = 0; i < 10; i++)
    {
        Console.WriteLine(i);
        await Task.Delay(1000, cancellationToken);
    }
}
```

## Task Combinators

The CLR includes two task combinators: **Task.WhenAny** and **Task.WhenAll**. In describing them, we’ll assume the following methods are defined:

```c#
async Task<int> Delay1() { await Task.Delay (1000); return 1; }
async Task<int> Delay2() { await Task.Delay (2000); return 2; }
async Task<int> Delay3() { await Task.Delay (3000); return 3; }
```
```c#
Task<int> winningTask = await Task.WhenAny (Delay1(), Delay2(), Delay3());
Console.WriteLine ("Done");
Console.WriteLine (winningTask.Result); // 1
```

Because Task.WhenAny itself returns a task, we await it, which returns the task that finished first. Our example is entirely nonblocking—including the last line when we access the Result property (because winningTask will already have finished). Nonetheless, it’s usually better to await the winningTask:

```c#
Console.WriteLine (await winningTask); // 1
```
because any exceptions are then re-thrown without an AggregateException wrapping. In fact, we can perform both awaits in one step:

```c#
int answer = await await Task.WhenAny (Delay1(), Delay2(), Delay3());
```
If a nonwinning task subsequently faults, the exception will go unobserved unless you subsequently await the task (or query its Exception property).

```c#
await Task.WhenAll (Delay1(), Delay2(), Delay3());
```

We could get a similar result by awaiting task1, task2 and task3 in turn rather than using WhenAll:

```c#
Task task1 = Delay1(), task2 = Delay2(), task3 = Delay3();
await task1; await task2; await task3;
```

The difference, is that should task1 fault, we’ll never get to await task2/task3, and any of their exceptions will go unobserved. In contrast, Task.WhenAll doesn’t complete until all tasks have completed—even when there’s a fault. And if there are multiple faults, their exceptions are combined into the task’s AggregateException (this is when AggregateException actually becomes useful—should you be interested in all the exceptions, that is).

