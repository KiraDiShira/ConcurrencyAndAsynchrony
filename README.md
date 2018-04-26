# Concurrency and asynchrony

Applications need to deal with more than one thing happening at a time (**concurrency**).

The mechanism by which a program can simultaneously execute code is called **multithreading**.

A **thread** is an execution path that can proceed independently of others. Each thread runs within an operating system **process**, which provides an isolated environment in which a program runs. 

With a single-threaded program, just one thread runs in the process’s isolated environment and so that thread has exclusive access to it. With a multithreaded program, multiple threads run in a single process, sharing the same execution environment (memory, in particular). This, in part, is why multithreading is useful: one thread can fetch data in the background, for instance, while another thread displays the data as it arrives. This data is referred to as **shared state**.
