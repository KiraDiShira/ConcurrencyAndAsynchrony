# Tasks

- [Introduction](#introduction)

## Introduction

A thread is a low-level tool for creating concurrency, and as such it has limitations. In particular

- While it’s easy to pass data into a thread that you start, there’s no easy way to get a “return value” back from a thread that you Join. You have to set up some
kind of shared field. And if the operation throws an exception, catching and propagating that exception is equally painful.
- You can’t tell a thread to start something else when it’s finished; instead you
must Join it (blocking your own thread in the process).

```c#

