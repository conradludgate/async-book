# Using a runtime

## Spawning

Spawning a task is analogous to spawning a thread. It creates a new top-level task with no parent for the runtime to process.
This usually returns a `JoinHandle`, similar again to the threaded version. Awaiting the join handle will pause the current task until the spawned task is
complete.

```rust,ignore
/// Downloads a set of pages concurrently
async fn download_pages(pages: Vec<String>) -> Vec<(String, Vec<u8>)> {
    let mut work = Vec::with_capacity(pages.len());
    for page in pages {
        // spawn each download job into it's own task.
        // this ensures that all the download tasks are run concurrently
        let handle = tokio::spawn(async {
            let bytes = download_page(&page).await;
            (page, bytes)
        });

        // store the handles
        work.push(handle);
    }

    let mut output = Vec::with_capacity(pages.len());
    for handle in work {
        // join lets us get the output of the task
        output.push(handle.await.unwrap())
    }

    output
}

/// Downloads the page contents at the URL
async fn download_page(url: &str) -> Vec<u8> {
    todo!()
}
```

Depending on the runtime, you may also get the ability to 'abort' or 'cancel' a task. This in theory should remove the task from any queues, but it's never
that simple in practice and it usually just sets a flag that this task can be skipped instead.

Given the similar API to thread::spawn, this requires that tasks have a `'static` lifetime.

## Network

Much like [`std::net`](https://doc.rust-lang.org/std/net/index.html), 
general purpose runtimes expose their own network primitives. 
Also, much like how std has [`Read`](https://doc.rust-lang.org/std/io/trait.Read.html)/[`Write`](https://doc.rust-lang.org/std/io/trait.Write.html) traits,
these runtimes will often have a similar [`AsyncRead`](https://docs.rs/smol/1.2.5/smol/prelude/trait.AsyncRead.html)/[`Write`](https://docs.rs/smol/1.2.5/smol/prelude/trait.AsyncWrite.html) traits

## Timers

Finally, most runtimes offer efficient alternatives to [`std::thread::sleep`](https://doc.rust-lang.org/nightly/std/thread/fn.sleep.html). 
This is often extended to provide interval clocks that repeatedly fire, as well as timeouts that can cancel tasks that take too long.
