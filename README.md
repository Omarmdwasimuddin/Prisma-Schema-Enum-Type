## Prisma-Schema-Enum-Type

```schema
generator client {
  provider = "prisma-client-js"
  output   = "../app/generated/prisma"
}

datasource db {
  provider = "postgresql"
}

model User {
  id Int @id @default(autoincrement())
  remarks UserType
}

enum UserType {
  New
  Existing
  VIP
  Regular
  Bed
}
```
---
