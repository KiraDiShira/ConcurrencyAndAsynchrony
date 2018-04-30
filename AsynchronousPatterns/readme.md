# Asynchronous Patterns

- [Cancellation](#cancellation)

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
