# Rust Multithreaded HTTP Server

A simple multithreaded HTTP server built in Rust, following the final project from [The Rust Book](https://doc.rust-lang.org/book/ch20-00-final-project-a-web-server.html).

## How It Works

The server listens on `127.0.0.1:7878` and processes TCP connections using a fixed-size thread pool of 4 workers. Each worker pulls jobs from a shared `mpsc` channel protected by an `Arc<Mutex<...>>`, so connections are distributed across threads without spawning a new thread per request.

**Note:** The server intentionally accepts only 2 requests (`.take(2)`) before initiating a graceful shutdown. This is a deliberate design choice from the tutorial to demonstrate the `Drop`-based shutdown mechanism.

### Routes

| Path              | Response                                                    |
|-------------------|-------------------------------------------------------------|
| `GET /`           | 200 OK — serves `hello.html`                               |
| `GET /sleep`      | 200 OK — sleeps 5 s, then serves `hello.html` (tests concurrency) |
| anything else     | 404 NOT FOUND — serves `404.html`                          |

### Project Structure

```
src/
  main.rs   — TCP listener, request routing, connection handler
  lib.rs    — ThreadPool and Worker implementation
hello.html  — served on successful requests
404.html    — served on unknown routes
```

### ThreadPool Design

- `ThreadPool::new(n)` spawns `n` worker threads, each sharing a single `mpsc` channel receiver via `Arc<Mutex<Receiver<Job>>>`.
- `ThreadPool::execute(f)` sends a boxed closure over the channel; the first free worker picks it up.
- Graceful shutdown via `Drop`:
  1. `drop(self.sender.take())` closes the sending end of the channel.
  2. Each worker's `recv()` returns `Err`, causing it to exit its loop.
  3. The `Drop` impl joins every worker thread, ensuring all in-flight jobs finish before the process exits.

## Usage

```bash
cargo run
```

Then open a browser or use `curl`. The server will shut down cleanly after 2 requests.

```bash
curl http://127.0.0.1:7878/        # Hello page (request 1)
curl http://127.0.0.1:7878/sleep   # Delayed response — tests concurrency (request 2)
curl http://127.0.0.1:7878/foobar  # 404 page
```

## Requirements

- Rust 2024 edition (`rustup default stable`)
