# 📦 Importer et exporter des composants en React

### 1. Pourquoi découper ses composants en fichiers ?

* Plus ton app grossit, plus tes composants s’accumulent.
* Si tu gardes tout dans **un seul fichier**, ça devient vite illisible.
* Solution : **chaque composant dans son propre fichier**.\
  👉 Avantages : organisation, réutilisation, clarté.

***

### 2. Le fichier de composant racine

Dans tes premiers exemples, tu avais **Profile** et **Gallery** définis dans **App.js** :

```jsx
function Profile() {
  return (
    <img
      src="https://i.imgur.com/MK3eW3As.jpg"
      alt="Katherine Johnson"
    />
  );
}

export default function Gallery() {
  return (
    <section>
      <h1>Scientifiques de renom</h1>
      <Profile />
      <Profile />
      <Profile />
    </section>
  );
}
```

👉 Ici, **Gallery** est exporté par défaut, donc c’est le **composant racine** de ce fichier.

⚠️ Selon le setup :

* En **React classique** (Vite, CRA…), ce sera `App.js`.
* En **Next.js** : chaque page est un **composant racine**, par ex. `pages/index.js` ou `app/page.tsx`.

***

### 3. Export par défaut (`export default`)

C’est l’export le plus courant pour **un seul composant principal** par fichier.

#### Exemple :

```jsx
// Profile.js
export default function Profile() {
  return <img src="https://i.imgur.com/MK3eW3As.jpg" alt="Katherine Johnson" />;
}
```

Et tu l’importes ainsi :

```jsx
// Gallery.js
import Profile from "./Profile";

export default function Gallery() {
  return (
    <section>
      <h1>Scientifiques de renom</h1>
      <Profile />
    </section>
  );
}
```

👉 **Avantage** : quand tu importes, tu choisis le nom (`Profile`, `Avatar`, `X`) car il n’y a **qu’un seul export par défaut**.\
👉 **Inconvénient** : ça peut semer la confusion si tu renommes sans faire exprès.

***

### 4. Export nommé (`export`)

Idéal quand tu veux exporter **plusieurs composants dans le même fichier**.

#### Exemple :

```jsx
// components.js
export function Profile() {
  return <img src="https://i.imgur.com/MK3eW3As.jpg" alt="Katherine Johnson" />;
}

export function Gallery() {
  return (
    <section>
      <h1>Scientifiques de renom</h1>
      <Profile />
    </section>
  );
}
```

Import :

```jsx
import { Profile, Gallery } from "./components";
```

👉 **Avantage** : lisible, clair, plusieurs exports possibles.\
👉 **Inconvénient** : tu dois respecter exactement le **même nom** que dans l’export (`Profile` ≠ `profile`).

***

### 5. Mélanger export par défaut + nommés

Tu peux combiner les deux dans un fichier, mais il vaut mieux garder ça **rare** (ça embrouille vite).

#### Exemple :

```jsx
// Gallery.js
export function Profile() {
  return <img src="..." alt="Katherine Johnson" />;
}

export default function Gallery() {
  return (
    <section>
      <h1>Scientifiques de renom</h1>
      <Profile />
    </section>
  );
}
```

Import :

```jsx
import Gallery, { Profile } from "./Gallery";
```

***

### 6. Quand choisir quoi ?

* **Export par défaut**\
  👉 Pour le **composant principal** d’un fichier (ex. `Gallery.js`, `Button.js`).
* **Export nommé**\
  👉 Pour les **petits composants utilitaires** regroupés ensemble (ex. `components.js` qui contient `Button`, `Input`, `Card`).
* **Mix**\
  👉 Rarement utile. Réserve ça aux cas particuliers.

***

### 🧠 TL;DR

1. Découpe tes composants en plusieurs fichiers → lisibilité ++.
2. `export default` → un seul composant par fichier, libre de renommer à l’import.
3. `export` (nommé) → plusieurs composants, import avec le même nom.
4. Next.js : chaque fichier de `pages/` ou `app/` est un **composant racine**.

## 📦 Exporter et importer un composant en React

### 1. Pourquoi séparer les composants ?

Si tu gardes **tous tes composants dans un seul fichier (`App.js`)**, ça devient vite un chaos 🌀.\
👉 En déplaçant chaque composant dans son propre fichier :

* tu rends ton code **modulaire** (chaque fichier = une brique claire),
* tu peux **réutiliser** tes composants facilement,
* tu gardes ton projet **facile à maintenir**.

***

### 2. Les 3 étapes

Pour déplacer un composant dans un nouveau fichier, il y a toujours le même rituel :

1. **Créer un fichier** → par ex. `Gallery.js`
2. **Exporter** le composant depuis ce fichier
   * `export default` (cas le plus courant pour le “composant principal”)
   * ou `export` nommé
3. **Importer** ce composant là où tu veux l’utiliser (`App.js`, une page Next.js, etc.)

***

### 3. Exemple : découper `App.js`

Avant : tout est dans `App.js` :

```jsx
function Profile() {
  return (
    <img
      src="https://i.imgur.com/MK3eW3As.jpg"
      alt="Katherine Johnson"
    />
  );
}

export default function Gallery() {
  return (
    <section>
      <h1>Scientifiques de renom</h1>
      <Profile />
      <Profile />
      <Profile />
    </section>
  );
}
```

***

#### Étape 1 → Créer `Gallery.js`

```jsx
// Gallery.js
function Profile() {
  return (
    <img
      src="https://i.imgur.com/MK3eW3As.jpg"
      alt="Katherine Johnson"
    />
  );
}

// ✅ On exporte Gallery par défaut
export default function Gallery() {
  return (
    <section>
      <h1>Scientifiques de renom</h1>
      <Profile />
      <Profile />
      <Profile />
    </section>
  );
}
```

👉 Ici, `Profile` est **interne** à `Gallery` (il n’est pas exporté car utilisé uniquement dans ce fichier).\
👉 `Gallery` est **exporté par défaut**, car c’est le composant principal du fichier.

***

#### Étape 2 → Modifier `App.js`

```jsx
// App.js
import Gallery from "./Gallery.js"; // ou "./Gallery"

export default function App() {
  return <Gallery />;
}
```

👉 Ici, on **importe Gallery** (export par défaut de `Gallery.js`)\
👉 Puis on définit `App` comme **composant racine** et on l’exporte par défaut.

***

### 4. Petit détail sur l’extension `.js`

*   Tu peux écrire :

    ```jsx
    import Gallery from "./Gallery.js";
    ```
*   ou simplement :

    ```jsx
    import Gallery from "./Gallery";
    ```

👉 Les deux marchent avec React, mais la version avec `.js` est plus proche du standard ES Modules natif (meilleure pratique si tu veux être rigoureux).

***

### 🧠 À retenir

1. **Découpe tes composants** en fichiers séparés pour plus de clarté.
2. `export default` → parfait pour le composant principal d’un fichier.
3. `export` nommé → pratique si tu veux exposer plusieurs composants dans le même fichier.
4. Tu peux importer un fichier **avec ou sans l’extension `.js`**.
5. Toujours garder ton **composant racine** (`App`, `page.tsx` en Next.js) simple, en déléguant aux autres composants.

## 📦 Exports par défaut vs. exports nommés

### 1. Deux manières d’exporter en JavaScript

En React (et en JavaScript en général), tu as **deux grands types d’exports** :

1. **Export par défaut** → un seul par fichier.
2. **Export nommé** → autant que tu veux dans un fichier.

👉 Et tu peux même combiner les deux (rare, mais possible).

***

### 2. Le tableau comparatif (place pour l’image 🖼️)

<figure><img src="../.gitbook/assets/image (6).png" alt=""><figcaption></figcaption></figure>

***

#### 📊 Tableau : différence d’écriture

| Syntaxe        | Déclaration d’export                  | Déclaration d’import                    |
| -------------- | ------------------------------------- | --------------------------------------- |
| **Par défaut** | `export default function Button() {}` | `import Button from './Button.js';`     |
| **Nommé**      | `export function Button() {}`         | `import { Button } from './Button.js';` |

***

### 3. Détails importants

#### ✅ Export par défaut

* Un fichier **ne peut avoir qu’un seul** export par défaut.
*   À l’import, tu peux mettre **le nom que tu veux**.

    ```jsx
    // Button.js
    export default function Button() {}

    // Import
    import Button from "./Button.js";   // classique
    import Banana from "./Button.js";   // marche aussi 🍌
    ```

👉 Le nom `Button` ou `Banana` n’a pas d’importance → ça pointe toujours vers l’export par défaut.

***

#### ✅ Export nommé

* Tu peux en avoir **plusieurs par fichier**.
*   À l’import, tu dois respecter **exactement le même nom**.

    ```jsx
    // components.js
    export function Button() {}
    export function Input() {}

    // Import
    import { Button, Input } from "./components.js"; // ✅ doit correspondre
    ```

👉 Ici, tu ne peux pas écrire `import { Banana } from ...` → ça plantera car `Banana` n’existe pas.

***

### 4. Quand utiliser quoi ?

* **Export par défaut** → quand ton fichier ne contient qu’un seul composant principal.
  * Exemple : `Button.js` → `export default function Button() {}`
* **Export nommé** → quand tu veux regrouper plusieurs composants ou utilitaires dans un même fichier.
  * Exemple : `components.js` → `export { Button, Input, Card }`

👉 Dans les deux cas :

* Donne toujours des **noms clairs** à tes composants ET à tes fichiers.
*   Évite les composants anonymes comme :

    ```jsx
    export default () => <button>Click</button>;
    ```

    ⚠️ Mauvaise pratique → ça rend le débogage beaucoup plus difficile.

***

### 🧠 TL;DR

1. **Par défaut** = un seul export, import librement nommé.
2. **Nommé** = plusieurs exports possibles, import avec le nom exact.
3. **Bonne pratique** :
   * `export default` pour un composant principal unique par fichier.
   * `export` nommé pour plusieurs composants/utilitaires.
4. Toujours préférer des **composants nommés** (pas anonymes) → débogage + lisibilité.

## 📦 Exporter et importer plusieurs composants

### 1. Le problème de départ

Tu as un fichier `Gallery.js` qui contient deux composants :

* `Gallery` (une galerie de profils)
* `Profile` (un profil individuel)

👉 Mais un fichier **ne peut avoir qu’un seul `export default`**.\
Donc si tu veux rendre **les deux disponibles à l’extérieur**, tu dois faire :

* `Gallery` → export par défaut
* `Profile` → export nommé

***

### 2. Exemple de code

#### 🔹 Gallery.js

```jsx
// Export nommé (pas de "default")
export function Profile() {
  return (
    <img
      src="https://i.imgur.com/MK3eW3As.jpg"
      alt="Katherine Johnson"
    />
  );
}

// Export par défaut
export default function Gallery() {
  return (
    <section>
      <h1>Scientifiques de renom</h1>
      <Profile />
      <Profile />
      <Profile />
    </section>
  );
}
```

👉 Ici :

* `Profile` est **exporté nommé**.
* `Gallery` est **exporté par défaut**.

***

#### 🔹 App.js

```jsx
// Import par défaut
import Gallery from './Gallery.js';

// Import nommé
import { Profile } from './Gallery.js';

export default function App() {
  return (
    <Profile />
    // ou <Gallery />
  );
}
```

👉 Dans ce fichier :

* `Gallery` est importé **sans accolades** (car export par défaut).
* `Profile` est importé **avec accolades** (car export nommé).

***

### 3. Résultat

* Si tu affiches `<Profile />` → tu vois **une seule image**.
* Si tu affiches `<Gallery />` → tu vois **la galerie de trois images**.

***

### 4. Récapitulatif

* **Gallery.js**
  * `export function Profile() {}` → export **nommé**
  * `export default function Gallery() {}` → export **par défaut**
* **App.js**
  * `import Gallery from "./Gallery.js";` → import **par défaut**
  * `import { Profile } from "./Gallery.js";` → import **nommé**

***

### ⚠️ Remarque importante

* Certaines équipes **choisissent de ne pas mélanger** export par défaut + export nommé dans un même fichier, car ça peut embrouiller.
* Tu peux adopter une convention claire pour ton projet :
  * soit **toujours export par défaut** (1 composant = 1 fichier),
  * soit **toujours export nommé** (plusieurs composants regroupés).
* Mais techniquement, les deux sont possibles.

***

### 🧠 TL;DR

1. Un fichier peut avoir **un seul export par défaut** mais **plusieurs exports nommés**.
2. Import par défaut → **sans accolades**.
3. Import nommé → **avec accolades**.
4. Tu peux mélanger les deux, mais fais-le avec parcimonie (clarté avant tout).
