```
npm i express-rate-limit
npm i -D @types/express-rate-limit
```

```ts
// middleware/rateLimiter.ts

import rateLimit from 'express-rate-limit'
import { Request, Response } from 'express'

const ratelimiter = (time: number, limit: number) =>
  rateLimit({
    windowMs: time * 1000, // seconds
    max: limit,
    standardHeaders: true,
    legacyHeaders: false,
    message: async (req: Request, res: Response) => {
      return res.status(429).json({ message: 'Too many requests from this IP, please try again after some time' })
    },
  })

export default ratelimiter
```
