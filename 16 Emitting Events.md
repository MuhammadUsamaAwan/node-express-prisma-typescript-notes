```ts
import EventEmitter from 'events'

class MyEmitter extends EventEmitter {}

const myEmitter = new MyEmitter()

myEmitter.on('run', (msg) => console.log(msg))
myEmitter.once('run-once', (msg) => console.log(msg)) // this will run only once

myEmitter.emit('run', 'log this')
```
