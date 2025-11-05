# Node.js Core Study Guide

> Study material for learning Node.js core APIs through conceptual understanding and hands-on practice

## Table of Contents

1. [Process & Globals](#process--globals)
2. [Events](#events)
3. [Buffers](#buffers)
4. [Streams](#streams)
5. [Modules](#modules)
6. [File System](#file-system)
7. [HTTP](#http)
8. [Child Processes](#child-processes)
9. [Event Loop & Async](#event-loop--async)
10. [Learning Path](#learning-path)

---

## Process & Globals

**Concept:** The `process` object is a global that provides info about the current Node.js process and control over it.

```javascript
// Environment and runtime info
console.log(process.version);      // Node version
console.log(process.platform);     // OS platform
console.log(process.cwd());        // Current working directory
console.log(process.argv);         // Command line arguments

// Exit codes (0 = success, 1+ = error)
process.exit(0);

// Environment variables
const port = process.env.PORT || 3000;
const env = process.env.NODE_ENV || 'development';
```

### Common Pitfalls
- **Forgetting exit codes matter** - Always use `process.exit(1)` for errors, `process.exit(0)` for success
- **Blocking the event loop** - Synchronous operations block everything; use async versions
- **Not handling signals** - Process can receive SIGTERM, SIGINT - handle them gracefully
- **Modifying `process.env` at runtime** - Changes don't persist; set before process starts

### Practice Exercises
1. Write a CLI that accepts `--name` and `--age` flags and prints them (parse `process.argv`)
2. Create a script that reads `DATABASE_URL` from env vars and exits with code 1 if missing
3. Handle SIGTERM signal to gracefully shutdown (close connections, cleanup, then exit)

**Docs:** https://nodejs.org/api/process.html

---

## Events

**Concept:** Node.js uses an event-driven architecture. Objects that emit events are instances of `EventEmitter`.

```javascript
const EventEmitter = require('events');

class DatabaseConnection extends EventEmitter {
  connect() {
    // Simulate async connection
    setTimeout(() => {
      this.emit('connected', { host: 'localhost', port: 5432 });
    }, 100);
  }

  query(sql) {
    if (!this.connected) {
      this.emit('error', new Error('Not connected'));
      return;
    }
    this.emit('query', sql);
  }
}

const db = new DatabaseConnection();

// Listeners
db.on('connected', (info) => console.log('Connected:', info));
db.once('error', (err) => console.error('Error:', err)); // Fires only once
db.on('query', (sql) => console.log('Query:', sql));

db.connect();
```

### Event Execution Model
- Events are **synchronous** - listeners execute in order, blocking
- Use `setImmediate()` inside emitter if you need async behavior
- `error` events are special - unhandled errors crash the process

### Common Pitfalls
- **Memory leaks from listeners** - Always remove listeners you don't need (`removeListener()`)
- **Not handling 'error' events** - Unhandled error events crash the process
- **Order matters** - Listeners execute in registration order
- **Synchronous execution** - Long-running listener blocks other listeners

### Practice Exercises
1. Create a `FileWatcher` EventEmitter that emits `change`, `add`, `delete` events
2. Implement max listener warning - emit warning if >10 listeners on same event
3. Build a simple pub/sub system with subscribe/unsubscribe and publish methods

**Docs:** https://nodejs.org/api/events.html

---

## Buffers

**Concept:** Buffers represent fixed-size chunks of memory for binary data. They exist outside V8's heap.

```javascript
// Creating buffers
const buf1 = Buffer.alloc(10);              // Safe, zero-filled
const buf2 = Buffer.allocUnsafe(10);        // Fast, uninitialized
const buf3 = Buffer.from('hello');          // From string
const buf4 = Buffer.from([0x68, 0x65]);     // From array

// Reading/writing
buf1.write('hello', 0, 'utf8');
console.log(buf1.toString('utf8'));
console.log(buf1.toString('hex'));
console.log(buf1.toString('base64'));

// Slicing (creates view, not copy)
const slice = buf3.slice(0, 2);
slice[0] = 0x48; // Modifies original buffer!

// Copying (creates new buffer)
const copy = Buffer.from(buf3);
```

### Understanding Buffer Memory
- Buffers are **fixed size** - cannot be resized
- `allocUnsafe` is faster but contains old memory - **must initialize before use**
- Slices share memory with parent - modifying slice modifies original
- Buffers use UTF-8 by default

### Common Pitfalls
- **Using `allocUnsafe` without initializing** - exposes old memory data (security risk)
- **Thinking slices are copies** - they're views; modifications affect original
- **String encoding assumptions** - always specify encoding explicitly
- **Modifying buffer length** - buffers are fixed size, length cannot change

### Practice Exercises
1. Write function to convert hex string to Buffer and back
2. Implement a function that XORs two buffers of same length
3. Create a buffer pool manager that reuses buffers to reduce allocations

**Docs:** https://nodejs.org/api/buffer.html

---

## Streams

**Concept:** Streams process data piece-by-piece instead of loading everything into memory.

### Four Types of Streams
1. **Readable** - Read data (e.g., `fs.createReadStream()`)
2. **Writable** - Write data (e.g., `fs.createWriteStream()`)
3. **Duplex** - Both readable and writable (e.g., TCP socket)
4. **Transform** - Modify data while reading/writing (e.g., compression)

```javascript
const fs = require('fs');
const { Transform } = require('stream');

// Basic piping
fs.createReadStream('input.txt')
  .pipe(fs.createWriteStream('output.txt'));

// Transform stream
const uppercase = new Transform({
  transform(chunk, encoding, callback) {
    this.push(chunk.toString().toUpperCase());
    callback();
  }
});

fs.createReadStream('input.txt')
  .pipe(uppercase)
  .pipe(process.stdout);

// Manual stream handling
const readable = fs.createReadStream('large-file.txt');

readable.on('data', (chunk) => {
  console.log(`Received ${chunk.length} bytes`);
});

readable.on('end', () => {
  console.log('No more data');
});

readable.on('error', (err) => {
  console.error('Error:', err);
});
```

### Backpressure
When writable stream can't handle data fast enough, it returns `false` from `write()`:

```javascript
const readable = fs.createReadStream('input.txt');
const writable = fs.createWriteStream('output.txt');

readable.on('data', (chunk) => {
  const canContinue = writable.write(chunk);
  if (!canContinue) {
    readable.pause(); // Stop reading
  }
});

writable.on('drain', () => {
  readable.resume(); // Continue reading
});
```

### Common Pitfalls
- **Ignoring backpressure** - causes memory issues; always use `.pipe()` or handle `drain`
- **Not handling errors** - streams don't forward errors through pipes
- **Memory leaks from paused streams** - always resume or destroy paused streams
- **Assuming pipe() handles errors** - it doesn't; attach error handlers to each stream

### Practice Exercises
1. Implement a Transform stream that counts words in flowing text
2. Create a Duplex stream that echoes input back with 1 second delay
3. Build a stream multiplexer that writes to multiple destinations with backpressure handling

**Docs:** https://nodejs.org/api/stream.html

---

## Modules

### CommonJS (Traditional)

**Concept:** Each file is a module. `require()` loads modules synchronously and caches them.

```javascript
// math.js
const add = (a, b) => a + b;
const multiply = (a, b) => a * b;

// Wrong way
exports = { add, multiply }; // Doesn't work!

// Correct ways
module.exports = { add, multiply };
// OR
exports.add = add;
exports.multiply = multiply;
```

**Module Resolution:**
```javascript
require('./math');        // Relative path - loads ./math.js
require('lodash');        // node_modules lookup
require('path');          // Core module
```

### ES Modules (Modern)

**Concept:** Modern module system with static imports analyzed at parse time.

```javascript
// math.mjs
export const add = (a, b) => a + b;
export const multiply = (a, b) => a * b;
export default class Calculator {}

// app.mjs
import Calculator, { add, multiply } from './math.mjs';
import * as math from './math.mjs';

// Dynamic import
const module = await import('./math.mjs');
```

**Enabling ESM:**
- Use `.mjs` extension, OR
- Add `"type": "module"` in package.json

### Module Caching

```javascript
// counter.js
let count = 0;
module.exports = {
  increment: () => ++count,
  get: () => count
};

// Modules are cached - same instance everywhere
const counter1 = require('./counter');
const counter2 = require('./counter');
counter1.increment();
console.log(counter2.get()); // 1 - same instance!
```

### Common Pitfalls
- **`exports` vs `module.exports`** - `exports` is just a reference; reassigning breaks it
- **Circular dependencies** - can result in incomplete module exports
- **Forgetting `.mjs` or package.json type** - ESM won't work
- **Mixing CommonJS and ESM** - ESM can import CommonJS, but not vice versa directly
- **Assuming fresh module each require** - modules are cached

### Practice Exercises
1. Demonstrate circular dependency issue and fix it with refactoring
2. Create a module that returns different instances vs singleton pattern
3. Build a simple plugin loader that dynamically requires modules from a directory

**Docs:** https://nodejs.org/api/modules.html | https://nodejs.org/api/esm.html

---

## File System

**Concept:** Three API styles: callback, promise, and synchronous.

```javascript
const fs = require('fs');
const fsPromises = require('fs/promises');

// Callback style
fs.readFile('file.txt', 'utf8', (err, data) => {
  if (err) throw err;
  console.log(data);
});

// Promise style (preferred for async/await)
async function readFile() {
  try {
    const data = await fsPromises.readFile('file.txt', 'utf8');
    console.log(data);
  } catch (err) {
    console.error(err);
  }
}

// Sync style (blocks event loop - use sparingly)
const data = fs.readFileSync('file.txt', 'utf8');
```

### File Descriptors (Low-level)
```javascript
// Open file descriptor
const fd = fs.openSync('file.txt', 'r');
const buffer = Buffer.alloc(100);

// Read into buffer
fs.readSync(fd, buffer, 0, 100, 0);

// Always close!
fs.closeSync(fd);
```

### Watching Files
```javascript
// Watch for changes
fs.watch('file.txt', (eventType, filename) => {
  console.log(`${filename} ${eventType}`);
});

// More reliable (but polls)
fs.watchFile('file.txt', { interval: 100 }, (curr, prev) => {
  console.log('Modified:', curr.mtime);
});
```

### Common Pitfalls
- **Using sync methods in production** - blocks entire event loop
- **Not closing file descriptors** - causes file descriptor leaks
- **Race conditions with fs.exists()** - file can be deleted between check and use
- **Forgetting error handling** - file operations fail often (permissions, missing files)
- **fs.watch() platform differences** - behavior varies across OS; not 100% reliable

### Practice Exercises
1. Implement recursive directory copy with proper error handling
2. Build a file-based queue that handles concurrent reads/writes safely
3. Create a log rotation system that archives files when they reach size limit

**Docs:** https://nodejs.org/api/fs.html

---

## HTTP

**Concept:** Create HTTP servers and clients using event-driven request/response objects.

### HTTP Server
```javascript
const http = require('http');

const server = http.createServer((req, res) => {
  // Request object is a readable stream
  // Response object is a writable stream

  console.log(`${req.method} ${req.url}`);
  console.log('Headers:', req.headers);

  // Parse URL
  const url = new URL(req.url, `http://${req.headers.host}`);
  console.log('Query params:', url.searchParams);

  // Read body
  let body = '';
  req.on('data', chunk => body += chunk);
  req.on('end', () => {
    console.log('Body:', body);

    res.writeHead(200, {
      'Content-Type': 'application/json',
      'X-Custom-Header': 'value'
    });

    res.end(JSON.stringify({ message: 'Success', data: body }));
  });
});

server.listen(3000, () => {
  console.log('Server running on port 3000');
});
```

### HTTP Client
```javascript
const http = require('http');

const options = {
  hostname: 'api.example.com',
  port: 80,
  path: '/data',
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
  }
};

const req = http.request(options, (res) => {
  console.log('Status:', res.statusCode);
  console.log('Headers:', res.headers);

  let data = '';
  res.on('data', chunk => data += chunk);
  res.on('end', () => console.log('Response:', data));
});

req.on('error', (err) => console.error('Error:', err));

req.write(JSON.stringify({ key: 'value' }));
req.end();
```

### Understanding Request/Response Lifecycle
1. Client connects → `connection` event
2. Request headers received → `request` event, callback fired
3. Request body streams in → `data` events on req
4. Request complete → `end` event on req
5. Response sent → `res.writeHead()`, `res.write()`, `res.end()`
6. Connection may stay alive (keep-alive) or close

### Common Pitfalls
- **Not consuming request body** - causes memory leaks; always drain or read body
- **Sending headers after body** - must call `writeHead()` before `write()`/`end()`
- **Not handling request errors** - unhandled errors crash server
- **Forgetting Content-Length** - can cause client hangs; Node usually sets it
- **Assuming req.url is parsed** - it's a string; use `new URL()` to parse

### Practice Exercises
1. Build a routing system that handles GET/POST to different paths
2. Implement request timeout - close connection if no data after 30s
3. Create an HTTP proxy that forwards requests to another server

**Docs:** https://nodejs.org/api/http.html

---

## Child Processes

**Concept:** Spawn external processes to run system commands or other Node.js scripts.

### Four Ways to Spawn
```javascript
const { exec, execFile, spawn, fork } = require('child_process');

// 1. exec - runs command in shell, buffers output
exec('ls -la', (error, stdout, stderr) => {
  if (error) throw error;
  console.log(stdout);
});

// 2. execFile - runs executable directly (no shell), buffers output
execFile('ls', ['-la'], (error, stdout, stderr) => {
  if (error) throw error;
  console.log(stdout);
});

// 3. spawn - streams output, doesn't use shell
const ls = spawn('ls', ['-la']);
ls.stdout.on('data', (data) => console.log(data.toString()));
ls.stderr.on('data', (data) => console.error(data.toString()));
ls.on('close', (code) => console.log(`Exit code: ${code}`));

// 4. fork - spawn Node.js process with IPC channel
const child = fork('./worker.js');
child.send({ task: 'process', data: [1, 2, 3] });
child.on('message', (result) => console.log('Result:', result));
```

### When to Use Which
- **exec**: Quick shell commands, small output
- **execFile**: Direct binary execution, more secure
- **spawn**: Large output, need streaming, long-running
- **fork**: Node.js child processes with message passing

### IPC with fork()
```javascript
// parent.js
const child = fork('./child.js');
child.send({ command: 'compute', data: 100 });
child.on('message', (msg) => console.log('From child:', msg));

// child.js
process.on('message', (msg) => {
  if (msg.command === 'compute') {
    const result = msg.data * 2;
    process.send({ result });
  }
});
```

### Common Pitfalls
- **Using exec with user input** - command injection vulnerability; use execFile
- **Not handling child errors** - errors don't propagate; listen to 'error' event
- **Buffering too much with exec** - use spawn for large output
- **Forgetting to kill children** - orphaned processes; use `child.kill()`
- **Blocking on sync versions** - `execSync`, `spawnSync` block event loop

### Practice Exercises
1. Create a worker pool that distributes tasks across multiple forked processes
2. Build a command runner with timeout - kill process if exceeds time limit
3. Implement process monitoring - restart child if it crashes

**Docs:** https://nodejs.org/api/child_process.html

---

## Event Loop & Async

**Concept:** Node.js runs JavaScript in a single thread using an event loop with multiple phases.

### Event Loop Phases
```
   ┌───────────────────────────┐
┌─>│           timers          │  setTimeout, setInterval callbacks
│  └─────────────┬─────────────┘
│  ┌─────────────┴─────────────┐
│  │     pending callbacks     │  I/O callbacks deferred to next loop
│  └─────────────┬─────────────┘
│  ┌─────────────┴─────────────┐
│  │       idle, prepare       │  Internal use only
│  └─────────────┬─────────────┘
│  ┌─────────────┴─────────────┐
│  │           poll            │  Retrieve new I/O events
│  └─────────────┬─────────────┘
│  ┌─────────────┴─────────────┐
│  │           check           │  setImmediate callbacks
│  └─────────────┬─────────────┘
│  ┌─────────────┴─────────────┐
└──┤      close callbacks      │  socket.on('close', ...)
   └───────────────────────────┘
```

### Microtasks vs Macrotasks
```javascript
console.log('1: Sync');

setTimeout(() => console.log('2: setTimeout'), 0);

setImmediate(() => console.log('3: setImmediate'));

Promise.resolve().then(() => console.log('4: Promise'));

process.nextTick(() => console.log('5: nextTick'));

console.log('6: Sync');

// Output:
// 1: Sync
// 6: Sync
// 5: nextTick      <- process.nextTick runs first
// 4: Promise       <- then microtasks (promises)
// 2: setTimeout    <- then macrotasks
// 3: setImmediate
```

### Execution Order
1. Synchronous code
2. `process.nextTick()` callbacks
3. Promise microtasks
4. Timer callbacks (setTimeout/setInterval)
5. `setImmediate()` callbacks

### Async Patterns
```javascript
// Callback style
fs.readFile('file.txt', (err, data) => {
  if (err) throw err;
  console.log(data);
});

// Promise style
const fsPromises = require('fs/promises');
fsPromises.readFile('file.txt')
  .then(data => console.log(data))
  .catch(err => console.error(err));

// Async/await style (preferred)
async function readFile() {
  try {
    const data = await fsPromises.readFile('file.txt');
    console.log(data);
  } catch (err) {
    console.error(err);
  }
}

// Promisify callback-based functions
const util = require('util');
const readFilePromise = util.promisify(fs.readFile);
```

### Common Pitfalls
- **Blocking the event loop** - long synchronous operations freeze everything
- **Uncaught promise rejections** - always use `.catch()` or `try/catch`
- **process.nextTick() recursion** - can starve I/O; use setImmediate instead
- **Assuming setTimeout(fn, 0) runs immediately** - runs after current phase completes
- **Not understanding timer order** - setTimeout vs setImmediate order depends on context

### Practice Exercises
1. Demonstrate blocking vs non-blocking with sync/async file reads and HTTP server
2. Create a function that returns a Promise and implement proper error handling
3. Build a task scheduler using setImmediate to avoid blocking event loop

**Docs:** https://nodejs.org/api/timers.html | https://nodejs.org/api/async_hooks.html

---

## Learning Path

### Week 1-2: Fundamentals
**Focus:** Runtime, modules, basic I/O
- [ ] Process object and environment
- [ ] CommonJS vs ES Modules
- [ ] File system operations (read, write, append)
- [ ] Path utilities

**Mini-project:** CLI tool that reads files, transforms content, and writes output

### Week 3-4: Event-Driven Programming
**Focus:** Events, streams, async patterns
- [ ] EventEmitter pattern
- [ ] Stream types and piping
- [ ] Backpressure handling
- [ ] Promises and async/await

**Mini-project:** Log file processor using streams with custom Transform stream

### Week 5-6: Networking
**Focus:** HTTP, networking, real-time
- [ ] HTTP server and client
- [ ] Request/response lifecycle
- [ ] Headers, cookies, routing
- [ ] TCP sockets (net module)

**Mini-project:** RESTful API with routing and JSON responses

### Week 7-8: Advanced Patterns
**Focus:** Child processes, performance, debugging
- [ ] Spawning processes (exec, spawn, fork)
- [ ] IPC between processes
- [ ] Event loop deep dive
- [ ] Performance profiling

**Mini-project:** Task queue system using worker processes

---

## Key Takeaways

**Core Concepts to Master:**
1. **Everything is async** - understand event loop, promises, callbacks
2. **Streams are powerful** - use them for large data, memory efficiency
3. **Error handling is critical** - always handle errors in callbacks, promises, events
4. **Node is single-threaded** - don't block the event loop
5. **Modules are cached** - same instance returned on subsequent requires

**When Studying:**
- Always code along - reading isn't enough
- Break things intentionally - see what errors look like
- Read Node.js source code for modules you use
- Profile and measure - don't assume, verify

**Documentation:** https://nodejs.org/api/
