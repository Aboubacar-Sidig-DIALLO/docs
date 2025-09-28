# Extraire la logique d’état dans un réducer

Quand un composant contient beaucoup de mises à jour d’état dispersées dans plusieurs gestionnaires d’événements, cela peut devenir difficile à lire et à maintenir.\
Dans ces cas, vous pouvez regrouper toute la logique de mise à jour de l’état **dans une seule fonction** externe au composant : un **réducer**.

***

### Vous allez apprendre

* Ce qu’est une **fonction réducer**
* Comment **refactoriser `useState` vers `useReducer`**
* Quand utiliser un **réducer**
* Comment **bien l’écrire**

***

### Consolider la logique d’état avec un réducer

À mesure qu’un composant grossit, il devient difficile de voir en un coup d’œil toutes les façons dont son état est mis à jour.\
Par exemple, ce composant **`TaskApp`** gère un tableau de tâches avec trois gestionnaires d’événements différents : ajouter, modifier et supprimer.

```jsx
// App.js
import { useState } from 'react';
import AddTask from './AddTask.js';
import TaskList from './TaskList.js';

export default function TaskApp() {
  const [tasks, setTasks] = useState(initialTasks);

  function handleAddTask(text) {
    setTasks([
      ...tasks,
      {
        id: nextId++,
        text: text,
        done: false,
      },
    ]);
  }

  function handleChangeTask(task) {
    setTasks(
      tasks.map((t) => {
        if (t.id === task.id) {
          return task;
        } else {
          return t;
        }
      })
    );
  }

  function handleDeleteTask(taskId) {
    setTasks(tasks.filter((t) => t.id !== taskId));
  }

  return (
    <>
      <AddTask onAddTask={handleAddTask} />
      <TaskList
        tasks={tasks}
        onChangeTask={handleChangeTask}
        onDeleteTask={handleDeleteTask}
      />
    </>
  );
}

let nextId = 3;
const initialTasks = [
  { id: 0, text: 'Visiter le Louvre', done: true },
  { id: 1, text: 'Faire les courses', done: false },
  { id: 2, text: 'Lire un livre', done: false },
];
```

***

### Problème

* Chaque gestionnaire d’événement appelle `setTasks` avec une logique différente.
* La logique de mise à jour est **éparpillée** dans tout le composant.
* Plus le composant grandit, plus il devient difficile à maintenir.

👉 Solution : **déplacer toute la logique dans un réducer**.

***

### Étapes pour migrer de `useState` à `useReducer`

#### 1. Passer de `setState` à `dispatch` d’actions

Au lieu de modifier l’état directement, on envoie une **action** qui décrit ce qu’on veut faire (`add`, `change`, `delete`, etc.).

#### 2. Écrire une fonction réducer

C’est une fonction **pure** qui prend l’état actuel et une action, puis renvoie **le nouvel état**.

```js
function tasksReducer(tasks, action) {
  switch (action.type) {
    case 'added': {
      return [
        ...tasks,
        {
          id: action.id,
          text: action.text,
          done: false,
        },
      ];
    }
    case 'changed': {
      return tasks.map((t) => {
        if (t.id === action.task.id) {
          return action.task;
        } else {
          return t;
        }
      });
    }
    case 'deleted': {
      return tasks.filter((t) => t.id !== action.id);
    }
    default: {
      throw Error('Action inconnue : ' + action.type);
    }
  }
}
```

#### 3. Utiliser le reducer dans le composant

```jsx
import { useReducer } from 'react';
import AddTask from './AddTask.js';
import TaskList from './TaskList.js';

export default function TaskApp() {
  const [tasks, dispatch] = useReducer(tasksReducer, initialTasks);

  function handleAddTask(text) {
    dispatch({
      type: 'added',
      id: nextId++,
      text,
    });
  }

  function handleChangeTask(task) {
    dispatch({
      type: 'changed',
      task,
    });
  }

  function handleDeleteTask(taskId) {
    dispatch({
      type: 'deleted',
      id: taskId,
    });
  }

  return (
    <>
      <AddTask onAddTask={handleAddTask} />
      <TaskList
        tasks={tasks}
        onChangeTask={handleChangeTask}
        onDeleteTask={handleDeleteTask}
      />
    </>
  );
}
```

***

✅ Résultat :

* Toute la logique de mise à jour est regroupée dans **une seule fonction (`tasksReducer`)**.
* Le composant est plus lisible : il ne fait qu’**envoyer des actions**.
* On réduit les risques de bugs et on prépare le terrain pour des fonctionnalités plus complexes (comme **undo/redo**).

### Étape 1 : Remplacer `setState` par `dispatch`

Avant, tes gestionnaires d’événements **décrivaient quoi faire à l’état** :

```js
function handleAddTask(text) {
  setTasks([
    ...tasks,
    {
      id: nextId++,
      text: text,
      done: false,
    },
  ]);
}

function handleChangeTask(task) {
  setTasks(
    tasks.map((t) => {
      if (t.id === task.id) {
        return task;
      } else {
        return t;
      }
    })
  );
}

function handleDeleteTask(taskId) {
  setTasks(tasks.filter((t) => t.id !== taskId));
}
```

Ici, chaque handler **connaît la logique de mise à jour complète** : ajout, modification, suppression.\
👉 Cela disperse la logique et la rend plus difficile à maintenir.

***

#### Après : on **décrit ce qui s’est passé** avec une **action**

Avec un reducer, tu **n’indiques plus quoi faire à l’état**, mais **ce que l’utilisateur vient de faire**.

Chaque gestionnaire **envoie une action** (objet JS simple) à `dispatch` :

```js
function handleAddTask(text) {
  dispatch({
    type: 'added',
    id: nextId++,
    text: text,
  });
}

function handleChangeTask(task) {
  dispatch({
    type: 'changed',
    task: task,
  });
}

function handleDeleteTask(taskId) {
  dispatch({
    type: 'deleted',
    id: taskId,
  });
}
```

***

#### Qu’est-ce qu’une **action** ?

Une **action** est simplement un **objet JavaScript**.\
Par convention :

* `type`: une **chaîne** décrivant l’événement (ex. `"added"`, `"deleted"`)
* d’autres champs : les données nécessaires pour effectuer la mise à jour

Exemple :

```js
dispatch({
  type: 'deleted',
  id: taskId,
});
```

Ici :

* `type` = ce qui s’est passé ("deleted")
* `id` = donnée nécessaire pour appliquer ce changement

***

#### Pourquoi c’est mieux ?

* **Lisibilité** → tu sais ce qui s’est passé (ajout, modif, suppression) en lisant l’action
* **Centralisation** → la logique de mise à jour de l’état sera regroupée **dans le reducer**
* **Évolutif** → plus facile d’ajouter de nouveaux cas (ex. "toggle", "undo", etc.)

***

👉 **Étape suivante :** écrire la fonction **reducer** qui saura traiter chaque type d’action (`added`, `changed`, `deleted`) et mettre à jour l’état correctement.

### ✨ Étape 2 : Écrire une fonction reducer

Un **reducer** est une fonction qui :

* prend **l’état actuel** (ici `tasks`)
* prend **l’action** (objet décrivant ce qui s’est passé)
* retourne **le nouvel état** (que React utilisera).

👉 Tu centralises **toute la logique de mise à jour** dans une seule fonction au lieu de la disperser dans chaque handler.

***

#### Exemple avec `if/else`

```js
function tasksReducer(tasks, action) {
  if (action.type === 'added') {
    return [
      ...tasks,
      {
        id: action.id,
        text: action.text,
        done: false,
      },
    ];
  } else if (action.type === 'changed') {
    return tasks.map((t) => {
      if (t.id === action.task.id) {
        return action.task;
      } else {
        return t;
      }
    });
  } else if (action.type === 'deleted') {
    return tasks.filter((t) => t.id !== action.id);
  } else {
    throw Error('Unknown action: ' + action.type);
  }
}
```

***

#### Version recommandée avec `switch`

C’est la convention, car plus lisible 👇

```js
function tasksReducer(tasks, action) {
  switch (action.type) {
    case 'added': {
      return [
        ...tasks,
        {
          id: action.id,
          text: action.text,
          done: false,
        },
      ];
    }
    case 'changed': {
      return tasks.map((t) => {
        if (t.id === action.task.id) {
          return action.task;
        } else {
          return t;
        }
      });
    }
    case 'deleted': {
      return tasks.filter((t) => t.id !== action.id);
    }
    default: {
      throw Error('Unknown action: ' + action.type);
    }
  }
}
```

⚡ Remarques importantes :

* Chaque `case` est entouré de `{ }` pour éviter les conflits de variables.
* Chaque `case` se termine par un `return`.
* Si aucun `case` ne correspond, on lève une erreur → pratique pour le debug.

***

#### Pourquoi "reducer" ?

Le mot vient de la méthode JavaScript `.reduce()`.\
Avec `reduce`, on accumule une valeur finale en appliquant une fonction à chaque élément d’un tableau :

```js
const arr = [1, 2, 3, 4, 5];

const sum = arr.reduce(
  (result, number) => result + number
); 
// Résultat = 15
```

👉 De la même façon, **un reducer React accumule les actions dans le temps** pour construire l’état.

Exemple avec nos `actions` :

```js
let initialState = [];
let actions = [
  { type: 'added', id: 1, text: 'Visiter le musée Kafka' },
  { type: 'added', id: 2, text: 'Voir un spectacle de marionnettes' },
  { type: 'deleted', id: 1 },
  { type: 'added', id: 3, text: 'Prendre une photo du mur Lennon' },
];

let finalState = actions.reduce(tasksReducer, initialState);

console.log(finalState);
// [
//   { id: 2, text: 'Voir un spectacle de marionnettes', done: false },
//   { id: 3, text: 'Prendre une photo du mur Lennon', done: false }
// ]
```

C’est exactement ce que fait React : appliquer ton reducer **à une suite d’actions au fil du temps**.

### ⚡ Étape 3 : Utiliser le reducer dans ton composant

👉 Au lieu d’utiliser `useState`, on va utiliser **useReducer** :

```js
import { useReducer } from 'react';
import AddTask from './AddTask.js';
import TaskList from './TaskList.js';

// Reducer séparé (tu peux le mettre dans un fichier tasksReducer.js)
function tasksReducer(tasks, action) {
  switch (action.type) {
    case 'added': {
      return [
        ...tasks,
        {
          id: action.id,
          text: action.text,
          done: false,
        },
      ];
    }
    case 'changed': {
      return tasks.map((t) =>
        t.id === action.task.id ? action.task : t
      );
    }
    case 'deleted': {
      return tasks.filter((t) => t.id !== action.id);
    }
    default: {
      throw Error('Unknown action: ' + action.type);
    }
  }
}

let nextId = 3;

const initialTasks = [
  { id: 0, text: 'Visit Kafka Museum', done: true },
  { id: 1, text: 'Watch a puppet show', done: false },
  { id: 2, text: 'Lennon Wall pic', done: false },
];

export default function TaskApp() {
  // 🔥 useReducer au lieu de useState
  const [tasks, dispatch] = useReducer(tasksReducer, initialTasks);

  function handleAddTask(text) {
    dispatch({
      type: 'added',
      id: nextId++,
      text: text,
    });
  }

  function handleChangeTask(task) {
    dispatch({
      type: 'changed',
      task: task,
    });
  }

  function handleDeleteTask(taskId) {
    dispatch({
      type: 'deleted',
      id: taskId,
    });
  }

  return (
    <>
      <h1>Prague itinerary</h1>
      <AddTask onAddTask={handleAddTask} />
      <TaskList
        tasks={tasks}
        onChangeTask={handleChangeTask}
        onDeleteTask={handleDeleteTask}
      />
    </>
  );
}
```

***

#### 🚀 Ce qui change avec `useReducer` :

* `useReducer(tasksReducer, initialTasks)` retourne **`tasks`** (état actuel) + **`dispatch`** (fonction pour envoyer une action).
* Chaque handler **ne contient plus de logique métier** → il se contente d’**envoyer une action** (`dispatch`).
* Toute la logique est centralisée dans le **reducer** (`tasksReducer`).

***

#### ✅ Avantages :

* Code plus lisible (handlers = simples dispatchs).
* La logique métier est **séparée du composant** → plus facile à maintenir et tester.
* Scalable : plus ton app grandit, plus `useReducer` rend la gestion d’état claire.

## 🔎 Comparaison : `useState` vs `useReducer`

#### 1. **Taille du code**

*   ✅ `useState` → moins de code à écrire au départ.\
    Exemple :

    ```js
    const [count, setCount] = useState(0);
    ```
* ⚡ `useReducer` → nécessite un reducer + des actions (plus verbeux).\
  Mais si ton composant contient **beaucoup de mises à jour similaires**, `useReducer` évite la duplication.

***

#### 2. **Lisibilité**

* ✅ `useState` → super lisible quand les mises à jour sont simples.
* ⚡ `useReducer` → brille quand les mises à jour sont **complexes** :
  * Handlers = simples `dispatch`
  * Toute la logique regroupée dans le **reducer**\
    → ton code devient plus clair et mieux structuré.

***

#### 3. **Débogage**

* ❌ `useState` → plus difficile de savoir _où_ l’état a mal été modifié.
* ✅ `useReducer` → tu peux `console.log(action, state)` dans le reducer → tu vois :
  * Quelle action a été envoyée
  * Pourquoi l’état a changé
  * Où est l’erreur (dans le reducer lui-même)

***

#### 4. **Tests**

* ❌ `useState` → logique dispersée dans le composant, donc plus dur à tester isolément.
*   ✅ `useReducer` → le reducer est une **fonction pure** → tu peux l’exporter et le tester seul.\
    Exemple :

    ```js
    expect(tasksReducer([], { type: 'added', id: 1, text: 'Hello' }))
      .toEqual([{ id: 1, text: 'Hello', done: false }]);
    ```

***

#### 5. **Préférence personnelle**

* Certaines personnes aiment `useReducer` pour sa **rigueur**.
* D’autres trouvent `useState` **plus simple** et suffisant.
* Tu peux même **mélanger les deux** dans un même composant :
  * `useReducer` pour la logique complexe (ex: gestion de tâches)
  * `useState` pour du simple (ex: état d’un champ texte).

***

#### ✅ Quand choisir ?

* **Utilise `useState`** si :
  * L’état est simple
  * Les mises à jour sont faciles à suivre
  * Peu de handlers
* **Utilise `useReducer`** si :
  * L’état est **complexe** (tableaux, objets, logique imbriquée)
  * Beaucoup de handlers mettent à jour l’état
  * Tu veux centraliser la logique et éviter les erreurs
  * Tu veux tester la logique séparément

***

👉 En résumé :

* `useState` = 🟢 simple, rapide, idéal pour du **local et basique**
* `useReducer` = 🔵 structuré, clair, idéal pour du **complexe et évolutif**

## ✍️ Bien écrire ses reducers

### 1. **Un reducer doit être pur**

* **Pur** = mêmes entrées → mêmes sorties, sans effets de bord.
* Interdit dans un reducer :\
  ❌ appels réseau (`fetch`, `axios`, etc.)\
  ❌ timers (`setTimeout`, `setInterval`)\
  ❌ mutations directes d’objets/arrays (`push`, `splice`, `=` direct sur props).

👉 Le reducer **ne fait qu’une seule chose** : transformer `state` en un **nouvel état** en fonction de l’`action`.

Exemple correct ✅ :

```js
function tasksReducer(tasks, action) {
  switch (action.type) {
    case 'added': {
      return [...tasks, { id: action.id, text: action.text, done: false }];
    }
    case 'deleted': {
      return tasks.filter(t => t.id !== action.id);
    }
    default: {
      throw Error('Unknown action: ' + action.type);
    }
  }
}
```

Exemple incorrect ❌ :

```js
function tasksReducer(tasks, action) {
  if (action.type === 'added') {
    fetch('/api/log', { method: 'POST', body: JSON.stringify(action) }); // ❌ effet de bord
    tasks.push({ id: action.id, text: action.text, done: false });       // ❌ mutation
    return tasks;
  }
}
```

***

### 2. **Chaque action doit représenter une interaction claire**

* Une action = un **événement utilisateur** (ou système).
* Même si ça entraîne plusieurs changements, regroupe-les dans **une seule action**.

Exemple 🟢 :\
Un formulaire avec 5 champs → bouton "Reset" → **1 seule action** :

```js
dispatch({ type: 'reset_form' });
```

Dans le reducer :

```js
case 'reset_form': {
  return initialFormState; // reset complet en une seule action
}
```

Exemple 🔴 à éviter :

```js
dispatch({ type: 'set_field', field: 'name', value: '' });
dispatch({ type: 'set_field', field: 'email', value: '' });
dispatch({ type: 'set_field', field: 'password', value: '' });
// etc...
```

👉 L’idée :\
Si tu **logs** toutes les actions, tu dois pouvoir reconstruire facilement ce que l’utilisateur a fait ("ajouté une tâche", "réinitialisé le formulaire", "supprimé une tâche").

***

✅ En résumé :

* **Pureté** : pas d’effets de bord → juste des transformations immuables de `state`.
* **Clarté des actions** : 1 action = 1 interaction utilisateur.

## ✍️ Écrire des reducers concis avec **Immer**

### 1. Le problème classique

Un reducer doit rester **pur** → interdiction de muter directement `state`.\
Donc, on est obligé d’écrire du code immuable parfois un peu lourd :

```js
function tasksReducer(tasks, action) {
  switch (action.type) {
    case 'added':
      return [
        ...tasks,
        { id: action.id, text: action.text, done: false }
      ];
    case 'changed':
      return tasks.map(t => 
        t.id === action.task.id ? action.task : t
      );
    case 'deleted':
      return tasks.filter(t => t.id !== action.id);
    default:
      throw Error('Unknown action: ' + action.type);
  }
}
```

Ça marche ✅ mais ça devient vite verbeux si l’état est complexe.

***

### 2. Solution : Immer + `useImmerReducer`

Avec Immer, tu peux **écrire comme si tu mutais l’état**, mais en réalité Immer gère les copies immuables pour toi.

👉 Exemple avec `useImmerReducer` (fourni par le package `use-immer`) :

```js
import { useImmerReducer } from "use-immer";

function tasksReducer(draft, action) {
  switch (action.type) {
    case 'added':
      draft.push({
        id: action.id,
        text: action.text,
        done: false
      });
      break;
    case 'changed': {
      const index = draft.findIndex(t => t.id === action.task.id);
      draft[index] = action.task;
      break;
    }
    case 'deleted': {
      return draft.filter(t => t.id !== action.id);
    }
    default:
      throw Error('Unknown action: ' + action.type);
  }
}

export default function TaskApp() {
  const [tasks, dispatch] = useImmerReducer(tasksReducer, initialTasks);

  function handleAddTask(text) {
    dispatch({ type: 'added', id: nextId++, text });
  }

  function handleChangeTask(task) {
    dispatch({ type: 'changed', task });
  }

  function handleDeleteTask(taskId) {
    dispatch({ type: 'deleted', id: taskId });
  }

  return (
    <>
      <h1>Prague itinerary</h1>
      <AddTask onAddTask={handleAddTask} />
      <TaskList
        tasks={tasks}
        onChangeTask={handleChangeTask}
        onDeleteTask={handleDeleteTask}
      />
    </>
  );
}
```

***

### 3. Pourquoi ça marche ?

* Dans un reducer classique : tu **dois retourner** un nouvel état.
* Dans un reducer avec Immer : tu reçois un `draft` (brouillon mutable).
  * Tu peux faire `push`, `splice`, `draft[i] = ...` sans problème.
  * Immer s’occupe de générer un nouvel état immuable à partir de ton brouillon.

⚡ Résultat → code plus concis et plus lisible, surtout avec des états complexes.

***

### 4. Récap ⚡

* 🔄 Migrer de `useState` → `useReducer` =
  1. Dispatcher des **actions** depuis tes handlers
  2. Déplacer la logique de mise à jour dans un **reducer**
  3. Brancher avec `useReducer`
* 🎯 Avantages des reducers :
  * Centraliser la logique
  * Debugging + facile
  * Testables isolément
* ⚠️ Règles :
  * Reducers doivent rester **purs** (pas d’effets de bord)
  * Une action = une **interaction utilisateur** claire
* ✨ Avec **Immer** (`useImmerReducer`) :
  * Tu écris du code comme si tu **mutais** ton état
  * Immer s’occupe de générer une version immuable propre
