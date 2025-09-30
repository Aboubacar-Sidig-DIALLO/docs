# 📌 MongoDB Adapter avec Better Auth

### 🔹 Rôle

* Contrairement à **Postgres/SQLite/Prisma**, MongoDB est **NoSQL** → il n’a pas de schéma strict.
* Better Auth gère automatiquement les **collections nécessaires** (`users`, `accounts`, `sessions`, etc.).
* **Aucune migration** à gérer (donc pas de `npx @better-auth/cli migrate` ici).

***

### 🔹 Exemple de configuration

```ts
// auth.ts
import { betterAuth } from "better-auth";
import { MongoClient } from "mongodb";
import { mongodbAdapter } from "better-auth/adapters/mongodb";

const client = new MongoClient("mongodb://localhost:27017/mydb"); // connexion
const db = client.db("mydb"); // sélection de la base

export const auth = betterAuth({
  database: mongodbAdapter(db, {
    // ⚡ Optionnel : si tu fournis le client, BetterAuth active les transactions MongoDB
    client,
  }),
});
```

***

### 🔹 Avantages MongoDB dans Better Auth

1. **Flexibilité** : pas besoin de schéma strict → utile si tu veux rajouter facilement des champs custom.
2. **Simplicité** : tu n’as pas besoin de migrations (`generate` ou `migrate`).
3. **Transactions supportées** si tu passes `client`. → pratique pour éviter les incohérences entre `users` et `accounts`.
4. **Performances** : adapté aux applis qui veulent du login **rapide et scalable**.

***

### 🔹 Collections créées automatiquement

Better Auth crée et gère en interne ces collections :

* `users` → infos de base (id, email, name, image, champs supplémentaires éventuels).
* `accounts` → comptes liés (Google, GitHub, credentials, etc.).
* `sessions` → sessions actives de l’utilisateur.
* `verificationTokens` → jetons de vérification d’email.
* `passwordResets` → jetons de réinitialisation de mot de passe.

> ⚠️ Tu n’as pas besoin de les créer manuellement, Better Auth s’occupe de ça.

***

### 🔹 Points importants à retenir

* Si tu viens d’un SQL (Postgres, SQLite), **MongoDB ne nécessite pas Prisma/Kysely** → tu passes directement un `db`.
* Pas de **schema generation** ni **migration** (juste les collections gérées auto).
* ⚡ **Astuce perf** : toujours utiliser un `client` partagé (`MongoClient`) au lieu de reconnecter la DB à chaque requête.
* Tu peux toujours combiner avec les **plugins Better Auth** (email/password, OAuth, 2FA, etc.) de la même manière que Postgres/SQLite.

## 📌 Exemple Complet : Better Auth + MongoDB (Next.js)

### 1. Installer les dépendances

```bash
npm install better-auth mongodb
```

***

### 2. Configurer Better Auth avec MongoDB

```ts
// lib/auth.ts
import { betterAuth } from "better-auth";
import { MongoClient } from "mongodb";
import { mongodbAdapter } from "better-auth/adapters/mongodb";

const client = new MongoClient(process.env.MONGO_URI || "mongodb://localhost:27017/mydb");
const db = client.db("mydb");

export const auth = betterAuth({
  database: mongodbAdapter(db, { client }),

  // ⚡ Exemple : activer email/password
  emailAndPassword: {
    enabled: true,
    requireEmailVerification: false,
  },
});
```

***

### 3. Auth Client (Frontend)

```ts
// lib/auth-client.ts
import { createAuthClient } from "better-auth/client";

export const authClient = createAuthClient({
  baseURL: "/api/auth", // où tes routes auth Next.js sont exposées
});
```

***

### 4. API Routes Next.js

Better Auth génère des endpoints API automatiquement.\
Tu dois juste les exposer dans Next.js :

```ts
// app/api/auth/[...all]/route.ts
import { auth } from "@/lib/auth";

export const { GET, POST } = auth.api;
```

👉 Ça expose automatiquement :

* `/api/auth/sign-up/email`
* `/api/auth/sign-in/email`
* `/api/auth/sign-out`
* `/api/auth/session`\
  etc.

***

### 5. Signup (Inscription)

```tsx
// app/signup/page.tsx
"use client";
import { authClient } from "@/lib/auth-client";
import { useState } from "react";

export default function SignupPage() {
  const [email, setEmail] = useState("");
  const [password, setPassword] = useState("");

  const handleSignup = async () => {
    const { data, error } = await authClient.signUp.email({
      name: "John Doe",
      email,
      password,
    });
    if (error) alert(error.message);
    else alert("Inscription réussie 🎉");
  };

  return (
    <div>
      <h1>Sign Up</h1>
      <input value={email} onChange={(e) => setEmail(e.target.value)} placeholder="email" />
      <input type="password" value={password} onChange={(e) => setPassword(e.target.value)} placeholder="password" />
      <button onClick={handleSignup}>Sign Up</button>
    </div>
  );
}
```

***

### 6. Signin (Connexion)

```tsx
// app/signin/page.tsx
"use client";
import { authClient } from "@/lib/auth-client";
import { useState } from "react";
import { useRouter } from "next/navigation";

export default function SigninPage() {
  const [email, setEmail] = useState("");
  const [password, setPassword] = useState("");
  const router = useRouter();

  const handleSignin = async () => {
    const { data, error } = await authClient.signIn.email({
      email,
      password,
    });
    if (error) alert(error.message);
    else router.push("/dashboard");
  };

  return (
    <div>
      <h1>Sign In</h1>
      <input value={email} onChange={(e) => setEmail(e.target.value)} placeholder="email" />
      <input type="password" value={password} onChange={(e) => setPassword(e.target.value)} placeholder="password" />
      <button onClick={handleSignin}>Sign In</button>
    </div>
  );
}
```

***

### 7. Signout (Déconnexion)

```tsx
// app/dashboard/page.tsx
"use client";
import { authClient } from "@/lib/auth-client";
import { useRouter } from "next/navigation";

export default function DashboardPage() {
  const router = useRouter();

  const handleSignout = async () => {
    await authClient.signOut({
      fetchOptions: {
        onSuccess: () => router.push("/signin"),
      },
    });
  };

  return (
    <div>
      <h1>Dashboard</h1>
      <button onClick={handleSignout}>Sign Out</button>
    </div>
  );
}
```

***

### 8. Gestion de Session (Exemple)

```tsx
// app/dashboard/page.tsx
"use client";
import { authClient } from "@/lib/auth-client";

export default function DashboardPage() {
  const { data: session } = authClient.useSession();

  if (!session) return <p>Chargement...</p>;

  return (
    <div>
      <h1>Bienvenue {session.user.name}</h1>
      <p>Email : {session.user.email}</p>
    </div>
  );
}
```

***

✅ Avec ça, tu as :

* **MongoDB adapter** branché.
* **Signup** (inscription).
* **Signin** (connexion).
* **Signout** (déconnexion).
* **Gestion de session** (avec `useSession`).
