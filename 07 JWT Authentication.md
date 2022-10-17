```
npm i cookie-parser argon2 jsonwebtoken
npm i -D @types/jsonwebtoken @types/cookie-parser
```

```ts
node
require('crypto').randomBytes(64).toString('hex')
```

```dockerfile
# .env
ACCESS_TOKEN_SECRET=paste
REFRESH_TOKEN_SECRET=paste
```

```ts
// server.ts

import cookieParser from 'cookie-parser'
app.use(cookieParser())
```

```ts
// auth/auth.controller.ts

import { Request, Response } from 'express'
import { PrismaClient } from '@prisma/client'
import { PrismaClientKnownRequestError } from '@prisma/client/runtime'
import * as argon from 'argon2'
import jwt from 'jsonwebtoken'

const prisma = new PrismaClient()

export const signup = async (req: Request, res: Response) => {
  const { email, password } = req.body
  const hashedPassword = await argon.hash(password)
  const refreshToken = getRefreshToken(email)
  try {
    await prisma.user.create({
      data: {
        email,
        password: hashedPassword,
        refreshToken,
      },
    })
    res.cookie('jwt', refreshToken, {
      httpOnly: true,
      sameSite: 'none',
      secure: true,
      maxAge: 24 * 60 * 60 * 1000,
    })
    return res.status(201).json({ accessToken: getAccessToken(email) })
  } catch (err) {
    if (err instanceof PrismaClientKnownRequestError)
      if (err.code === 'P2002') {
        return res.status(409).json({ message: 'Email already in use' })
      }
  }
}

export const login = async (req: Request, res: Response) => {
  const { email, password } = req.body
  const user = await prisma.user.findUnique({
    where: {
      email,
    },
  })
  if (user && (await argon.verify(user.password, password))) {
    const refreshToken = getRefreshToken(email)
    await prisma.user.update({
      where: {
        email,
      },
      data: {
        refreshToken,
      },
    })
    res.cookie('jwt', refreshToken, {
      httpOnly: true,
      sameSite: 'none',
      secure: true,
      maxAge: 24 * 60 * 60 * 1000,
    })
    return res.status(200).json({ accessToken: getAccessToken(email) })
  } else return res.status(400).json({ message: 'Invalid crendentials' })
}

export const sendRefreshToken = async (req: Request, res: Response) => {
  const cookies = req.cookies
  if (!cookies?.jwt) return res.sendStatus(401)
  const refreshToken = cookies.jwt
  const user = await prisma.user.findFirst({
    where: {
      refreshToken,
    },
  })
  if (!user) return res.sendStatus(403)
  return res.status(200).json({ accessToken: getAccessToken(user.email) })
}

export const logout = async (req: Request, res: Response) => {
  const cookies = req.cookies
  if (!cookies?.jwt) return res.sendStatus(204)
  const refreshToken = cookies.jwt
  const user = await prisma.user.findFirst({
    where: {
      refreshToken,
    },
  })
  if (user) {
    await prisma.user.update({
      where: {
        email: user.email,
      },
      data: {
        refreshToken: '',
      },
    })
  }
  res.clearCookie('jwt', { httpOnly: true, sameSite: 'none', secure: true })
  return res.sendStatus(204)
}

const getRefreshToken = (email: string) =>
  jwt.sign({ email }, process.env.REFRESH_TOKEN_SECRET as string, {
    expiresIn: '1d',
  })

const getAccessToken = (email: string) =>
  jwt.sign({ email }, process.env.ACCESS_TOKEN_SECRET as string, {
    expiresIn: '15min',
  })
```

```ts
// auth/auth.schema.ts

import * as yup from 'yup'

export const singupSchema = yup.object({
  body: yup.object({
    email: yup.string().email().required(),
    password: yup.string().min(5).max(30).required(),
  }),
})

export const loginSchema = yup.object({
  body: yup.object({
    email: yup.string().email().required(),
    password: yup.string().min(5).max(30).required(),
  }),
})
```

```ts
// auth/auth.routes.ts

import { Router } from 'express'
import validate from '../middleware/validate'
import { login, logout, sendRefreshToken, signup } from './auth.controller'
import { loginSchema, singupSchema } from './auth.schema'

const router = Router()
export default router

router.post('/signup', validate(loginSchema), login)
router.post('/login', validate(singupSchema), signup)
router.post('/refresh', sendRefreshToken)
router.post('/logout', logout)
```

```ts
// types/express/index.d.ts

import express from 'express'
import { User } from '@prisma/client'

declare global {
  namespace Express {
    interface Request {
      user?: User | null
    }
  }
}
```

```json
// tsconfig.json

"typeRoots": ["./src/types"]
```

```ts
// middleware/jwtGuard.ts

import { Request, Response, NextFunction } from 'express'
import { PrismaClient } from '@prisma/client'
import jwt from 'jsonwebtoken'

const prisma = new PrismaClient()

const jwtGuard = async (req: Request, res: Response, next: NextFunction) => {
  const authorization = req.headers.authorization
  if (authorization && authorization.startsWith('Bearer')) {
    try {
      const token = authorization.split(' ')[1]
      const decoded = jwt.verify(token, process.env.ACCESS_TOKEN_SECRET as string) as any
      req.user = await prisma.user.findUnique({
        where: {
          email: decoded.email,
        },
      })
      next()
    } catch (err) {
      return res.status(403).json({ message: 'Not authorized' })
    }
  } else return res.status(401).json({ message: 'Not authorized, no token' })
}

export default jwtGuard
```
