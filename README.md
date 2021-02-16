# obs-websocket-js

<p align="center"><i>
obs-websocket-js allows Javascript-based connections to the Open Broadcaster plugin <a href="https://github.com/Palakis/obs-websocket">obs-websocket</a>.
</i></p>

This is a fork of the original project available at <a href="https://github.com/haganbmj/obs-websocket-js">haganbmj/obs-websocket-js</a>.

## Installation

```sh
npm install @streamdasher/obs-websocket-js --save
```

Typescript definitions are included in this package, and are automatically generated to match the latest `obs-websocket` release.

## Usage
#### Instantiation
The web distributable exposes a global named `OBSWebSocket`.  

```html
<script type='text/javascript' src='./dist/obs-websocket.js'></script>
```

In node...  

```js
const OBSWebSocket = require('obs-websocket-js');
```

Create a new WebSocket connection using the following.
- Address is optional; defaults to `localhost` with a port of `4444`.  
- Password is optional.  

```js
const obs = new OBSWebSocket();
obs.connect({ address: 'localhost:4444', password: '$up3rSecretP@ssw0rd' });
```

#### Sending Requests
All requests support the following two Syntax options where both `err` and `data` will contain the raw response from the WebSocket plugin.  
_Note that all response objects will supply both the original [obs-websocket][link-obswebsocket] response items in their original format (ex: `'response-item'`), but also camelCased (ex: `'responseItem'`) for convenience._  
- RequestName must exactly match what is defined by the [`obs-websocket`][link-obswebsocket] plugin.  
- `{args}` are optional. Note that both `request-type` and `message-id` will be bound automatically.  
- To use callbacks instead of promises, use the `sendCallback` method instead of `send`.

```js
// Promise API
obs.send('RequestName', {args}) // returns Promise

// Callback API
obs.sendCallback('RequestName', {args}, callback(err, data)) // no return value

// The following are additional supported requests.
obs.connect({ address: 'address', password: 'password' }) // returns Promise
obs.disconnect();
```

#### Receiving Events
For all events, `data` will contain the raw response from the WebSocket plugin.  
_Note that all response objects will supply both the original [obs-websocket][link-obswebsocket] response items in their original format (ex: `'response-item'`), but also camelCased (ex: `'responseItem'`) for convenience._  
- EventName must exactly match what is defined by the [`obs-websocket`][link-obswebsocket] plugin.

```js
const callback = (data) => {
	console.log(data);
};

obs.on('EventName', (data) => callback(data));

// The following are additional supported events.
obs.on('ConnectionOpened', (data) => callback(data));
obs.on('ConnectionClosed', (data) => callback(data));
obs.on('AuthenticationSuccess', (data) => callback(data));
obs.on('AuthenticationFailure', (data) => callback(data));
```

#### Handling Errors
By default, certain types of WebSocket errors will be thrown as uncaught exceptions.
To ensure that you are handling every error, you must do the following:
1. Add a `.catch()` handler to every returned Promise.
2. Add a `error` event listener to the `OBSWebSocket` object. By default only errors on the initial socket connection will be caught. Any subsequent errors will be emit here and will be considered uncaught without this handler.

```js
// You must add this handler to avoid uncaught exceptions.
obs.on('error', err => {
    console.error('socket error:', err);
});
```

#### Example
See more examples in [`\samples`](samples).
```js
const OBSWebSocket = require('obs-websocket-js');

const obs = new OBSWebSocket();
obs.connect({
        address: 'localhost:4444',
        password: '$up3rSecretP@ssw0rd'
    })
    .then(() => {
        console.log(`Success! We're connected & authenticated.`);

        return obs.send('GetSceneList');
    })
    .then(data => {
        console.log(`${data.scenes.length} Available Scenes!`);

        data.scenes.forEach(scene => {
            if (scene.name !== data.currentScene) {
                console.log(`Found a different scene! Switching to Scene: ${scene.name}`);

                obs.send('SetCurrentScene', {
                    'scene-name': scene.name
                });
            }
        });
    })
    .catch(err => { // Promise convention dicates you have a catch on every chain.
        console.log(err);
    });

obs.on('SwitchScenes', data => {
    console.log(`New Active Scene: ${data.sceneName}`);
});

// You must add this handler to avoid uncaught exceptions.
obs.on('error', err => {
    console.error('socket error:', err);
});
```

#### Debugging
To enable debug logging, set the `DEBUG` environment variable:

```sh
# Enables debug logging for all modules of osb-websocket-js
DEBUG=obs-websocket-js:*

# on Windows
set DEBUG=obs-websocket-js:*
```

If you have multiple libraries or application which use the `DEBUG` environment variable, they can be joined with commas:

```sh
DEBUG=foo,bar:*,obs-websocket-js:*

# on Windows
set DEBUG=foo,bar:*,obs-websocket-js:*
```

Browser debugging uses `localStorage`

```js
localStorage.debug = 'obs-websocket-js:*';

localStorage.debug = 'foo,bar:*,obs-websocket-js:*';
```

For more information, see the [`debug`][link-debug] documentation.



  [link-obswebsocket]: https://github.com/Palakis/obs-websocket "OBS WebSocket Plugin"
  [link-debug]: https://github.com/visionmedia/debug "Debug Documentation"
