# Choisir la structure de l’état

Bien structurer l’état (**state**) peut faire la différence entre :

* 🔹 un composant agréable à modifier et à déboguer
* 🔹 et un composant qui devient une source constante de bugs ⚠️

C’est pourquoi il est important de réfléchir à la manière dont on organise son état dans React.

***

### Dans cette section, tu apprendras :

* 🟢 Quand utiliser une **seule variable d’état** (state variable) vs **plusieurs**.
* 🟢 Ce qu’il faut **éviter** lorsque tu organises ton état.
* 🟢 Comment **corriger les problèmes courants** liés à une mauvaise structure de l’état.

***

👉 La règle d’or : **moins il y a de duplication dans ton état, mieux c’est**.\
Un état mal structuré peut introduire des paradoxes ou des incohérences, surtout quand certaines valeurs peuvent être déduites d’autres.

## Principes pour structurer l’état

Lorsque tu écris un composant qui contient un **state**, tu dois faire des choix :

* Combien de variables d’état utiliser ?
* Quelle forme (shape) doivent avoir ces données ?

Il est possible de faire fonctionner ton code même avec une structure d’état sous-optimale, mais tu risques d’introduire des incohérences et des bugs.\
Heureusement, il existe quelques **principes de base** pour t’aider à prendre les meilleures décisions.

***

### 🔹 1. Grouper les états liés

Si tu mets à jour **toujours deux ou plusieurs variables d’état en même temps**, c’est le signe qu’elles devraient probablement être regroupées dans **une seule variable d’état** (par exemple un objet).

***

### 🔹 2. Éviter les contradictions

Une mauvaise structure peut conduire à ce que deux morceaux de state se contredisent.\
👉 Exemple : `isTyping` et `isEmpty` qui pourraient être incohérents.\
Essaie d’organiser ton state pour que ce genre de contradiction ne soit pas possible.

***

### 🔹 3. Éviter l’état redondant

Si une information peut être **calculée à partir des props** ou de l’état existant **pendant le rendu**, elle **ne doit pas être stockée** dans le state.\
⚡ Exemple classique : stocker `fullName` alors qu’on a déjà `firstName` et `lastName`.

***

### 🔹 4. Éviter la duplication

Si la même donnée est **dupliquée** entre plusieurs variables d’état (ou dans des objets imbriqués), il sera difficile de les garder synchronisées.\
👉 Réduis la duplication autant que possible.

***

### 🔹 5. Éviter un état trop profondément imbriqué

Un état très **hiérarchique et profondément imbriqué** est pénible à mettre à jour (par exemple un objet contenant un tableau d’objets qui contiennent eux-mêmes d’autres objets).\
⚡ Préfère une structure **plus plate** quand c’est possible.

***

✅ L’objectif derrière ces principes est simple :\
👉 Avoir un état **facile à mettre à jour sans introduire d’erreurs**.

En supprimant les données **redondantes** et **dupliquées**, tu t’assures que toutes les parties du state restent synchronisées.\
C’est exactement comme en **conception de base de données**, où on “normalise” les tables pour réduire les incohérences.

💡 Pour paraphraser Einstein :\
&#xNAN;**“Fais ton state aussi simple que possible — mais pas plus simple.”**

## Regrouper les états liés

Parfois, tu n’es pas sûr s’il vaut mieux utiliser **plusieurs variables d’état** ou **une seule variable regroupée**.

Par exemple :

```jsx
// Option 1 : deux états séparés
const [x, setX] = useState(0);
const [y, setY] = useState(0);

// Option 2 : un seul état regroupé
const [position, setPosition] = useState({ x: 0, y: 0 });
```

👉 Techniquement, les deux approches sont possibles.\
Mais **si deux variables changent toujours ensemble**, il est préférable de les **unifier dans un seul objet**.\
Ainsi, tu ne risques pas d’oublier de les mettre à jour en même temps.

***

#### Exemple : un point rouge qui suit la souris

Ici, `position.x` et `position.y` sont regroupés dans un objet :

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
        setPosition({
          x: e.clientX,
          y: e.clientY
        });
      }}
      style={{
        position: 'relative',
        width: '100vw',
        height: '100vh',
      }}
    >
      <div
        style={{
          position: 'absolute',
          backgroundColor: 'red',
          borderRadius: '50%',
          transform: `translate(${position.x}px, ${position.y}px)`,
          left: -10,
          top: -10,
          width: 20,
          height: 20,
        }}
      />
    </div>
  );
}
```

Résultat 👉 le point rouge suit le curseur, car `x` et `y` changent toujours ensemble.

***

#### Quand regrouper dans un objet ou un tableau ?

Tu regrouperas les données dans un **objet** ou un **tableau** :

* Quand elles changent **ensemble** (comme `x` et `y`).
* Quand tu ne sais pas **combien d’éléments** tu auras (par ex. un formulaire où l’utilisateur peut ajouter des champs dynamiques).

***

⚠️ **Attention : piège courant**\
Si ton state est un **objet**, tu ne peux pas mettre à jour **un seul champ** sans recopier les autres.

❌ Exemple incorrect :

```jsx
setPosition({ x: 100 }); // y sera perdu !
```

✅ Correct :

```jsx
setPosition({ ...position, x: 100 });
```

Ou bien, si tu préfères, tu peux garder deux states séparés :

```jsx
setX(100);
```

***

👉 Résumé : **si tes données évoluent ensemble, regroupe-les**.\
Sinon, garde-les séparées pour éviter du code inutilement compliqué.

## ⚖️ Éviter les contradictions dans l’état

Prenons un formulaire de feedback pour un hôtel. Ici, le développeur utilise **deux états distincts** :

```jsx
const [isSending, setIsSending] = useState(false);
const [isSent, setIsSent] = useState(false);
```

👉 Problème :\
Ces deux variables peuvent **entrer en contradiction**.\
Par exemple, si tu oublies de bien synchroniser les deux setters, tu pourrais te retrouver avec :

* `isSending = true`
* **et** `isSent = true`

➡️ Ce qui n’a aucun sens : un message ne peut pas être **envoyé** et **en cours d’envoi** en même temps.

***

#### Exemple avec contradiction possible

```jsx
import { useState } from 'react';

export default function FeedbackForm() {
  const [text, setText] = useState('');
  const [isSending, setIsSending] = useState(false);
  const [isSent, setIsSent] = useState(false);

  async function handleSubmit(e) {
    e.preventDefault();
    setIsSending(true);
    await sendMessage(text);
    setIsSending(false);
    setIsSent(true);
  }

  if (isSent) {
    return <h1>Merci pour votre feedback !</h1>
  }

  return (
    <form onSubmit={handleSubmit}>
      <p>Comment s’est passé votre séjour au Poney Fringant ?</p>
      <textarea
        disabled={isSending}
        value={text}
        onChange={e => setText(e.target.value)}
      />
      <br />
      <button disabled={isSending} type="submit">
        Envoyer
      </button>
      {isSending && <p>Envoi en cours...</p>}
    </form>
  );
}

function sendMessage(text) {
  return new Promise(resolve => setTimeout(resolve, 2000));
}
```

⚠️ Ici, rien n’empêche une incohérence entre `isSending` et `isSent`.

***

#### ✅ Solution : unifier dans une seule variable `status`

Au lieu de deux états séparés, on utilise une seule variable **statut** qui ne peut prendre que des valeurs bien définies :

* `'typing'` (saisie en cours — état initial)
* `'sending'` (envoi en cours)
* `'sent'` (envoi terminé)

```jsx
import { useState } from 'react';

export default function FeedbackForm() {
  const [text, setText] = useState('');
  const [status, setStatus] = useState('typing'); // 'typing' | 'sending' | 'sent'

  async function handleSubmit(e) {
    e.preventDefault();
    setStatus('sending');
    await sendMessage(text);
    setStatus('sent');
  }

  const isSending = status === 'sending';
  const isSent = status === 'sent';

  if (isSent) {
    return <h1>Merci pour votre feedback !</h1>
  }

  return (
    <form onSubmit={handleSubmit}>
      <p>Comment s’est passé votre séjour au Poney Fringant ?</p>
      <textarea
        disabled={isSending}
        value={text}
        onChange={e => setText(e.target.value)}
      />
      <br />
      <button disabled={isSending} type="submit">
        Envoyer
      </button>
      {isSending && <p>Envoi en cours...</p>}
    </form>
  );
}

function sendMessage(text) {
  return new Promise(resolve => setTimeout(resolve, 2000));
}
```

***

#### ✅ Avantages de cette approche

* Pas de **paradoxes possibles** (un seul statut actif à la fois).
* Plus **lisible** : tu sais toujours dans quel état logique se trouve le composant.
* Tu peux même créer des constantes dérivées (`isSending`, `isSent`) pour plus de lisibilité, mais **elles ne sont pas dans le state**, donc pas de risque de désynchronisation.

***

👉 En résumé :

* Si deux variables d’état risquent de **se contredire**, regroupe-les en une seule **variable de statut**.
* Cela rend ton composant plus robuste et plus simple à maintenir.

## 🚫 Éviter l’état redondant

Un principe important : **si une donnée peut être calculée à partir des props ou d’un autre état déjà existant, elle ne doit pas être stockée dans le state**.

Sinon, tu risques :

* de dupliquer l’information,
* de devoir la mettre à jour à plusieurs endroits,
* et donc de créer des bugs si tu oublies une mise à jour.

***

#### Exemple avec état redondant ❌

Ici, le formulaire stocke **firstName**, **lastName**, et **fullName** :

```jsx
import { useState } from 'react';

export default function Form() {
  const [firstName, setFirstName] = useState('');
  const [lastName, setLastName] = useState('');
  const [fullName, setFullName] = useState(''); // ❌ Redondant

  function handleFirstNameChange(e) {
    setFirstName(e.target.value);
    setFullName(e.target.value + ' ' + lastName);
  }

  function handleLastNameChange(e) {
    setLastName(e.target.value);
    setFullName(firstName + ' ' + e.target.value);
  }

  return (
    <>
      <h2>Enregistrement</h2>
      <label>
        Prénom :{' '}
        <input
          value={firstName}
          onChange={handleFirstNameChange}
        />
      </label>
      <label>
        Nom :{' '}
        <input
          value={lastName}
          onChange={handleLastNameChange}
        />
      </label>
      <p>
        Billet émis pour : <b>{fullName}</b>
      </p>
    </>
  );
}
```

⚠️ Ici, `fullName` est **calculable** à partir de `firstName` et `lastName`, donc il est inutile de l’avoir dans le state.

***

#### ✅ Solution sans état redondant

On calcule `fullName` **au moment du rendu** :

```jsx
import { useState } from 'react';

export default function Form() {
  const [firstName, setFirstName] = useState('');
  const [lastName, setLastName] = useState('');

  const fullName = firstName + ' ' + lastName; // ✅ calculé à la volée

  return (
    <>
      <h2>Enregistrement</h2>
      <label>
        Prénom :{' '}
        <input
          value={firstName}
          onChange={e => setFirstName(e.target.value)}
        />
      </label>
      <label>
        Nom :{' '}
        <input
          value={lastName}
          onChange={e => setLastName(e.target.value)}
        />
      </label>
      <p>
        Billet émis pour : <b>{fullName}</b>
      </p>
    </>
  );
}
```

➡️ Quand `firstName` ou `lastName` change, React re-render, et `fullName` est automatiquement recalculé.\
Aucune duplication, aucun risque d’incohérence ✅.

***

### 🔎 Deep Dive : Ne pas “dupliquer” les props dans le state

Une erreur fréquente : recopier une prop dans un state interne :

```jsx
function Message({ messageColor }) {
  const [color, setColor] = useState(messageColor); // ❌
}
```

⚠️ Ici, si le parent change `messageColor`, le state `color` ne sera pas mis à jour.

✅ Solution : utiliser directement la prop, ou la stocker dans une constante :

```jsx
function Message({ messageColor }) {
  const color = messageColor; // ✅ pas de duplication
}
```

👉 On ne “duplique” une prop dans un state que si on veut **ignorer ses changements futurs**.\
Dans ce cas, par convention, on nomme la prop `initialSomething` ou `defaultSomething` :

```jsx
function Message({ initialColor }) {
  const [color, setColor] = useState(initialColor); 
  // Ici `color` ne suivra PAS les mises à jour de `initialColor`.
}
```

***

✅ **Règle d’or :**\
N’ajoute pas d’état si tu peux le calculer à partir d’un autre état ou d’une prop.

## 🚫 Éviter la duplication dans l’état

Un problème classique : **stocker deux fois la même information** dans des variables d’état différentes.\
Quand l’une change mais pas l’autre, cela crée des incohérences et des bugs.

***

#### Exemple ❌ avec duplication

Ici, le menu stocke :

* un tableau d’items (`items`)
* et l’item sélectionné complet (`selectedItem`)

```jsx
import { useState } from 'react';

const initialItems = [
  { title: 'Bretzels', id: 0 },
  { title: 'Algues croustillantes', id: 1 },
  { title: 'Barre de céréales', id: 2 },
];

export default function Menu() {
  const [items, setItems] = useState(initialItems);
  const [selectedItem, setSelectedItem] = useState(items[0]); // ❌ duplication

  return (
    <>
      <h2>Quel est ton snack de voyage ?</h2>
      <ul>
        {items.map(item => (
          <li key={item.id}>
            {item.title}{' '}
            <button onClick={() => setSelectedItem(item)}>
              Choisir
            </button>
          </li>
        ))}
      </ul>
      <p>Tu as choisi : {selectedItem.title}.</p>
    </>
  );
}
```

⚠️ Ici `selectedItem` est **le même objet** que dans `items`.\
Résultat → si tu modifies un titre dans `items`, `selectedItem` reste figé et ne se met pas à jour.

***

#### Exemple où le bug apparaît

Ajoutons la possibilité de modifier les titres :

```jsx
function handleItemChange(id, e) {
  setItems(items.map(item => {
    if (item.id === id) {
      return { ...item, title: e.target.value };
    } else {
      return item;
    }
  }));
}
```

👉 Si tu avais choisi un item, son titre en bas **ne se met pas à jour** car `selectedItem` n’a pas changé.

***

#### ✅ Solution : ne stocker que l’ID

Au lieu de stocker tout l’objet sélectionné, on stocke uniquement son **ID** (`selectedId`).\
Puis, pendant le rendu, on retrouve l’objet avec `find`.

```jsx
import { useState } from 'react';

const initialItems = [
  { title: 'Bretzels', id: 0 },
  { title: 'Algues croustillantes', id: 1 },
  { title: 'Barre de céréales', id: 2 },
];

export default function Menu() {
  const [items, setItems] = useState(initialItems);
  const [selectedId, setSelectedId] = useState(0); // ✅ seulement l’ID

  const selectedItem = items.find(item => item.id === selectedId);

  function handleItemChange(id, e) {
    setItems(items.map(item => {
      if (item.id === id) {
        return { ...item, title: e.target.value };
      } else {
        return item;
      }
    }));
  }

  return (
    <>
      <h2>Quel est ton snack de voyage ?</h2>
      <ul>
        {items.map(item => (
          <li key={item.id}>
            <input
              value={item.title}
              onChange={e => handleItemChange(item.id, e)}
            />
            {' '}
            <button onClick={() => setSelectedId(item.id)}>
              Choisir
            </button>
          </li>
        ))}
      </ul>
      <p>Tu as choisi : {selectedItem.title}.</p>
    </>
  );
}
```

***

#### 📝 Comparaison avant / après

Avant :

```js
items = [{ id: 0, title: 'Bretzels' }, ...]
selectedItem = { id: 0, title: 'Bretzels' } // ❌ doublon
```

Après :

```js
items = [{ id: 0, title: 'Bretzels' }, ...]
selectedId = 0 // ✅ pas de doublon
```

***

✅ Résultat :

* Plus aucune duplication.
* Si on modifie le titre d’un item, `selectedItem` est recalculé automatiquement.
* Le code est plus simple et plus fiable.

## ⚠️ Éviter l’état profondément imbriqué

Il est tentant de représenter ton état sous forme d’objets très imbriqués (par ex. une arborescence `planète → continent → pays`).\
Mais **mettre à jour un élément profondément enfoui oblige à recopier toute la chaîne de parents** → le code devient vite lourd et source d’erreurs.

***

### Exemple ❌ État imbriqué

Ici, chaque `place` contient directement ses enfants :

```jsx
export const initialTravelPlan = {
  id: 0,
  title: '(Racine)',
  childPlaces: [{
    id: 1,
    title: 'Terre',
    childPlaces: [{
      id: 2,
      title: 'Afrique',
      childPlaces: [
        { id: 3, title: 'Botswana', childPlaces: [] },
        { id: 4, title: 'Égypte', childPlaces: [] },
        { id: 5, title: 'Kenya', childPlaces: [] },
      ]
    }]
  }]
};
```

⚠️ Pour supprimer `Kenya`, il faudrait :

* créer une nouvelle version d’`Afrique` sans `Kenya`,
* recréer `Terre` avec la nouvelle `Afrique`,
* recréer la racine avec la nouvelle `Terre`.

C’est verbeux et fragile.

***

### ✅ Solution : état aplati (flat / normalized state)

On remplace les enfants imbriqués par **une table d’objets indexés par ID**.\
Chaque place ne stocke que ses `childIds`.

```jsx
export const initialTravelPlan = {
  0: { id: 0, title: '(Racine)', childIds: [1] },
  1: { id: 1, title: 'Terre', childIds: [2] },
  2: { id: 2, title: 'Afrique', childIds: [3, 4, 5] },
  3: { id: 3, title: 'Botswana', childIds: [] },
  4: { id: 4, title: 'Égypte', childIds: [] },
  5: { id: 5, title: 'Kenya', childIds: [] }
};
```

***

### Suppression d’un enfant (mise à jour plus simple)

```jsx
import { useState } from "react";
import { initialTravelPlan } from "./places.js";

export default function TravelPlan() {
  const [plan, setPlan] = useState(initialTravelPlan);

  function handleComplete(parentId, childId) {
    const parent = plan[parentId];

    // Nouveau parent sans l’enfant supprimé
    const nextParent = {
      ...parent,
      childIds: parent.childIds.filter(id => id !== childId)
    };

    // Mettre à jour uniquement ce qu’il faut
    setPlan({
      ...plan,
      [parentId]: nextParent
    });
  }

  const root = plan[0];
  return (
    <>
      <h2>Endroits à visiter</h2>
      <ol>
        {root.childIds.map(id => (
          <PlaceTree
            key={id}
            id={id}
            parentId={0}
            placesById={plan}
            onComplete={handleComplete}
          />
        ))}
      </ol>
    </>
  );
}
```

👉 Ici, seule la racine et le parent direct sont recréés → plus besoin de recopier toute la hiérarchie.

***

### 📝 Récapitulatif des principes

* Si deux états changent toujours ensemble → fusionne-les.
* Évite les états contradictoires (ex. deux booléens qui ne devraient pas être vrais en même temps).
* Ne stocke pas ce que tu peux calculer à partir d’autres états ou props.
* Supprime la duplication → stocke des **ID** plutôt que des objets complets.
* ⚠️ Si ton état est trop imbriqué → **applatis-le** en le normalisant (comme une base de données).
