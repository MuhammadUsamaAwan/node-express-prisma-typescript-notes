```ts
// todo/todo.controller.ts

import { Request, Response } from 'express'
import { PrismaClient } from '@prisma/client'

const prisma = new PrismaClient()

export const getTodos = async (req: Request, res: Response) => {
  const todos = await prisma.todo.findMany()
  return res.status(200).json(todos)
}

export const getTodo = async (req: Request, res: Response) => {
  const todos = await prisma.todo.findUnique({
    where: {
      id: req.params.id,
    },
  })
  return res.status(200).json(todos)
}

export const createTodo = async (req: Request, res: Response) => {
  const todo = await prisma.todo.create({
    data: {
      name: req.body.name,
    },
  })
  return res.status(201).json(todo)
}

export const updateTodo = async (req: Request, res: Response) => {
  const todo = await prisma.todo.update({
    where: {
      id: req.params.id,
    },
    data: {
      name: req.body.name,
    },
  })
  return res.status(200).json(todo)
}

export const deleteTodo = async (req: Request, res: Response) => {
  await prisma.todo.delete({
    where: {
      id: req.params.id,
    },
  })
  return res.sendStatus(204)
}
```

```ts
// todo/todo.routes.ts

import { Router } from 'express'
import { createTodo, deleteTodo, getTodo, getTodos, updateTodo } from './todo.controller'

const router = Router()
export default router

router.route('/').get(getTodos).post(createTodo)
router.route('/:id').get(getTodo).patch(updateTodo).delete(deleteTodo)
```

```ts
// router.ts

import { Router } from 'express'
import todo from './todo/todo.routes'

const router = Router()

router.use('/todos', todo)

export default router
```

```ts
// server.ts
import routes from './router'

app.use('/', routes)
```
