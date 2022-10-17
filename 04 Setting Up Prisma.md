```
npm i -D prisma
npm i @prisma/client
npx prisma init
```

After that change the env & then create a prisma model

```prisma
// example model

model User {
  id        Int      @id @default(autoincrement())
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
  email String @unique
  password  String
  firstName String?
  lastName  String?
}
```

```json
// package.json

"prisma:migrate": "npx prisma migrate dev",
"prisma:dev:deploy": "prisma migrate deploy",
"dev:db:up": "docker compose up dev-db -d",
"dev:db:rm": "docker compose rm dev-db -s -f -v",
"dev:db:restart": "npm run dev:db:rm && npm run dev:db:up && sleep 1 && npm run prisma:dev:deploy",
"redis:up": "docker compose up redis -d",
"redis:rm": "docker compose rm redis -s -f -v"
```

```
npx prisma migrate dev --name dev
```

```ts
// use prisma

import { PrismaClient } from '@prisma/client'

const prisma = new PrismaClient()

// enjoy
```
