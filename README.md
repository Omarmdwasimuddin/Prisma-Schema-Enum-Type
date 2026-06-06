# 📘 Prisma Schema — `enum` Type Complete Guide

> **লক্ষ্য:** এই ডকুমেন্টটি পড়ার পর তুমি বুঝতে পারবে — Prisma-তে `enum` কী, কেন ব্যবহার করা হয়, এবং কীভাবে ব্যবহার করতে হয়।

---

## 🔷 ১. `enum` কী?

`enum` মানে **Enumeration** — অর্থাৎ **নির্দিষ্ট কিছু মানের একটি তালিকা** যেখান থেকে শুধুমাত্র একটিই বেছে নেওয়া যাবে।

### বাস্তব জীবনের উদাহরণ:

| পরিস্থিতি | সম্ভাব্য মানগুলো |
|-----------|-----------------|
| ট্র্যাফিক সিগনাল | 🔴 Red, 🟡 Yellow, 🟢 Green |
| অর্ডার স্ট্যাটাস | Pending, Processing, Shipped, Delivered |
| ইউজার টাইপ | New, Existing, VIP, Regular, Bed |

> ✅ **মূল কথা:** enum ব্যবহার করলে ডেটাবেজে শুধু **নির্ধারিত মানগুলোই** save হতে পারবে — অন্য কিছু নয়।

---

## 🔷 ২. আমাদের Schema বিশ্লেষণ

```prisma
generator client {
  provider = "prisma-client-js"
  output   = "../app/generated/prisma"
}

datasource db {
  provider = "postgresql"
}

model User {
  id      Int      @id @default(autoincrement())
  remarks UserType
}

enum UserType {
  New
  Existing
  VIP
  Regular
  Bad
}
```

---

## 🔷 ৩. Schema-এর  ব্যাখ্যা

### 🟦 `model User` — টেবিল সংজ্ঞা

```prisma
model User {
  id      Int      @id @default(autoincrement())
  remarks UserType
}
```

| Field | Type | Attribute | অর্থ |
|-------|------|-----------|------|
| `id` | `Int` | `@id @default(autoincrement())` | Primary key, auto-increment |
| `remarks` | `UserType` | — | ইউজার কোন ক্যাটাগরির তা নির্দেশ করে |

> ✅ `remarks` field-এর type হলো `UserType` — যেটা নিচে `enum` হিসেবে define করা হয়েছে।

---

### 🟦 `enum UserType` — মূল enum সংজ্ঞা

```prisma
enum UserType {
  New
  Existing
  VIP
  Regular
  Bad
}
```

এই enum-এর ৫টি সম্ভাব্য মান:

| মান | অর্থ (অনুমান) |
|-----|--------------|
| `New` | নতুন ইউজার |
| `Existing` | পুরনো ইউজার |
| `VIP` | বিশেষ সুবিধাপ্রাপ্ত ইউজার |
| `Regular` | সাধারণ ইউজার |
| `Bad` | ⚠️ সম্ভবত টাইপো — `Bad` হওয়া উচিত ছিল? |

---

## 🔷 ৪. ডেটাবেজে কী হয়?

`npx prisma migrate dev` রান করার পর PostgreSQL-এ নিচের মতো SQL তৈরি হয়:

```sql
-- enum type তৈরি হয়
CREATE TYPE "UserType" AS ENUM (
    'New',
    'Existing',
    'VIP',
    'Regular',
    'Bad'
);

-- User টেবিল তৈরি হয়
CREATE TABLE "User" (
    "id"      SERIAL       NOT NULL,
    "remarks" "UserType"   NOT NULL,

    CONSTRAINT "User_pkey" PRIMARY KEY ("id")
);
```

---

## 🔷 ৫. Prisma Client দিয়ে ব্যবহার

### ✅ নতুন ইউজার তৈরি করা

```typescript
import { PrismaClient, UserType } from '../app/generated/prisma';

const prisma = new PrismaClient();

// নতুন ইউজার তৈরি
const newUser = await prisma.user.create({
  data: {
    remarks: UserType.VIP,   // ✅ শুধু enum মানই দেওয়া যাবে
  },
});

console.log(newUser);
// Output: { id: 1, remarks: 'VIP' }
```

### ✅ ইউজার খোঁজা (filter by enum)

```typescript
// সব VIP ইউজার খোঁজো
const vipUsers = await prisma.user.findMany({
  where: {
    remarks: UserType.VIP,
  },
});
```

### ❌ ভুল মান দিলে কী হয়?

```typescript
// ❌ এটা TypeScript error দেবে — compile হবে না
const badUser = await prisma.user.create({
  data: {
    remarks: "Premium",   // ❌ 'Premium' enum-এ নেই!
  },
});
```

> ✅ **সুবিধা:** TypeScript + Prisma মিলে compile time-এই ভুল ধরে ফেলে।

---

## 🔷 ৬. enum-এ নতুন মান যোগ করা

ধরো, `Premium` নামের নতুন একটি type যোগ করতে চাও:

### ধাপ ১ — Schema আপডেট করো

```prisma
enum UserType {
  New
  Existing
  VIP
  Regular
  Bed
  Premium    // ← নতুন মান যোগ করা হলো
}
```

## ✅ সারসংক্ষেপ

| বিষয় | মনে রাখো |
|------|----------|
| `enum` কী | নির্দিষ্ট মানের তালিকা |
| কোথায় define করে | `schema.prisma`-এ, `model`-এর বাইরে |
| কোথায় ব্যবহার করে | `model`-এর field type হিসেবে |
| সুবিধা | Type safety + DB validation |
| PostgreSQL-এ | `CREATE TYPE ... AS ENUM (...)` |
| TypeScript-এ | `import { MyEnum }` করে ব্যবহার |

---
