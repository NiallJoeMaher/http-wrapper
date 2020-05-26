# Deno HTTP Wrapper

### _Simple Server/Router wrapper around Deno's HTTP module_

[Example app using Vite and Deno](https://arcane-cove-31742.herokuapp.com/)

[Example GitHub repo](https://github.com/lindsaykwardell/http-wrapper-example)

## Intent

I really like Deno as a concept, and I especially like the default HTTP library. I feel like it doesn't need all the abstraction put on top of it to build something pretty good.

The intent of this library is to build a lightweight wrapper around it that provides a more intuitive API around the `listenAndServe()` function.

Some of the API is inspired by Express.js.

## Install

As with all Deno modules, `http_wrapper` is imported via URL, and not with a package manager:

```javascript
import {
  Server,
  Router,
  Socket,
} from "https://deno.land/x/http_wrapper@v0.3.0/mod.ts";
```

## Example

```javascript
import {
  Server,
  Router,
  Socket,
} from "https://deno.land/x/http_wrapper@v0.3.0/mod.ts";

// Create a new route
const router = new Router();

// Add an endpoint to the route
// Deno's default ServerRequest object is used

// This endpoint will be accessible at /
router.get("/", async (req: ServerRequest) => {
  // Respond using default methods
  req.respond({
    status: 200,
    headers: new Headers({
      "content-type": "text/html",
    }),
    body: await Deno.open("./index.html"),
  });
});

// Create a second endpoint
const bobRouter = new Router("/bob");

// This endpoint will be accessible at /bob
bobRouter.get("/", (req) => {
  req.respond({
    status: 200,
    headers: new Headers({
      "content-type": "application/json",
    }),
    body: JSON.stringify({
      test: "This is a test",
    }),
  });
});

// Create a web socket route at /ws
const socket = new Socket("/ws");

// Add a message event to listen for
socket.on("message", (msg: Broadcast, connId: string): void => {
  console.log("Received");
  socket.emit("message", `[${connId}]: ${msg.body}`);
});

// Create the server
const app = new Server();

// Add routes to server
app.use(router.routes);
app.use(bobRouter.routes);
app.use(socket.routes);

// Add static assets folder
app.static("static", "/static");

// Start the server
app
  .start({ port: 3000 })
  .then((config) => console.log(`Server running on localhost:${config.port}`));
```

## Use

There are two classes: `Server` and `Router`.

### Router

`Router` currently supports seven HTTP methods:

- GET
- POST
- PUT
- DELETE
- PATCH
- OPTIONS
- HEAD

New routes are created as follows:

```javascript
const router = new Router();
```

`new Router()` accepts a string for its constructor of new route (see example above).

A new endpoint for the given route can be added by using its specific method:

```javascript
router.get("/", (req) => {
  // Perform actions on request
});
```

The request is the standard `ServerRequest` object native to Deno. It is not modified in any way, and you can interact with it directly. The purpose of this library is to make it easy to work with, not to change the interface.

### Socket

The `Socket` class is a wrapper around `Router`, and is initialized the same way.

```javascript
// Create a new socket endpoint
const socket = new Socket("/ws");
```

Internally, web socket connections to this endpoint are handled with Deno's standard library:

- WebSocket
- acceptable
- acceptWebSocket
- isWebSocketCloseEvent

This class wraps the above functionality, and acts as a router around certain events. Messages are stringified JSON, using the below structure:

```javascript
type Broadcast = {
  from?: string,
  to?: string,
  event?: string,
  body: any,
};
```

There is currently no client developed to handle this object, just the server implementation, but it should be simple enough to write one.

Socket events are implemented as follows:

```javascript
const socket = new Socket("/ws");

socket.on("message", (msg: Broadcast, connId: string): void => {
  console.log("Received");
  socket.emit("message", `[${connId}]: ${msg.body}`);
});
```

An additional `to` or `from` value can be set as a third parameter:

```javascript
const socket = new Socket("/ws");

socket.on("message", (msg: Broadcast, connId: string): void => {
  console.log("Received");
  socket.emit("message", `[${connId}]: ${msg.body}`, {
    to: anotherUserId,
    from: connId,
  });
});
```

### Server

To add a static file folder (useful for serving HTML/CSS/JS files), use the following:

```javascript
const app = new Server();

// First attribute is local folder where files are located, second is the route to load the files from
app.static("static", "/static");
```

When you are ready to apply your routes and start your server, run the following:

```javascript
const app = new Server();

app.use(router.routes); // router.routes is a getter, so you do not need to invoke it as a function.
app.use(socket.routes); // Sockets are added to the server in the same way as routers.

// Using promises to know when the server is up
app
  .start({ port: 3000 })
  .then((config) => console.log(`Server running on localhost:${config.port}`));

// Using await to know when the server is up
await app.start({ port: 3000 });

console.log("Server running on localhost:3000");
```

The benefit of using promises is having the config object returned that you passed in, but it isn't important if you don't want to see it.

## License

This project is MIT Licensed.
