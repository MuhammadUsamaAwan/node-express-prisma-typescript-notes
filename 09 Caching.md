```
npm i redis
```

```ts
// config/redisClient.ts

import { createClient } from 'redis'

const redisClient = createClient()

export default redisClient
```

```ts
// config/connectRedis.ts

import redisClient from './redisClient'

const connectRedis = async () => {
  redisClient.on('error', (err) => console.log('Redis Client Error', err))
  await redisClient.connect()
}

export default connectRedis
```

```ts
// server.ts

import connectRedis from './config/connectRedis'

connectRedis()
```

```ts
// helper/useCache.ts

import redisClient from '../config/redisClient'

const useCache = async (key: string, cb: Function) => {
  const data = await redisClient.get(key)
  if (data) return JSON.parse(data)
  const newData = await cb()
  redisClient.set(key, JSON.stringify(newData))
}

export default useCache
```

```ts
// todo/todo.controller.ts

import useCache from '../helper/useCache'

export const getTodos = async (req: Request, res: Response) => {
  const todos = await useCache('todos', async () => await prisma.todo.findMany())
  return res.status(200).json(todos)
}
```
