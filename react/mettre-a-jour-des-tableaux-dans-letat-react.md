# 🔢 Mettre à jour des tableaux dans l’état React

En JavaScript, les **tableaux sont mutables** :

```js
const arr = [1, 2, 3];
arr.push(4); // ✅ autorisé en JS
```

👉 Mais en **React**, comme pour les objets, il faut les traiter comme **immuables**.\
Cela veut dire : **ne jamais modifier le tableau original** stocké dans l’état (`push`, `splice`, `sort`, …).\
À la place, on **crée un nouveau tableau** et on le passe au setter (`setState`).

***

#### ✨ Ce que tu vas apprendre

1. ➕ **Ajouter** des éléments dans un tableau d’état
2. ➖ **Supprimer** des éléments
3. ✏️ **Modifier** un élément existant
4. 🔄 **Mettre à jour un objet à l’intérieur d’un tableau**
5. ⚡ Utiliser **Immer** pour rendre le code plus concis

***

#### ⚠️ Pourquoi ?

* Comme pour les objets, si tu modifies directement un tableau existant, React **ne détectera pas le changement**, et donc **ne refera pas de rendu**.
* Créer un **nouveau tableau** permet à React de comparer (référence différente) et de déclencher un nouveau rendu.

***

👉 Exemple simple : ajouter un élément

```jsx
const [items, setItems] = useState([1, 2, 3]);

function addItem() {
  // ❌ Mauvais : items.push(4)
  // ✅ Bon : créer un nouveau tableau
  setItems([...items, 4]);
}
```

## 🔄 Mettre à jour les tableaux sans mutation

En JavaScript, les **tableaux** sont en réalité un type particulier d’**objet**. Comme pour les objets, en React il faut les considérer comme **lecture seule** (read-only).

👉 Cela signifie :

* ❌ ne pas réassigner directement un élément (`arr[0] = "bird"`)
* ❌ ne pas utiliser de méthodes mutables comme `push()` ou `pop()`

✅ Au lieu de ça, à chaque fois que tu veux modifier un tableau, tu dois **créer un nouveau tableau** et le passer à ta fonction de mise à jour d’état (`setState`).

Tu peux créer un nouveau tableau :

* en utilisant des méthodes **non mutantes** (`filter()`, `map()`, `slice()`, …)
* ou en utilisant l’opérateur **spread** (`[...arr, newItem]`).

***

### 📊 Tableau de référence des opérations sur les tableaux en React

| 🚫 À éviter (mutations)                  | ✅ À utiliser (nouveau tableau)        |
| ---------------------------------------- | ------------------------------------- |
| **Ajouter** : `push`, `unshift`          | `concat`, `[...]` (spread syntax)     |
| **Supprimer** : `pop`, `shift`, `splice` | `filter`, `slice`                     |
| **Remplacer** : `splice`, `arr[i] = ...` | `map`                                 |
| **Trier** : `reverse`, `sort`            | copier d’abord le tableau, puis trier |

***

#### ⚠️ Attention : `slice` vs `splice`

* `slice` → **ne modifie pas** le tableau → retourne une copie (✅ recommandé en React).
* `splice` → **modifie** le tableau existant (❌ à éviter en React).

***

👉 Alternative :\
Tu peux utiliser **Immer** qui te permet d’écrire du code comme si tu faisais des mutations (`push`, `pop`, etc.), mais en réalité Immer crée un **nouveau tableau immuable** en arrière-plan.

## ➕ Ajouter un élément à un tableau en React

En **JavaScript classique**, on ferait :

```js
artists.push({ id: nextId++, name: name });
```

👉 Mais `push()` **modifie** (mute) directement le tableau existant. En React, c’est à éviter ❌ car React ne saura pas qu’il doit re-render ton composant.

***

### ✅ La bonne approche : créer un **nouveau tableau**

Avec l’opérateur **spread (`...`)**, tu peux recopier l’ancien tableau et y ajouter le nouvel élément.

#### Ajouter à la fin (équivalent de `push()`)

```js
setArtists([
  ...artists, // copie les anciens
  { id: nextId++, name: name } // ajoute le nouveau
]);
```

***

#### Ajouter au début (équivalent de `unshift()`)

```js
setArtists([
  { id: nextId++, name: name }, // ajoute en premier
  ...artists // puis les anciens
]);
```

***

### Exemple complet

```jsx
import { useState } from 'react';

let nextId = 0;

export default function List() {
  const [name, setName] = useState('');
  const [artists, setArtists] = useState([]);

  return (
    <>
      <h1>Inspiring sculptors:</h1>
      <input
        value={name}
        onChange={e => setName(e.target.value)}
      />
      <button onClick={() => {
        setArtists([
          ...artists,
          { id: nextId++, name: name }
        ]);
      }}>
        Add
      </button>
      <ul>
        {artists.map(artist => (
          <li key={artist.id}>{artist.name}</li>
        ))}
      </ul>
    </>
  );
}
```

***

✅ Résultat :

* À chaque clic sur **Add**, un nouvel artiste est ajouté.
* React voit qu’un **nouveau tableau** a été créé → déclenche un re-render.

## 🗑️ Supprimer un élément d’un tableau en React

En **JavaScript classique**, on aurait pu utiliser `splice()` :

```js
artists.splice(index, 1);
```

👉 Mais `splice()` **mute le tableau existant** ❌ → pas bon pour React.

***

### ✅ La bonne approche : utiliser `filter()`

`filter()` crée un **nouveau tableau** contenant uniquement les éléments qui respectent la condition.

Exemple :

```jsx
setArtists(
  artists.filter(a => a.id !== artist.id)
);
```

👉 Ici :

* `a.id !== artist.id` → garde tous les artistes sauf celui qu’on veut supprimer.
* Comme un **nouveau tableau** est créé, React détecte le changement et déclenche un re-render.

***

### Exemple complet

```jsx
import { useState } from 'react';

let initialArtists = [
  { id: 0, name: 'Marta Colvin Andrade' },
  { id: 1, name: 'Lamidi Olonade Fakeye' },
  { id: 2, name: 'Louise Nevelson' },
];

export default function List() {
  const [artists, setArtists] = useState(initialArtists);

  return (
    <>
      <h1>Inspiring sculptors:</h1>
      <ul>
        {artists.map(artist => (
          <li key={artist.id}>
            {artist.name}{' '}
            <button onClick={() => {
              setArtists(
                artists.filter(a => a.id !== artist.id)
              );
            }}>
              Delete
            </button>
          </li>
        ))}
      </ul>
    </>
  );
}
```

***

✅ Résultat :

* Chaque bouton **Delete** supprime uniquement l’artiste correspondant.
* `filter()` ne touche pas au tableau original → React peut re-render proprement.

***

⚡ Astuce :

* `filter()` = **garder ce qui est vrai**.
* Donc pour supprimer un élément → on filtre en gardant tout **sauf** lui.

## 🔄 Transformer un tableau avec `map()`

👉 Rappel :

* `map()` crée un **nouveau tableau** de la même taille.
* Chaque élément peut être **transformé** ou **laissé tel quel**.
* Tu peux décider quoi faire en fonction de ses données (`shape.type`, `shape.id`, etc.).

***

### Exemple complet : déplacer uniquement les cercles ⭕⬇️

```jsx
import { useState } from 'react';

let initialShapes = [
  { id: 0, type: 'circle', x: 50, y: 100 },
  { id: 1, type: 'square', x: 150, y: 100 },
  { id: 2, type: 'circle', x: 250, y: 100 },
];

export default function ShapeEditor() {
  const [shapes, setShapes] = useState(initialShapes);

  function handleClick() {
    const nextShapes = shapes.map(shape => {
      if (shape.type === 'square') {
        // On ne change pas les carrés
        return shape;
      } else {
        // On retourne un **nouvel objet**
        return {
          ...shape,      // copie toutes les propriétés
          y: shape.y + 50 // mais décale les cercles de 50px
        };
      }
    });
    setShapes(nextShapes); // nouveau tableau → re-render
  }

  return (
    <>
      <button onClick={handleClick}>
        Move circles down!
      </button>
      {shapes.map(shape => (
        <div
          key={shape.id}
          style={{
            position: 'absolute',
            left: shape.x,
            top: shape.y,
            width: 50,
            height: 50,
            borderRadius: shape.type === 'circle' ? '50%' : '0',
            background: 'lightblue',
          }}
        />
      ))}
    </>
  );
}
```

***

### 🚀 Points clés

1. **Immutabilité respectée** → on crée un nouveau tableau avec `map()`.
2. **Éléments non concernés** → renvoyés tels quels (`return shape`).
3. **Éléments à modifier** → recréés avec `...spread` + propriétés modifiées.

***

✅ Résultat :

* Les cercles descendent de 50px à chaque clic.
* Les carrés restent à leur place.

## 🔄 1. Remplacer un élément avec `map()`

👉 Mauvaise pratique :

```js
arr[0] = "bird" // ❌ mutation → React ne détecte rien
```

👉 Bonne pratique avec **`map()`** :

* `map()` parcourt le tableau.
* Tu renvoies **l’élément inchangé** ou **une nouvelle valeur** si l’index correspond.

#### Exemple : incrémenter seulement le compteur cliqué ✅

```jsx
import { useState } from 'react';

let initialCounters = [0, 0, 0];

export default function CounterList() {
  const [counters, setCounters] = useState(initialCounters);

  function handleIncrementClick(index) {
    const nextCounters = counters.map((c, i) => {
      if (i === index) {
        return c + 1; // remplace uniquement ce compteur
      } else {
        return c;     // les autres restent identiques
      }
    });
    setCounters(nextCounters); // nouveau tableau → re-render
  }

  return (
    <ul>
      {counters.map((counter, i) => (
        <li key={i}>
          {counter}{' '}
          <button onClick={() => handleIncrementClick(i)}>+1</button>
        </li>
      ))}
    </ul>
  );
}
```

✅ Chaque clic incrémente uniquement le compteur concerné.\
⚡ L’immutabilité est respectée car on génère un **nouveau tableau**.

***

## ➕ 2. Insérer un élément à une position précise

👉 Avec `slice()` et l’opérateur `...spread` :

* `slice(0, insertAt)` = partie avant.
* `slice(insertAt)` = partie après.
* On insère le nouvel élément **entre les deux**.

#### Exemple : insérer un artiste à l’index `1`

```jsx
import { useState } from 'react';

let nextId = 3;
const initialArtists = [
  { id: 0, name: 'Marta Colvin Andrade' },
  { id: 1, name: 'Lamidi Olonade Fakeye'},
  { id: 2, name: 'Louise Nevelson'},
];

export default function List() {
  const [name, setName] = useState('');
  const [artists, setArtists] = useState(initialArtists);

  function handleClick() {
    const insertAt = 1; // position d’insertion
    const nextArtists = [
      ...artists.slice(0, insertAt),       // avant
      { id: nextId++, name: name },        // nouvel élément
      ...artists.slice(insertAt)           // après
    ];
    setArtists(nextArtists);
    setName('');
  }

  return (
    <>
      <h1>Inspiring sculptors:</h1>
      <input
        value={name}
        onChange={e => setName(e.target.value)}
      />
      <button onClick={handleClick}>Insert at 1</button>
      <ul>
        {artists.map(artist => (
          <li key={artist.id}>{artist.name}</li>
        ))}
      </ul>
    </>
  );
}
```

✅ Résultat : le nouvel artiste est inséré **entre le 1er et le 2e** élément du tableau.

***

## 📝 À retenir

* `map()` → idéal pour **remplacer** un ou plusieurs éléments.
* `slice()` + `...spread` → parfait pour **insérer** à une position donnée.
* Toujours **créer un nouveau tableau** (immutabilité respectée).

En React, pour respecter l’**immutabilité**, il faut d’abord **copier le tableau** avant de le modifier.

***

## ✅ Exemple correct avec `reverse()`

```jsx
import { useState } from 'react';

const initialList = [
  { id: 0, title: 'Big Bellies' },
  { id: 1, title: 'Lunar Landscape' },
  { id: 2, title: 'Terracotta Army' },
];

export default function List() {
  const [list, setList] = useState(initialList);

  function handleClick() {
    // copie du tableau avec spread
    const nextList = [...list];
    // on peut maintenant utiliser une méthode mutante
    nextList.reverse();
    // mise à jour de l’état avec le nouveau tableau
    setList(nextList);
  }

  return (
    <>
      <button onClick={handleClick}>Reverse</button>
      <ul>
        {list.map(artwork => (
          <li key={artwork.id}>{artwork.title}</li>
        ))}
      </ul>
    </>
  );
}
```

👉 Ici, `list` reste intact, et seul `nextList` est modifié.

***

## ⚠️ Problème des objets partagés

Le spread `[...]` fait une **copie superficielle** (_shallow copy_).\
Ça veut dire que le nouveau tableau est bien un nouvel objet, mais ses éléments **pointent toujours vers les mêmes objets internes**.

Exemple problématique :

```js
const nextList = [...list];
nextList[0].seen = true; // ❌ mutation !
setList(nextList);
```

Même si `nextList` est un nouveau tableau, `nextList[0]` et `list[0]` **pointent vers le même objet** → tu viens de muter l’ancien état.

***

## ✅ Solution : copier aussi les éléments que tu modifies

Pour mettre à jour **un objet dans un tableau**, il faut créer une nouvelle version de cet objet avec le spread :

```jsx
function handleMarkAsSeen(id) {
  setList(
    list.map(artwork => {
      if (artwork.id === id) {
        // retourne une copie mise à jour
        return { ...artwork, seen: true };
      } else {
        // garde l’original inchangé
        return artwork;
      }
    })
  );
}
```

👉 Ici :

* `list.map()` génère un nouveau tableau.
* Seul l’objet ciblé (`artwork`) est copié et modifié.
* Les autres restent identiques (et réutilisés).

***

## 📝 À retenir

* 🔄 `reverse()`, `sort()`, `splice()`, etc. → à utiliser seulement sur une **copie** (`[...array]` ou `array.slice()`).
* 📦 Spread `[...list]` = copie superficielle → attention aux objets internes !
* 🎯 Pour modifier un objet d’un tableau → utiliser `map()` et retourner une **nouvelle copie de l’objet ciblé**.

En React :

* Les **tableaux** doivent être traités comme immuables.
* Mais leurs **éléments** (les objets qu’ils contiennent) doivent l’être aussi !

***

### ⚠️ Le problème classique

Dans ton premier exemple, tu fais ceci :

```js
const myNextList = [...myList]; // nouvelle copie du tableau
const artwork = myNextList.find(a => a.id === artworkId);
artwork.seen = nextSeen; // ❌ mutation d’un objet existant
setMyList(myNextList);
```

👉 Ici, tu as bien copié le tableau (`[...myList]`), mais **pas ses éléments**.\
Comme `artwork` pointe vers **le même objet** qui existait déjà, tu modifies aussi :

* `myList` original,
* et même `yourList` si celui-ci contient la même référence.

Résultat : **les deux listes partagent l’état** → un bug invisible et très difficile à déboguer.

***

### ✅ La solution : recréer une nouvelle version de l’objet

Pour corriger ça, tu dois remplacer **l’objet entier** avec un nouveau :

```js
setMyList(
  myList.map(artwork => {
    if (artwork.id === artworkId) {
      // retourne une nouvelle copie avec la modif
      return { ...artwork, seen: nextSeen };
    } else {
      // sinon garde l’original
      return artwork;
    }
  })
);
```

👉 Ici :

* `map()` crée un nouveau tableau.
* L’objet ciblé est cloné (`{ ...artwork, seen: nextSeen }`).
* Les autres objets ne changent pas, donc React ne va pas les re-render inutilement.

***

### 📝 Règles pratiques à retenir

* 🔄 **Copie toujours à partir du point où tu veux modifier, jusqu’en haut.**\
  (objet → tableau → état)
* 📦 Tu peux **muter uniquement les objets que tu viens de créer** (ex : lors d’un `push` dans un nouveau tableau).
* 🚫 Ne **jamais muter un objet déjà en state** → ça casse l’immutabilité, et donc la logique de React.
* ⚡ Si tu as des objets très imbriqués → **Immer** simplifie énormément le code.

## ✍️ Écrire une logique de mise à jour concise avec Immer

Mettre à jour des tableaux imbriqués **sans mutation** peut vite devenir répétitif.\
Comme pour les objets :

* En général, vous ne devriez pas avoir besoin de mettre à jour l’état à plus de deux ou trois niveaux de profondeur.
* Si vos objets d’état sont **trop profondément imbriqués**, il vaut mieux les **restructurer** pour les aplatir.
* Mais si vous ne voulez pas changer la structure de votre état, vous pouvez utiliser **Immer**, qui vous permet d’écrire du code avec une syntaxe pratique (comme si vous mutiez directement), tout en gérant pour vous la création des copies immuables.

***

### 🎨 Exemple : _Liste d’œuvres à voir_ avec Immer

#### 📦 `package.json`

```json
{
  "dependencies": {
    "immer": "1.7.3",
    "react": "latest",
    "react-dom": "latest",
    "react-scripts": "latest",
    "use-immer": "0.5.1"
  },
  "scripts": {
    "start": "react-scripts start",
    "build": "react-scripts build",
    "test": "react-scripts test --env=jsdom",
    "eject": "react-scripts eject"
  },
  "devDependencies": {}
}
```

***

#### 📝 `App.js` (exemple avec Immer)

```js
import { useImmer } from 'use-immer';

export default function BucketList() {
  const [maListe, mettreAJourMaListe] = useImmer([
    { id: 0, titre: 'Grosses ventres', vu: false },
    { id: 1, titre: 'Paysage lunaire', vu: false },
    { id: 2, titre: 'Armée de terre cuite', vu: true },
  ]);

  function basculerEtat(artworkId, prochainVu) {
    mettreAJourMaListe(draft => {
      const oeuvre = draft.find(o => o.id === artworkId);
      oeuvre.vu = prochainVu; // ✅ autorisé avec Immer
    });
  }

  return (
    <ul>
      {maListe.map(oeuvre => (
        <li key={oeuvre.id}>
          <label>
            <input
              type="checkbox"
              checked={oeuvre.vu}
              onChange={e => basculerEtat(oeuvre.id, e.target.checked)}
            />
            {oeuvre.titre}
          </label>
        </li>
      ))}
    </ul>
  );
}
```

***

### ✅ Pourquoi `oeuvre.vu = prochainVu` est permis ici ?

Parce que vous **ne mutiez pas l’état original** !\
Immer vous fournit un **draft** (une version provisoire de l’état sous forme de Proxy).

* Vous pouvez modifier ce draft librement (assignations, `push()`, `pop()`, etc.).
* En arrière-plan, **Immer reconstruit un nouvel état immuable** basé sur vos changements.

👉 Résultat : un code **clair, concis et lisible**, sans casser les règles d’immutabilité de React.

***

## 🔄 Récapitulatif

* Vous pouvez stocker des **tableaux** dans l’état, mais vous ne devez pas les **muter directement**.
* Au lieu de muter un tableau, créez **une nouvelle version** et passez-la à la fonction de mise à jour d’état.
* Utilisez :
  * `[...]` (spread syntax) pour ajouter des éléments.
  * `filter()` et `map()` pour filtrer ou transformer un tableau.
* Utilisez **Immer** pour éviter du code répétitif quand vos objets ou tableaux sont imbriqués.

