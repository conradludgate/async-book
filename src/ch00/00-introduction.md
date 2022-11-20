# Introduction to Async

Async code exists to allow a programming language to offer more control
in the way that code can run concurrently.
A famous example of an async programming language is JS,
which can only execute code using 1 thread, but still needs
to support allowing multiple streams of work to happen at the same time.

For instance, say you want to make a network request,
but while you're communicating with the external service, you want
to update the web page with a live updating progress bar.

One way we can accomplish this in a single threaded setup is by allowing multiplexing.
That is, allowing one task to temporarily pause to allow another task to resume.
Eventually, both tasks will be run to completion, but neither will block the other.

## Threading vs Async

I've already hinted at another mechanism to allow concurrency, threads!
Rust allows multithreading, so why do we need async code?
Well, there's many reasons.
Maybe you're writing code on an embedded device that only supports a single threaded operations.
Maybe you're working in the Linux kernel itself and don't have access to just create threads as you please.
Maybe you just want more control about what tasks can run and in which order,
and to not be at the mercy of your OS scheduler.

Whatever your reasoning, async rust should be general enough to support your needs.

## Concurrency vs Parallelism

Concurrency and Parallelism are different, but related concepts.

Concurrency is the process of performing tasks out-of-order. For example, if you have two tasks,
a simple setup is to perform task 1, then task 2. But if these tasks involve any waiting, then you're wasting time.
Instead, concurrency is the idea that you can do some of task 1 and some of task 2 in any order. Eventually both
will be complete and the same end goal is reached, but in potentially much less time.

Parallelism is the process of performing tasks at the same time. This would involve multiple workers
(CPU cores, services, threads, people etc) performing their single task. Using the example above, tasks 1 & 2 will be started immediately
and will complete as soon as possible, however it might take double the resources in the worst case.

Both concepts have the same goal of reducing the time a combination of tasks may take, but in different ways.
Parallelism requires more resources, but concurrency can't achieve the same throughput levels. 

Async Rust provides you with **concurrency**. However, most async runtimes make use of thread pools in order to utilise
a fixed amount of parallelism. In theory, this lets you achieve as much throughput as possible with the least resources available
in your system.

## Async Rust

Under the hood, async-await in Rust is syntax sugar that makes it easy to build cooperative-coroutines, unlike the preemptive green threads
you might see in Go or Java. This means that you must expliclty mark where a task may pause using `.await`. The runtime isn't able to
pause a task because it's taking a long time. For that reason, you typically have to be careful to not do too much CPU bound work in between
`.await` lines. 
