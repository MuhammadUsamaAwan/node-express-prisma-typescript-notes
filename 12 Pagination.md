```ts
export const getTodos = async (req: Request, res: Response) => {
  const page = parseInt(req.query.page as string)
  const limit = parseInt(req.query.limit as string)
  let todos
  if (page && limit)
    todos = await prisma.todo.findMany({
      skip: (page - 1) * limit,
      take: limit,
    })
  else todos = await prisma.todo.findMany()
  return res.status(200).json(todos)
}
```
