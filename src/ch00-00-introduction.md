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
