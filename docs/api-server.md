# Web-Socket Server

The [`Server`](#server) and [`ClusterServer`](#clusterserver) have pretty much the same API. 

They're responsible for providing the WebSocket server to enable communication between server and client.

## Server

Recommended for development and prototyping environments.

### `register (name: string, handler: Room, options?: any)`

Register a new session handler.

**Parameters:**

- `name: string` - The public name of the room. You'll use this name when joining the room from the client-side.
- `handler: Room` - Reference to the `Room` handler class.
- `options?: any` - Custom options for room initialization.

```typescript
// Register "chat" room
gameServer.register("chat", ChatRoom);

// Register "battle" room
gameServer.register("battle", BattleRoom);
```

!!! Tip
    You may register the same room handler multiple times with different `options`. When [Room#onInit()](http://localhost:8000/api-room/#oninit-options) is called, the `options` will contain the merged values you specified on [Server#register()](api-server/#register-name-string-handler-room-options-any) + the options provided by the first client on `client.join()`

#### Listening to matchmake events

The `register` method will return the registered handler instance, which you can listen to match-making events from outside the room instance scope. Such as:

- `"create"` - when a room has been created
- `"dispose"` - when a room has been disposed
- `"join"` - when a client join a room
- `"leave"` - when a client leave a room
- `"lock"` - when a room has been locked
- `"unlock"` - when a room has been unlocked

**Usage:**

```typescript
gameServer.register("chat", ChatRoom).
  on("create", (room) => console.log("room created:", room.roomId)).
  on("dispose", (room) => console.log("room disposed:", room.roomId)).
  on("join", (room, client) => console.log(client.id, "joined", room.roomId)).
  on("leave", (room, client) => console.log(client.id, "left", room.roomId));
```

!!! Warning
    It's completely discouraged to manipulate a room's state through these events. Use the [abstract methods](api-room/#abstract-methods) in your room handler instead.

### `attach (options: any)`

Attaches or creates the WebSocket server.

- `options.server`: The HTTP server to attach the WebSocket server on.
- `options.ws`: An existing WebSocket server to be re-used.

```javascript fct_label="Express"
import * as express from "express";
import { Server } from "colyseus";

const app = new express();
const gameServer = new Server();

gameServer.attach({ server: app });
```

```javascript fct_label="http.createServer"
import * as http from "http";
import { Server } from "colyseus";

const httpServer = http.createServer();
const gameServer = new Server();

gameServer.attach({ server: httpServer });
```

```javascript fct_label="WebSocket.Server"
import * as http from "http";
import * as express from "express";
import * as ws from "ws";
import { Server } from "colyseus";

const app = express();
const server = http.createServer(app);
const wss = new WebSocket.Server({
    // your custom WebSocket.Server setup.
});

const gameServer = new Server();
gameServer.attach({ ws: wss });
```


### `listen (port: number)`

Binds the WebSocket server into the specified port.

### `onShutdown (callback: Function)`

Register a callback that should be called before the process shut down. See [graceful shutdown](api-graceful-shutdown/) for more details.

## ClusterServer

Recommended for production environment. 

The `ClusterServer` has the same functionality of `Server`, with some caveats. You'll need to use the `"cluster"` module by yourself and call its methods in the right node type. 

See [Clustered environment](concept-worker-processes/#clustered-environment) for more details.

### `fork (workers?: number)`

Specify the number of session workers to spawn. By default it uses the number of available CPUs.

```typescript fct_label="TypeScript"
import * as cluster from "cluster";
import { ClusterServer } from "colyseus";

let gameServer = new ClusterServer();

if (cluster.isMaster) {
    gameServer.listen(8080);
    gameServer.fork(4);

} else {
    // ...
}
```

```javascript fct_label="JavaScript"
const cluster = require("cluster");
const ClusterServer = require("colyseus").ClusterServer;

let gameServer = new ClusterServer();

if (cluster.isMaster) {
    gameServer.listen(8080);
    gameServer.fork(4);

} else {
    // ...
}
```