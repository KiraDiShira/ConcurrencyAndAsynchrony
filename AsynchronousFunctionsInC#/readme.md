[index](https://github.com/KiraDiShira/ConcurrencyAndAsynchrony/blob/master/README.md#concurrency-and-asynchrony)

# Asynchronous Functions in C#

- [Awaiting](#awaiting)
- [Writing Asynchronous Functions](#writing-asynchronous-functions)
- [Returning Task< TResult >]
- [Asynchronous Lambda Expressions](#asynchronous-lambda-expressions)
- [Asynchrony and Synchronization Contexts](#asynchrony-and-synchronization-contexts)

## Awaiting

The await keyword simplifies the attaching of continuations. The compiler expands:

```c#

var result = await expression;
statement(s);

```

into something functionally similar to:

```c#

var awaiter = expression.GetAwaiter();
awaiter.OnCompleted(() =>
{
    var result = awaiter.GetResult();
    statement(s);
});

```

Example:

```c#

async void DisplayPrimesCount()
{
    int result = await GetPrimesCountAsync(2, 1000000);
    Console.WriteLine(result);
}

Task<int> GetPrimesCountAsync(int start, int count)
{
    return Task.Run(() =>
        ParallelEnumerable.Range(start, count).Count(n =>
            Enumerable.Range(2, (int)Math.Sqrt(n) - 1).All(i => n % i > 0)));
}

```

The async modifier tells the compiler to treat await as a keyword rather than an identifier should an ambiguity arise within that method (this ensures that code written prior to C# 5 that might use await as an identifier will still compile without error). The async modifier can be applied only to methods (and lambda expressions) that return void or (as we’ll see later) a Task or Task<TResult>.

The expression upon which you await is typically a task; however, any object with a GetAwaiter method that returns an awaitable object (implementing INotifyCompletion.OnCompleted and with an appropriately typed GetResult method and a bool IsCompleted property) will satisfy the compiler.

## Writing Asynchronous Functions

With any asynchronous function, you can replace the void return type with a Task to make the method itself usefully asynchronous (and awaitable).

```c#
async Task PrintAnswerToLife() // We can return Task instead of void
{
    await Task.Delay(5000);
    int answer = 21 * 2;
    Console.WriteLine(answer);
}
```
Notice that we don’t explicitly return a task in the method body. The compiler manufactures the task, which it signals upon completion of the method (or upon an unhandled exception). This makes it easy to create asynchronous call chains:
```c#
async Task Go()
{
    await PrintAnswerToLife();
    Console.WriteLine("Done");
}
```

And because we’ve declared Go with a Task return type, Go itself is awaitable.

The compiler expands asynchronous functions that return tasks into code that leverages TaskCompletionSource to create a task that it then signals or faults.

## Returning Task< TResult >

You can return a Task< TResult > if the method body returns TResult:

```c#
async Task<int> GetAnswerToLife()
{
    await Task.Delay(5000);
    int answer = 21 * 2;
    return answer; // Method has return type Task<int> we return int
}
```
Internally, this results in the TaskCompletionSource being signaled with a value rather than null.

The compiler’s ability to manufacture tasks for asynchronous functions means that for the most part, you need to explicitly instantiate a TaskCompletionSource only in bottom-level methods that initiate I/O-bound concurrency. (And for methods that initiate compute-bound currency, you create the task with Task.Run.)
    
## Asynchronous Lambda Expressions

Just as ordinary named methods can be asynchronous:

```c#
async Task NamedMethod()
{
    await Task.Delay(1000);
    Console.WriteLine("Foo");
}
```
so can unnamed methods (lambda expressions and anonymous methods), if preceded by the async keyword:

```c#
Func<Task> unnamed = async () =>
{
    await Task.Delay(1000);
    Console.WriteLine("Foo");
};
```
## Asynchrony and Synchronization Contexts

```c#
async void ButtonClick(object sender, RoutedEventArgs args)
{
    await Task.Delay(1000);
    throw new Exception("Will this be ignored?");
}
```

When the button is clicked and the event handler runs, execution returns normally to the message loop after the await statement, and the exception that’s thrown a second later cannot be caught by the catch block in the message loop.

To mitigate this problem, AsyncVoidMethodBuilder catches unhandled exceptions (in void-returning asynchronous functions), and posts them to the synchronization context if present, ensuring that global exception-handling events still fire.

The compiler applies this logic only to void-returning asynchronous functions. So if we changed ButtonClick to return a Task instead of void, the unhandled exception would fault the resultant Task, which would then have nowhere to go (resulting in an unobserved exception).

An interesting nuance is that it makes no difference whether you throw before or after an await. So in the following example, the exception is posted to the synchronization context (if present) and never to the caller:

```c#
async void Foo() { throw null; await Task.Delay(1000); }
```
