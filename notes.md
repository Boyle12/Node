# Node.js Core Concepts & Architecture Notes

## 1. Web Application Structure
Every web application has two main parts:
* **Frontend (Client Side):** What the user sees in the browser or mobile app. Built using HTML, CSS, JavaScript, React, etc.
* **Backend (Server Side):** Where data processing happens. It has two parts:
    1. **Server:** A **software** (not hardware) that is always running and ready to serve user requests.
    2. **Database:** Used to store data permanently. Examples: MySQL/PostgreSQL (SQL) and MongoDB (NoSQL).

---

## 2. Ports and APIs
* **Virtual Ports:** A computer has thousands of virtual ports. A server software connects to a specific port (like `3000` or `8080`) and constantly listens for incoming internet traffic.
* **APIs (Application Programming Interface):** Created on the server so the frontend can talk to a specific port, request data, and get a filtered response.

---

## 3. What is Node.js?
JavaScript was originally made to run only inside browsers. It could not touch local files or systems. 
**Node.js** is a runtime environment that gives JavaScript superpowers to run on servers. It contains:
1. **V8 Engine:** Google's engine that runs JavaScript code at high speed.
2. **C/C++ Libraries:** Background systems like `libuv` (for asynchronous operations) and `crypto` (for security).

---

## 4. Built-in Modules (Core Libraries)
Node.js provides built-in modules out of the box using the `require()` syntax:

### 1. `fs` (File System)
Used to read, write, and append files.
* **Synchronous (`readFileSync`):** Blocks the main thread. The system waits until the file operation finishes.
* **Asynchronous (`readFile`):** Runs in the background without stopping other tasks.

```javascript
const fs = require('fs');

// Synchronous Read
const syncData = fs.readFileSync('tasks.json', 'utf-8');

// Asynchronous Read
fs.readFile('tasks.json', 'utf-8', (err, data) => {
    if (err) console.log("Error reading file");
    else console.log(data);
});

// Appending data to a file
fs.appendFileSync('log.txt', 'New log entry\n');
```

2. path:
Helps handle file paths across different operating systems cleanly.

```javascript
JavaScript
const path = require('path');

// Safely joining directories and filenames
const absolutePath = path.join(__dirname, 'TODO', 'task.json'); 
console.log(absolutePath); // Outputs the full directory path

// Getting file extension
const ext = path.extname('server.js'); // .js
```

3. os (Operating System):
Provides information about the server machine's hardware and state.

```javascript
JavaScript
const os = require('os');

console.log("Total Memory: " + os.totalmem());
console.log("Free Memory: " + os.freemem());
console.log("CPU Architecture: " + os.arch());
```

4. http:
Used to open network ports and build web servers.

```javascript
JavaScript
const http = require('http');

const server = http.createServer((req, res) => {
    res.writeHead(200, { 'Content-Type': 'text/plain' });
    res.end('Hello from custom server!');
});

server.listen(3000, () => {
    console.log('Server is running on port 3000');
});
```

5. MIME Types (Multipurpose Internet Mail Extensions):
What it is: A label or identifier that tells the browser what kind of file format the server is sending.

Why it matters: Servers cannot guess file types automatically. We must mention it in the header so browsers render it correctly.

```javascript
JavaScript
// Managing MIME types inside code
const mimeTypes = {
    '.html': 'text/html',
    '.css': 'text/css',
    '.js': 'application/javascript',
    '.png': 'image/png'
};

// Default fallback if extension type is unknown
const contentType = mimeTypes['.html'] || 'application/octet-stream';
```

6. NGINX & Web Servers:

What it is: NGINX is a highly popular, fast web server used in production.

Main Job: Its primary job is to serve Static Assets (HTML, CSS, JS, Images) instantly when a user visits a website URL. Our custom http.createServer() routing acts similarly to a basic NGINX server.

7. Event-Driven Architecture:
Node.js works on an Event-Driven Architecture. The application waits quietly until an "Event" happens (like a user clicking a button or visiting a route), and then executes a specific callback function.

```javascript
Code Example:
JavaScript
const EventEmitter = require('events');

// Creating a custom class that extends EventEmitter
class Logger extends EventEmitter {
    log(message) {
        // Emit/Broadcast the event named 'message'
        this.emit('message', { id: 1, text: message });
    }
}

const logger = new Logger();

// Listener: Catches the 'message' event and executes action
logger.on('message', (eventData) => {
    console.log('Listener caught an event with data:', eventData);
});

// Triggering the log method
logger.log('Application Started');
```

8. HTTP Headers, Bodies, and Status Codes:
An HTTP response sent by a server has two main parts:

Response Head (Metadata): Sent using res.writeHead(). It contains status codes and content types.

Response Body (Content): Sent using res.end(). It contains the actual data or HTML page.

```javascript
Code Example:
JavaScript
const http = require('http');

http.createServer((req, res) => {
    if (req.url === '/') {
        // Success Header (200 OK)
        res.writeHead(200, { 'Content-Type': 'text/html' });
        res.end('<h1>Welcome to Home Page</h1>');
    } else {
        // Error Header (404 Not Found)
        res.writeHead(404, { 'Content-Type': 'text/plain' });
        res.end('404 Page Not Found Bro!');
    }
}).listen(3000);
```

9. Important Coding Insights & Common Bugs:

a. Data Buffers
The fs.readFileSync() method returns raw binary data (a Buffer), not a string. You must convert it using .toString() before parsing it with JSON.parse().

```javascript
JavaScript
// ❌ WRONG: Will cause parsing errors
// const data = JSON.parse(fs.readFileSync('tasks.json'));

//  CORRECT
const bufferData = fs.readFileSync('tasks.json');
const stringData = bufferData.toString();
const finalObject = JSON.parse(stringData);
```

b. process.argv
An array holding command-line arguments passed when running a file (e.g., node todo.js add "Buy Milk").

```javascript
JavaScript
// Index 0: Node binary path
// Index 1: Executed file path
// Index 2: Custom command ('add', 'list')
// Index 3: Custom argument ('Buy Milk')

const command = process.argv[2];
const argument = process.argv[3];
```

c. Circular Structure Error
This crash happens during JSON.stringify() if you accidentally push an array inside itself continuously (creating an infinite loop). Always push clean, individual objects.

```javascript
JavaScript
let tasks = [];
let singleTask = "Record Video";

// ❌ WRONG: Do not push the parent array into itself
// tasks.push(tasks); 

//  CORRECT
tasks.push({ task: singleTask });
const dataJSON = JSON.stringify(tasks);