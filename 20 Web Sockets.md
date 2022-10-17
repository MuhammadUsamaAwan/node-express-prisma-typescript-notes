```
npm i socket.io
```

```ts
// helper/users

const users: { id: string; username: string; room: string }[] = []

export const joinUser = (id: string, username: string, room: string) =>
  users.push({ id, username, room })

export const getRoomUsers = (room: string) =>
  users.filter(user => user.room === room)

export const removeUser = (id: string) => {
  const index = users.findIndex(user => user.id === id)
  if (index !== -1) {
    return users.splice(index, 1)[0]
  }
}
```

```ts
import express from 'express'
import { joinUser, getRoomUsers, removeUser } from './helper/users'
const app = express()
import http from 'http'
const server = http.createServer(app)
import { Server } from 'socket.io'
const io = new Server(server)
const PORT = process.env.PORT || 5000

app.use(express.static('public'))

io.on('connection', socket => {
  // join room
  socket.on('join-room', (username, room) => {
    joinUser(socket.id, username, room)
    socket.join(room)
    io.to(room).emit('user-joined', username, new Date())
    io.to(room).emit('room-users', getRoomUsers(room))
  })
  // disconnect
  socket.on('disconnect', () => {
    const user = removeUser(socket.id)
    if (user) {
      io.to(user.room).emit('user-left', user.username, new Date())
      io.to(user.room).emit('room-users', getRoomUsers(user.room))
      socket.broadcast.to(user.room).emit('typing-message-remove')
    }
  })
  // new messge
  socket.on('new-message', (message, username, room) => {
    io.to(room).emit('receive-message', username, new Date(), message)
  })
  // user is typing
  socket.on('user-typing', (username, room) => {
    socket.broadcast.to(room).emit('typing-message', username)
  })
  // user typing false
  socket.on('user-typing-false', room => {
    socket.broadcast.to(room).emit('typing-message-remove')
  })
})

server.listen(PORT, () => console.log(`Server started on port ${PORT}`))
```
