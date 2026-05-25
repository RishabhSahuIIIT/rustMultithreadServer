# Rust Multithreaded HTTP Server

A simple multithreaded HTTP server built in Rust, following the project from [The Rust Book](https://doc.rust-lang.org/book/ch20-00-final-project-a-web-server.html).

## How It Works

The server listens on `127.0.0.1:7878` and handles incoming TCP connections using a fixed-size thread pool of 4 workers. Each worker thread pulls jobs from a shared channel protected by a `Mutex`, so connections are distributed across threads without spawning a new thread per request.

### Routes

| Path     | Response                          |
|----------|-----------------------------------|
| `GET /`  | 200 OK — serves `hello.html`      |
| `GET /sleep` | 200 OK — sleeps 5s, then serves `hello.html` (useful for testing concurrency) |
| anything else | 404 NOT FOUND — serves `404.html` |

### Project Structure

```
src/
  main.rs   — TCP listener, request routing, connection handler
  lib.rs    — ThreadPool and Worker implementation
hello.html  — served on successful requests
404.html    — served on unknown routes
```

### ThreadPool Design

- `ThreadPool::new(n)` spawns `n` worker threads sharing a single `mpsc` channel receiver behind an `Arc<Mutex<...>>`.
- `ThreadPool::execute(f)` sends a boxed closure over the channel; whichever worker is free picks it up.
- `Drop` on `ThreadPool` joins all worker threads for clean shutdown.

## Usage

```bash
cargo run
```

Then open your browser or use curl:

```bash
curl http://127.0.0.1:7878/        # Hello page
curl http://127.0.0.1:7878/sleep   # Delayed response (tests concurrency)
curl http://127.0.0.1:7878/foobar  # 404 page
```

## Requirements

- Rust 2024 edition (rustup default stable)
