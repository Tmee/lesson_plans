---
title: Building Real-Time Applications
length: 90
tags: json, javascript, websockets, jquery, socket.io, node
---

**Nota bene:** This is the successor to the Pub/Sub in the Browser lesson.

## Learning Goals

* Set up a basic Node.js server using Express
* Understand how web sockets work
* Implement pub/sub in the browser using Socket.io

## Resources

* [right-now][rn] repository  in [turingschool-examples][org]

[org]: https://github.com/turingschool-examples
[rn]: https://github.com/turingschool-examples/right-now

## Structure

* 5 - Warm up
* 20 - Lecture
* 5 - Break
* 25 - Experiment  #1: Right Now
* 5 - Break
* 25 - Experiment #2: Twitter Stream
* 5 - Wrap Up

## Warm Up

Last week we learned about pub/sub on the server. Reflect on the following questions before we begin:

* What are the limitations of using pub/sub to enable communication between the server and the client?
* How would you implement pub/sub in the browser?

## Lecture

* Explain WebSockets vs. the HTTP request/response cycle
  * A WebSocket is a persistent two-way connection between the server and the client
  * The server can push an event without having to receive a request from the client
  * The client can send messages to the server without having to wait for a response
* In what contexts would you want to consider WebSockets?
  * Chat/instant messaging
  * Real-time analytics
  * Document collaboration
  * Streaming
* WebSockets work best when there is an event-driven server on the backend
  * Ruby with [EventMachine][]
  * [Node.js][]
* [Socket.io][] and [Faye][] will fallback to other methods if it can't make a WebSocket connection
  * Some examples include: long-polling, Adobe Flash sockets
* ActionCable

[Socket.io]: http://socket.io/
[Faye]: http://faye.jcoglan.com/
[Node.js]:http://nodejs.org
[EventMachine]: http://rubyeventmachine.com/

## Experiment: Right Now

### Getting Started

Fist, clone [this repository][rn] if you have not already.  It contains a simple little Express app that server a static `index.html` page. It also has [Socket.io][] hooked up by default—despite the fact that we're not using it at this moment.

Once you have cloned the repo, you will probably need to `sudo npm install`.  Let npm install all the dependencies that are needed for the app.  If you want to see what those are, check out the `package.json`.  Once the dependencies are loaded, you can fire up the server with `npm start`.

```shell
$npm start

> right-now@1.0.0 start /Users/YourName/right-now
> ./node_modules/nodemon/bin/nodemon.js index.js

15 Jul 13:16:08 - [nodemon] v1.3.7
15 Jul 13:16:08 - [nodemon] to restart at any time, enter `rs`
15 Jul 13:16:08 - [nodemon] watching: *.*
15 Jul 13:16:08 - [nodemon] starting `node index.js`
Your server is up and running on Port 3000. Good job!
```

Let's start with a simple "hello world" implementation.

In our server, add the following code:

```js
// index.js
io.on('connection', function (socket) {
  console.log('Someone has connected.');
});
```

When a connection is made to via WebSocket, you're server will log it to the console. The next step, of course is create a connection via WebSocket, right?

```js
// public/application.js
var socket = io();
```

You should see the following in your terminal after you refresh the page:

```shell
Your server is up and running on Port 3000. Good job!
Someone has connected.
```

We can also let the client celebrate our new conneciton.

```js
// public/application.js
socket.on('connect', function () {
  console.log('You have connected!');
});
```

One thing that you've probably picked up on is that we have an `io` object on the server- as well as the client-side of our application.

So, let's send a message over the wire when a user connects.

```js
// index.js
io.on('connection', function (socket) {
  console.log('Someone has connected.');
  socket.emit('message', {user: 'turingbot', text: 'Hello, world!'});
});
```

Like everything with WebSockets, this is a two-part affair. The server is now emitting an event on the `message` channel. (This is arbitrary.) We now need to do something when the client receives that message.

```js
socket.on('message', function (message) {
  console.log('Something came along on the "message" channel:', message);
});
```

Super cool. You did the thing! Let's shoot some stuff over the wire on a regular interval.

```js
// index.js
io.on('connection', function (socket) {

  var interval = setInterval(function () {
    socket.emit('message', {user: 'turingbot', text: 'I am a banana.'});
  }, 1000);

  socket.on('disconnect', function () {
    clearInterval(interval);
  });
});
```

### Your Turn

* Can you write some jQuery to append these messages to the DOM?

### Talking Back to the Server

WebSockets are a two-way street. We can send something back to the server over `socket.send`.

```js
// public/application.js
socket.on('connect', function () {
  console.log('You have connected!');
  socket.send('message', {
    username: 'yournamehere',
    text: 'I did the thing.'
  });
});
```

Let's also write a listener on the server.

```js
// index.js
io.on('connection', function (socket) {

  var interval = setInterval(function () {
    socket.emit('message', {user: 'turingbot', text: 'I am a banana.'});
  }, 1000);

  socket.on('message', function (channel, message) {
    console.log(channel + ':', message);
  });

  socket.on('disconnect', function () {
    clearInterval(interval);
  });
});
```

### Your Turn

* Write the functionality on the client that sends something over the `gamenight` channel.
* Write the functionality on the server that listens on the `gamenight` channel and logs it to the console.

Here is a little bit of code to point you in the right direction.

```js
io.on('connection', function(socket) {
  socket.on('message', function (channel, message) {
    console.log(message);
  });
});
```

### Adding Some Nuance

So far, we've had one server talking to one client. This has a lot of practical value, but what if we wanted to create add more functionality to our little application? Socket.io has a few other APIs for fine-tuning your application.

```js
// Send to current request socket client
socket.emit('message', {user: 'turingbot', text: 'You can do the thing.'});

// Sending to all clients, include sender
io.sockets.emit('message', {user: 'turingbot', text: 'You can do the thing.'});

// Sending to all clients except sender
socket.broadcast.emit('message', {user: 'turingbot', text: 'You can do the thing.'});
```

You can also start to break your messaging out in additional channels. Here's an example.

```js
socket.on('new message', addMessageToPage);
socket.on('new connection', updateStatus.bind(null, 'A new user has connected.'));
socket.on('lost connection', updateStatus.bind(null, 'Someone has disconnected.'));
```

There are also some helpful methods for seeing how many clients are currently connected.

### Your Turn

* When a user connects. Broadcast a message to all of the other clients connected announcing that someone new has connected.
* When a user disconnects. Broadcast a message to all of the other clients connected announcing that someone new has disconnected.
* When a message comes in from a user. Broadcast it out to all users.

## Hooking Things Up with Redis

Open up a new terminal window and navigate to the root directory of the Right-Now app we've been working on.

Let's install the `redis` library in Node.

```
npm install redis --save
```

Then start the Redis server in that same window

```
redis-server
```

If that worked you should see something like this:

```shell
$redis-server
# Warning: no config file specified, using the default config. In order to specify a config file use redis-server /path/to/redis.conf
* Increased maximum number of open files to 10032 (it was originally set to 2560).
                _._
           _.-``__ ''-._
      _.-``    `.  `_.  ''-._           Redis 2.8.17 (00000000/0) 64 bit
  .-`` .-```.  ```\/    _.,_ ''-._
 (    '      ,       .-`  | `,    )     Running in stand alone mode
 |`-._`-...-` __...-.``-._|'` _.-'|     Port: 6379
 |    `-._   `._    /     _.-'    |     PID: 52027
  `-._    `-._  `-./  _.-'    _.-'
 |`-._`-._    `-.__.-'    _.-'_.-'|
 |    `-._`-._        _.-'_.-'    |           http://redis.io
  `-._    `-._`-.__.-'_.-'    _.-'
 |`-._`-._    `-.__.-'    _.-'_.-'|
 |    `-._`-._        _.-'_.-'    |
  `-._    `-._`-.__.-'_.-'    _.-'
      `-._    `-.__.-'    _.-'
          `-._        _.-'
              `-.__.-'

# Server started, Redis version 2.8.17
* The server is now ready to accept connections on port 6379
```

Head back to `index.js` so we can set up Redis. Require Redis like so:

```js
const redis = require('redis');
```

Next, we can create a client that connects:

```js
const client = redis.createClient();
```

Now, we'll connect to the channel `community`.  The app we will use to talk to our Express app will publish to a community channel by default. Remember the following code needs to be inside the function that has access to the socket:

```js
client.subscribe("community");
```

So, now let's log a message when it comes in over Redis:

```js
client.on("message", function (channel, message) {
  console.log(channel, message);
});
```

The next step is for us to fire up one of the [Slacker][] publishers (preferably `talker.rb`) and publish some messages.  Open up a new terminal window and clone that repo then `bundle install`.  Slacker is a Sinatra app so, to start up the publisher enter: `ruby publishers/talker.rb`.  To send a messege to the community channel, simply type a message at the prompt and press enter.  If your Express app was still running check the browser's console, otherwise `npm start` that server and send another message.  Look! Ruby is talking to Node. This is so amazing. Barriers: broken.

### Your Turn

Can you take a message from Slacker (via Redis) and push it over a socket to the client? Try adding the message onto the DOM.

[Slacker]: http://github.com/turingschool-examples/slacker

## Pair Project

We're going to build a small chat room (like [this one][ch]) using Socket.io and jQuery. Users should be able to fill out a little form, which will send their message over the WebSocket to the server, which will broadcast it out to all of the connected clients.

[ch]: https://fullstack-denver.herokuapp.com/websockets/

### Extensions

* Can you take advantage of [node-tweet-stream][nts] to stream tweets to your chatroom?

[nts]: https://www.npmjs.com/package/node-tweet-stream
