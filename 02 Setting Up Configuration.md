## Code Formatting

```json
// .prettierrc
{
  "useTabs": false,
  "semi": false,
  "trailingComma": "es5",
  "singleQuote": true,
  "printWidth": 120,
  "tabWidth": 2,
  "arrowParens": "avoid"
}
```

## Environment Variables

```dockerfile
#.env

NODE_ENV=development
PORT=5000
```

## .gitignore

```dockerfile
node_modules
*.log
.env
dist
```

## Error Handling

```
npm i express-async-errors
```

```ts
// server.ts

import 'express-async-errors'
```

```ts
// middleware/errorHandler.ts

import { Request, Response, NextFunction } from 'express'

const errorHandler = (err: Error, req: Request, res: Response, next: NextFunction) => {
  const status = res.statusCode || 500
  if (process.env.NODE_ENV === 'development') console.log(err.stack)
  return res.status(status).json({ message: err.message })
}

export default errorHandler
```

```ts
// server.ts

// below all routes
app.use(errorHandler)
```

## Not Found

```ts
// server.ts

app.all('*', (req: Request, res: Response) => {
  res.status(404).json({ message: '404 - No Route Found' })
})
```

## CORS

```
npm i cors
npm i -D @types/cors
```

```ts
// config/allowedOrigins.ts

const allowedOrigins: string[] = ['http://localhost:3000']

export default allowedOrigins
```

```ts
// config/corsOptions.ts

import allowedOrigins from './allowedOrigins'

const corsOptions = {
  origin: (origin: any, callback: Function) => {
    if (allowedOrigins.indexOf(origin) !== -1 || !origin) {
      callback(null, true)
    } else {
      callback(new Error('Not allowed by CORS'))
    }
  },
  credentials: true,
  optionsSuccessStatus: 200,
}

export default corsOptions
```

```ts
// server.ts

import cors from 'cors'

app.use(cors(corsOptions))
```

## Logger

```
npm i uuid
npm i -D @types/uuid
```

```ts
// loggerOptions.ts

export const loggerOptions = {
  useLogger: false,
  console: false,
  requestMethods: ['GET', 'POST', 'PUT', 'PATCH', 'DELETE'],
}

export interface loggerInterface {
  useLogger: boolean
  console: boolean
  requestMethods: string[]
}
```

```ts
// middleware/logger.ts

import { Request, Response, NextFunction } from 'express'
import { join } from 'path'
import { existsSync } from 'fs'
import fsPromises from 'fs/promises'
import { v4 as uuid } from 'uuid'
import { loggerInterface } from '../config/loggerOptions'

const dateTime = () =>
  new Intl.DateTimeFormat('en-US', {
    year: 'numeric',
    month: 'numeric',
    day: 'numeric',
    hour: 'numeric',
    minute: 'numeric',
    second: 'numeric',
  }).format(new Date())

export const logEvents = async (message: string, logFileName: string) => {
  const logItem = `${dateTime()}\t${uuid()}\t${message}\n`
  try {
    if (!existsSync(join(__dirname, '..', 'logs'))) {
      await fsPromises.mkdir(join(__dirname, '..', 'logs'))
    }
    await fsPromises.appendFile(join(__dirname, '..', 'logs', logFileName), logItem)
  } catch (err) {
    console.log(err)
  }
}

export const logger = (options: loggerInterface) => (req: Request, res: Response, next: NextFunction) => {
  if (options.useLogger) {
    if (options.logMethods.includes(req.method))
      logEvents(`${req.method}\t${req.url}\t${req.headers.origin}`, 'reqLog.log')
  }
  if (options.console) console.log(`${req.method}\t${req.url}\t${req.headers.origin}`)
  next()
}
```
