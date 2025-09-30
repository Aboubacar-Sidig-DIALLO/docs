# 📌 Prisma avec Better Auth

**Prisma ORM** est un ORM moderne, **type-safe** et intuitif, qui facilite la gestion et l’accès aux bases de données.\
Better Auth propose un **adaptateur officiel Prisma** pour s’intégrer directement avec tes modèles Prisma.

***

### ⚡ Exemple d’utilisation

Assure-toi d’avoir **Prisma installé et configuré** (`npx prisma init`), puis connecte-le à Better Auth avec l’adaptateur.

```ts
// auth.ts
import { betterAuth } from "better-auth";
import { prismaAdapter } from "better-auth/adapters/prisma";
import { PrismaClient } from "@prisma/client";

const prisma = new PrismaClient();

export const auth = betterAuth({
  database: prismaAdapter(prisma, {
    provider: "sqlite", // ou "postgresql" | "mysql" | "mongodb"
  }),
});
```

***

### ⚙️ Schéma Prisma avec Better Auth

Tu dois définir un schéma Prisma (`schema.prisma`) compatible. Better Auth peut générer ce schéma automatiquement avec la CLI.

#### Exemple (SQLite) :

```prisma
// schema.prisma
datasource db {
  provider = "sqlite"
  url      = "file:./dev.db"
}

generator client {
  provider = "prisma-client-js"
}

model User {
  id        String   @id @default(cuid())
  email     String   @unique
  name      String?
  image     String?
  accounts  Account[]
  sessions  Session[]
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
}

model Account {
  id             String  @id @default(cuid())
  provider       String
  providerId     String
  userId         String
  accessToken    String?
  refreshToken   String?
  user           User    @relation(fields: [userId], references: [id])
}

model Session {
  id        String   @id @default(cuid())
  userId    String
  user      User     @relation(fields: [userId], references: [id])
  expiresAt DateTime
  createdAt DateTime @default(now())
}
```

***

### 🚀 Génération & Migration avec la CLI

Better Auth fournit une **CLI** pour générer le schéma Prisma.

#### Générer le schéma

```bash
npx @better-auth/cli@latest generate
```

✅ **Schema Generation** : supporté\
❌ **Schema Migration** : non supporté (utilise Prisma directement)

👉 Pour migrer ta base :

```bash
npx prisma migrate dev --name init
```

***

### ℹ️ Informations supplémentaires

* Si tu utilises un **output personnalisé** dans `schema.prisma` :

```prisma
generator client {
  provider = "prisma-client-js"
  output   = "../src/generated/prisma"
}
```

➡️ importe le client depuis `../src/generated/prisma` au lieu de `@prisma/client`.

* Prisma est idéal si tu veux :
  * ✅ TypeScript **type-safe**
  * ✅ Des **requêtes lisibles**
  * ✅ Un **workflow clair** avec migrations
  * ✅ Une intégration facile avec **Next.js / NestJS**

**schéma Prisma complet prêt à l’emploi** pour une intégration robuste avec **Better Auth**.\
Il inclut **User, Account, Session, PasswordReset, VerificationToken** pour couvrir **authentification, sessions, reset password et email verification**.

***

### 📂 `schema.prisma`

```prisma
// schema.prisma
datasource db {
  provider = "postgresql" // ou "sqlite" | "mysql" | "mongodb"
  url      = env("DATABASE_URL")
}

generator client {
  provider = "prisma-client-js"
}

model User {
  id        String    @id @default(cuid())
  email     String    @unique
  name      String?
  image     String?
  accounts  Account[]
  sessions  Session[]
  createdAt DateTime  @default(now())
  updatedAt DateTime  @updatedAt
}

model Account {
  id           String   @id @default(cuid())
  provider     String
  providerId   String
  userId       String
  accessToken  String?
  refreshToken String?
  user         User     @relation(fields: [userId], references: [id])
}

model Session {
  id        String   @id @default(cuid())
  userId    String
  user      User     @relation(fields: [userId], references: [id])
  expiresAt DateTime
  createdAt DateTime @default(now())
}

model PasswordReset {
  id        String   @id @default(cuid())
  userId    String
  token     String   @unique
  expiresAt DateTime
  user      User     @relation(fields: [userId], references: [id])
}

model VerificationToken {
  id        String   @id @default(cuid())
  identifier String
  token      String   @unique
  expiresAt  DateTime
}
```

***

### ⚡ Utilisation avec Better Auth

```ts
// auth.ts
import { betterAuth } from "better-auth";
import { prismaAdapter } from "better-auth/adapters/prisma";
import { PrismaClient } from "@prisma/client";

const prisma = new PrismaClient();

export const auth = betterAuth({
  database: prismaAdapter(prisma, {
    provider: "postgresql", // adapte selon ta base
  }),
  emailAndPassword: {
    enabled: true,
    requireEmailVerification: true,
  },
  emailVerification: {
    sendVerificationEmail: async ({ user, url }) => {
      console.log(`Lien de vérification envoyé à ${user.email}: ${url}`);
      // 👉 ici tu ajoutes ton service d'email (SendGrid, Postmark, Nodemailer…)
    },
  },
});
```

***

### 🚀 Migration

```bash
npx prisma migrate dev --name init_auth
```

Cela va créer toutes les tables nécessaires.

## 📌 1. Rôle et importance des tables

#### 🔹 `User`

* **Contient les informations principales du compte utilisateur.**
* Champs importants :
  * `id` : identifiant unique.
  * `email`, `name`, `image` : infos du profil.
  * Relations vers `Account` (méthodes d’auth), `Session` (sessions actives).
* **Importance** : c’est la table centrale de ton auth, tout tourne autour d’elle.

***

#### 🔹 `Account`

* **Stocke les méthodes d’authentification associées à un `User`.**
* Exemple :
  * Un utilisateur peut avoir :
    * un compte `email/password`
    * un compte `google`
    * un compte `github`
    * tous liés au **même `User`**.
* Champs importants :
  * `provider` : type (google, github, credentials…)
  * `providerId` : identifiant unique côté provider
  * `accessToken` / `refreshToken` : pour recontacter l’API du provider.
* **Importance** : permet l’account linking (= un seul user avec plusieurs façons de se connecter).

***

#### 🔹 `Session`

* **Représente une connexion active (session utilisateur).**
* Exemple :
  * Quand un utilisateur se connecte, on crée une `Session` avec une date d’expiration (`expiresAt`).
* Champs importants :
  * `id` : identifiant de session (souvent stocké dans un cookie/token).
  * `userId` : lien vers l’utilisateur.
  * `expiresAt` : fin de validité de la session.
* **Importance** : gère les connexions actives, permet le sign-out (invalidation de session), multi-device login, etc.

***

#### 🔹 `PasswordReset`

* **Stocke les jetons temporaires de réinitialisation de mot de passe.**
* Exemple :
  * Tu envoies un mail avec `https://app.com/reset?token=xyz`
  * Ce `token` est stocké ici avec une `expiration`.
* **Importance** : sécurise la procédure de réinitialisation (token unique et limité dans le temps).

***

#### 🔹 `VerificationToken`

* **Stocke les jetons de vérification d’email.**
* Exemple :
  * Lors de l’inscription, tu envoies un mail : `https://app.com/verify?token=xyz`
  * Ce `token` est stocké ici, avec une `expiration`.
* **Importance** : assure que l’utilisateur possède bien l’adresse email (lutte contre spam, faux comptes, sécurité).

***

## 📌 2. Endpoints avec Next.js API Routes

👉 Exemple dans un projet **Next.js 14 App Router** (`/app/api/.../route.ts`).

***

#### 🔹 Signup

`/app/api/auth/signup/route.ts`

```ts
import { auth } from "@/lib/auth";
import { NextResponse } from "next/server";

export async function POST(req: Request) {
  const body = await req.json();
  try {
    const { data, error } = await auth.signUp.email({
      name: body.name,
      email: body.email,
      password: body.password,
    });

    if (error) return NextResponse.json({ error: error.message }, { status: 400 });

    return NextResponse.json({ user: data.user });
  } catch (err: any) {
    return NextResponse.json({ error: err.message }, { status: 500 });
  }
}
```

***

#### 🔹 Signin

`/app/api/auth/signin/route.ts`

```ts
import { auth } from "@/lib/auth";
import { NextResponse } from "next/server";

export async function POST(req: Request) {
  const body = await req.json();
  try {
    const { data, error } = await auth.signIn.email({
      email: body.email,
      password: body.password,
    });

    if (error) return NextResponse.json({ error: error.message }, { status: 400 });

    return NextResponse.json({ session: data.session });
  } catch (err: any) {
    return NextResponse.json({ error: err.message }, { status: 500 });
  }
}
```

***

#### 🔹 Signout

`/app/api/auth/signout/route.ts`

```ts
import { auth } from "@/lib/auth";
import { NextResponse } from "next/server";

export async function POST(req: Request) {
  try {
    await auth.signOut();
    return NextResponse.json({ success: true });
  } catch (err: any) {
    return NextResponse.json({ error: err.message }, { status: 500 });
  }
}
```

***

#### 🔹 Reset Password (request)

`/app/api/auth/request-password-reset/route.ts`

```ts
import { auth } from "@/lib/auth";
import { NextResponse } from "next/server";

export async function POST(req: Request) {
  const body = await req.json();
  try {
    await auth.requestPasswordReset({
      email: body.email,
      redirectTo: "https://your-app.com/reset-password",
    });

    return NextResponse.json({ message: "Reset email sent" });
  } catch (err: any) {
    return NextResponse.json({ error: err.message }, { status: 500 });
  }
}
```

***

#### 🔹 Reset Password (confirm)

`/app/api/auth/reset-password/route.ts`

```ts
import { auth } from "@/lib/auth";
import { NextResponse } from "next/server";

export async function POST(req: Request) {
  const body = await req.json();
  try {
    const { data, error } = await auth.resetPassword({
      token: body.token,
      newPassword: body.newPassword,
    });

    if (error) return NextResponse.json({ error: error.message }, { status: 400 });

    return NextResponse.json({ success: true });
  } catch (err: any) {
    return NextResponse.json({ error: err.message }, { status: 500 });
  }
}
```

***

#### 🔹 Email Verification

`/app/api/auth/verify-email/route.ts`

```ts
import { auth } from "@/lib/auth";
import { NextResponse } from "next/server";

export async function POST(req: Request) {
  const body = await req.json();
  try {
    const { data, error } = await auth.verifyEmail({
      token: body.token,
    });

    if (error) return NextResponse.json({ error: error.message }, { status: 400 });

    return NextResponse.json({ verified: true });
  } catch (err: any) {
    return NextResponse.json({ error: err.message }, { status: 500 });
  }
}
```

***

✅ Avec ça tu as :

* **un schéma Prisma robuste**
* **une explication claire des tables et leur rôle**
* **des endpoints REST prêts à l’emploi avec Next.js API Routes**
