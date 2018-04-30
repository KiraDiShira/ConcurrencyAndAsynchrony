[index](https://github.com/KiraDiShira/ConcurrencyAndAsynchrony/blob/master/README.md#concurrency-and-asynchrony)

# Asynchronous Functions in C#

- [Awaiting](#awaiting)

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

## Returning Task<TResult>

