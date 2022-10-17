```yml
// docker-compose.yml

version: '3.8'
services:
  dev-db:
    image: postgres
    ports:
      - 5432:5432
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: 123
      POSTGRES_DB: node
  redis:
    image: redis
    ports:
      - 6379:6379
```
