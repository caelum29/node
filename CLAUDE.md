# Node.js Study Guide

> A comprehensive guide for studying the Node.js API and ecosystem
> Based on [Node.js Official Documentation](https://nodejs.org/api/)

## Table of Contents

1. [Getting Started](#getting-started)
2. [Core Fundamentals](#core-fundamentals)
3. [Module Systems](#module-systems)
4. [I/O & Networking](#io--networking)
5. [System & Environment](#system--environment)
6. [Async Programming](#async-programming)
7. [Development & Debugging](#development--debugging)
8. [Advanced Topics](#advanced-topics)
9. [Learning Path](#learning-path)

---

## Getting Started

### Prerequisites
- Basic JavaScript knowledge
- Understanding of asynchronous programming concepts
- Familiarity with command line/terminal

### Installation & Setup
```bash
# Check Node.js version
node --version

# Check npm version
npm --version

# Run a Node.js file
node app.js

# Start REPL (Read-Eval-Print Loop)
node
```

---

## Core Fundamentals

### 1. Process
**Documentation:** https://nodejs.org/api/process.html

The `process` object provides information about and control over the current Node.js process.

**Key Concepts:**
- `process.argv` - Command line arguments
- `process.env` - Environment variables
- `process.cwd()` - Current working directory
- `process.exit([code])` - Exit the process
- `process.nextTick(callback)` - Defer execution

**Study Points:**
- Environment variable management
- Process signals (SIGTERM, SIGINT)
- Standard streams (stdin, stdout, stderr)
- Process lifecycle and exit codes

### 2. Events
**Documentation:** https://nodejs.org/api/events.html

Node.js uses an event-driven architecture where many objects emit events.

**Key Concepts:**
```javascript
const EventEmitter = require('events');
class MyEmitter extends EventEmitter {}

const myEmitter = new MyEmitter();
myEmitter.on('event', () => console.log('Event fired!'));
myEmitter.emit('event');
```

**Study Points:**
- Creating custom EventEmitters
- `on()`, `once()`, `emit()`, `removeListener()`
- Error event handling
- Event ordering and synchronous vs asynchronous

### 3. Buffer
**Documentation:** https://nodejs.org/api/buffer.html

Buffers handle binary data directly in Node.js.

**Key Concepts:**
```javascript
// Creating buffers
const buf1 = Buffer.from('hello world');
const buf2 = Buffer.alloc(10);
const buf3 = Buffer.allocUnsafe(10);

// Working with buffers
buf1.toString(); // Convert to string
buf1.length;     // Get length
```

**Study Points:**
- Buffer vs String encoding (utf8, base64, hex)
- Buffer pooling and memory management
- TypedArrays and ArrayBuffer relationship
- Performance considerations

### 4. Streams
**Documentation:** https://nodejs.org/api/stream.html

Streams are collections of data that might not be available all at once.

**Types of Streams:**
- **Readable** - Read data (e.g., `fs.createReadStream()`)
- **Writable** - Write data (e.g., `fs.createWriteStream()`)
- **Duplex** - Both readable and writable (e.g., `net.Socket`)
- **Transform** - Modify data while reading/writing (e.g., `zlib.createGzip()`)

**Key Concepts:**
```javascript
const fs = require('fs');

// Pipe data from one stream to another
const readStream = fs.createReadStream('input.txt');
const writeStream = fs.createWriteStream('output.txt');
readStream.pipe(writeStream);

// Handle events
readStream.on('data', (chunk) => console.log(chunk));
readStream.on('end', () => console.log('Done'));
readStream.on('error', (err) => console.error(err));
```

**Study Points:**
- Backpressure and flow control
- Object mode vs binary mode
- Pipeline and error handling
- Stream performance benefits

### 5. Utilities
**Documentation:** https://nodejs.org/api/util.html

Utility functions for common programming tasks.

**Key Functions:**
- `util.promisify()` - Convert callback-based functions to promises
- `util.inspect()` - Debug object representation
- `util.types` - Type checking utilities
- `util.format()` - String formatting

---

## Module Systems

### 1. CommonJS Modules
**Documentation:** https://nodejs.org/api/modules.html

The traditional Node.js module system.

**Key Concepts:**
```javascript
// Exporting
module.exports = function() { /* ... */ };
exports.myFunction = function() { /* ... */ };

// Importing
const myModule = require('./myModule');
const { myFunction } = require('./myModule');
```

**Study Points:**
- `require()` caching mechanism
- Circular dependencies
- Module resolution algorithm
- `module.exports` vs `exports`

### 2. ES Modules (ESM)
**Documentation:** https://nodejs.org/api/esm.html

Modern JavaScript module system.

**Key Concepts:**
```javascript
// Exporting
export function myFunction() { /* ... */ }
export default class MyClass { /* ... */ }

// Importing
import { myFunction } from './myModule.mjs';
import MyClass from './myModule.mjs';
```

**Study Points:**
- File extensions (.mjs, .cjs, .js)
- Package.json "type" field
- Top-level await
- Interoperability with CommonJS
- Dynamic imports

### 3. Packages
**Documentation:** https://nodejs.org/api/packages.html

**Study Points:**
- package.json structure and fields
- Conditional exports
- Dual package hazards
- Package entry points
- Semver versioning

---

## I/O & Networking

### 1. File System (fs)
**Documentation:** https://nodejs.org/api/fs.html

File system operations for reading, writing, and managing files.

**Key Concepts:**
```javascript
const fs = require('fs');
const fsPromises = require('fs/promises');

// Callback style
fs.readFile('file.txt', 'utf8', (err, data) => {
  if (err) throw err;
  console.log(data);
});

// Promise style
async function readFile() {
  const data = await fsPromises.readFile('file.txt', 'utf8');
  console.log(data);
}

// Synchronous
const data = fs.readFileSync('file.txt', 'utf8');
```

**Study Points:**
- Async vs sync operations
- File descriptors and low-level operations
- Directory operations (mkdir, readdir, rmdir)
- File stats and permissions
- Watch files for changes
- Working with file streams

### 2. Path
**Documentation:** https://nodejs.org/api/path.html

Utilities for working with file and directory paths.

**Key Functions:**
```javascript
const path = require('path');

path.join('/foo', 'bar', 'baz');      // Join path segments
path.resolve('foo', 'bar');            // Resolve to absolute path
path.basename('/foo/bar/file.txt');    // Get filename
path.dirname('/foo/bar/file.txt');     // Get directory
path.extname('file.txt');              // Get extension
```

**Study Points:**
- Platform differences (Windows vs POSIX)
- Relative vs absolute paths
- Path normalization

### 3. HTTP/HTTPS
**Documentation:** https://nodejs.org/api/http.html

Create HTTP servers and make HTTP requests.

**Key Concepts:**
```javascript
const http = require('http');

// Create server
const server = http.createServer((req, res) => {
  res.writeHead(200, { 'Content-Type': 'text/plain' });
  res.end('Hello World\n');
});

server.listen(3000, () => {
  console.log('Server running on port 3000');
});

// Make request
http.get('http://example.com', (res) => {
  let data = '';
  res.on('data', (chunk) => data += chunk);
  res.on('end', () => console.log(data));
});
```

**Study Points:**
- Request and response objects
- HTTP methods and status codes
- Headers and cookies
- Keep-alive connections
- HTTP/2 support

### 4. Net (TCP)
**Documentation:** https://nodejs.org/api/net.html

Create TCP servers and clients.

**Study Points:**
- Socket programming
- TCP vs UDP
- Connection lifecycle
- Half-open connections

### 5. DNS
**Documentation:** https://nodejs.org/api/dns.html

DNS lookups and name resolution.

### 6. URL
**Documentation:** https://nodejs.org/api/url.html

URL parsing and formatting.

**Key Concepts:**
```javascript
const { URL } = require('url');

const myURL = new URL('https://example.com:8080/path?query=value#hash');
console.log(myURL.hostname);  // example.com
console.log(myURL.pathname);  // /path
console.log(myURL.searchParams.get('query')); // value
```

---

## System & Environment

### 1. OS
**Documentation:** https://nodejs.org/api/os.html

Operating system-related utility methods.

**Key Functions:**
```javascript
const os = require('os');

os.platform();    // Operating system platform
os.arch();        // CPU architecture
os.cpus();        // CPU information
os.totalmem();    // Total memory
os.freemem();     // Free memory
os.homedir();     // Home directory
os.tmpdir();      // Temp directory
```

### 2. Child Processes
**Documentation:** https://nodejs.org/api/child_process.html

Spawn child processes to run system commands.

**Key Methods:**
```javascript
const { exec, spawn, fork } = require('child_process');

// Execute command
exec('ls -la', (error, stdout, stderr) => {
  console.log(stdout);
});

// Spawn process
const ls = spawn('ls', ['-la']);
ls.stdout.on('data', (data) => console.log(data.toString()));

// Fork Node.js process
const child = fork('./child.js');
child.send({ message: 'Hello' });
```

**Study Points:**
- exec vs spawn vs fork
- IPC (Inter-Process Communication)
- Process management and signals
- Security considerations

### 3. Cluster
**Documentation:** https://nodejs.org/api/cluster.html

Create child processes that share server ports.

**Study Points:**
- Multi-core utilization
- Load balancing strategies
- Worker process management
- Zero-downtime restarts

### 4. Worker Threads
**Documentation:** https://nodejs.org/api/worker_threads.html

Run JavaScript in parallel using threads.

**Study Points:**
- Threads vs processes
- SharedArrayBuffer and Atomics
- Message passing
- CPU-intensive operations

---

## Async Programming

### 1. Async Hooks
**Documentation:** https://nodejs.org/api/async_hooks.html

Track asynchronous resources throughout their lifecycle.

**Study Points:**
- Async context tracking
- Performance monitoring
- Resource lifecycle

### 2. Async Context Tracking
**Documentation:** https://nodejs.org/api/async_context.html

Maintain context across async operations.

### 3. Promises & Timers
**Documentation:** https://nodejs.org/api/timers.html

**Key Functions:**
```javascript
// Timers
setTimeout(() => console.log('After 1s'), 1000);
setInterval(() => console.log('Every 1s'), 1000);
setImmediate(() => console.log('Immediate'));

// Promise-based timers
const { setTimeout: setTimeoutPromise } = require('timers/promises');
await setTimeoutPromise(1000);
```

**Study Points:**
- Event loop phases
- Timer precision and drift
- `setImmediate()` vs `process.nextTick()` vs `setTimeout(..., 0)`

---

## Development & Debugging

### 1. Console
**Documentation:** https://nodejs.org/api/console.html

Debugging output similar to browser console.

**Key Methods:**
```javascript
console.log('Info');
console.error('Error');
console.warn('Warning');
console.table([{ a: 1, b: 2 }]);
console.time('label');
// ... code ...
console.timeEnd('label');
console.trace('Stack trace');
```

### 2. Assert
**Documentation:** https://nodejs.org/api/assert.html

Assertion testing for unit tests.

**Key Functions:**
```javascript
const assert = require('assert');

assert.strictEqual(actual, expected);
assert.deepStrictEqual(obj1, obj2);
assert.throws(() => { throw new Error(); });
assert.ok(value); // Truthy check
```

### 3. Test Runner
**Documentation:** https://nodejs.org/api/test.html

Built-in testing framework (Node.js 18+).

**Key Concepts:**
```javascript
const test = require('node:test');
const assert = require('assert');

test('synchronous test', (t) => {
  assert.strictEqual(1 + 1, 2);
});

test('async test', async (t) => {
  const result = await asyncFunction();
  assert.strictEqual(result, expected);
});

test('subtests', async (t) => {
  await t.test('subtest 1', (t) => {
    assert.ok(true);
  });
});
```

### 4. Debugger
**Documentation:** https://nodejs.org/api/debugger.html

Built-in debugging capabilities.

**Commands:**
```bash
# Start with debugger
node inspect app.js

# Debug commands
cont, c    - Continue execution
next, n    - Step next
step, s    - Step in
out, o     - Step out
pause      - Pause execution
```

**Study Points:**
- Chrome DevTools integration
- VS Code debugging
- Breakpoints and watch expressions

### 5. Inspector
**Documentation:** https://nodejs.org/api/inspector.html

Access to the V8 inspector.

### 6. Performance Hooks
**Documentation:** https://nodejs.org/api/perf_hooks.html

Measure performance of operations.

**Key Concepts:**
```javascript
const { performance, PerformanceObserver } = require('perf_hooks');

// Measure time
performance.mark('start');
// ... operation ...
performance.mark('end');
performance.measure('My operation', 'start', 'end');

// Observe measurements
const obs = new PerformanceObserver((items) => {
  console.log(items.getEntries()[0].duration);
});
obs.observe({ entryTypes: ['measure'] });
```

---

## Advanced Topics

### 1. Crypto
**Documentation:** https://nodejs.org/api/crypto.html

Cryptographic functionality (OpenSSL's hash, HMAC, cipher, decipher, sign, verify).

**Key Concepts:**
```javascript
const crypto = require('crypto');

// Hash
const hash = crypto.createHash('sha256');
hash.update('data to hash');
console.log(hash.digest('hex'));

// Random bytes
crypto.randomBytes(16, (err, buf) => {
  console.log(buf.toString('hex'));
});

// Encryption
const algorithm = 'aes-256-cbc';
const key = crypto.randomBytes(32);
const iv = crypto.randomBytes(16);
const cipher = crypto.createCipheriv(algorithm, key, iv);
```

**Study Points:**
- Hashing vs encryption
- Symmetric vs asymmetric encryption
- HMAC and digital signatures
- Secure random number generation
- Certificate management

### 2. VM
**Documentation:** https://nodejs.org/api/vm.html

Execute JavaScript in a sandboxed environment.

**Study Points:**
- V8 contexts
- Script compilation and execution
- Security considerations

### 3. C++ Addons
**Documentation:** https://nodejs.org/api/addons.html

Extend Node.js with C++ code.

**Study Points:**
- Node-API (N-API)
- Memory management
- Async operations in native code
- Building and distributing addons

### 4. Zlib
**Documentation:** https://nodejs.org/api/zlib.html

Compression using Gzip and Deflate/Inflate.

**Key Concepts:**
```javascript
const zlib = require('zlib');
const fs = require('fs');

// Compress file
const gzip = zlib.createGzip();
const input = fs.createReadStream('input.txt');
const output = fs.createWriteStream('input.txt.gz');
input.pipe(gzip).pipe(output);
```

### 5. REPL
**Documentation:** https://nodejs.org/api/repl.html

Create custom REPLs for interactive programming.

### 6. SQLite
**Documentation:** https://nodejs.org/api/sqlite.html

Built-in SQLite database support (experimental).

### 7. WebAssembly
**Documentation:** https://nodejs.org/api/wasi.html

Run WebAssembly modules in Node.js.

---

## Learning Path

### Beginner (Weeks 1-2)
1. **Process & Globals** - Understand the Node.js runtime
2. **CommonJS Modules** - Learn module system basics
3. **File System (fs)** - Read and write files
4. **Path** - Work with file paths
5. **Events** - Event-driven programming
6. **Console & Debugging** - Debug your applications

**Practice Projects:**
- File reader/writer CLI tool
- Log file analyzer
- Simple task manager with file storage

### Intermediate (Weeks 3-4)
1. **HTTP/HTTPS** - Create web servers
2. **Streams** - Efficient data handling
3. **Buffer** - Binary data manipulation
4. **Child Processes** - Execute system commands
5. **ES Modules** - Modern module syntax
6. **Error Handling** - Proper error management
7. **Async/Await** - Promise-based async programming

**Practice Projects:**
- RESTful API server
- File upload/download service
- CLI tool that spawns processes
- Web scraper

### Advanced (Weeks 5-8)
1. **Cluster & Worker Threads** - Scalable applications
2. **Performance Hooks** - Optimize performance
3. **Crypto** - Security and encryption
4. **Test Runner** - Write comprehensive tests
5. **Async Hooks** - Advanced async patterns
6. **Net (TCP/UDP)** - Low-level networking
7. **HTTP/2** - Modern protocols
8. **VM** - Code execution sandboxing

**Practice Projects:**
- Load-balanced web server
- Real-time chat application
- Encrypted file storage system
- Custom testing framework
- Process manager like PM2

### Expert (Ongoing)
1. **C++ Addons** - Native extensions
2. **V8 API** - Deep runtime integration
3. **WebAssembly** - High-performance modules
4. **Custom Streams** - Advanced stream implementations
5. **Security** - Best practices and hardening
6. **Performance Tuning** - Profiling and optimization

---

## Best Practices

### 1. Error Handling
```javascript
// Always handle errors
fs.readFile('file.txt', (err, data) => {
  if (err) {
    console.error('Error reading file:', err);
    return;
  }
  // Process data
});

// Use try-catch with async/await
async function readFile() {
  try {
    const data = await fsPromises.readFile('file.txt');
    return data;
  } catch (err) {
    console.error('Error:', err);
    throw err; // Or handle appropriately
  }
}

// Handle unhandled rejections
process.on('unhandledRejection', (reason, promise) => {
  console.error('Unhandled Rejection:', reason);
});
```

### 2. Async Operations
```javascript
// Prefer async/await over callbacks
// BAD - Callback hell
fs.readFile('file1.txt', (err, data1) => {
  fs.readFile('file2.txt', (err, data2) => {
    fs.writeFile('output.txt', data1 + data2, (err) => {
      console.log('Done');
    });
  });
});

// GOOD - Async/await
async function processFiles() {
  const data1 = await fsPromises.readFile('file1.txt');
  const data2 = await fsPromises.readFile('file2.txt');
  await fsPromises.writeFile('output.txt', data1 + data2);
  console.log('Done');
}
```

### 3. Streams for Large Data
```javascript
// BAD - Load entire file into memory
const data = fs.readFileSync('large-file.txt');
process(data);

// GOOD - Stream the file
const stream = fs.createReadStream('large-file.txt');
stream.on('data', (chunk) => process(chunk));
```

### 4. Environment Variables
```javascript
// Use environment variables for configuration
const PORT = process.env.PORT || 3000;
const DB_URL = process.env.DATABASE_URL || 'mongodb://localhost:27017';

// Use dotenv for development
require('dotenv').config();
```

### 5. Security
- Validate all user input
- Use HTTPS in production
- Keep dependencies updated
- Use environment variables for secrets
- Implement rate limiting
- Sanitize user data before database operations
- Use helmet for HTTP security headers

---

## Resources

### Official Documentation
- [Node.js API Documentation](https://nodejs.org/api/)
- [Node.js Guides](https://nodejs.org/en/docs/guides/)
- [npm Documentation](https://docs.npmjs.com/)

### Learning Resources
- [Node.js Best Practices](https://github.com/goldbergyoni/nodebestpractices)
- [You Don't Know Node](https://github.com/azat-co/you-dont-know-node)
- [Node.js Design Patterns](https://www.nodejsdesignpatterns.com/)

### Tools & Frameworks
- **Express** - Web application framework
- **Fastify** - Fast web framework
- **NestJS** - Progressive Node.js framework
- **Jest** - Testing framework
- **ESLint** - Code linting
- **PM2** - Process manager
- **nodemon** - Auto-restart during development

---

## Contributing to Node.js

Interested in contributing to Node.js core?

1. Read the [Contributing Guide](CONTRIBUTING.md)
2. Check [Good First Issues](https://github.com/nodejs/node/labels/good%20first%20issue)
3. Join the [Node.js Slack](https://www.nodeslackers.com/)
4. Attend [Node.js working group meetings](https://github.com/nodejs/node/tree/main/doc/contributing)

---

## Quick Reference

### Common Patterns
```javascript
// Module exports
module.exports = { /* ... */ };
exports.func = function() { /* ... */ };

// Error-first callbacks
callback(err, result);

// Event emitter
emitter.on('event', handler);
emitter.emit('event', data);

// Streams
readable.pipe(writable);
stream.on('data', chunk => {});
stream.on('end', () => {});

// Promises
promise.then(result => {}).catch(err => {});

// Async/await
const result = await asyncFunction();
```

### Useful Commands
```bash
# Version management
node -v                    # Check Node version
npm -v                     # Check npm version

# Running scripts
node script.js             # Run script
node --inspect script.js   # Debug script
node --trace-warnings      # Show warning traces

# npm commands
npm init                   # Initialize package.json
npm install package        # Install package
npm install -g package     # Install globally
npm run script             # Run npm script
npm test                   # Run tests
npm audit                  # Check for vulnerabilities

# Environment
NODE_ENV=production node app.js
DEBUG=* node app.js        # Enable debug logs
```

---

## Study Tips

1. **Build Real Projects** - Apply concepts in practical applications
2. **Read the Docs** - Official documentation is comprehensive
3. **Experiment in REPL** - Test ideas quickly in the Node.js REPL
4. **Read Source Code** - Learn from popular packages on npm
5. **Debug Actively** - Use debugger to understand execution flow
6. **Performance Test** - Benchmark different approaches
7. **Stay Updated** - Follow Node.js release notes
8. **Join Community** - Participate in forums and discussions

---

**Happy Learning!** 🚀

Remember: Node.js is all about asynchronous, event-driven programming. Master these concepts and you'll master Node.js!
