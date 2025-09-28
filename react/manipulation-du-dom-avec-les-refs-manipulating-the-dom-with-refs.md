# Manipulation du DOM avec les Refs (Manipulating the DOM with Refs)

En React, **c’est React qui contrôle le DOM** : tu décris ce que tu veux voir, et React synchronise automatiquement le DOM avec ton rendu. 👉 Donc en principe, **tu n’as pas besoin de toucher directement au DOM**.

Mais parfois, tu dois **sortir de React** pour accéder au DOM : donner le focus à un input, faire défiler une zone, mesurer une taille, lancer une animation, etc. Et là, les **refs DOM** deviennent utiles.

***

## 🎯 Ce que tu vas apprendre

1. **Comment accéder à un nœud DOM géré par React avec l’attribut `ref`**
2. **Comment l’attribut JSX `ref` est lié au Hook `useRef`**
3. **Comment accéder au DOM d’un autre composant**
4. **Dans quels cas il est sûr de modifier le DOM géré par React**

***

## 1️⃣ Accéder à un élément DOM avec `ref`

👉 Exemple : mettre le focus sur un champ texte quand on clique sur un bouton.

```jsx
import { useRef } from 'react';

export default function Form() {
  const inputRef = useRef(null);

  function handleClick() {
    inputRef.current.focus(); // accès direct au DOM
  }

  return (
    <>
      <input ref={inputRef} />
      <button onClick={handleClick}>
        Focus sur l’input
      </button>
    </>
  );
}
```

* Ici, `ref={inputRef}` demande à React :\
  &#xNAN;_« Mets l’élément DOM \<input> dans `inputRef.current` »_.
* Ensuite, on peut appeler les méthodes natives du DOM (`focus`, `scrollIntoView`, `getBoundingClientRect`, etc.).

***

## 2️⃣ Comment `ref` est lié à `useRef`

* `useRef(initialValue)` crée un **objet ref** : `{ current: initialValue }`.
* L’attribut JSX `ref={...}` permet de **lier cet objet à un nœud DOM**.
* Quand React **monte** l’élément → il remplit `ref.current` avec le nœud DOM réel.
* Quand l’élément est **supprimé** du DOM → React remet `ref.current = null`.

***

## 3️⃣ Accéder au DOM d’un autre composant

Tu ne peux pas directement faire :

```jsx
<MyComponent ref={...} />
```

Car par défaut, un `ref` pointe seulement vers un **élément DOM**, pas un composant.

👉 Pour exposer un nœud DOM depuis un composant personnalisé, tu dois utiliser **`forwardRef`** :

```jsx
import { forwardRef, useRef } from 'react';

const CustomInput = forwardRef((props, ref) => {
  return <input {...props} ref={ref} />;
});

export default function App() {
  const inputRef = useRef(null);

  return (
    <>
      <CustomInput ref={inputRef} />
      <button onClick={() => inputRef.current.focus()}>
        Focus custom input
      </button>
    </>
  );
}
```

Ici, `CustomInput` "transmet" la ref jusqu’à son `<input>` interne.

***

## 4️⃣ Quand est-ce sûr de modifier le DOM soi-même ?

⚠️ React veut **gérer le DOM pour toi** → si tu modifies un élément que React contrôle (par ex. `innerHTML`, `value` d’un input) tu risques des incohérences.

✅ Bon usage :

* donner le focus (`element.focus()`),
* défiler (`element.scrollIntoView()`),
* lire des tailles (`element.offsetWidth`),
* lancer une animation (`element.animate(...)`).

❌ Mauvais usage :

* changer du texte avec `element.innerText = ...`,
* injecter du HTML avec `element.innerHTML = ...`,
* modifier directement des attributs que React gère déjà.

***

## 🔑 Récap

* `useRef` + `ref={...}` = moyen officiel d’accéder aux nœuds DOM créés par React.
* `ref.current` = le nœud DOM réel.
* Pour exposer le DOM d’un composant perso → `forwardRef`.
* Tu peux lire ou appeler des méthodes DOM sans danger.
* Mais **évite de modifier** ce que React contrôle déjà (contenu, attributs).

## 🔎 Obtenir une référence (`ref`) vers un nœud DOM

Pour accéder à un nœud DOM géré par React, on utilise le Hook `useRef`.

***

### Étapes

1. **Importer `useRef`** :

```jsx
import { useRef } from 'react';
```

2. **Déclarer une ref** dans ton composant avec une valeur initiale :

```jsx
const myRef = useRef(null);
```

3. **Passer la ref à un élément JSX** via l’attribut `ref` :

```jsx
<div ref={myRef}></div>
```

* `useRef` retourne un objet de la forme :

```js
{ 
  current: null // au départ
}
```

* Quand React crée le DOM pour `<div>`, il met une **référence directe au nœud DOM** dans `myRef.current`.

👉 Tu peux ensuite utiliser les **APIs natives du navigateur** dessus :

```js
myRef.current.scrollIntoView();
```

***

### Exemple 1 : Focus sur un champ texte

```jsx
import { useRef } from 'react';

export default function Form() {
  const inputRef = useRef(null);

  function handleClick() {
    inputRef.current.focus(); // donne le focus au champ
  }

  return (
    <>
      <input ref={inputRef} />
      <button onClick={handleClick}>
        Mettre le focus
      </button>
    </>
  );
}
```

#### Comment ça marche :

1. On déclare `inputRef` avec `useRef`.
2. On passe `ref={inputRef}` à l’élément `<input>`.
3. React met **le nœud DOM de l’input** dans `inputRef.current`.
4. Dans `handleClick`, on appelle `inputRef.current.focus()`.

***

### Exemple 2 : Faire défiler jusqu’à un élément (`scrollIntoView`)

Ici, on a un carrousel de 3 images. Chaque bouton défile jusqu’à une image donnée :

```jsx
import { useRef } from 'react';

export default function Gallery() {
  const firstCatRef = useRef(null);
  const secondCatRef = useRef(null);
  const thirdCatRef = useRef(null);

  function handleScrollToFirstCat() {
    firstCatRef.current.scrollIntoView({ behavior: 'smooth', block: 'nearest', inline: 'center' });
  }

  function handleScrollToSecondCat() {
    secondCatRef.current.scrollIntoView({ behavior: 'smooth', block: 'nearest', inline: 'center' });
  }

  function handleScrollToThirdCat() {
    thirdCatRef.current.scrollIntoView({ behavior: 'smooth', block: 'nearest', inline: 'center' });
  }

  return (
    <>
      <nav>
        <button onClick={handleScrollToFirstCat}>Neo</button>
        <button onClick={handleScrollToSecondCat}>Millie</button>
        <button onClick={handleScrollToThirdCat}>Bella</button>
      </nav>
      <div>
        <ul>
          <li>
            <img src="https://placecats.com/neo/300/200" alt="Neo" ref={firstCatRef} />
          </li>
          <li>
            <img src="https://placecats.com/millie/200/200" alt="Millie" ref={secondCatRef} />
          </li>
          <li>
            <img src="https://placecats.com/bella/199/200" alt="Bella" ref={thirdCatRef} />
          </li>
        </ul>
      </div>
    </>
  );
}
```

***

### 🧩 Cas avancé : liste dynamique avec callback refs

Si tu as une **liste d’éléments dont tu ne connais pas la taille**, tu ne peux pas écrire :

```jsx
{items.map(item => {
  const ref = useRef(null); // ❌ interdit dans une boucle !
  return <li ref={ref}>{item.name}</li>;
})}
```

👉 La solution : utiliser une **ref callback** qui sera appelée par React quand le nœud est monté/démonté :

```jsx
<li
  key={cat.id}
  ref={(node) => {
    const map = getMap();
    if (node) {
      map.set(cat.id, node);    // on ajoute le nœud
    } else {
      map.delete(cat.id);       // on nettoie quand démonté
    }
  }}
>
  <img src={cat.imageUrl} alt={cat.name} />
</li>
```

Avec ça, tu peux gérer une **Map de refs** et accéder à n’importe quel élément.

***

### 🔑 Récap

* `useRef(null)` crée un objet `{ current: null }`.
* `ref={myRef}` dans JSX = associer le nœud DOM à `myRef.current`.
* Tu peux manipuler le DOM avec les APIs natives (`focus`, `scrollIntoView`, etc.).
* Plusieurs refs → plusieurs éléments.
* Pour une liste dynamique → **callback refs**.

## 📌 Accéder aux nœuds DOM d’un autre composant avec les refs

⚠️ **Attention** : Les refs sont une _échappatoire_ (“escape hatch”).\
Manipuler directement le DOM d’un autre composant peut rendre ton code fragile et difficile à maintenir. Mais dans certains cas particuliers (focus, scroll, animation, etc.), c’est utile.

***

### 1. Passer une ref du parent à l’enfant

Tu peux passer une ref comme **prop** d’un composant parent vers un composant enfant. L’enfant transmet ensuite cette ref à un élément DOM natif (`<input>`, `<div>`, etc.).

#### Exemple :

```jsx
import { useRef } from 'react';

function MyInput({ inputRef }) {
  return <input ref={inputRef} />;
}

export default function MyForm() {
  const inputRef = useRef(null);

  function handleClick() {
    inputRef.current.focus(); // focus sur l’input de l’enfant
  }

  return (
    <>
      <MyInput inputRef={inputRef} />
      <button onClick={handleClick}>
        Focus sur le champ
      </button>
    </>
  );
}
```

✅ Ici :

* `MyForm` crée une ref `inputRef`.
* Elle est transmise à `MyInput` via la prop `inputRef`.
* `MyInput` attache cette ref au DOM `<input>`.
* Depuis `MyForm`, on peut appeler `inputRef.current.focus()`.

***

### 2. ⚡ Utiliser `forwardRef`

Par convention, React recommande d’utiliser `forwardRef` plutôt que de donner une prop spéciale `inputRef`. Cela permet de passer directement `ref` au composant enfant.

#### Exemple avec `forwardRef` :

```jsx
import { useRef, forwardRef } from 'react';

const MyInput = forwardRef((props, ref) => {
  return <input ref={ref} {...props} />;
});

export default function MyForm() {
  const inputRef = useRef(null);

  function handleClick() {
    inputRef.current.focus();
  }

  return (
    <>
      <MyInput ref={inputRef} />
      <button onClick={handleClick}>Focus</button>
    </>
  );
}
```

👉 Ici, `MyInput` utilise `forwardRef`, ce qui permet au parent de passer directement `ref` comme si c’était un élément natif.

***

### 3. 🔒 Restreindre l’API exposée avec `useImperativeHandle`

Par défaut, quand tu passes une ref à un composant, le parent a accès **à tout le nœud DOM**.\
Mais parfois, tu veux limiter ce que le parent peut faire (par ex. exposer seulement `focus()`, mais pas `.style` ou `.value`).

#### Exemple :

```jsx
import { useRef, forwardRef, useImperativeHandle } from "react";

const MyInput = forwardRef((props, ref) => {
  const realInputRef = useRef(null);

  useImperativeHandle(ref, () => ({
    focus() {
      realInputRef.current.focus(); // on expose uniquement focus()
    },
  }));

  return <input ref={realInputRef} {...props} />;
});

export default function MyForm() {
  const inputRef = useRef(null);

  function handleClick() {
    inputRef.current.focus(); // ici, seul "focus" est dispo
  }

  return (
    <>
      <MyInput ref={inputRef} />
      <button onClick={handleClick}>Focus</button>
    </>
  );
}
```

✅ Ici :

* `realInputRef` garde la vraie référence DOM.
* `useImperativeHandle(ref, () => ({ focus() {...} }))` expose seulement une API restreinte.
* Depuis le parent, `inputRef.current` **ne contient que** `focus()`.

***

### 🔑 Récap

* **Refs dans un parent** peuvent être passées à un enfant pour accéder à un élément DOM.
* **forwardRef** est la manière idiomatique de faire ça en React.
* **useImperativeHandle** permet de contrôler ce que le parent peut faire avec la ref.
* Utilise cette technique uniquement si nécessaire → sinon, préfère les props et la gestion déclarative.

## Accéder aux nœuds DOM d’un autre composant avec les refs

⚠️ **Attention** : Les refs sont une **échappatoire** (« escape hatch »). Manipuler manuellement le DOM d’un autre composant peut rendre ton code fragile. Il faut donc les utiliser uniquement quand c’est nécessaire (focus d’un champ, lecture de dimensions, scroll automatique, etc.).

***

### Exemple simple : transmettre une ref du parent à l’enfant

En React, la prop `ref` n’est **pas un prop comme les autres**. Pour qu’un composant personnalisé accepte une ref, il faut utiliser **`forwardRef`**.

```jsx
import { useRef, forwardRef } from 'react';

// forwardRef permet à l’enfant d’accepter la ref du parent
const MyInput = forwardRef((props, ref) => {
  return <input ref={ref} {...props} />;
});

export default function MyForm() {
  const inputRef = useRef(null);

  function handleClick() {
    inputRef.current.focus(); // on accède directement à <input>
  }

  return (
    <>
      <MyInput ref={inputRef} />
      <button onClick={handleClick}>
        Mettre le focus sur l’input
      </button>
    </>
  );
}
```

👉 Ici :

* `MyForm` crée une ref (`inputRef`).
* `forwardRef` permet à `MyInput` de la transmettre à son `<input>`.
* Le bouton du parent peut appeler `focus()` directement.

***

### Exemple avancé : limiter ce que le parent peut faire

Parfois, tu ne veux pas que le parent puisse modifier complètement le DOM de ton composant enfant. Tu peux **exposer uniquement certaines méthodes** grâce à **`useImperativeHandle`**.

```jsx
import { useRef, forwardRef, useImperativeHandle } from "react";

const MyInput = forwardRef((props, ref) => {
  const realInputRef = useRef(null);

  // On limite ce que le parent peut utiliser
  useImperativeHandle(ref, () => ({
    focus() {
      realInputRef.current.focus();
    },
  }));

  return <input ref={realInputRef} {...props} />;
});

export default function Form() {
  const inputRef = useRef(null);

  function handleClick() {
    inputRef.current.focus(); // fonctionne car "focus" est exposé
  }

  return (
    <>
      <MyInput ref={inputRef} />
      <button onClick={handleClick}>
        Mettre le focus sur l’input
      </button>
    </>
  );
}
```

👉 Ici :

* `realInputRef` pointe sur le vrai `<input>`.
* `useImperativeHandle` expose uniquement la méthode `focus`.
* Le parent **ne peut pas** modifier `.style` ou `.value`, seulement utiliser `focus()`.

***

### ✅ Récapitulatif

* `ref` **n’est pas** une prop classique → il faut **`forwardRef`** pour l’utiliser avec un composant enfant.
* `useRef` → permet de pointer sur un élément DOM.
* `useImperativeHandle` → limite l’API exposée au parent (plus sûr).
* ⚠️ Les refs doivent rester un outil exceptionnel, car elles contournent le flux normal de données React (props → rendu).

## Quand React attache les refs

En React, chaque mise à jour se déroule en **deux phases** :

1. **Phase de rendu (render)** → React appelle tes composants pour déterminer _quoi_ doit être affiché.
2. **Phase de commit (commit)** → React applique effectivement les changements dans le **DOM**.

👉 Il est **trop tôt pour accéder à une ref pendant le rendu** :

* Lors du **premier rendu**, le DOM n’existe pas encore → `ref.current` vaut `null`.
* Lors des mises à jour, le DOM n’a pas encore été modifié → tu ne peux pas encore lire les nouvelles valeurs.

✅ React met à jour les refs pendant la **phase de commit** :

* Avant de modifier le DOM → React met les `ref.current` concernés à `null`.
* Après avoir modifié le DOM → React assigne les bons nœuds DOM dans `ref.current`.

En pratique :

* Tu utilises les **refs dans les gestionnaires d’événements** (ex. clic → focus).
* Si tu dois agir _juste après un rendu_ sans événement, utilise un **Hook d’effet (`useEffect`)**.

***

### ⚡ Exemple : problème classique sans `flushSync`

Ici, on ajoute un élément à une liste et on scrolle jusqu’au dernier élément :

```jsx
import { useState, useRef } from 'react';

export default function TodoList() {
  const listRef = useRef(null);
  const [text, setText] = useState('');
  const [todos, setTodos] = useState(initialTodos);

  function handleAdd() {
    const newTodo = { id: nextId++, text: text };
    setText('');
    setTodos([...todos, newTodo]);

    // ❌ Mauvais : la liste n’a pas encore été mise à jour dans le DOM
    listRef.current.lastChild.scrollIntoView({
      behavior: 'smooth',
      block: 'nearest',
    });
  }

  return (
    <>
      <button onClick={handleAdd}>Add</button>
      <input
        value={text}
        onChange={(e) => setText(e.target.value)}
      />
      <ul ref={listRef}>
        {todos.map((todo) => (
          <li key={todo.id}>{todo.text}</li>
        ))}
      </ul>
    </>
  );
}
```

👉 Résultat : le scroll est toujours en **retard d’un élément**, car `setTodos` est **asynchrone** et le DOM n’est pas encore mis à jour au moment du `scrollIntoView()`.

***

### ✅ Solution : forcer une mise à jour synchrone avec `flushSync`

React propose `flushSync` (de `react-dom`) pour **forcer la mise à jour du DOM immédiatement après un `setState`**.

```jsx
import { useState, useRef } from 'react';
import { flushSync } from 'react-dom';

export default function TodoList() {
  const listRef = useRef(null);
  const [text, setText] = useState('');
  const [todos, setTodos] = useState(initialTodos);

  function handleAdd() {
    const newTodo = { id: nextId++, text: text };

    // ✅ flushSync : applique la mise à jour du DOM immédiatement
    flushSync(() => {
      setText('');
      setTodos([...todos, newTodo]);
    });

    listRef.current.lastChild.scrollIntoView({
      behavior: 'smooth',
      block: 'nearest',
    });
  }

  return (
    <>
      <button onClick={handleAdd}>Add</button>
      <input
        value={text}
        onChange={(e) => setText(e.target.value)}
      />
      <ul ref={listRef}>
        {todos.map((todo) => (
          <li key={todo.id}>{todo.text}</li>
        ))}
      </ul>
    </>
  );
}
```

👉 Ici, grâce à `flushSync`, le dernier élément est **déjà rendu dans le DOM** au moment du scroll → le comportement est correct.

***

### 📝 Récapitulatif

* `ref.current` est mis à jour pendant la **phase de commit**, jamais pendant le rendu.
* Accède aux refs depuis un **event handler** ou un **useEffect**.
* ⚡ Si tu dois agir immédiatement après une mise à jour d’état (avant que React n’ait le temps de différer le rendu) → utilise `flushSync`.

## Bonnes pratiques pour la manipulation du DOM avec les refs

Les **refs** sont un **échappatoire (escape hatch)**. Tu dois les utiliser uniquement quand il est nécessaire de « sortir de React » pour interagir directement avec le navigateur ou une API que React ne gère pas.

👉 Cas typiques :

* Gérer le **focus** d’un champ de formulaire.
* Contrôler la **position de scroll**.
* Utiliser une **API du navigateur** que React n’expose pas.

***

### ✅ Actions sûres avec les refs

Si tu utilises les refs pour des actions **non destructives** comme :

* `inputRef.current.focus()` (mettre le focus)
* `elementRef.current.scrollIntoView()` (scroller)
* Lire des dimensions avec `elementRef.current.getBoundingClientRect()`

➡️ Pas de problème, car tu ne modifies pas la structure que React contrôle.

***

### ⚠️ Exemple de problème : manipuler le DOM en dehors de React

Ici, on a un message affiché de façon conditionnelle.

* Le premier bouton utilise `setState` pour basculer l’affichage (**React gère le DOM**).
* Le second bouton force la suppression de l’élément avec `.remove()` (**en dehors de React**).

```jsx
import { useState, useRef } from 'react';

export default function Counter() {
  const [show, setShow] = useState(true);
  const ref = useRef(null);

  return (
    <div>
      <button onClick={() => setShow(!show)}>
        Toggle with setState
      </button>
      <button onClick={() => ref.current.remove()}>
        Remove from the DOM
      </button>
      {show && <p ref={ref}>Hello world</p>}
    </div>
  );
}
```

👉 Étapes :

1. Clique sur **Toggle with setState** → le message apparaît/disparaît normalement.
2. Clique sur **Remove from the DOM** → l’élément est supprimé **en dehors du contrôle de React**.
3. Clique à nouveau sur **Toggle with setState** → 💥 Crash ou comportement incohérent.

➡️ Cela se produit car **React ne sait plus gérer le DOM** qu’il pense contrôler.

***

### 🔒 Règles à respecter

1. **Ne modifie pas le DOM géré par React** (supprimer, ajouter, changer la hiérarchie).
   * Ex. : ne jamais utiliser `.remove()`, `.appendChild()`, `.innerHTML = ...` sur des nœuds rendus par React.
   * Sinon → incohérences ou plantages.
2. Tu peux modifier le DOM uniquement si React **n’a aucune raison de le mettre à jour**.
   *   Exemple sûr :

       ```jsx
       <div ref={containerRef}></div>
       ```

       Comme React ne met jamais de contenu dans ce `<div>`, tu peux y injecter tes propres éléments avec `appendChild()`.
3. Utilise les refs pour des actions **non destructives** :
   * Focus, scroll, mesures → ✅.
   * Manipuler le contenu rendu par React → ❌ sauf si tu sais que React n’y touchera jamais.

***

### 📝 Récapitulatif

* Les **refs** servent surtout à stocker des **éléments DOM**.
* Tu lies une ref à un élément avec `<div ref={myRef}>`.
* Tu dois les utiliser pour des **actions légères** (focus, scroll, lecture de dimensions).
* Par défaut, un composant n’expose pas son DOM → il faut passer une prop `ref`.
* ⚠️ **Évite de modifier le DOM géré par React**.
* Si tu dois manipuler le DOM, ne touche qu’à des parties que React ne met jamais à jour.
