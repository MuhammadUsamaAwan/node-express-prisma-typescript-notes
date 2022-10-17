```
npm i yup
```

```ts
// middleware/validate.ts

import { Request, Response, NextFunction } from 'express'

const validate = (schema: any) => async (req: Request, res: Response, next: NextFunction) => {
  try {
    await schema.validate({
      body: req.body,
      query: req.query,
      params: req.params,
    })
    next()
  } catch (err: any) {
    return res.status(400).json({ message: err.errors[0].split('.')[1] })
  }
}

export default validate
```

```ts
// todo/todo.schema.ts

import * as yup from 'yup'

export const createTodoSchema = yup.object({
  body: yup.object({
    name: yup.string().min(4).max(100).required(),
  }),
})

export const updateTodoSchema = yup.object({
  body: yup.object({
    name: yup.string().min(4).max(100).required(),
  }),
  params: yup.object({
    id: yup.string().uuid(),
  }),
})

export const delteTodoSchema = yup.object({
  params: yup.object({
    id: yup.string().uuid(),
  }),
})
```

```ts
// routes/todo/todo.routes.ts

import { Router } from 'express'
import validate from '../middleware/validate'
import { createTodo, deleteTodo, getTodo, getTodos, updateTodo } from './todo.controller'
import { createTodoSchema, delteTodoSchema, updateTodoSchema } from './todo.schema'

const router = Router()
export default router

router.route('/').get(getTodos).post(validate(createTodoSchema), createTodo)
router
  .route('/:id')
  .get(getTodo)
  .patch(validate(updateTodoSchema), updateTodo)
  .delete(validate(delteTodoSchema), deleteTodo)
```
