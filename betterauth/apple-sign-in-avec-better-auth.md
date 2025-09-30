# 🍏 Apple Sign-In avec Better Auth

L’intégration de **Sign in with Apple** demande un peu plus d’étapes que les autres providers, car Apple impose une configuration stricte dans son **Developer Portal** et l’utilisation d’un **clientSecret JWT** généré dynamiquement.

***

### 🛠️ Étapes de configuration côté Apple Developer Portal

1. **Créer un App ID**
   * Accéder à _Certificates, Identifiers & Profiles → Identifiers_
   * Cliquer sur **+ → App IDs**
   * Type : **App** → Continue
   * Renseigner :
     * **Description** : nom de ton app (`My Awesome App`)
     * **Bundle ID** : format `com.entreprise.application` (par ex. `com.mycompany.myapp.ai`)
   * Cocher **Sign In with Apple** dans _Capabilities_
   * **Register**

***

2. **Créer un Service ID**
   * Toujours dans _Identifiers_ → **+ → Service IDs**
   * **Description** : ton app (ex. `My Awesome App Service`)
   * **Identifier** : un ID unique (`com.mycompany.myapp.si`) → c’est ton **clientId**
   * **Register**

***

3. **Configurer le Service ID**
   * Cliquer sur ton Service ID nouvellement créé
   * Activer **Sign In with Apple** → **Configure**
   * Associer ton **Primary App ID** (celui créé à l’étape 1)
   * Ajouter :
     * **Domains** : ex. `example.com`
     * **Return URLs** : ex. `https://example.com/api/auth/callback/apple`
   * **Save**

***

4. **Créer une clé privée (.p8)**
   * Aller dans **Keys → +**
   * Nommer la clé (ex. `Apple Sign In Key`)
   * Activer **Sign In with Apple** et sélectionner ton **App ID**
   * **Register** et télécharger le fichier `.p8` (⚠️ téléchargeable une seule fois)
   * Note ton **Key ID** et ton **Team ID** (dans ton compte Apple Developer)

***

5.  **Générer le Client Secret (JWT)**

    * Apple exige que ton `clientSecret` soit un **JWT signé** avec :
      * la clé privée `.p8`
      * ton **Team ID**
      * ton **Key ID**
      * ton **clientId (Service ID)**

    Exemple de payload JWT attendu par Apple :

    ```json
    {
      "iss": "TEAM_ID",
      "iat": 1700000000,
      "exp": 1700003600,
      "aud": "https://appleid.apple.com",
      "sub": "com.mycompany.myapp.si"
    }
    ```

    Ce JWT est signé avec l’algorithme `ES256` et sert de `clientSecret`.

***

### ⚡ Intégration avec Better Auth

Une fois tes credentials obtenus :

```ts
import { betterAuth } from "better-auth";

export const auth = betterAuth({
  socialProviders: {
    apple: {
      clientId: "com.mycompany.myapp.si", // Service ID
      clientSecret: async () => {
        // Génère dynamiquement le JWT ici avec ton .p8
        const token = generateAppleClientSecret({
          teamId: "TEAM_ID",
          clientId: "com.mycompany.myapp.si",
          keyId: "KEY_ID",
          privateKey: process.env.APPLE_PRIVATE_KEY!, // contenu du .p8
        });
        return token;
      },
    },
  },
});
```

#### 🍏 **Configurer le provider Apple dans Better Auth**

Une fois que tu as généré ton **Service ID (clientId)** et ton **clientSecret JWT**, tu dois déclarer le provider **Apple** dans la configuration de ton instance Better Auth.

***

### ✅ Exemple de configuration

```ts
import { betterAuth } from "better-auth";

export const auth = betterAuth({
  socialProviders: {
    apple: { 
      clientId: process.env.APPLE_CLIENT_ID as string, // Service ID (ex: com.mycompany.myapp.si)
      clientSecret: process.env.APPLE_CLIENT_SECRET as string, // JWT signé
      // ⚠️ Obligatoire si tu veux utiliser Apple Sign In côté iOS natif
      appBundleIdentifier: process.env.APPLE_APP_BUNDLE_IDENTIFIER as string, 
    }, 
  },

  // Autoriser Apple comme domaine de confiance
  trustedOrigins: ["https://appleid.apple.com"], 
});
```

***

### ⚠️ Points importants

1.  **`clientId`**

    * Côté **Web**, c’est ton **Service ID** (`com.mycompany.myapp.si`).
    * Côté **iOS natif**, Apple attend **le Bundle ID de ton app** (`com.mycompany.myapp`).

    👉 D’où l’option `appBundleIdentifier`.
2. **`clientSecret`**
   * Doit être un **JWT signé dynamiquement** avec ta clé `.p8`.
   * Peut être généré avec `jsonwebtoken` ou `jose`.
3. **`trustedOrigins`**
   * Tu dois inclure `https://appleid.apple.com` pour autoriser la communication OAuth.
4. **Erreur fréquente**
   *   Si tu utilises le **Service ID** comme `clientId` en natif iOS, tu verras :

       ```
       JWTClaimValidationFailed: unexpected "aud" claim value
       ```
   * Solution 👉 Fournir `appBundleIdentifier` pour le cas natif.

#### 🍏 **Sign In with Apple – Usage dans Better Auth**

Une fois ton provider **Apple** configuré côté serveur (`auth.ts`), tu peux l’utiliser côté **client** avec la méthode `signIn.social`.

***

### ✅ Exemple côté **client**

```ts
import { createAuthClient } from "better-auth/client"

const authClient = createAuthClient()

// 🔑 Sign In avec Apple
const signInWithApple = async () => {
  try {
    const { data, error } = await authClient.signIn.social({
      provider: "apple", // ⚡ Indique qu'on utilise Apple
    })

    if (error) {
      console.error("Erreur de connexion Apple:", error.message)
      return
    }

    console.log("Connexion réussie via Apple ✅", data)
  } catch (err) {
    console.error("Erreur inattendue:", err)
  }
}
```

***

### ⚙️ Notes importantes

* `provider: "apple"` 👉 doit correspondre à la clé déclarée dans `socialProviders.apple` dans ton `auth.ts`.
* Si tu es sur **iOS natif** avec ID Token (par ex. `signInWithApple` de `expo-apple-authentication`), assure-toi d’avoir fourni `appBundleIdentifier` dans la config côté serveur.
* Tu peux aussi ajouter un `callbackURL` pour rediriger après login si nécessaire.

#### 🍏 **Sign In with Apple avec ID Token (Better Auth)**

Ce mode est utilisé quand tu obtiens déjà un **ID Token Apple côté client** (par ex. via **React Native / Expo** ou un SDK natif), et que tu veux le transmettre directement à ton backend Better Auth **sans redirection**.

***

### ✅ Exemple d’utilisation

```ts
// auth-client.ts
import { authClient } from "@/lib/auth-client"

const signInWithAppleIdToken = async (idToken: string, nonce?: string, accessToken?: string) => {
  try {
    const { data, error } = await authClient.signIn.social({
      provider: "apple",
      idToken: {
        token: idToken, // ⚡ L’ID Token Apple récupéré côté client
        nonce,          // (optionnel) Nonce utilisé pour la requête
        accessToken,    // (optionnel) Access Token Apple
      },
    })

    if (error) {
      console.error("❌ Erreur de connexion Apple:", error.message)
      return
    }

    console.log("✅ Connexion réussie via Apple ID Token:", data)
  } catch (err) {
    console.error("🚨 Erreur inattendue:", err)
  }
}
```

***

### ⚙️ Notes importantes

* **Pas de redirection** : avec `idToken`, l’utilisateur est directement connecté.
* **iOS natif** : sur mobile, Apple fournit souvent un **ID Token + nonce** via `expo-apple-authentication` ou le SDK natif.
* **Server config** :
  * Si tu es sur mobile (iOS natif), utilise bien `appBundleIdentifier` comme `clientId` côté serveur (et non le Service ID).
  * Ajoute `https://appleid.apple.com` dans `trustedOrigins` de ton `auth.ts`.

#### 🍏 Génération du **Apple Client Secret (JWT)** pour Better Auth

Apple exige que le **clientSecret** soit un **JWT signé** (avec ta clé privée `.p8`, ton **Key ID**, ton **Team ID**, et ton **Service ID / App ID**).

Tu dois générer ce JWT dynamiquement côté serveur. Voici un exemple en **Node.js avec `jsonwebtoken`** :

***

### ✅ Exemple en TypeScript / Node.js

```ts
import jwt from "jsonwebtoken";

/**
 * Génère un Apple Client Secret (JWT)
 * @param clientId - Service ID (ex: com.yourdomain.app)
 * @param teamId - Team ID Apple Developer
 * @param keyId - Key ID de ta clé privée (.p8)
 * @param privateKey - Contenu du fichier .p8
 * @returns JWT string
 */
export function generateAppleClientSecret({
  clientId,
  teamId,
  keyId,
  privateKey,
}: {
  clientId: string;
  teamId: string;
  keyId: string;
  privateKey: string;
}): string {
  const now = Math.floor(Date.now() / 1000);

  // Durée de validité : 6 mois max (Apple ne permet pas plus)
  const expiration = now + 60 * 60 * 24 * 180;

  const payload = {
    iss: teamId,      // Team ID
    iat: now,         // Issued at
    exp: expiration,  // Expiration
    aud: "https://appleid.apple.com", // Audience fixée par Apple
    sub: clientId,    // Service ID ou App ID
  };

  return jwt.sign(payload, privateKey, {
    algorithm: "ES256", // Apple utilise ES256
    keyid: keyId,       // ID de ta clé
  });
}
```

***

### ⚙️ Exemple d’utilisation

```ts
import fs from "fs";
import path from "path";

const privateKey = fs.readFileSync(
  path.join(__dirname, "AuthKey_F6G7H8I9J0.p8"), 
  "utf8"
);

const clientSecret = generateAppleClientSecret({
  clientId: "com.yourdomain.app",   // Service ID (ou App ID si iOS natif)
  teamId: "A1B2C3D4E5",             // Team ID Apple
  keyId: "F6G7H8I9J0",              // Key ID de la clé .p8
  privateKey,                       // Contenu du fichier .p8
});

console.log("✅ Apple Client Secret:", clientSecret);
```

***

### 🚨 Points importants

* **clientId** :
  * **Service ID** (`com.yourdomain.app.service`) pour web.
  * **App ID / Bundle ID** (`com.yourdomain.app`) pour iOS natif.
* **Durée max** : le JWT ne peut pas dépasser **6 mois** (≈ 15777000 secondes).
* **Algorithme** : toujours `ES256`.
* **Regénération** : tu devras régénérer ce clientSecret périodiquement.
