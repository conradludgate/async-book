# What is a runtime

At a high level, in order to run through your async functions, you need a runtime to manage which tasks are ready,
as well as getting the tasks processed in an efficient way.

The two most popular runtimes at the time of writing are [tokio](https://tokio.rs/), [smol](https://docs.rs/smol/1.2.5/smol/), and [async-std](https://async.rs/).
All three achieve similar end results by utilising work-stealing and IO polling.

## Anatomy of a runtime

A runtime is collection of worker threads that share some state between them. At it's core, the runtime
will has a shared queue of ready tasks, and possible a shared queue of waiting tasks. The job of the worker
is to progress a task forward, poll the OS to see if any other tasks that were waiting for IO are ready, and countdown any waiting timers.

For efficiency, most runtime workers will have their own local queue of tasks, and employ tactics like work distribution and work stealing in order
to not starve any workers so they can maintain the maximum parallelism and throughput.

Put into some psuedo-rust-code, this worker process might look like

```rust,ignore
loop {
    // check if anything new is ready
    timers.count_down();
    os.poll();

    // get a task from the local queue
    let mut task = local_queue.pop_front();

    // if our local queue has no tasks, take some from the global queue
    if task.is_none() {
        local_queue.take_from(&mut global_queue);
        task = local_queue.pop_front();
    }

    // if there still isn't a task available, check the other workers
    for worker in &mut workers {
        if task.is_some() { break }

        local_queue.steal_from(worker);
        task = local_queue.pop_front();
    }

    // if there was a task, run it.
    // otherwise, pause this thread.
    if let Some(task) = task {
        task.progress();
    } else {
        pause();
    }
}
```

In reality, there's a bit more logic involved to maximise CPU efficiency, but that's the gist of it.

## Configuration

You might be working on platforms without access to the std library. This means you don't have access to the OS,
or access to threads. In which case, you can simplify that event loop down significantly as you have no global queue
or workers to steal from. You also don't need to worry about OS polling or pausing. 

Similarly, you might be using a 'thread-per-core' architecture, which removes the potentially expensive work-stealing step,
but can reduce throughput in some workloads. This also allows you to run tasks that are `!Send` (can't be sent between threads).
This is the technique employed by [actix-rt](https://docs.rs/actix-rt/2.7.0/actix_rt/)
by utilising [Tokio's `LocalSet`](https://docs.rs/tokio/1.22.0/tokio/task/struct.LocalSet.html).
[Glommio](https://www.datadoghq.com/blog/engineering/introducing-glommio/) is another such runtime with this feature

## Choosing a runtime

Unfortunately, while there's a lot of options for runtimes, there's often not a lot of choice you as a user can make.

If you want to use the `axum`/`reqwest` libraries as shown in [the previous example](./01-example.md), you **must** use `tokio`. This is because
internally these use `tokio::spawn`, `tokio::net` and `tokio::time` modules. All of these must be called in the context of a tokio runtime
and will panic if the thread local runtime global is not set.

Some libraries are generic over runtimes. `hyper`, the HTTP engine that `axum` and `reqwest` are based on, is generic over executors using
traits like [Executor](https://docs.rs/hyper/0.14.23/hyper/rt/trait.Executor.html) to abstract away the `tokio::spawn`, 
and [Accept](https://docs.rs/hyper/0.14.23/hyper/server/accept/trait.Accept.html) to abstract away the `tokio::net::TcpListener`.

Similarly, there exists [executor-trait](https://docs.rs/executor-trait/2.1.0/executor_trait/)
and [reactor-trait](https://docs.rs/reactor-trait/1.1.0/reactor_trait/index.html) which are used by [`lapin`](https://crates.io/crates/lapin
in order to allow the async-runtime to be configured in trait-objects. This requires more allocation overhead but potentially can
speed up compile times if you don't need extreme performance.

Because `tokio` is orders of magnitudes more popular than `smol` and `async-std`, there also exists compatibility layers like
[async-compat](https://docs.rs/async-compat/latest/async_compat/) which run tokio-based async code within a tokio context.
This comes with the added cost that any tokio actions are spawned into an extra single threaded runtime, reducing overall resource
efficiency.
