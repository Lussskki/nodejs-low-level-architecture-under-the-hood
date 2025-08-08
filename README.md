## Node.js  Pipe-line Architecture
```
┌──────────────────────────────────────────────┐
│                Your JS Code                  │
│      (Runs in V8 JavaScript Engine)          │
└────────────┬─────────────────────────────────┘
             │ JS bindings / native module APIs
             ▼
┌──────────────────────────────────────────────┐
│               Node.js Core (C++)             │
│ - Built-in modules (fs, http, net, etc)      │
│ - JS-to-native bindings (node_fs.cc, etc)    │
└────────────┬─────────────────────────────────┘
             │ C API calls
             ▼
┌──────────────────────────────────────────────┐
│                 libuv (C)                    │
│ - Event Loop                                  │
│ - Thread Pool                                 │
│ - Async I/O + Timer Management                │
└────────────┬─────────────────────────────────┘
             │ OS-specific APIs
             ▼
┌──────────────────────────────────────────────┐
│              Operating System APIs           │
│  (epoll, kqueue, IOCP, select, etc.)         │
└──────────────────────────────────────────────┘
```
## Node.js Core (C++) and libuv (C) 
 - Node.js Core is primarily written in C++ and provides the bridge between JavaScript code (running on the V8 engine) and low-level system functionality.
## Responsibilities:
 - Implements built-in modules like:
 - fs (file system access)
 - http (HTTP server/client)
 - net (TCP sockets)
 - child_process (spawning processes)
 - Acts as a wrapper around native system APIs so they can be exposed to JavaScript.
 - Uses V8 (Google's high-performance JavaScript engine) to run your JS code.
 - Converts JavaScript calls to native operations using bindings (like node_file.cc for fs, node_http.cc for http, etc).
## libuv (C)
 - libuv is a multi-platform support library written in C that provides Node.js with its non-blocking I/O model.
Key Features:
 - Event Loop: Manages the lifecycle of asynchronous operations (I/O, timers, etc.).
 = Thread Pool: For operations that can’t be handled non-blockingly by the OS (like file system operations), libuv uses a thread pool.
## Cross-platform abstraction:
 - On Linux: uses epoll
 - On macOS/BSD: uses kqueue
 - On Windows: uses IOCP (I/O Completion Ports)
Handles:
 - File I/O
 - DNS resolution
 - TCP/UDP sockets
 - Pipes and TTYs
 - Timers
- Signals
- Threading and synchronization
## Additional Components to Include
 ## 1. V8 Engine
 - Developed by Google in C++.
 - Executes your JavaScript code and compiles it into machine code via Just-In-Time (JIT) compilation.
 - Works tightly with Node.js core to allow JavaScript to invoke C++ bindings.

 ## 2. Event Loop Phases lifecycle managed by libuv:
 - timers	Executes callbacks scheduled by setTimeout() or setInterval()
 - pending callbacks	Executes I/O callbacks deferred to the next loop
 - idle / prepare	Internal use
 - poll	Waits for I/O events
 - check	Executes setImmediate() callbacks
 - close callbacks	Handles socket.on('close', ...) events

 ## 3. libuv Thread Pool
 - Default size: 4 threads (configurable via UV_THREADPOOL_SIZE env variable).
 - Used for blocking tasks like:
 - fs.readFile()
 - crypto.pbkdf2()
 - DNS lookups (dns.lookup())

 ## 4. Native Addons (Node-API / N-API)
 - Can write C/C++ modules and expose them to JavaScript.
 - Used for performance-critical code or interacting with native libraries.
  ```
  // Example: myaddon.cc
  #include <napi.h>
  
  Napi::Number Add(const Napi::CallbackInfo& info) {
    return Napi::Number::New(info.Env(), 42);
  }
  
  Napi::Object Init(Napi::Env env, Napi::Object exports) {
    exports.Set("add", Napi::Function::New(env, Add));
    return exports;
  }
  
  NODE_API_MODULE(addon, Init)
  
  ```
 ## 5. Node.js Execution Flow
 - Load the script.
 - Initialize Node.js and V8.
 - Start executing synchronous code.
 - Register async operations (fs, net, timers).
 - Run the event loop (libuv).
 - Execute callbacks when ready.
 - Exit when there are no more tasks to run.

 ## 6. Why This Matters
 - Understanding this architecture helps explain:
 - Why Node.js is single-threaded but non-blocking
 - How high concurrency is achieved with low overhead
 - Why CPU-bound tasks slow down Node
 - Why some APIs (like fs.readFileSync) block the event loop
 ##  Table
  ```
  | Component    | Language | Purpose                                         |
  | ------------ | -------- | ----------------------------------------------- |
  | V8           | C++      | Run JavaScript (JIT compiler)                   |
  | Node.js Core | C++      | Bind JavaScript to system APIs                  |
  | libuv        | C        | Event loop, async I/O, thread pool              |
  | OS APIs      | C        | Native event notification (epoll, kqueue, IOCP) |
  ```   
 ## Garbage Collector (GC) — V8 Memory Management
The Garbage Collector (GC) is part of the V8 engine and is responsible for automatically managing memory in Node.js.

Responsibilities:
 - Automatically allocates and frees memory so developers don’t need to manually manage it.
 - Reclaims memory occupied by unreachable objects (no longer referenced in code).
 - Prevents memory leaks and helps avoid crashes due to memory overflow.
 ## How It Works:
V8 uses a Generational Garbage Collection strategy:
 - Splits memory into regions:
 - New Space – stores short-lived objects.
 -Old Space – stores long-lived objects.
 - Uses different algorithms for each space:
 - Scavenge (Copying Collector) for New Space.
 - Mark-and-Sweep / Mark-and-Compact for Old Space.
 ## GC Lifecycle:
 - Allocate memory for new JS objects in New Space.
 - If New Space fills → minor GC is triggered (Scavenge).
 - Surviving objects are promoted to Old Space.
 - Periodically, V8 runs major GC on Old Space using Mark-and-Sweep.
## My Notes:
 - GC is automatic, but you can trigger it manually (for testing/debug only):
   ```
    node --expose-gc
    global.gc();

   ```
 -  Monitor GC behavior using:
   ```
    node --trace-gc

   ```
 ## When Memory Leaks Happen:
 - Even with GC, memory leaks can occur if:
 - Variables are kept in global scope unintentionally.
 - Timers or intervals are not cleared (setInterval or setTimeout).
 - Closures unintentionally hold references to outer scope.
 - You store large data in memory (e.g. cache, logs) without limits.
 ## Why It Matters:
 - Explains how Node.js apps stay efficient without manual memory handling.
 - Helps debug performance issues or memory leaks.
 - Crucial for long-running apps (e.g. servers, background workers).   
