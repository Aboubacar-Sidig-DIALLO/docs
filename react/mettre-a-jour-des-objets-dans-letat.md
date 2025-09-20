# 📝 Mettre à jour des objets dans l’état

L’état peut contenir **n’importe quelle valeur JavaScript**, y compris des **objets**.\
⚠️ Mais attention : vous **ne devez jamais modifier directement** un objet stocké dans l’état React.

👉 Au lieu de ça, quand vous voulez le modifier :

* Créez un **nouvel objet** (ou une **copie** de l’existant).
* Puis passez cette nouvelle version à votre setter d’état (`setState`).

***

### 🚀 Vous allez apprendre

* ✅ Comment mettre à jour correctement un objet dans l’état React.
* ✅ Comment modifier un objet **imbriqué** sans le muter.
* ✅ Ce qu’est **l’immutabilité** et pourquoi il ne faut pas la casser.
* ✅ Comment utiliser **Immer** pour rendre la copie d’objets moins répétitive.

***

👉 Exemple concret (mauvaise pratique vs bonne pratique) :

#### ❌ Mauvaise pratique (mutation directe)

```js
const [person, setPerson] = useState({ name: "Alice", age: 25 });

// ❌ Mauvais : on modifie directement l’objet
person.age = 26;
setPerson(person);
```

➡️ Ici, React **ne détecte pas la mutation**, car la référence de `person` n’a pas changé.

***

#### ✅ Bonne pratique (copie immuable)

```js
const [person, setPerson] = useState({ name: "Alice", age: 25 });

// ✅ Bon : on crée un nouvel objet en copiant l’ancien
setPerson({
  ...person,  // copie de toutes les clés existantes
  age: person.age + 1  // mise à jour
});
```

➡️ Ici, React reçoit un **nouvel objet** → il comprend qu’il doit re-render.

***

⚡ Question :\
Veux-tu que je continue en détaillant **les 2 prochains cas pratiques** :

1. 🔄 Mettre à jour un objet simple (exemple profil utilisateur).
2. 🏗️ Mettre à jour un objet **imbriqué** (exemple : coordonnées avec `{user: {address: {city}}}`).

## 🔎 Qu’est-ce qu’une **mutation** ?

En React, tu peux stocker **n’importe quelle valeur JavaScript** dans l’état :

```js
const [x, setX] = useState(0);
```

👉 Jusqu’ici, on a surtout manipulé des **primitifs** :

* nombres (`0`, `42`)
* chaînes (`"Hello"`)
* booléens (`true`, `false`)

Ces valeurs sont **immuables** → c’est-à-dire **inchangées** et **en lecture seule**.\
Si tu veux les “modifier”, en réalité tu les **remplaces par une nouvelle valeur** :

```js
setX(5); // x passe de 0 à 5
```

⚡ Remarque : le nombre `0` lui-même ne change pas → il reste `0` pour toujours en mémoire.\
En JavaScript, on ne peut pas modifier un primitif existant.

***

### 🎯 Et pour les objets ?

Prenons un objet stocké dans l’état :

```js
const [position, setPosition] = useState({ x: 0, y: 0 });
```

Contrairement aux primitives, un objet **est techniquement mutable** :\
tu pourrais écrire :

```js
position.x = 5; // ⚠️ mutation directe !
```

👉 Ça s’appelle une **mutation** → tu modifies directement le contenu de l’objet.

***

### 🚫 Pourquoi éviter les mutations dans React ?

Même si **techniquement** tu peux muter un objet, en React il faut toujours les traiter comme **immuables** (comme si c’étaient des nombres ou des chaînes).\
➡️ En pratique : **ne jamais modifier un objet existant dans l’état**.\
Au lieu de ça → **crée une copie**, modifie la copie, puis remplace l’ancienne valeur :

```js
setPosition({
  ...position,   // copie l’ancien objet
  x: 5           // nouvelle valeur
});
```

***

✅ Résumé rapide :

* Primitifs (`number`, `string`, `boolean`) → **toujours immuables**.
* Objets et tableaux → **mutables en JavaScript**, mais en React → **tu dois les traiter comme immuables**.

.

***

## 📌 Traiter l’état comme **en lecture seule**

En d’autres termes, tout objet JavaScript que tu stockes dans l’état doit être considéré comme **non modifiable directement**.

***

### ❌ Exemple qui ne marche pas

Ici on stocke un objet `position` dans l’état. On voudrait déplacer un point rouge en suivant la souris, mais… il reste bloqué au coin de l’écran :

```jsx
import { useState } from 'react';

export default function MovingDot() {
  const [position, setPosition] = useState({
    x: 0,
    y: 0
  });
  return (
    <div
      onPointerMove={e => {
        position.x = e.clientX; // ⚠️ mutation directe
        position.y = e.clientY; // ⚠️ mutation directe
      }}
      style={{
        position: 'relative',
        width: '100vw',
        height: '100vh',
      }}>
      <div style={{
        position: 'absolute',
        backgroundColor: 'red',
        borderRadius: '50%',
        transform: `translate(${position.x}px, ${position.y}px)`,
        left: -10,
        top: -10,
        width: 20,
        height: 20,
      }} />
    </div>
  );
}
```

***

#### 🚨 Le problème

Ici on écrit :

```js
position.x = e.clientX;
position.y = e.clientY;
```

👉 Ça **modifie directement** l’objet `position` du rendu précédent.\
Mais comme on n’appelle **pas** `setPosition`, React n’a aucun moyen de savoir que quelque chose a changé → il ne refait pas de rendu. Résultat : rien ne bouge !

***

### ✅ La bonne pratique : recréer un nouvel objet

Il faut **créer un nouvel objet** et passer cet objet à la fonction de mise à jour `setPosition` :

```jsx
onPointerMove={e => {
  setPosition({
    x: e.clientX,
    y: e.clientY
  });
}}
```

Ce que ça dit à React :

1. « Remplace `position` par ce **nouvel objet** »
2. « Refais le rendu avec cette nouvelle valeur »

👉 Et là, le point rouge suit bien la souris 🎉

***

### 🔎 Détail : mutation locale = OK

La mutation n’est un problème que si tu modifies un **objet déjà existant dans l’état**.

Exemple à éviter ❌ :

```js
position.x = e.clientX; // mutation directe d’un objet déjà en état
```

Mais ceci ✅ est correct car tu modifies un objet **tout neuf** :

```js
const nextPosition = {};
nextPosition.x = e.clientX;
nextPosition.y = e.clientY;
setPosition(nextPosition);
```

En fait, c’est équivalent à :

```js
setPosition({
  x: e.clientX,
  y: e.clientY
});
```

👉 Tant que l’objet vient d’être créé et que rien d’autre ne le référence encore, tu peux le modifier librement avant de l’envoyer à `setPosition`.\
C’est ce qu’on appelle une **mutation locale** → totalement sûre et même pratique.

***

✅ À retenir :

* **Ne modifie jamais un objet déjà stocké dans l’état.**
* **Crée toujours une copie** (ou un nouvel objet) puis mets-le dans l’état avec `setState`.
* Les mutations locales (sur des objets fraîchement créés) sont parfaitement valides.

## 📌 Copier des objets avec la syntaxe _spread_ (`...`)

Dans l’exemple précédent, on recréait un objet `position` entièrement à chaque mouvement.\
Mais dans la vraie vie, tu veux souvent **mettre à jour une seule propriété d’un objet** tout en conservant les autres inchangées.\
👉 Exemple typique : un **formulaire**.

***

### ❌ Mauvaise pratique : mutation directe

```jsx
person.firstName = e.target.value; // ⚠️ mutation directe
```

Cela **modifie l’objet existant** → React ne détecte pas le changement, donc il ne refait pas de rendu.

***

### ✅ Bonne pratique : recréer un nouvel objet avec _spread_

```jsx
setPerson({
  ...person,              // copie toutes les anciennes propriétés
  firstName: e.target.value // écrase uniquement celle-ci
});
```

👉 `...person` signifie : **copie toutes les propriétés existantes** dans un nouvel objet.\
Puis on remplace uniquement `firstName` par la nouvelle valeur.

***

### Exemple complet : formulaire

```jsx
import { useState } from 'react';

export default function Form() {
  const [person, setPerson] = useState({
    firstName: 'Barbara',
    lastName: 'Hepworth',
    email: 'bhepworth@sculpture.com'
  });

  function handleFirstNameChange(e) {
    setPerson({
      ...person,
      firstName: e.target.value
    });
  }

  function handleLastNameChange(e) {
    setPerson({
      ...person,
      lastName: e.target.value
    });
  }

  function handleEmailChange(e) {
    setPerson({
      ...person,
      email: e.target.value
    });
  }

  return (
    <>
      <label>
        First name:
        <input value={person.firstName} onChange={handleFirstNameChange} />
      </label>
      <label>
        Last name:
        <input value={person.lastName} onChange={handleLastNameChange} />
      </label>
      <label>
        Email:
        <input value={person.email} onChange={handleEmailChange} />
      </label>
      <p>
        {person.firstName} {person.lastName} ({person.email})
      </p>
    </>
  );
}
```

***

### ⚡ Optimisation : un seul gestionnaire générique

Plutôt que d’écrire une fonction par champ (`handleFirstNameChange`, `handleLastNameChange`…), tu peux utiliser un seul **gestionnaire dynamique** grâce à `[]` pour les clés :

```jsx
function handleChange(e) {
  setPerson({
    ...person,
    [e.target.name]: e.target.value
  });
}
```

Et dans les inputs :

```jsx
<input name="firstName" value={person.firstName} onChange={handleChange} />
<input name="lastName" value={person.lastName} onChange={handleChange} />
<input name="email" value={person.email} onChange={handleChange} />
```

👉 Ici, `e.target.name` correspond à l’attribut `name` de chaque `<input>`.\
Ça permet de mettre à jour automatiquement la bonne clé dans l’objet d’état.

***

### ⚠️ Attention : _spread_ est **peu profond**

Le spread `...person` copie uniquement les **propriétés directes** (shallow copy).\
Si tu as des objets imbriqués, tu dois répéter la copie à chaque niveau ou utiliser une librairie comme **Immer**.

***

✅ À retenir :

* Ne **mute** jamais directement un objet dans l’état.
* Utilise `...spread` pour copier l’ancien objet et ne remplacer que ce qui change.
* Pour les formulaires, c’est pratique d’avoir un seul `useState` avec un objet plutôt que plein de `useState`.

## 🔹 Mise à jour d’un objet imbriqué dans l’état

En React, tu dois **toujours traiter l’état comme immuable**.\
Ça veut dire que même si JavaScript te permet de faire :

```js
person.artwork.city = 'New Delhi'; // ❌ mutation interdite
```

👉 En React, tu ne dois **jamais modifier directement** un objet déjà stocké dans l’état.\
Sinon React ne détecte pas le changement → pas de nouveau rendu.

***

### ✅ Bonne pratique : recréer les objets à chaque niveau

Pour modifier `person.artwork.city`, tu dois recréer **d’abord** un nouvel objet `artwork`, puis un nouvel objet `person` :

```js
const nextArtwork = { ...person.artwork, city: 'New Delhi' };
const nextPerson = { ...person, artwork: nextArtwork };
setPerson(nextPerson);
```

Ou en version compacte :

```js
setPerson({
  ...person,
  artwork: {
    ...person.artwork,
    city: 'New Delhi'
  }
});
```

***

### 📝 Exemple : formulaire complet

```jsx
import { useState } from 'react';

export default function Form() {
  const [person, setPerson] = useState({
    name: 'Niki de Saint Phalle',
    artwork: {
      title: 'Blue Nana',
      city: 'Hamburg',
      image: 'https://i.imgur.com/Sd1AgUOm.jpg',
    }
  });

  function handleNameChange(e) {
    setPerson({
      ...person,
      name: e.target.value
    });
  }

  function handleTitleChange(e) {
    setPerson({
      ...person,
      artwork: {
        ...person.artwork,
        title: e.target.value
      }
    });
  }

  function handleCityChange(e) {
    setPerson({
      ...person,
      artwork: {
        ...person.artwork,
        city: e.target.value
      }
    });
  }

  function handleImageChange(e) {
    setPerson({
      ...person,
      artwork: {
        ...person.artwork,
        image: e.target.value
      }
    });
  }

  return (
    <>
      <label>
        Name:
        <input value={person.name} onChange={handleNameChange} />
      </label>
      <label>
        Title:
        <input value={person.artwork.title} onChange={handleTitleChange} />
      </label>
      <label>
        City:
        <input value={person.artwork.city} onChange={handleCityChange} />
      </label>
      <label>
        Image:
        <input value={person.artwork.image} onChange={handleImageChange} />
      </label>

      <p>
        <b>{person.name}</b>
        {' — '}
        <i>{person.artwork.title}</i>
        ({person.artwork.city})
      </p>
      <img src={person.artwork.image} alt={person.artwork.title} />
    </>
  );
}
```

👉 Chaque champ met à jour **une propriété précise** de `person.artwork` sans toucher aux autres.

***

### 💡 À retenir

* Les objets en JavaScript **ne sont pas réellement “imbriqués”** :\
  `artwork` est juste une référence à un autre objet.\
  Donc si deux personnes pointent vers le même objet et que tu le modifies, tu modifies les deux.
* C’est pour ça que React recommande toujours de **recréer une copie** de chaque objet concerné.

## 🔹 Mise à jour concise de l’état avec **Immer**

Quand ton état est **profondément imbriqué**, utiliser le spread `...` devient vite **verbeux** et répétitif :

```js
setPerson({
  ...person,
  artwork: {
    ...person.artwork,
    city: "New Delhi"
  }
});
```

👉 Avec **Immer**, tu peux écrire du code qui ressemble à une mutation directe (ex. `draft.artwork.city = 'Lagos'`), mais Immer se charge de **créer une copie immuable** et d’appliquer uniquement les changements.

***

### ✅ Exemple avec **use-immer**

#### Installation

```bash
npm install use-immer
```

#### Utilisation

```jsx
import { useImmer } from "use-immer";

export default function Form() {
  const [person, updatePerson] = useImmer({
    name: "Niki de Saint Phalle",
    artwork: {
      title: "Blue Nana",
      city: "Hamburg",
      image: "https://i.imgur.com/Sd1AgUOm.jpg",
    }
  });

  function handleNameChange(e) {
    updatePerson(draft => {
      draft.name = e.target.value;
    });
  }

  function handleCityChange(e) {
    updatePerson(draft => {
      draft.artwork.city = e.target.value;
    });
  }

  function handleTitleChange(e) {
    updatePerson(draft => {
      draft.artwork.title = e.target.value;
    });
  }

  return (
    <>
      <label>
        Name:
        <input value={person.name} onChange={handleNameChange} />
      </label>
      <label>
        City:
        <input value={person.artwork.city} onChange={handleCityChange} />
      </label>
      <label>
        Title:
        <input value={person.artwork.title} onChange={handleTitleChange} />
      </label>

      <p>
        <b>{person.name}</b> — <i>{person.artwork.title}</i> ({person.artwork.city})
      </p>
      <img src={person.artwork.image} alt={person.artwork.title} />
    </>
  );
}
```

👉 Ici, au lieu d’écrire de longs `...spread`, tu modifies directement `draft` comme si c’était mutable. Immer génère en interne une **copie immuable** avec les modifications.

***

### 🔍 Comment ça marche ?

* `draft` est un **Proxy** spécial créé par Immer.
* Tu peux modifier `draft` librement (`draft.name = "..."`, `draft.artwork.city = "..."`).
* Immer enregistre uniquement ce qui a changé.
* Il produit un **nouvel objet immuable** avec les modifications → parfait pour React.

***

### 💡 Pourquoi c’est recommandé ?

1. **Lisibilité** : le code est beaucoup plus court et clair.
2. **Pas de mutation réelle** : tu gardes tous les avantages de l’immutabilité (undo/redo, débogage clair, perf optimisées).
3. **Flexibilité** : tu peux mélanger `useState` et `useImmer` dans le même composant.

### 📝 Récapitulatif : Mise à jour d’objets dans l’état React

* ✅ Considère **tout état dans React comme immuable**.
* ⚠️ Si tu stockes des **objets** dans l’état et que tu les **mutations directement**, React :
  * ne déclenchera **aucun nouveau rendu**,
  * et tu modifieras aussi les « instantanés » des anciens rendus → ce qui casse le modèle mental de React.
* ✅ À la place, **crée une nouvelle version de l’objet** et passe-la au setter d’état (`setState`) → cela déclenche un **nouveau rendu**.
* ✅ Tu peux utiliser la **syntaxe spread** `{ ...obj, propriété: nouvelleValeur }` pour copier un objet et remplacer une propriété.
* ⚠️ Attention : le spread est **peu profond (shallow)** → il ne copie qu’un seul niveau.
* ✅ Pour mettre à jour un **objet imbriqué**, tu dois copier chaque niveau jusqu’à la racine.
* ✅ Pour éviter un code trop répétitif (beaucoup de `...spread`), tu peux utiliser **Immer** → qui permet d’écrire du code comme une mutation (`draft.x = ...`) mais qui génère automatiquement un nouvel objet immuable.

***

👉 En résumé :

* Pas de mutation directe,
* Toujours créer une copie,
* Spread pour les cas simples,
* Immer pour les objets imbriqués.
