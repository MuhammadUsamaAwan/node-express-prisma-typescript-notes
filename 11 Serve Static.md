```ts
app.use(express.static('uploads')) // this is the absoulte path
app.use('/images', jwtGuard, express.static('uploads')) // with authentication and path
```
