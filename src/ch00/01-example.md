# Example of Async

```rust
use axum::{
    routing::get,
    Router,
};

#[tokio::main]
async fn main() {
    // build our application with a single route
    let app = Router::new().route("/", get(index));

    axum::Server::bind(&"0.0.0.0:3000".parse().unwrap())
        .serve(app.into_make_service())
        .await
        .unwrap();
}

async fn index() {
    let client = reqwest::Client::new();
    let res = client.post("http://example.com/counter")
        .send()
        .await;

    match res {
        Ok(_) => "Welcome to my site!",
        Err(_) => "Error registering your visit",
    }
}
```

This is a very typical async application,
featuring [axum](https://docs.rs/axum/latest/axum/) as the HTTP server library,
[reqwest](https://docs.rs/reqwest/latest/reqwest/) as the HTTP client library,
and [tokio](https://tokio.rs/) as the async runtime.

The key feature here is that each incomming server request happen 'asynchronously'.
While each request handler is waiting on the response, it yields control
of the thread over to another request that needs to be handled.

Let's break down what that means:

As a request comes in, our server will launch the request handler as a new async 'task'.

```rust
// Task 1:
    let client = reqwest::Client::new();
    let res = client.post("http://example.com/counter")
        .send()
        .await;
        // register interest in sending and receiving data.
        // Our runtime will pause this task

// Task 2:
    let client = reqwest::Client::new();
    let res = client.post("http://example.com/counter")
        .send()
        .await;
        // register interest in sending and receiving data.
        // Our runtime will pause this task

// Task 1:
    // our runtime has monitored the OS and found that the data is now ready,
    // resumes task 1.
    match res {
        Ok(_) => "Welcome to my site!",
        Err(_) => "Error registering your visit",
    }

// Task 2:
    // our runtime has monitored the OS and found that the data is now ready,
    // resumes task 2.
    match res {
        Ok(_) => "Welcome to my site!",
        Err(_) => "Error registering your visit",
    }
```

Paired with multiple threads to fully utilise the CPU cores, a good runtime can achieve
hundreds of thousands requests per second, with many many thousands concurrent requests.

Over the course of this book, we will find out how these actually work, from a high level
down to some very low level implementation details
