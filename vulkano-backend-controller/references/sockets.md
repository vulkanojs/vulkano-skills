## Sockets

1. Register the event in `app/config/sockets/events.js`:

```js
module.exports = {
  eventName: 'sockets.NameController.method'
};
```

2. Implement it in `app/controllers/sockets/NameController.js`:

```js
module.exports = {
  method({ socket, body }, callback) {
    callback({/* result */});
  }
};
```

Handler signature is always `({ socket, body }, callback)`. Requires `app/config/sockets/config.js` with `enabled: true` (adapter defaults to in-memory; redis/mongodb available). Optional `app/config/sockets/cors.js` and `app/config/sockets/middlewares/auth.js` for origin/auth checks.
