# 🧠 L’état : la mémoire d’un composant

👉 Les composants doivent parfois **se souvenir d’informations** qui changent au fil du temps, en fonction :

* d’une **interaction utilisateur** (saisie, clic, etc.),
* ou d’un **processus interne** (timer, résultat d’une requête, etc.).

C’est ce qu’on appelle **l’état local (state)** en React.

***

## ✨ Exemples concrets d’état

* La valeur saisie dans un **champ de formulaire**.
* L’image **active** dans un carrousel.
* Les produits dans le **panier d’achat**.

***

## ⚡ Le Hook `useState`

React fournit un **Hook** appelé `useState` qui permet d’ajouter de la mémoire aux composants.

```jsx
import { useState } from "react";

export default function Counter() {
  // 1️⃣ Déclare une variable d’état
  const [count, setCount] = useState(0);

  // 2️⃣ Utilisation et mise à jour de l’état
  return (
    <button onClick={() => setCount(count + 1)}>
      Vous avez cliqué {count} fois
    </button>
  );
}
```

***

## 📌 Explications

* `const [count, setCount] = useState(0)`
  * **`count`** → la valeur actuelle de l’état (ici : 0 au départ).
  * **`setCount`** → la fonction pour mettre à jour l’état.
  * **`0`** → la valeur initiale.
* Quand tu cliques sur le bouton, `setCount(count + 1)` **met à jour l’état** → React relance le rendu du composant → le bouton affiche la nouvelle valeur.

***

## 🔑 Points importants

* ⚡ **Chaque composant a son propre état.**
* ⚡ L’état est **local** : il appartient uniquement au composant qui le définit.
* ⚡ Quand l’état change, React **refait le rendu** du composant pour afficher la nouvelle valeur.

## 🚫 Quand une variable classique ne suffit pas

Prenons ton exemple avec `Gallery` :

```jsx
import { sculptureList } from './data.js';

export default function Gallery() {
  let index = 0;

  function handleClick() {
    index = index + 1; // ❌ change une variable locale
  }

  let sculpture = sculptureList[index];
  return (
    <>
      <button onClick={handleClick}>Suivant</button>
      <h2>
        <i>{sculpture.name} </i> par {sculpture.artist}
      </h2>
      <h3>
        ({index + 1} sur {sculptureList.length})
      </h3>
      <img src={sculpture.url} alt={sculpture.alt} />
      <p>{sculpture.description}</p>
    </>
  );
}
```

***

### ❌ Pourquoi ça ne marche pas ?

1. **Les variables locales disparaissent à chaque rendu.**\
   Quand React ré-exécute la fonction `Gallery`, il remet `let index = 0`. Donc tu ne gardes pas la valeur entre deux clics.
2. **Changer une variable locale ne déclenche pas de rendu.**\
   React ne « surveille » pas tes variables classiques → il ne sait pas qu’il doit redessiner le composant.

***

## ✅ La solution : `useState`

Tu dois utiliser **l’état (state)** pour :

* **mémoriser** la valeur entre deux rendus,
* et **demander à React** de refaire le rendu quand la valeur change.

```jsx
import { useState } from "react";
import { sculptureList } from "./data.js";

export default function Gallery() {
  // 1️⃣ Déclarer une variable d’état
  const [index, setIndex] = useState(0);

  function handleClick() {
    // 2️⃣ Mettre à jour l’état → React relance le rendu
    setIndex(index + 1);
  }

  let sculpture = sculptureList[index];
  return (
    <>
      <button onClick={handleClick}>Suivant</button>
      <h2>
        <i>{sculpture.name} </i> par {sculpture.artist}
      </h2>
      <h3>
        ({index + 1} sur {sculptureList.length})
      </h3>
      <img src={sculpture.url} alt={sculpture.alt} />
      <p>{sculpture.description}</p>
    </>
  );
}
```

***

## 🔑 Différence clé

* `let index = 0` → **variable locale** (oubliée après chaque rendu).
* `const [index, setIndex] = useState(0)` → **état persistant** (mémorisé et lié au cycle de rendu de React).

## ⚡ Ajouter une variable d’état avec `useState`

1.  **Importer `useState`** :

    ```js
    import { useState } from "react";
    ```
2.  **Déclarer l’état** au lieu d’une variable classique :

    ```js
    const [index, setIndex] = useState(0);
    ```

    * `index` 👉 la **valeur actuelle de l’état**.
    * `setIndex` 👉 la **fonction qui met à jour l’état** (et redéclenche le rendu).
    * `0` 👉 la **valeur initiale**.
3.  **Mettre à jour l’état** dans un gestionnaire :

    ```js
    function handleClick() {
      setIndex(index + 1); // ✅ met à jour et déclenche un rendu
    }
    ```

***

## ✅ Exemple complet corrigé

```jsx
import { useState } from "react";
import { sculptureList } from "./data.js";

export default function Gallery() {
  // 🎯 index est la valeur d’état, 0 la valeur initiale
  const [index, setIndex] = useState(0);

  function handleClick() {
    // 🔄 met à jour l’état → React refait le rendu
    setIndex(index + 1);
  }

  let sculpture = sculptureList[index];

  return (
    <>
      <button onClick={handleClick}>Suivant</button>
      <h2>
        <i>{sculpture.name}</i> par {sculpture.artist}
      </h2>
      <h3>
        ({index + 1} sur {sculptureList.length})
      </h3>
      <img src={sculpture.url} alt={sculpture.alt} />
      <p>{sculpture.description}</p>
    </>
  );
}
```

***

## 🎣 Dites bonjour à votre premier **Hook**

👉 `useState` est un **Hook React**.

* Les Hooks sont des **fonctions spéciales** qui commencent par `use`.
* Ils ne s’utilisent **que dans la phase de rendu** d’un composant.
* Ils permettent d’« accrocher » votre composant à une fonctionnalité de React (ici, **l’état**).

Exemples d’autres Hooks que tu verras ensuite :

* `useEffect` → effets de bord
* `useContext` → contexte
* `useReducer` → gestion avancée d’état

***

## ⚠️ Règles importantes des Hooks

* Toujours les appeler **à la racine du composant**.\
  ❌ Pas dans une boucle, une condition ou une fonction imbriquée.
* Toujours commencer par `use` (React s’appuie sur ce préfixe).

👉 Les Hooks, c’est comme les `import` : tu les déclares en haut et sans conditions.

## 🧩 Anatomie de `useState`

👉 Quand tu écris :

```js
const [index, setIndex] = useState(0);
```

Il se passe **exactement** ceci :

1. **React crée une paire `[valeur, fonctionDeMiseÀJour]`**.
   * `index` → la valeur actuelle de l’état.
   * `setIndex` → la fonction pour mettre à jour l’état.
2. **Tu donnes une valeur initiale** (ici `0`).
   * Lors du premier rendu → React renvoie `[0, setIndex]`.
   * Lors d’un rendu suivant → React renvoie `[valeurDéjàMémorisée, setIndex]`.
3. **Quand tu appelles `setIndex`** →
   * React met à jour la valeur stockée.
   * Puis il **déclenche un nouveau rendu** du composant.

***

## 📚 Exemple simple

```jsx
import { useState } from "react";

export default function Counter() {
  const [count, setCount] = useState(0); // 🟢 état avec valeur initiale = 0

  function handleClick() {
    setCount(count + 1); // 🔄 met à jour l’état
  }

  return (
    <div>
      <p>Compteur : {count}</p>
      <button onClick={handleClick}>+1</button>
    </div>
  );
}
```

👉 À chaque clic :

* React **mémorise la nouvelle valeur** de `count`.
* Le composant **se re-rend** avec cette valeur à jour.

***

## 🎛 Plusieurs variables d’état

Un composant peut avoir **autant de variables d’état que nécessaire** :

```jsx
const [index, setIndex] = useState(0);     // nombre
const [showMore, setShowMore] = useState(false); // booléen
```

* Si tes données **sont indépendantes** → plusieurs états séparés (comme `index` et `showMore`).
* Si tes données **sont liées** → un seul état sous forme d’**objet** ou **tableau**.

Exemple avec un formulaire 👇

```js
const [form, setForm] = useState({
  name: "",
  email: "",
});
```

***

## 🔍 Comment React sait quel état rendre ?

Tu l’as remarqué : on n’écrit pas d’`id` quand on appelle `useState`.\
Alors… comment React fait pour ne pas se tromper ? 🤔

👉 **Réponse** : React s’appuie sur **l’ordre des appels aux Hooks**.

* Lors du premier rendu, il stocke chaque `useState` dans un tableau interne.
* À chaque rendu suivant, il lit les Hooks **dans le même ordre**.
* D’où la règle ⚠️ : **toujours appeler les Hooks à la racine du composant, jamais dans une boucle, condition ou fonction imbriquée**.

***

## 🛠 Illustration simplifiée (pseudo-code interne React)

```js
let hooks = [];
let currentIndex = 0;

function useState(initialValue) {
  let state = hooks[currentIndex] ?? initialValue;

  function setState(newValue) {
    hooks[currentIndex] = newValue;
    rerender();
  }

  hooks[currentIndex] = state;
  currentIndex++;
  return [state, setState];
}
```

👉 En clair :

* React garde un **tableau de valeurs d’état**.
* Chaque appel à `useState` prend **la prochaine case du tableau**.
* À chaque rendu → l’ordre doit être le même → sinon tout se décale ❌.

***

## ✅ À retenir

* `useState` renvoie toujours une **paire `[valeur, setter]`**.
* Tu peux en avoir **autant que tu veux**, à condition de respecter **l’ordre d’appel**.
* Les Hooks fonctionnent **par position**, pas par nom → d’où la règle de ne pas les mettre dans des conditions/boucles.

## 🔒 L’état est **isolé** et **privé**

### 1. Chaque composant a **son propre état**

Quand tu écris :

```jsx
function Gallery() {
  const [index, setIndex] = useState(0);
  // ...
}
```

👉 Chaque **instance** de `<Gallery />` affichée à l’écran a **sa propre mémoire** pour `index`.

* Si tu mets deux `<Gallery />` dans la même page → elles ne partagent **pas** leur état.
* Cliquer sur une n’influence pas l’autre.

Exemple :

```jsx
import Gallery from './Gallery';

export default function Page() {
  return (
    <div className="Page">
      <Gallery /> {/* état #1 */}
      <Gallery /> {/* état #2, indépendant */}
    </div>
  );
}
```

👉 Ici, tu as **deux galeries indépendantes** avec leur état `index` isolé.

***

### 2. Différence avec une variable classique

* Si tu avais écrit `let index = 0` **hors du composant**, cette variable serait **partagée** entre toutes les instances → bug ❌.
* Avec `useState`, l’état est **lié à l’instance affichée à l’écran** → pas de mélange ✅.

C’est ce qui rend React **prédictible** : chaque composant gère **son petit bout de mémoire**.

***

### 3. L’état est **privé**

* **Un composant parent n’a pas accès à l’état de son enfant.**
* Seul le composant qui déclare un état peut le modifier.
* Le parent peut passer des **props** mais ne peut pas « forcer » un changement d’état interne.

👉 C’est l’inverse des props :

* **props** → données qui viennent de l’extérieur (le parent).
* **state** → mémoire interne, privée au composant.

***

### 4. Et si on veut partager un état ?

Parfois, tu veux que **plusieurs composants gardent un état synchronisé** (par ex. deux `<Gallery />` qui affichent la même image active).

👉 La bonne pratique :

* **remonter l’état** au parent commun (lift state up).
* Puis redistribuer la valeur et les fonctions de mise à jour via des **props**.

Exemple :

```jsx
function Page() {
  const [index, setIndex] = useState(0);

  return (
    <div className="Page">
      <Gallery index={index} setIndex={setIndex} />
      <Gallery index={index} setIndex={setIndex} />
    </div>
  );
}
```

Ici, les deux `<Gallery />` partagent le même état car il est **géré par leur parent** `Page`.

***

✅ **À retenir :**

* Chaque composant a son **propre état isolé**.
* L’état est **privé** → seul le composant qui le définit peut le modifier.
* Pour le partager, on **remonte l’état** à l’ancêtre commun.
