# 🔐 Two-Factor Authentication (2FA) dans Better Auth

### 1. Installation du plugin

```ts
// auth.ts
import { betterAuth } from "better-auth";
import { twoFactor } from "better-auth/plugins";

export const auth = betterAuth({
  // ... ton adapter DB et autres options
  plugins: [
    twoFactor({
      otp: {
        enabled: true, // Active OTP (par email ou SMS)
        sendOTP: async ({ user, otp }) => {
          // Implémente ton envoi (email / SMS)
          console.log(`Envoyer OTP ${otp} à ${user.email}`);
        },
      },
      totp: {
        enabled: true, // Active TOTP
      },
      backupCodes: {
        enabled: true, // Active backup codes
      },
      trustedDevices: {
        enabled: true, // Active trusted devices
      },
    }),
  ],
});
```

***

### 2. OTP (One-Time Password)

Un **code temporaire** (par défaut 6 chiffres) envoyé par email ou SMS.

#### Activer OTP pour un utilisateur

```ts
await authClient.twoFactor.enableOTP();
```

#### Vérifier OTP lors du login

```ts
const { data, error } = await authClient.twoFactor.verifyOTP({
  code: "123456",
});
```

***

### 3. TOTP (Google Authenticator / Authy)

Un **code basé sur le temps** généré par une app mobile.

#### Générer un QR Code pour activer TOTP

```ts
const { data } = await authClient.twoFactor.enableTOTP();
console.log(data.qrCode); // → afficher QR code dans ton UI
```

👉 L’utilisateur scanne ce QR Code avec son app d’authentification.

#### Vérifier le code

```ts
await authClient.twoFactor.verifyTOTP({
  code: "123456",
});
```

***

### 4. Backup Codes (codes de secours)

Permettre à l’utilisateur de se connecter même sans accès à OTP/TOTP.

#### Générer

```ts
const { data } = await authClient.twoFactor.generateBackupCodes();
console.log(data.codes); // affiche la liste des codes
```

#### Vérifier

```ts
await authClient.twoFactor.verifyBackupCode({
  code: "XXXX-XXXX",
});
```

***

### 5. Trusted Devices

Un **device de confiance** évite de redemander le 2FA sur ce navigateur.

#### Activer

```ts
await authClient.twoFactor.trustDevice();
```

👉 En général, tu ajoutes un bouton **"Ne plus demander sur cet appareil"**.

***

### 6. Login Flow avec 2FA

👉 Exemple typique avec **email + password + OTP** :

```ts
// Étape 1 : login classique
const { data, error } = await authClient.signIn.email({
  email: "user@email.com",
  password: "password123",
});

if (error?.code === "TWO_FACTOR_REQUIRED") {
  // Étape 2 : demande OTP ou TOTP
  const otp = prompt("Entrez votre code 2FA");
  const verify = await authClient.twoFactor.verifyOTP({ code: otp });
  if (verify.error) {
    alert("Code invalide !");
  } else {
    alert("Connexion réussie 🎉");
  }
}
```

***

### 7. Cas d’usage avancés

* 🔁 **Réinitialiser 2FA** si l’utilisateur perd ses accès.
* 🔑 **Forcer 2FA** pour certains rôles (`admin`).
* 📱 **Mix OTP + TOTP** → OTP par défaut, TOTP recommandé.
* 🔒 **Trusted Devices + Backup Codes** = meilleure UX + sécurité.

## 🔐 Installation du plugin Two-Factor (2FA)

### 1. Ajouter le plugin côté serveur

Dans ton fichier `auth.ts`, tu déclares le plugin **twoFactor** et précises ton `appName` (nom qui apparaîtra comme _issuer_ dans les apps type Google Authenticator).

```ts
// auth.ts
import { betterAuth } from "better-auth";
import { twoFactor } from "better-auth/plugins";

export const auth = betterAuth({
  // ... autres options
  appName: "My App", // utilisé comme "issuer" pour TOTP
  plugins: [
    twoFactor(), // active le support 2FA (OTP, TOTP, backup codes, trusted devices)
  ],
});
```

***

### 2. Migrer la base de données

Le plugin ajoute de nouvelles colonnes/tables (par ex. stockage des secrets TOTP, OTP, trusted devices).\
Il faut donc lancer une migration ou générer le schéma.

```bash
# Migration directe
npx @better-auth/cli migrate

# ou génération du schéma
npx @better-auth/cli generate
```

👉 Sans cette migration, tu auras des erreurs au moment d’activer OTP/TOTP.

***

### 3. Ajouter le plugin côté client

Côté frontend, tu dois inclure le **twoFactorClient** pour que le client sache quoi faire lorsqu’un utilisateur doit entrer un second facteur.

```ts
// auth-client.ts
import { createAuthClient } from "better-auth/client";
import { twoFactorClient } from "better-auth/client/plugins";

export const authClient = createAuthClient({
  plugins: [
    twoFactorClient(), // gère redirections + appels OTP/TOTP
  ],
});
```

📌 Ici, tu peux aussi configurer **où rediriger l’utilisateur** quand une vérification 2FA est requise (par ex. `/verify-2fa`).

***

✅ Résumé :

* **Serveur** → active le plugin 2FA (`twoFactor()`) + ajoute `appName`.
* **DB** → migration pour stocker les infos 2FA.
* **Client** → ajoute `twoFactorClient()` pour gérer OTP/TOTP/backup.

## 🔐 Usage du Two-Factor Authentication (2FA) avec BetterAuth

***

### 1. **Activer la 2FA (Enabling 2FA)**

👉 Lorsqu’un utilisateur active la 2FA, un **secret TOTP** et des **codes de secours (backupCodes)** sont générés.

```ts
// Client-side
const { data, error } = await authClient.twoFactor.enable({
  password: "secure-password", // 🔑 le mot de passe de l'utilisateur (obligatoire)
  issuer: "my-app-name",       // (optionnel) surcharge l'appName de ta config
});
```

#### Retour de `enable` :

* `totpURI` → URI que tu peux transformer en **QR Code** (pour Google Authenticator, Authy, etc.)
* `backupCodes` → codes uniques à usage unique si l’utilisateur perd l’accès à son app TOTP

⚠️ Important :

* `twoFactorEnabled` ne sera pas encore `true` tant que l’utilisateur n’a pas **validé son premier TOTP**.
* Si tu veux **skipper cette étape** (risqué), tu peux mettre `skipVerificationOnEnable: true` dans ton `plugin config`.

📌 À ce jour, la 2FA ne fonctionne **que pour les comptes avec mot de passe** (`credential accounts`), pas pour les comptes sociaux (Google, GitHub, etc.), car ces providers gèrent déjà leur propre 2FA.

***

### 2. **Connexion avec 2FA activée (Sign In with 2FA)**

Lorsqu’un utilisateur avec 2FA activée se connecte avec email/password :\
👉 La réponse de `signIn` contiendra `twoFactorRedirect: true`.

Cela veut dire que tu dois **demander le code OTP/TOTP** avant de valider la connexion.

#### Exemple avec callback inline

```ts
await authClient.signIn.email(
  {
    email: "user@example.com",
    password: "password123",
  },
  {
    async onSuccess(context) {
      if (context.data.twoFactorRedirect) {
        // 👉 Ici, tu affiches ton écran de saisie du code 2FA
      }
    },
  }
);
```

***

#### Exemple avec un handler global (via le plugin client)

```ts
import { createAuthClient } from "better-auth/client";
import { twoFactorClient } from "better-auth/client/plugins";

const authClient = createAuthClient({
  plugins: [
    twoFactorClient({
      onTwoFactorRedirect() {
        // 👉 Redirection globale vers /verify-2fa
      },
    }),
  ],
});
```

***

#### Exemple côté serveur (`auth.api`)

```ts
const response = await auth.api.signInEmail({
  body: {
    email: "test@test.com",
    password: "test",
  },
});

if ("twoFactorRedirect" in response) {
  // 👉 Demande du code OTP/TOTP
}
```

***

✅ Résumé :

1. `enable` → génère `totpURI` (QR Code) + `backupCodes`.
2. `signIn` → peut retourner `twoFactorRedirect: true`, que tu dois gérer.
3. Tu peux choisir de gérer la redirection **au cas par cas** ou **globalement** via le plugin client.

## 🔐 Désactiver la Two-Factor Authentication (2FA)

***

### 1. **Pourquoi désactiver ?**

Un utilisateur peut souhaiter désactiver la 2FA lorsqu’il change de téléphone, qu’il n’a plus accès à son application TOTP (Google Authenticator, Authy, etc.), ou simplement s’il ne veut plus de cette protection supplémentaire.

👉 **Important** : on demande toujours le mot de passe pour confirmer l’identité de l’utilisateur avant de désactiver la 2FA.

***

### 2. **Comment désactiver ?**

#### Côté client

```ts
const { data, error } = await authClient.twoFactor.disable({
  password: "secure-password", // 🔑 obligatoire
});
```

#### Côté serveur (`auth.api`)

```ts
const response = await auth.api.twoFactorDisable({
  body: { password: "secure-password" },
  headers: req.headers, // inclure les headers pour valider la session
});
```

***

### 3. **Propriétés**

| Prop       | Description                   | Type   |
| ---------- | ----------------------------- | ------ |
| `password` | Mot de passe de l’utilisateur | string |

***

### 4. **Ce qu’il se passe après désactivation**

* Le champ `twoFactorEnabled` du compte utilisateur passe à `false`.
* Le secret TOTP et les codes de secours associés sont **supprimés** de la base de données.
* À la prochaine connexion, l’utilisateur **ne sera plus redirigé** vers l’étape de vérification 2FA.

## ⏱️ TOTP (Time-based One-Time Password) avec BetterAuth

***

### 1. **Qu’est-ce que TOTP ?**

* Génère un code **unique toutes les 30 secondes** (par défaut).
* Fonctionne **hors-ligne** → basé uniquement sur le temps et un secret partagé.
* S’utilise via une **application d’authentification** (Google Authenticator, Authy, 1Password, etc.).
* Avantage : évite les failles SMS/email (sim-swap, phishing, etc.).

***

### 2. **Récupérer l’URI TOTP**

Après avoir activé la 2FA (`twoFactor.enable`), tu peux récupérer l’URI TOTP (format standard `otpauth://...`) pour générer un QR Code à scanner.

#### Exemple client

```ts
const { data, error } = await authClient.twoFactor.getTotpUri({
  password: "secure-password", // 🔑 mot de passe requis
});
```

#### Exemple serveur

```ts
const response = await auth.api.twoFactorGetTotpUri({
  body: { password: "secure-password" },
  headers: req.headers,
});
```

***

### 3. **Propriétés**

| Prop       | Description                                    | Type   |
| ---------- | ---------------------------------------------- | ------ |
| `password` | Le mot de passe de l’utilisateur (obligatoire) | string |

***

### 4. **Afficher le QR Code en React**

Une fois que tu as `totpURI`, tu peux générer un QR code avec une lib comme `react-qr-code` :

```tsx
import QRCode from "react-qr-code";
import { useQuery } from "@tanstack/react-query";
import { authClient } from "@/lib/auth-client"; // ton instance BetterAuth client

export default function UserCard({ password }: { password: string }) {
  const { data: session } = authClient.useSession();

  const { data: qr } = useQuery({
    queryKey: ["two-factor-qr"],
    queryFn: async () => {
      const res = await authClient.twoFactor.getTotpUri({ password });
      return res.data;
    },
    enabled: !!session?.user.twoFactorEnabled, // activer si l’utilisateur a la 2FA
  });

  return <QRCode value={qr?.totpURI || ""} />;
}
```

👉 L’utilisateur scanne ce QR Code dans son appli d’authentification → il pourra générer les codes 2FA.

***

### 5. **Issuer (Émetteur)**

* Par défaut : `appName` défini dans ton `auth.ts`.
* Sinon → `Better Auth`.
* Tu peux aussi le **surcharger** :

```ts
twoFactor({
  issuer: "MonApplicationSécurisée",
})
```

***

⚡ Prochaine étape logique après ça → **vérifier le code TOTP saisi par l’utilisateur pour finaliser l’activation** (car `twoFactorEnabled` reste `false` tant que le TOTP n’a pas été vérifié).

## 🔐 Vérification du TOTP (6 chiffres)

Lorsqu’un utilisateur active la 2FA :

1. Tu génères le QR code avec le `totpURI`.
2. L’utilisateur le scanne dans son app (Google Authenticator, Authy, etc.).
3. Il obtient un code à 6 chiffres qu’il doit saisir pour confirmer.

***

### 1. Vérifier le code TOTP

#### Côté client

```ts
const { data, error } = await authClient.twoFactor.verify({
  code: "123456", // le code à 6 chiffres de l'app
});
```

#### Côté serveur

```ts
const response = await auth.api.twoFactorVerify({
  body: {
    code: "123456",
  },
  headers: req.headers,
});
```

***

### 2. Propriétés

| Prop | Description                                                           | Type   |
| ---- | --------------------------------------------------------------------- | ------ |
| code | Le code TOTP (6 chiffres) généré par l’application d’authentification | string |

***

### 3. Résultat attendu

* Si le code est correct → ✅ `twoFactorEnabled` passe à **true** dans l’objet `user`.
* Si le code est invalide → ❌ erreur `INVALID_TOTP_CODE`.

***

### 4. Exemple en React (formulaire de vérification)

```tsx
"use client";
import { useState } from "react";
import { authClient } from "@/lib/auth-client";

export default function Verify2FA() {
  const [code, setCode] = useState("");
  const [message, setMessage] = useState("");

  const handleVerify = async () => {
    const { data, error } = await authClient.twoFactor.verify({ code });
    if (error) {
      setMessage("❌ Code invalide, réessaie !");
    } else {
      setMessage("✅ 2FA activée avec succès !");
    }
  };

  return (
    <div>
      <input
        type="text"
        placeholder="Entrez votre code 2FA"
        value={code}
        onChange={(e) => setCode(e.target.value)}
      />
      <button onClick={handleVerify}>Vérifier</button>
      <p>{message}</p>
    </div>
  );
}
```

***

👉 Une fois validé, ton utilisateur a officiellement **2FA activée** sur son compte (`twoFactorEnabled: true`).\
Au prochain login → il devra fournir **email + mot de passe + code TOTP**.

## 🔑 Backup Codes

Les _backup codes_ permettent à un utilisateur de se connecter si :

* il n’a plus accès à son application TOTP (perte de téléphone, réinstallation, etc.).
* chaque code ne peut être utilisé **qu’une seule fois**.

#### Exemple d’activation

Quand l’utilisateur active 2FA :

```ts
const { data, error } = await authClient.twoFactor.enable({
  password: "secure-password",
});

// data.backupCodes contient une liste de codes uniques
console.log(data.backupCodes);
/*
[
  "3KD9-XY7Z",
  "JH27-PLQ8",
  ...
]
*/
```

👉 Ces codes doivent être affichés **une seule fois** à l’utilisateur (et idéalement téléchargés en PDF/texte ou copiés dans un gestionnaire).

***

#### Utiliser un backup code

```ts
const { data, error } = await authClient.twoFactor.verify({
  code: "3KD9-XY7Z", // un backup code valide
});
```

Si valide → `twoFactorEnabled` reste actif, mais ce code est **invalidé** et ne pourra plus être réutilisé.

***

## 📱 Trusted Devices

Tu peux permettre à l’utilisateur de marquer un appareil comme _de confiance_.\
👉 Ainsi il ne devra **pas saisir son code 2FA à chaque connexion** depuis cet appareil.

#### Activation côté client

Lors du sign-in 2FA réussi, tu peux passer l’option :

```ts
await authClient.twoFactor.verify({
  code: "123456",
  trustedDevice: true, // optionnel
});
```

#### Résultat

* L’appareil est marqué comme “trusted” (un token lié au device est stocké).
* Lors des prochaines connexions, si ce device est reconnu → pas besoin de retaper le TOTP.

***

## ⚙️ Bonnes pratiques

* Toujours proposer **backup codes** dès l’activation.
* Donner une option **“Gérer mes appareils de confiance”** dans le compte.
* Permettre de **révoquer tous les appareils** si suspicion de piratage.

## 🔐 Vérification du TOTP avec BetterAuth

Après avoir activé le **Two-Factor Authentication (2FA)**, l’utilisateur doit entrer un **code TOTP** (généré via Google Authenticator, Authy, etc.).\
BetterAuth fournit la méthode `twoFactor.verifyTotp` pour valider ce code.

***

### ✅ Exemple côté client

```ts
import { authClient } from "@/lib/auth-client";

async function verifyTotp(code: string) {
  const { data, error } = await authClient.twoFactor.verifyTotp({
    code,           // Code TOTP à 6 chiffres
    trustDevice: true, // Facultatif : mémoriser l’appareil 30 jours
  });

  if (error) {
    console.error("Erreur de vérification 2FA :", error.message);
    alert("Code invalide ou expiré.");
    return;
  }

  alert("✅ 2FA vérifié avec succès !");
}
```

***

### 📌 Exemple React avec un formulaire

```tsx
"use client";
import { useState } from "react";
import { authClient } from "@/lib/auth-client";

export default function VerifyTotpForm() {
  const [code, setCode] = useState("");

  const handleVerify = async () => {
    const { data, error } = await authClient.twoFactor.verifyTotp({
      code,
      trustDevice: true,
    });

    if (error) {
      alert("❌ Code incorrect ou expiré.");
    } else {
      alert("✅ 2FA validé, appareil marqué comme de confiance.");
    }
  };

  return (
    <div className="p-4 border rounded space-y-3">
      <h2 className="text-lg font-bold">Vérifier votre code TOTP</h2>
      <input
        type="text"
        placeholder="Entrez votre code à 6 chiffres"
        value={code}
        onChange={(e) => setCode(e.target.value)}
        className="border p-2 rounded w-full"
      />
      <button
        onClick={handleVerify}
        className="bg-blue-600 text-white px-4 py-2 rounded"
      >
        Vérifier
      </button>
    </div>
  );
}
```

***

### 🔎 Points importants

* BetterAuth accepte le **code actuel**, celui du **créneau précédent**, et celui du **créneau suivant** → tolérance aux décalages d’horloge.
* L’option `trustDevice: true` permet de **mémoriser l’appareil** pendant **30 jours** (cookie spécial).
* Si l’utilisateur supprime ses cookies ou change de navigateur → il devra **ressaisir son TOTP**.

## 🔐 OTP (One-Time Password) avec BetterAuth

L’**OTP** est un **code aléatoire et temporaire** envoyé par email, SMS ou autre canal défini par ton application.\
Contrairement au **TOTP**, il n’est **pas basé sur l’heure**, mais généré côté serveur et envoyé à l’utilisateur.

***

### ⚙️ Configuration du plugin

Avant d’utiliser OTP comme second facteur, tu dois définir une fonction `sendOTP` dans ton instance BetterAuth.\
C’est elle qui enverra le code à l’utilisateur.

```ts
// auth.ts
import { betterAuth } from "better-auth";
import { twoFactor } from "better-auth/plugins";

export const auth = betterAuth({
  plugins: [
    twoFactor({
      otpOptions: {
        async sendOTP({ user, otp }, request) {
          // Exemple : envoi par email
          await sendEmail({
            to: user.email,
            subject: "Votre code de connexion",
            text: `Votre code OTP est : ${otp}`,
          });

          // Tu pourrais aussi envoyer par SMS via Twilio par ex.
        },
      },
    }),
  ],
});
```

***

### ✉️ Envoi d’un OTP

Côté client, on appelle `twoFactor.sendOtp()`.\
Cela déclenche ton `sendOTP` défini dans la config.

```ts
// Client
const { data, error } = await authClient.twoFactor.sendOtp({
  trustDevice: true, // Facultatif : mémoriser l’appareil 30 jours
});

if (data) {
  // Redirige vers une page ou affiche un champ pour entrer le code
}
```

📌 **Paramètres :**

* `trustDevice?` → si vrai, l’appareil sera marqué comme "de confiance" pendant 30 jours (renouvelé à chaque connexion).

***

### 🔑 Vérification de l’OTP

Après saisie du code par l’utilisateur, on appelle `twoFactor.verifyOtp`.

```ts
// Client
const { data, error } = await authClient.twoFactor.verifyOtp({
  code: "123456",  // Code OTP saisi par l’utilisateur
  trustDevice: true, // Facultatif : mémoriser l’appareil
});

if (error) {
  alert("❌ Code invalide ou expiré");
} else {
  alert("✅ Connexion réussie avec OTP");
}
```

📌 **Paramètres :**

* `code` → le code OTP à 6 chiffres (ou plus, selon ta config).
* `trustDevice?` → idem que pour l’envoi.

***

### 🔎 Comparaison rapide avec TOTP

| 🔐 Méthode | 🛠 Fonctionnement                                                           | 📱 Exemple d’usage                                                                         |
| ---------- | --------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------ |
| **TOTP**   | Généré localement par une app (Google Authenticator, Authy) toutes les 30s. | Idéal pour sécurité forte sans dépendre d’email/SMS.                                       |
| **OTP**    | Généré côté serveur puis envoyé (email, SMS, etc.).                         | Plus simple pour les utilisateurs mais moins sécurisé (si email piraté ou SMS intercepté). |

un mini **module React complet** qui montre :

1. **Envoi OTP** (clic sur un bouton → reçoit un code par email/SMS).
2. **Saisie & Vérification OTP** (champ input → valider le code).
3. Gestion des erreurs et succès.

***

## 🔐 Exemple React : Authentification OTP avec BetterAuth

```tsx
// otp-login.tsx
"use client";

import { useState } from "react";
import { authClient } from "@/lib/auth-client"; // ton instance BetterAuth côté client

export default function OtpLogin() {
  const [step, setStep] = useState<"send" | "verify">("send");
  const [code, setCode] = useState("");
  const [loading, setLoading] = useState(false);
  const [message, setMessage] = useState("");

  // 1️⃣ Envoi OTP
  const handleSendOtp = async () => {
    setLoading(true);
    setMessage("");
    try {
      const { error } = await authClient.twoFactor.sendOtp({
        trustDevice: true, // optionnel : "appareil de confiance"
      });
      if (error) throw error;
      setStep("verify");
      setMessage("✅ Code envoyé ! Vérifie ton email ou SMS.");
    } catch (err: any) {
      setMessage("❌ Erreur lors de l’envoi du code");
    } finally {
      setLoading(false);
    }
  };

  // 2️⃣ Vérification OTP
  const handleVerifyOtp = async () => {
    setLoading(true);
    setMessage("");
    try {
      const { error } = await authClient.twoFactor.verifyOtp({
        code,
        trustDevice: true,
      });
      if (error) throw error;
      setMessage("🎉 Connexion réussie !");
    } catch (err: any) {
      setMessage("❌ Code invalide ou expiré");
    } finally {
      setLoading(false);
    }
  };

  return (
    <div className="max-w-md mx-auto mt-10 p-6 rounded-2xl shadow-md bg-white">
      <h1 className="text-xl font-bold mb-4 text-center">🔐 Connexion par OTP</h1>

      {step === "send" && (
        <button
          onClick={handleSendOtp}
          disabled={loading}
          className="w-full py-2 px-4 bg-blue-600 text-white rounded-lg hover:bg-blue-700 disabled:opacity-50"
        >
          {loading ? "Envoi..." : "📩 Envoyer le code OTP"}
        </button>
      )}

      {step === "verify" && (
        <div className="space-y-4">
          <input
            type="text"
            value={code}
            onChange={(e) => setCode(e.target.value)}
            placeholder="Entrez le code OTP"
            className="w-full px-3 py-2 border rounded-lg"
          />
          <button
            onClick={handleVerifyOtp}
            disabled={loading}
            className="w-full py-2 px-4 bg-green-600 text-white rounded-lg hover:bg-green-700 disabled:opacity-50"
          >
            {loading ? "Vérification..." : "✅ Vérifier le code"}
          </button>
        </div>
      )}

      {message && (
        <p className="mt-4 text-center text-sm font-medium">{message}</p>
      )}
    </div>
  );
}
```

***

## ⚡ Fonctionnement du flow

1. **Envoi OTP** → appelle `authClient.twoFactor.sendOtp()` → envoie le code via ta fonction `sendOTP()` (email/SMS).
2. **Saisie du code** → l’utilisateur entre son OTP.
3. **Vérification OTP** → `authClient.twoFactor.verifyOtp()` valide le code et connecte l’utilisateur.
4. **Option trustDevice** → permet de mémoriser l’appareil pour **30 jours** (pas besoin de redemander OTP à chaque fois).

## 🔐 Backup Codes avec BetterAuth

Les **backup codes** sont un mécanisme de secours qui permettent à l’utilisateur de se connecter même s’il perd l’accès à son téléphone ou email (utile en cas de perte du smartphone avec TOTP ou du mail OTP).

***

### 1️⃣ Générer des Backup Codes

Lorsqu’un utilisateur active la 2FA, tu peux générer une série de codes de secours à lui afficher **une seule fois** (il devra les sauvegarder).

```ts
// client-side
const { data, error } = await authClient.twoFactor.generateBackupCodes({
  password: "user-password", // nécessaire pour vérifier l’identité
});

if (data) {
  console.log("🔑 Backup Codes générés :", data.backupCodes);
  // ⚠️ Important : montrer à l’utilisateur et lui dire de les stocker en lieu sûr
}
```

📌 **Notes importantes** :

* Les anciens codes sont supprimés et remplacés.
* Ces codes sont **à usage unique** (dès qu’un code est utilisé, il disparaît).
* Tu ne devrais jamais regénérer les codes en boucle (sinon l’utilisateur perd ceux déjà notés).

***

### 2️⃣ Vérifier un Backup Code

Si l’utilisateur perd son accès principal (TOTP ou OTP), il peut saisir un **backup code** :

```ts
// client-side
const { data, error } = await authClient.twoFactor.verifyBackupCode({
  code: "123456",   // code saisi par l’utilisateur
  disableSession: false, 
  trustDevice: true, // optionnel : mémorise l’appareil 30j
});

if (!error) {
  console.log("✅ Connexion réussie avec un backup code !");
}
```

⚡ Dès qu’un backup code est utilisé :

* il est supprimé de la base (non réutilisable).
* la session est validée comme si l’utilisateur avait passé le 2FA.

***

### 3️⃣ Voir les Backup Codes (backend uniquement)

Pour des raisons de sécurité, **seul le serveur** peut accéder aux codes existants, et seulement pour un utilisateur ayant une **session fraîche** (connexion récente).

```ts
// server-side
const data = await auth.api.viewBackupCodes({
  body: {
    userId: "user-id", // si null, prend l’utilisateur connecté
  },
});

console.log("🔑 Codes de secours actifs :", data.backupCodes);
```

👉 Cas d’usage : une page "Mon compte" où l’utilisateur peut revoir ses codes **juste après activation de la 2FA**.

***

### 4️⃣ Exemple UI React (affichage codes après génération)

```tsx
"use client";

import { useState } from "react";
import { authClient } from "@/lib/auth-client";

export default function BackupCodes() {
  const [codes, setCodes] = useState<string[]>([]);

  const generateCodes = async () => {
    const { data, error } = await authClient.twoFactor.generateBackupCodes({
      password: "secure-password", // ⚠️ demandé à l’utilisateur
    });
    if (!error && data) {
      setCodes(data.backupCodes);
    }
  };

  return (
    <div className="p-6 bg-white shadow-md rounded-xl">
      <h2 className="text-lg font-bold mb-4">🔑 Codes de secours</h2>
      <button
        onClick={generateCodes}
        className="px-4 py-2 bg-blue-600 text-white rounded-lg"
      >
        Générer de nouveaux codes
      </button>

      {codes.length > 0 && (
        <ul className="mt-4 space-y-2">
          {codes.map((code, i) => (
            <li
              key={i}
              className="font-mono text-lg bg-gray-100 p-2 rounded-md text-center"
            >
              {code}
            </li>
          ))}
        </ul>
      )}
    </div>
  );
}
```

***

### 🚨 Bonnes pratiques

1. 🔒 **Afficher les codes une seule fois** → ne jamais permettre de "revoir" les anciens.
2. 📜 **Informer l’utilisateur** → il doit noter ces codes dans un gestionnaire de mots de passe ou les imprimer.
3. ❌ **Supprimer les anciens codes** → BetterAuth le fait automatiquement.
4. 🔐 **Ne pas stocker les codes côté client** → ils doivent rester uniquement côté utilisateur.

## 🔐 Exemple Complet : TOTP + Backup Codes avec BetterAuth

👉 Objectif :

* L’utilisateur active la 2FA
* On lui affiche un **QR Code** pour scanner avec Google Authenticator
* On lui génère aussi des **codes de secours** (à noter ou imprimer)

***

### 1️⃣ Backend (auth.ts)

```ts
import { betterAuth } from "better-auth";
import { twoFactor } from "better-auth/plugins";

export const auth = betterAuth({
  appName: "My Secure App",
  plugins: [
    twoFactor({
      otpOptions: {
        async sendOTP({ user, otp }, request) {
          // tu peux l’envoyer par email/SMS
          console.log(`📩 OTP envoyé à ${user.email} : ${otp}`);
        },
      },
    }),
  ],
});
```

***

### 2️⃣ Génération du TOTP (QR Code)

```tsx
"use client";

import { useState } from "react";
import QRCode from "react-qr-code";
import { authClient } from "@/lib/auth-client";

export default function TwoFactorSetup() {
  const [totpURI, setTotpURI] = useState<string | null>(null);
  const [backupCodes, setBackupCodes] = useState<string[]>([]);

  const enable2FA = async () => {
    // 1. Activer 2FA
    await authClient.twoFactor.enable({
      password: "secure-password", // ⚠️ demandé à l’utilisateur
    });

    // 2. Récupérer le TOTP URI (QR code)
    const totpRes = await authClient.twoFactor.getTotpUri({
      password: "secure-password",
    });
    setTotpURI(totpRes.data?.totpURI || null);

    // 3. Générer les backup codes
    const backupRes = await authClient.twoFactor.generateBackupCodes({
      password: "secure-password",
    });
    setBackupCodes(backupRes.data?.backupCodes || []);
  };

  return (
    <div className="p-6 bg-white shadow-md rounded-xl space-y-6">
      <h2 className="text-lg font-bold">🔐 Activer la 2FA</h2>

      <button
        onClick={enable2FA}
        className="px-4 py-2 bg-blue-600 text-white rounded-lg"
      >
        Activer 2FA
      </button>

      {totpURI && (
        <div className="space-y-2">
          <h3 className="font-semibold">📱 Scanne ce QR code :</h3>
          <QRCode value={totpURI} size={180} />
        </div>
      )}

      {backupCodes.length > 0 && (
        <div className="space-y-2">
          <h3 className="font-semibold">🔑 Codes de secours :</h3>
          <ul className="grid grid-cols-2 gap-2">
            {backupCodes.map((code, i) => (
              <li
                key={i}
                className="font-mono text-lg bg-gray-100 p-2 rounded-md text-center"
              >
                {code}
              </li>
            ))}
          </ul>
          <p className="text-sm text-red-600">
            ⚠️ Sauvegarde ces codes ! Tu ne pourras pas les revoir plus tard.
          </p>
        </div>
      )}
    </div>
  );
}
```

***

### 3️⃣ Vérification du TOTP (connexion)

Après la saisie du code 2FA par l’utilisateur :

```ts
const { data, error } = await authClient.twoFactor.verifyTotp({
  code: "123456", // le code entré depuis Google Authenticator
  trustDevice: true,
});

if (!error) {
  console.log("✅ 2FA validée, utilisateur connecté !");
}
```

***

### ✅ Résumé du flow utilisateur

1. L’utilisateur active la 2FA → un **QR Code TOTP** et des **Backup Codes** sont affichés.
2. Lors de la connexion :
   * Il saisit son mot de passe.
   * Si 2FA activée → il doit entrer son **code TOTP**.
   * S’il a perdu son téléphone → il peut utiliser un **backup code**.

👉 Ça couvre **100% des cas** (sécurité + récupération).

## 📱 Trusted Devices avec BetterAuth

### 👉 Principe

* Quand un utilisateur **vérifie son TOTP ou OTP**, tu peux ajouter `trustDevice: true`.
* Résultat : l’appareil est marqué comme **fiable** pour **60 jours**.
* Pendant cette période → pas besoin de ressaisir le code 2FA.
* Chaque connexion réussie **rafraîchit le compteur** ⏳.

***

### Exemple avec **TOTP**

```ts
const verify2FA = async (code: string) => {
  const { data, error } = await authClient.twoFactor.verifyTotp({
    code,
    callbackURL: "/dashboard",
    trustDevice: true, // ✅ On marque le device comme "trusted"
  });

  if (data) {
    console.log("✅ 2FA vérifiée et appareil marqué comme fiable !");
  } else {
    console.error("❌ Erreur de vérification 2FA :", error?.message);
  }
};
```

***

### Exemple avec **OTP**

```ts
const verifyOtp = async (code: string) => {
  const { data, error } = await authClient.twoFactor.verifyOtp({
    code,
    trustDevice: true, // ✅ Même principe que pour TOTP
  });

  if (data) {
    console.log("✅ OTP vérifié et appareil marqué comme fiable !");
  }
};
```

***

### ⚠️ Points Importants

1. **Durée par défaut :** 60 jours → configurable côté BetterAuth (si tu veux modifier).
2. **Refresh automatique :** à chaque connexion réussie, le délai de 60 jours est repoussé.
3. **Sécurité :**
   * Tu peux afficher dans le **profil utilisateur** une liste des _devices de confiance_.
   * Tu peux permettre de **révoquer un device** à distance (exemple : téléphone volé).

## ⚙️ Gestion des Appareils de Confiance (Trusted Devices)

### 1️⃣ Stockage côté serveur

BetterAuth garde une **empreinte de l’appareil** (cookie + infos device) quand tu utilises `trustDevice: true`.\
Tu peux exposer une route pour **lister et révoquer** les appareils. Exemple :

```ts
// routes/trusted-devices.ts
import { auth } from "@/auth";

export async function GET(req: Request) {
  const session = await auth.api.getSession({ req });

  if (!session) {
    return new Response("Unauthorized", { status: 401 });
  }

  // Récupère la liste des devices de confiance
  const devices = await auth.api.listTrustedDevices({
    body: { userId: session.user.id },
  });

  return Response.json(devices);
}

export async function DELETE(req: Request) {
  const session = await auth.api.getSession({ req });
  const { deviceId } = await req.json();

  if (!session) {
    return new Response("Unauthorized", { status: 401 });
  }

  await auth.api.revokeTrustedDevice({
    body: { userId: session.user.id, deviceId },
  });

  return Response.json({ success: true });
}
```

***

### 2️⃣ Affichage côté client (React / Next.js)

```tsx
"use client";
import { useEffect, useState } from "react";

export default function TrustedDevices() {
  const [devices, setDevices] = useState<any[]>([]);

  useEffect(() => {
    fetch("/api/trusted-devices")
      .then((res) => res.json())
      .then(setDevices);
  }, []);

  const revokeDevice = async (deviceId: string) => {
    await fetch("/api/trusted-devices", {
      method: "DELETE",
      body: JSON.stringify({ deviceId }),
    });
    setDevices(devices.filter((d) => d.id !== deviceId));
  };

  return (
    <div className="p-4 border rounded-xl shadow">
      <h2 className="text-lg font-semibold mb-2">Appareils de confiance</h2>
      <ul className="space-y-2">
        {devices.map((device) => (
          <li
            key={device.id}
            className="flex justify-between items-center border p-2 rounded-lg"
          >
            <div>
              <p className="font-medium">{device.deviceName}</p>
              <p className="text-sm text-gray-500">
                Dernière connexion : {new Date(device.lastUsed).toLocaleString()}
              </p>
            </div>
            <button
              onClick={() => revokeDevice(device.id)}
              className="px-3 py-1 bg-red-500 text-white rounded-lg hover:bg-red-600"
            >
              Révoquer
            </button>
          </li>
        ))}
      </ul>
    </div>
  );
}
```

***

### 3️⃣ UX Conseillée

* ✅ Ajouter cette section dans `/settings/security`.
* ✅ Afficher : _nom du device_ (ex: Chrome - Windows 11), _dernière utilisation_.
* ✅ Bouton "Révoquer" → supprime l’appareil de la liste et force la saisie du 2FA la prochaine fois.
* ✅ Option : bouton _Révoquer tous les appareils_ sauf celui en cours.

## 🔐 Révocation Globale des Appareils de Confiance

### 1️⃣ API Route pour tout révoquer

On ajoute une route `DELETE /api/trusted-devices/all` :

```ts
// routes/trusted-devices.ts
import { auth } from "@/auth";

export async function DELETE(req: Request) {
  const session = await auth.api.getSession({ req });

  if (!session) {
    return new Response("Unauthorized", { status: 401 });
  }

  // Révoque tous les devices sauf l'appareil actuel
  await auth.api.revokeAllTrustedDevices({
    body: { userId: session.user.id, keepCurrent: true },
  });

  return Response.json({ success: true });
}
```

***

### 2️⃣ Côté client (React / Next.js)

On ajoute un bouton pour déclencher la révocation globale :

```tsx
"use client";
import { useEffect, useState } from "react";

export default function TrustedDevices() {
  const [devices, setDevices] = useState<any[]>([]);

  useEffect(() => {
    fetch("/api/trusted-devices")
      .then((res) => res.json())
      .then(setDevices);
  }, []);

  const revokeDevice = async (deviceId: string) => {
    await fetch("/api/trusted-devices", {
      method: "DELETE",
      body: JSON.stringify({ deviceId }),
    });
    setDevices(devices.filter((d) => d.id !== deviceId));
  };

  const revokeAllDevices = async () => {
    await fetch("/api/trusted-devices/all", {
      method: "DELETE",
    });
    setDevices([]); // vide la liste
  };

  return (
    <div className="p-4 border rounded-xl shadow">
      <h2 className="text-lg font-semibold mb-2">Appareils de confiance</h2>
      <ul className="space-y-2 mb-4">
        {devices.map((device) => (
          <li
            key={device.id}
            className="flex justify-between items-center border p-2 rounded-lg"
          >
            <div>
              <p className="font-medium">{device.deviceName}</p>
              <p className="text-sm text-gray-500">
                Dernière connexion : {new Date(device.lastUsed).toLocaleString()}
              </p>
            </div>
            <button
              onClick={() => revokeDevice(device.id)}
              className="px-3 py-1 bg-red-500 text-white rounded-lg hover:bg-red-600"
            >
              Révoquer
            </button>
          </li>
        ))}
      </ul>
      {devices.length > 0 && (
        <button
          onClick={revokeAllDevices}
          className="w-full px-4 py-2 bg-red-600 text-white rounded-lg hover:bg-red-700"
        >
          Révoquer tous les appareils
        </button>
      )}
    </div>
  );
}
```

***

### 3️⃣ UX Conseillée

* ✅ **Confirmation modale** avant la révocation globale (_"Êtes-vous sûr ? Vous devrez ressaisir un code 2FA sur vos prochains appareils."_).
* ✅ **Garder l’appareil actuel actif** (`keepCurrent: true`) pour éviter de bloquer l’utilisateur.
* ✅ **Notification** après succès → “Tous vos appareils de confiance ont été révoqués.”

## 🛡️ Admin – Gestion des Appareils de Confiance

### 1️⃣ API côté serveur

On expose une route admin sécurisée :

```ts
// routes/admin/trusted-devices.ts
import { auth } from "@/auth";

// ✅ L’admin doit être authentifié et avoir un rôle "admin"
async function requireAdmin(req: Request) {
  const session = await auth.api.getSession({ req });
  if (!session || session.user.role !== "admin") {
    throw new Response("Forbidden", { status: 403 });
  }
  return session;
}

// 🔹 Lister les appareils d’un utilisateur
export async function GET(req: Request) {
  await requireAdmin(req);
  const { searchParams } = new URL(req.url);
  const userId = searchParams.get("userId");

  if (!userId) return new Response("Missing userId", { status: 400 });

  const devices = await auth.api.listTrustedDevices({ body: { userId } });

  return Response.json(devices);
}

// 🔹 Révoquer un appareil précis
export async function DELETE(req: Request) {
  await requireAdmin(req);
  const { userId, deviceId } = await req.json();

  await auth.api.revokeTrustedDevice({ body: { userId, deviceId } });

  return Response.json({ success: true });
}

// 🔹 Révoquer tous les appareils
export async function DELETE_ALL(req: Request) {
  await requireAdmin(req);
  const { userId } = await req.json();

  await auth.api.revokeAllTrustedDevices({
    body: { userId, keepCurrent: false },
  });

  return Response.json({ success: true });
}
```

***

### 2️⃣ UI côté Admin

Un composant React pour afficher et gérer les appareils :

```tsx
"use client";
import { useEffect, useState } from "react";

export default function AdminTrustedDevices({ userId }: { userId: string }) {
  const [devices, setDevices] = useState<any[]>([]);

  useEffect(() => {
    fetch(`/api/admin/trusted-devices?userId=${userId}`)
      .then((res) => res.json())
      .then(setDevices);
  }, [userId]);

  const revokeDevice = async (deviceId: string) => {
    await fetch("/api/admin/trusted-devices", {
      method: "DELETE",
      body: JSON.stringify({ userId, deviceId }),
    });
    setDevices(devices.filter((d) => d.id !== deviceId));
  };

  const revokeAll = async () => {
    await fetch("/api/admin/trusted-devices/all", {
      method: "DELETE",
      body: JSON.stringify({ userId }),
    });
    setDevices([]);
  };

  return (
    <div className="p-4 border rounded-xl shadow bg-white">
      <h2 className="text-lg font-semibold mb-4">
        Appareils de confiance de l’utilisateur
      </h2>
      {devices.length === 0 ? (
        <p className="text-gray-500">Aucun appareil trouvé</p>
      ) : (
        <ul className="space-y-2 mb-4">
          {devices.map((device) => (
            <li
              key={device.id}
              className="flex justify-between items-center border p-2 rounded-lg"
            >
              <div>
                <p className="font-medium">{device.deviceName}</p>
                <p className="text-sm text-gray-500">
                  Dernière utilisation :{" "}
                  {new Date(device.lastUsed).toLocaleString()}
                </p>
              </div>
              <button
                onClick={() => revokeDevice(device.id)}
                className="px-3 py-1 bg-red-500 text-white rounded-lg hover:bg-red-600"
              >
                Révoquer
              </button>
            </li>
          ))}
        </ul>
      )}
      {devices.length > 0 && (
        <button
          onClick={revokeAll}
          className="w-full px-4 py-2 bg-red-600 text-white rounded-lg hover:bg-red-700"
        >
          Révoquer tous les appareils
        </button>
      )}
    </div>
  );
}
```

***

### 3️⃣ Bonnes pratiques de sécurité

* ✅ Vérifier que **seuls les admins** ont accès à cette route (`role: "admin"`).
* ✅ Logger chaque action de révocation (`userId`, `adminId`, `timestamp`) pour traçabilité.
* ✅ Notifier l’utilisateur par email : _“Tous vos appareils de confiance ont été révoqués par un administrateur.”_

## 🏷️ Issuer (Nom d’application dans 2FA)

L’**issuer** est le nom de votre application qui s’affiche dans l’application d’authentification de l’utilisateur (Google Authenticator, Authy, 1Password, etc.).

Par défaut, Better Auth utilise `Better Auth` comme nom.\
👉 Mais vous pouvez personnaliser ce champ afin que vos utilisateurs voient **le nom de votre app** lorsqu’ils configurent leur 2FA.

***

### Exemple de configuration

```ts
import { betterAuth } from "better-auth";
import { twoFactor } from "better-auth/plugins";

export const auth = betterAuth({
  appName: "My App", // valeur globale par défaut
  plugins: [
    twoFactor({
      issuer: "my-app-name", // nom affiché dans Google Auth ou autres apps TOTP
    }),
  ],
});
```

***

### Résultat côté utilisateur

* Sans `issuer` → dans Google Authenticator, l’entrée apparaît comme **Better Auth**.
* Avec `issuer: "my-app-name"` → elle apparaît comme **my-app-name (email@user.com)**.

***

✅ **Bonnes pratiques**

* Choisissez un nom **clair et unique**, idéalement celui de votre application (`MonSuperSaaS`, `ShopSecure`, etc.).
* Gardez le même `issuer` partout pour éviter la confusion si un utilisateur configure plusieurs comptes.

Alors voici comment générer et afficher un **QR Code avec l’`issuer` intégré** pour que tes utilisateurs puissent scanner directement dans leur app d’authentification (Google Authenticator, Authy, etc.).

***

### 📌 Étape 1 : Activer 2FA et récupérer l’URI TOTP

Quand un utilisateur active la 2FA, Better Auth renvoie une **TOTP URI**.\
Cette URI contient :

* le **secret partagé**
* l’**issuer** (ton appName)
* l’**identité de l’utilisateur** (ex: email)

Exemple côté client :

```ts
const { data, error } = await authClient.twoFactor.getTotpUri({
  password: "secure-password", // le mot de passe de l'utilisateur
});

if (data) {
  console.log(data.totpURI);
  // Exemple d’URI :
  // otpauth://totp/my-app-name:user@email.com?secret=JBSWY3DPEHPK3PXP&issuer=my-app-name
}
```

***

### 📌 Étape 2 : Générer le QR Code dans React

Avec la librairie [`react-qr-code`](https://www.npmjs.com/package/react-qr-code) :

```tsx
import { useState, useEffect } from "react";
import QRCode from "react-qr-code";
import { authClient } from "./auth-client";

export default function TwoFactorSetup({ password }: { password: string }) {
  const [uri, setUri] = useState<string | null>(null);

  useEffect(() => {
    const fetchUri = async () => {
      const { data } = await authClient.twoFactor.getTotpUri({ password });
      if (data?.totpURI) {
        setUri(data.totpURI);
      }
    };
    fetchUri();
  }, [password]);

  return (
    <div>
      <h2>Configurer la 2FA</h2>
      {uri ? (
        <>
          <QRCode value={uri} />
          <p>Scannez ce QR Code dans Google Authenticator ou Authy.</p>
        </>
      ) : (
        <p>Chargement du QR Code...</p>
      )}
    </div>
  );
}
```

***

### 📌 Résultat côté utilisateur

Dans **Google Authenticator** ou **Authy**, l’utilisateur verra apparaître une entrée comme :

```
my-app-name (user@email.com)
```

## 🔐 Workflow Complet 2FA avec BetterAuth

### 1. ⚙️ Configuration du backend (`auth.ts`)

```ts
import { betterAuth } from "better-auth";
import { twoFactor } from "better-auth/plugins";

export const auth = betterAuth({
  appName: "MySecureApp", // 👈 ton nom d’app (issuer dans Google Auth)
  plugins: [
    twoFactor({
      issuer: "MySecureApp", // affiché dans Google Authenticator
      skipVerificationOnEnable: false, // l’utilisateur doit vérifier le code après activation
      otpOptions: {
        async sendOTP({ user, otp }, request) {
          // Exemple simple : envoi par email
          await sendEmail({
            to: user.email,
            subject: "Votre code OTP",
            text: `Votre code est : ${otp}`,
          });
        },
      },
    }),
  ],
});
```

***

### 2. 📌 Activation de la 2FA

Quand l’utilisateur veut activer la 2FA (depuis son profil par ex.) :

```ts
const { data, error } = await authClient.twoFactor.enable({
  password: "secure-password", // requis
});

// 🔑 Retourne :
// data.totpURI → à transformer en QR Code
// data.backupCodes → à stocker/sauvegarder
```

Ensuite, tu génères un QR Code pour que l’utilisateur scanne l’`uri`.

***

### 3. 📲 Vérification du code TOTP

L’utilisateur ouvre son app Authenticator → il saisit son code.

```ts
const { data, error } = await authClient.twoFactor.verifyTotp({
  code: "123456", // code à 6 chiffres de l’app
  trustDevice: true, // ✅ option : marque l’appareil comme fiable (pas de re-2FA avant 30j)
});

if (data) {
  console.log("2FA activée et vérifiée !");
}
```

👉 Si le code est correct → la colonne `twoFactorEnabled` passe à `true`.

***

### 4. 🔑 Connexion avec 2FA

Lorsqu’un utilisateur se connecte et qu’il a la 2FA activée :

* Le **login classique** (`signIn.email`) retourne `twoFactorRedirect: true`.
* Tu rediriges l’utilisateur vers une **page de saisie du code 2FA**.

```ts
const { data } = await authClient.signIn.email({
  email: "user@example.com",
  password: "password123",
}, {
  async onSuccess(ctx) {
    if (ctx.data.twoFactorRedirect) {
      router.push("/two-factor"); // page où saisir le code
    }
  },
});
```

Sur la page `/two-factor`, tu appelles `verifyTotp` ou `verifyOtp` selon ton setup.

***

### 5. 🔒 Backup Codes

Tu peux générer des **codes de secours** si l’utilisateur perd son téléphone :

```ts
const { data } = await authClient.twoFactor.generateBackupCodes({
  password: "secure-password",
});

// data.backupCodes → liste de codes uniques
```

Pour les utiliser :

```ts
await authClient.twoFactor.verifyBackupCode({
  code: "ABCD-1234", // code de secours
});
```

Chaque code utilisé est supprimé.

***

### 6. 📱 Trusted Devices

Lors de la vérification (TOTP/OTP/backup code), si `trustDevice: true`,\
BetterAuth stocke un **token d’appareil fiable valable 30 jours**.

Ainsi → l’utilisateur ne verra plus l’écran 2FA sur ce device pendant 30j.

***

## ✅ Résumé du Flow Utilisateur

1. **Activation** : utilisateur entre son password → reçoit un QR Code + backup codes.
2. **Vérification** : utilisateur saisit le premier code TOTP pour activer la 2FA.
3. **Connexion** :
   * Login normal → si 2FA activée → redirection page 2FA.
   * L’utilisateur saisit son code TOTP/OTP → accès OK.
4. **Confort** : possibilité de marquer l’appareil comme _fiable_ + générer des backup codes.
5. **Sécurité** : possibilité de désactiver 2FA en entrant son password.

**exemple complet de page Next.js `/two-factor.tsx`** pour gérer tout le flow 2FA avec **BetterAuth** (QR Code, saisie du code, gestion des erreurs).

***

## 📄 Exemple `/two-factor.tsx`

```tsx
"use client";

import { useState } from "react";
import { useRouter } from "next/navigation";
import { authClient } from "@/lib/auth-client"; // ton authClient configuré
import QRCode from "react-qr-code";

export default function TwoFactorPage() {
  const [password, setPassword] = useState("");
  const [qrUri, setQrUri] = useState<string | null>(null);
  const [backupCodes, setBackupCodes] = useState<string[]>([]);
  const [code, setCode] = useState("");
  const [error, setError] = useState<string | null>(null);
  const router = useRouter();

  // 🔹 Étape 1 : Activer la 2FA
  const enable2FA = async () => {
    setError(null);
    try {
      const res = await authClient.twoFactor.enable({
        password,
      });
      if (res?.data) {
        setQrUri(res.data.totpURI);
        setBackupCodes(res.data.backupCodes);
      }
    } catch (err: any) {
      setError(err.message || "Erreur lors de l’activation 2FA");
    }
  };

  // 🔹 Étape 2 : Vérifier le code saisi
  const verifyCode = async () => {
    setError(null);
    try {
      const res = await authClient.twoFactor.verifyTotp({
        code,
        trustDevice: true, // marque l’appareil comme fiable 30j
      });
      if (res?.data) {
        router.push("/dashboard"); // redirection après succès
      }
    } catch (err: any) {
      setError("Code invalide, réessaie !");
    }
  };

  return (
    <div className="min-h-screen flex flex-col items-center justify-center bg-gray-50 p-6">
      <div className="bg-white shadow-lg rounded-2xl p-8 w-full max-w-md">
        <h1 className="text-2xl font-bold text-center mb-6">
          🔐 Activer la double authentification
        </h1>

        {/* Étape 1 - Activation */}
        {!qrUri && (
          <>
            <p className="text-gray-600 text-sm mb-4">
              Entre ton mot de passe pour activer la 2FA :
            </p>
            <input
              type="password"
              placeholder="Mot de passe"
              value={password}
              onChange={(e) => setPassword(e.target.value)}
              className="w-full border rounded-md px-3 py-2 mb-4"
            />
            <button
              onClick={enable2FA}
              className="w-full bg-blue-600 text-white py-2 rounded-md hover:bg-blue-700"
            >
              Activer 2FA
            </button>
          </>
        )}

        {/* Étape 2 - QR Code */}
        {qrUri && (
          <div className="text-center">
            <p className="mb-4 text-gray-700">
              Scanne ce QR Code dans Google Authenticator ou Authy :
            </p>
            <QRCode value={qrUri} size={180} className="mx-auto mb-4" />
            <p className="text-sm text-gray-500 mb-2">
              Ou ajoute manuellement : <br />
              <span className="font-mono text-xs break-all">{qrUri}</span>
            </p>

            {/* Backup Codes */}
            {backupCodes.length > 0 && (
              <div className="bg-gray-100 rounded-md p-4 text-left my-4">
                <p className="font-semibold mb-2">🛠 Codes de secours :</p>
                <ul className="space-y-1">
                  {backupCodes.map((bc) => (
                    <li
                      key={bc}
                      className="font-mono text-sm bg-white p-2 rounded"
                    >
                      {bc}
                    </li>
                  ))}
                </ul>
                <p className="text-xs text-gray-500 mt-2">
                  Sauvegarde bien ces codes ! Un seul usage par code.
                </p>
              </div>
            )}

            {/* Vérification du code */}
            <input
              type="text"
              placeholder="Code 2FA"
              value={code}
              onChange={(e) => setCode(e.target.value)}
              className="w-full border rounded-md px-3 py-2 mb-4 text-center tracking-widest font-mono"
            />
            <button
              onClick={verifyCode}
              className="w-full bg-green-600 text-white py-2 rounded-md hover:bg-green-700"
            >
              Vérifier
            </button>
          </div>
        )}

        {error && (
          <p className="text-red-600 text-sm text-center mt-4">{error}</p>
        )}
      </div>
    </div>
  );
}
```

***

## 🚀 Fonctionnalités incluses

* ✅ **Étape d’activation** : saisie du mot de passe → génération du QR Code.
* ✅ **QR Code affiché** (scan via Google Auth/Authy).
* ✅ **Backup Codes affichés** et sauvegardables.
* ✅ **Vérification du code** avec option `trustDevice`.
* ✅ **Gestion des erreurs** (mot de passe faux, code invalide).
* ✅ **Redirection** vers `/dashboard` après succès.

## 📑 **Schéma nécessaire pour 2FA**

### 1. 🔹 Table `user`

On ajoute **un champ supplémentaire** :

| Champ              | Type    | Clé | Description                                                                 |
| ------------------ | ------- | --- | --------------------------------------------------------------------------- |
| `twoFactorEnabled` | boolean | –   | Indique si l’utilisateur a activé la 2FA (`true` = activée, `false` = non). |

👉 Ce champ permet à BetterAuth de savoir si un utilisateur doit être redirigé vers l’étape 2FA lors de la connexion.

***

### 2. 🔹 Nouvelle table `twoFactor`

Une table séparée pour stocker les secrets et codes nécessaires à la 2FA :

| Champ         | Type   | Clé | Description                                                                 |
| ------------- | ------ | --- | --------------------------------------------------------------------------- |
| `id`          | string | PK  | Identifiant unique de l’enregistrement 2FA (clé primaire).                  |
| `userId`      | string | FK  | Référence vers `user.id` (clé étrangère).                                   |
| `secret`      | string | ?   | Secret généré pour calculer les codes TOTP (via app Google Auth, Authy, …). |
| `backupCodes` | string | ?   | Liste des **codes de secours** générés pour récupérer l’accès.              |

👉 Cette table centralise tout ce qui est lié à la 2FA d’un utilisateur (secret TOTP, backup codes).

***

## ⚙️ Exemple Prisma Schema

Si tu utilises **Prisma**, ton `schema.prisma` doit ressembler à ceci :

```prisma
model User {
  id               String    @id @default(cuid())
  email            String    @unique
  name             String?
  password         String?
  twoFactorEnabled Boolean   @default(false)  // 👈 ajouté pour 2FA
  sessions         Session[]
  accounts         Account[]
  twoFactor        TwoFactor?
}

model TwoFactor {
  id          String @id @default(cuid())
  userId      String @unique
  user        User   @relation(fields: [userId], references: [id], onDelete: Cascade)
  secret      String?
  backupCodes String?
}
```

***

## 🎯 Rôle de chaque élément

* **`user.twoFactorEnabled`** → active ou non la 2FA pour l’utilisateur.
* **`twoFactor.secret`** → utilisé pour générer et valider les TOTP (Google Authenticator, etc.).
* **`twoFactor.backupCodes`** → codes de secours en cas de perte d’accès au téléphone.
* **`twoFactor.userId`** → garantit que chaque enregistrement est lié à un utilisateur unique.

## ⚙️ **Options 2FA dans BetterAuth**

### 🔹 **Options générales**

| Option                     | Type    | Défaut                       | Description                                                                                                                 |
| -------------------------- | ------- | ---------------------------- | --------------------------------------------------------------------------------------------------------------------------- |
| `twoFactorTable`           | string  | `"twoFactor"`                | Nom de la table utilisée pour stocker les données 2FA. Tu peux le changer si tu veux un autre nom de table.                 |
| `skipVerificationOnEnable` | boolean | `false`                      | Si `true`, la 2FA sera activée directement sans que l’utilisateur doive vérifier son premier code TOTP (⚠️ moins sécurisé). |
| `issuer`                   | string  | `"Better Auth"` ou `appName` | Nom de ton application qui apparaîtra dans l’app d’authentification (Google Authenticator, Authy…).                         |

***

### 🔹 **TOTP Options** (codes générés par apps comme Google Auth)

Ces options s’appliquent uniquement au **TOTP** (Time-based One-Time Password).

| Propriété | Type   | Défaut | Description                                                              |
| --------- | ------ | ------ | ------------------------------------------------------------------------ |
| `digits`  | number | `6`    | Nombre de chiffres du code TOTP (6 ou 8 en général).                     |
| `period`  | number | `30`   | Durée de validité d’un code TOTP en secondes (30s par défaut, standard). |

***

### 🔹 **OTP Options** (codes envoyés par email/SMS)

Ces options s’appliquent au **OTP** simple (One-Time Password envoyé par email, SMS, …).

| Propriété  | Type     | Défaut    | Description                                                                    |
| ---------- | -------- | --------- | ------------------------------------------------------------------------------ |
| `sendOTP`  | function | –         | Fonction obligatoire pour envoyer le code OTP (email, SMS…).                   |
| `period`   | number   | `3`       | Durée de validité de l’OTP (en minutes).                                       |
| `storeOTP` | string   | `"plain"` | Mode de stockage : `"plain"` (non chiffré, déconseillé en prod) ou `"hashed"`. |

***

### 🔹 **Backup Code Options** (codes de secours)

Les **codes de secours** permettent à l’utilisateur de récupérer son compte si son téléphone est perdu.

| Propriété                   | Type     | Défaut    | Description                                                                          |
| --------------------------- | -------- | --------- | ------------------------------------------------------------------------------------ |
| `amount`                    | number   | `10`      | Nombre de codes générés.                                                             |
| `length`                    | number   | `10`      | Longueur de chaque code (nombre de caractères).                                      |
| `customBackupCodesGenerate` | function | –         | Fonction personnalisée si tu veux générer tes propres codes (ex. format spécifique). |
| `storeBackupCodes`          | string   | `"plain"` | Mode de stockage : `"plain"` ou `"hashed"`.                                          |

***

## 📌 Exemple complet de configuration

```ts
import { betterAuth } from "better-auth";
import { twoFactor } from "better-auth/plugins";

export const auth = betterAuth({
  appName: "My Secure App",
  plugins: [
    twoFactor({
      twoFactorTable: "user_two_factor",
      skipVerificationOnEnable: false, // ⚠️ à garder sur false pour la sécurité
      issuer: "My Secure App", // affiché dans Google Authenticator

      totpOptions: {
        digits: 6,
        period: 30, // code valide 30 secondes
      },

      otpOptions: {
        async sendOTP({ user, otp }) {
          // Ici tu envoies le code par email/SMS
          await sendEmail(user.email, `Votre code est : ${otp}`);
        },
        period: 5, // valide 5 minutes
        storeOTP: "hashed", // meilleur pour la sécurité
      },

      backupCodeOptions: {
        amount: 12, // 12 codes
        length: 8, // 8 caractères chacun
        storeBackupCodes: "hashed",
      },
    }),
  ],
});
```

***

👉 En résumé :

* **TOTP** → sécurisé, utilisable via apps d’authentification.
* **OTP** → pratique par mail/SMS mais plus vulnérable.
* **Backup Codes** → indispensable pour récupération de compte.
* **Options avancées** → te permettent d’adapter la durée de validité, la sécurité (hashage), et le nombre de codes.

## 🖥️ **Client – Intégration du plugin 2FA**

Pour que ton **frontend** gère correctement la double authentification, il faut ajouter le plugin `twoFactorClient` dans la config de ton `authClient`.

### Exemple complet

```ts
// auth-client.ts
import { createAuthClient } from "better-auth/client";
import { twoFactorClient } from "better-auth/client/plugins";

const authClient = createAuthClient({
  plugins: [
    twoFactorClient({
      onTwoFactorRedirect() {
        // Lorsque BetterAuth détecte que l’utilisateur doit entrer son code 2FA
        window.location.href = "/2fa"; 
      },
    }),
  ],
});

export default authClient;
```

***

### ⚙️ **Options disponibles**

#### 🔹 `onTwoFactorRedirect`

* **Type**: `() => void`
* **Description**:\
  Callback déclenchée automatiquement quand un utilisateur doit passer par la vérification 2FA.\
  Exemple d’utilisation :
  * Redirection vers une page `/2fa`
  * Affichage d’un modal de saisie du code
  * Ouverture d’un composant React dédié

***

### 🚦 **Workflow côté client**

1. L’utilisateur **tente de se connecter** (`signIn.email`, `signIn.social`, …).
2. Si la 2FA est activée sur son compte :
   * La réponse contient `twoFactorRedirect: true`.
   * **BetterAuth déclenche `onTwoFactorRedirect`**.
3. Tu rediriges l’utilisateur vers la page `/2fa` (ou un modal) pour qu’il saisisse son **code TOTP/OTP/Backup Code**.
4. Une fois validé (`verifyTotp`, `verifyOtp`, `verifyBackupCode`), l’utilisateur est loggé normalement.

***

### ✅ Exemple côté React

```tsx
"use client";

import { authClient } from "@/lib/auth-client";

export default function LoginForm() {
  const handleLogin = async () => {
    const res = await authClient.signIn.email({
      email: "john@example.com",
      password: "password123",
    });

    if (res?.data?.twoFactorRedirect) {
      // Si jamais tu veux gérer le redirect manuellement
      window.location.href = "/2fa";
    }
  };

  return (
    <button onClick={handleLogin}>
      Se connecter
    </button>
  );
}
```

***

👉 En résumé :

* **`twoFactorClient`** est indispensable côté client pour intercepter la redirection 2FA.
* **`onTwoFactorRedirect`** te donne le contrôle sur ce qui se passe quand un utilisateur doit vérifier son 2FA.

**exemple complet d’une page `/2fa.tsx`** avec saisie du code 2FA (TOTP ou OTP) et vérification via `verifyTotp`.

***

## 📄 **Page `/2fa.tsx`**

```tsx
"use client";

import { useState } from "react";
import authClient from "@/lib/auth-client"; // ton instance authClient
import { useRouter } from "next/navigation";

export default function TwoFactorPage() {
  const [code, setCode] = useState("");
  const [error, setError] = useState<string | null>(null);
  const router = useRouter();

  const handleVerify = async () => {
    setError(null);

    const { data, error } = await authClient.twoFactor.verifyTotp({
      code, // le code TOTP généré par Google Authenticator / Authy
      trustDevice: true, // optionnel : mémoriser ce device pendant 30 jours
    });

    if (error) {
      setError(error.message || "Invalid code, try again.");
      return;
    }

    if (data) {
      // ✅ Authentification réussie
      router.push("/dashboard");
    }
  };

  return (
    <div className="flex min-h-screen items-center justify-center bg-gray-50">
      <div className="w-full max-w-sm rounded-2xl bg-white p-6 shadow-lg">
        <h1 className="mb-4 text-xl font-bold text-gray-800">
          Vérification 2FA
        </h1>
        <p className="mb-4 text-sm text-gray-600">
          Entrez le code à 6 chiffres généré par votre application
          d’authentification.
        </p>

        <input
          type="text"
          value={code}
          onChange={(e) => setCode(e.target.value)}
          maxLength={6}
          placeholder="Code 2FA"
          className="mb-3 w-full rounded-lg border px-3 py-2 text-center text-lg tracking-widest focus:border-blue-500 focus:outline-none"
        />

        {error && (
          <p className="mb-3 text-sm text-red-600">
            ⚠️ {error}
          </p>
        )}

        <button
          onClick={handleVerify}
          className="w-full rounded-lg bg-blue-600 py-2 font-semibold text-white hover:bg-blue-700 transition"
        >
          Vérifier
        </button>
      </div>
    </div>
  );
}
```

***

## 🚦 **Fonctionnement**

1. L’utilisateur est redirigé vers `/2fa` après connexion si son compte a 2FA activé.
2. Il saisit son code généré par son **application d’authentification (Google Authenticator, Authy, etc.)**.
3. `authClient.twoFactor.verifyTotp` envoie le code au serveur.
4. Si c’est correct → l’utilisateur est redirigé vers `/dashboard`.
5. Si c’est incorrect → message d’erreur affiché.
