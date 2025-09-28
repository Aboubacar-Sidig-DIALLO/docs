# 🚀 Monter en échelle avec Reducer et Context(Scaling Up with Reducer and Context)

👉 Les **reducers** permettent de centraliser la logique de mise à jour de l’état.\
👉 Les **contexts** permettent de partager des données profondément dans l’arbre sans avoir à passer les props de parent en enfant (_prop drilling_).

En combinant les deux, on obtient une architecture **claire, scalable et maintenable**.

***

### 🎯 Objectifs

* Combiner un **reducer** avec un **context**
* Éviter le _prop drilling_ (`tasks` et `dispatch` à travers toute la hiérarchie)
* Garder **la logique d’état et le contexte** dans des fichiers séparés

***

### 🔹 Exemple initial (uniquement reducer)

```jsx
import { useReducer } from 'react';
import AddTask from './AddTask.js';
import TaskList from './TaskList.js';

export default function TaskApp() {
  const [tasks, dispatch] = useReducer(
    tasksReducer,
    initialTasks
  );

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
      task: task
    });
  }

  function handleDeleteTask(taskId) {
    dispatch({
      type: 'deleted',
      id: taskId
    });
  }

  return (
    <>
      <h1>Jour de repos à Kyoto</h1>
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

➡️ Ici, **TaskApp** doit passer `tasks` et toutes les fonctions (`onChangeTask`, `onDeleteTask`) via les props.\
⚠️ Cela fonctionne pour un petit projet, mais devient vite ingérable sur une grosse app (_prop drilling_).

***

### 🔹 Étape suivante : Context + Reducer

Nous allons placer **`tasks` et `dispatch` dans un contexte** pour les rendre accessibles **depuis n’importe quel composant** de l’arbre.

***

#### 🛠️ Étape 1 : Créer les contextes

👉 Bonne pratique : séparer le **state** et le **dispatch** dans deux contextes distincts.

```js
// TasksContext.js
import { createContext } from 'react';

export const TasksContext = createContext(null);        // Pour lire l’état
export const TasksDispatchContext = createContext(null); // Pour envoyer des actions
```

***

#### 🛠️ Étape 2 : Fournir les contextes dans TaskApp

```jsx
import { useReducer } from 'react';
import { TasksContext, TasksDispatchContext } from './TasksContext.js';
import tasksReducer from './tasksReducer.js';
import AddTask from './AddTask.js';
import TaskList from './TaskList.js';

export default function TaskApp() {
  const [tasks, dispatch] = useReducer(tasksReducer, initialTasks);

  return (
    <TasksContext.Provider value={tasks}>
      <TasksDispatchContext.Provider value={dispatch}>
        <h1>Jour de repos à Kyoto</h1>
        <AddTask />
        <TaskList />
      </TasksDispatchContext.Provider>
    </TasksContext.Provider>
  );
}
```

***

#### 🛠️ Étape 3 : Utiliser les contextes dans les enfants

**Dans `TaskList.js` :**

```jsx
import { useContext } from 'react';
import { TasksContext } from './TasksContext.js';
import Task from './Task.js';

export default function TaskList() {
  const tasks = useContext(TasksContext);

  return (
    <ul>
      {tasks.map(task => (
        <li key={task.id}>
          <Task task={task} />
        </li>
      ))}
    </ul>
  );
}
```

**Dans `Task.js` :**

```jsx
import { useContext } from 'react';
import { TasksDispatchContext } from './TasksContext.js';

export default function Task({ task }) {
  const dispatch = useContext(TasksDispatchContext);

  return (
    <>
      <input
        type="text"
        value={task.text}
        onChange={e =>
          dispatch({
            type: 'changed',
            task: { ...task, text: e.target.value }
          })
        }
      />
      <button onClick={() => dispatch({ type: 'deleted', id: task.id })}>
        Supprimer
      </button>
    </>
  );
}
```

***

### ✅ Résultat

* `TaskApp` fournit **l’état** (`tasks`) et **la fonction de mise à jour** (`dispatch`) via deux Providers.
* `TaskList` et `Task` récupèrent directement ce dont ils ont besoin via `useContext`.
* 👉 Plus besoin de passer des props interminables. 🎉

***

### 📝 Récapitulatif

* **Reducer** : centraliser la logique d’état.
* **Context** : partager les données dans l’arbre.
* Ensemble = une **mini-architecture type Redux**, mais intégrée à React.
* Bonne pratique → séparer `state` et `dispatch` dans **deux contextes distincts** pour limiter les re-renders inutiles.



Jusqu’ici :

* Nous avons un **reducer** (`tasksReducer`) qui gère la logique des tâches.
* Nous avons créé deux **contexts** :
  * `TasksContext` (contient la liste des tâches)
  * `TasksDispatchContext` (contient la fonction `dispatch`)

Cela nous permet d’éviter le **prop drilling** (passer `tasks` et toutes les fonctions à travers plusieurs composants).

***

### 🔹 Étape 1 : Créer les contextes

👉 Crée un fichier `TasksContext.js` :

```js
// TasksContext.js
import { createContext } from 'react';

export const TasksContext = createContext(null);        // Pour lire la liste des tâches
export const TasksDispatchContext = createContext(null); // Pour envoyer des actions
```

Ici on met `null` par défaut, mais les vraies valeurs seront fournies par le composant `TaskApp`.

***

### 🔹 Étape 2 : Fournir state + dispatch via context

👉 Dans `TaskApp.js` :

```jsx
import { useReducer } from 'react';
import AddTask from './AddTask.js';
import TaskList from './TaskList.js';
import { TasksContext, TasksDispatchContext } from './TasksContext.js';
import tasksReducer from './tasksReducer.js';

export default function TaskApp() {
  const [tasks, dispatch] = useReducer(tasksReducer, initialTasks);

  return (
    <TasksContext.Provider value={tasks}>
      <TasksDispatchContext.Provider value={dispatch}>
        <h1>Jour de repos à Kyoto</h1>
        <AddTask />
        <TaskList />
      </TasksDispatchContext.Provider>
    </TasksContext.Provider>
  );
}
```

👉 Ici, **tout composant enfant** peut accéder à :

* `tasks` → via `TasksContext`
* `dispatch` → via `TasksDispatchContext`

***

### 🔹 Étape 3 : Lire et modifier les données avec `useContext`

#### Dans `TaskList.js` :

```jsx
import { useContext } from 'react';
import { TasksContext } from './TasksContext.js';
import Task from './Task.js';

export default function TaskList() {
  const tasks = useContext(TasksContext);

  return (
    <ul>
      {tasks.map(task => (
        <li key={task.id}>
          <Task task={task} />
        </li>
      ))}
    </ul>
  );
}
```

#### Dans `Task.js` :

```jsx
import { useState, useContext } from 'react';
import { TasksDispatchContext } from './TasksContext.js';

export default function Task({ task }) {
  const [isEditing, setIsEditing] = useState(false);
  const dispatch = useContext(TasksDispatchContext);

  let content;
  if (isEditing) {
    content = (
      <>
        <input
          value={task.text}
          onChange={e =>
            dispatch({
              type: 'changed',
              task: { ...task, text: e.target.value }
            })
          }
        />
        <button onClick={() => setIsEditing(false)}>Enregistrer</button>
      </>
    );
  } else {
    content = (
      <>
        {task.text}
        <button onClick={() => setIsEditing(true)}>Modifier</button>
      </>
    );
  }

  return (
    <label>
      <input
        type="checkbox"
        checked={task.done}
        onChange={e =>
          dispatch({
            type: 'changed',
            task: { ...task, done: e.target.checked }
          })
        }
      />
      {content}
      <button onClick={() => dispatch({ type: 'deleted', id: task.id })}>
        Supprimer
      </button>
    </label>
  );
}
```

#### Dans `AddTask.js` :

```jsx
import { useState, useContext } from 'react';
import { TasksDispatchContext } from './TasksContext.js';

let nextId = 3;

export default function AddTask() {
  const [text, setText] = useState('');
  const dispatch = useContext(TasksDispatchContext);

  return (
    <>
      <input
        placeholder="Nouvelle tâche"
        value={text}
        onChange={e => setText(e.target.value)}
      />
      <button
        onClick={() => {
          dispatch({
            type: 'added',
            id: nextId++,
            text: text
          });
          setText('');
        }}
      >
        Ajouter
      </button>
    </>
  );
}
```

***

### ✅ Résultat final

* `TaskApp` fournit **state + dispatch** via Context
* `TaskList`, `Task` et `AddTask` consomment ces données avec `useContext`
* ➡️ Plus besoin de passer `props` partout

***

### 📝 Récapitulatif

* `useReducer` centralise la logique d’état
* `Context` évite de propager inutilement les props
* Avec les deux combinés : une mini-architecture **type Redux**, mais **native React** 🎉

## 🚀 Déplacer tout le câblage dans un seul fichier

Jusqu’ici :

* Tu avais séparé ton **reducer** (`tasksReducer`) et tes **contexts** (`TasksContext`, `TasksDispatchContext`).
* Chaque composant (TaskApp, TaskList, AddTask, Task, etc.) devait importer à la fois le reducer et les contextes.

👉 Tu peux encore simplifier en regroupant **tout le câblage dans un seul fichier** (`TasksContext.js`).

***

### 🔹 Étape 1 : Créer `TasksProvider` dans le même fichier

Dans `TasksContext.js` :

```jsx
import { createContext, useContext, useReducer } from 'react';

// --- 1. Création des Contexts ---
export const TasksContext = createContext(null);
export const TasksDispatchContext = createContext(null);

// --- 2. Reducer ---
function tasksReducer(tasks, action) {
  switch (action.type) {
    case 'added': {
      return [
        ...tasks,
        { id: action.id, text: action.text, done: false }
      ];
    }
    case 'changed': {
      return tasks.map(t =>
        t.id === action.task.id ? action.task : t
      );
    }
    case 'deleted': {
      return tasks.filter(t => t.id !== action.id);
    }
    default: {
      throw Error('Action inconnue : ' + action.type);
    }
  }
}

// --- 3. Données initiales ---
const initialTasks = [
  { id: 0, text: 'Visiter le temple Kiyomizu-dera', done: true },
  { id: 1, text: 'Boire du thé matcha', done: false },
  { id: 2, text: 'Explorer Gion', done: false },
];

// --- 4. Provider global ---
export function TasksProvider({ children }) {
  const [tasks, dispatch] = useReducer(tasksReducer, initialTasks);

  return (
    <TasksContext.Provider value={tasks}>
      <TasksDispatchContext.Provider value={dispatch}>
        {children}
      </TasksDispatchContext.Provider>
    </TasksContext.Provider>
  );
}

// --- 5. Custom Hooks ---
export function useTasks() {
  return useContext(TasksContext);
}

export function useTasksDispatch() {
  return useContext(TasksDispatchContext);
}
```

***

### 🔹 Étape 2 : Utiliser le `TasksProvider` dans ton app

👉 Dans `App.js` (ou `TaskApp.js`) :

```jsx
import AddTask from './AddTask.js';
import TaskList from './TaskList.js';
import { TasksProvider } from './TasksContext.js';

export default function TaskApp() {
  return (
    <TasksProvider>
      <h1>Jour de repos à Kyoto</h1>
      <AddTask />
      <TaskList />
    </TasksProvider>
  );
}
```

***

### 🔹 Étape 3 : Lire et modifier les tâches avec les **Custom Hooks**

#### Dans `TaskList.js` :

```jsx
import { useTasks } from './TasksContext.js';
import Task from './Task.js';

export default function TaskList() {
  const tasks = useTasks();

  return (
    <ul>
      {tasks.map(task => (
        <li key={task.id}>
          <Task task={task} />
        </li>
      ))}
    </ul>
  );
}
```

#### Dans `Task.js` :

```jsx
import { useState } from 'react';
import { useTasksDispatch } from './TasksContext.js';

export default function Task({ task }) {
  const [isEditing, setIsEditing] = useState(false);
  const dispatch = useTasksDispatch();

  let content;
  if (isEditing) {
    content = (
      <>
        <input
          value={task.text}
          onChange={e =>
            dispatch({
              type: 'changed',
              task: { ...task, text: e.target.value }
            })
          }
        />
        <button onClick={() => setIsEditing(false)}>Enregistrer</button>
      </>
    );
  } else {
    content = (
      <>
        {task.text}
        <button onClick={() => setIsEditing(true)}>Modifier</button>
      </>
    );
  }

  return (
    <label>
      <input
        type="checkbox"
        checked={task.done}
        onChange={e =>
          dispatch({
            type: 'changed',
            task: { ...task, done: e.target.checked }
          })
        }
      />
      {content}
      <button onClick={() => dispatch({ type: 'deleted', id: task.id })}>
        Supprimer
      </button>
    </label>
  );
}
```

#### Dans `AddTask.js` :

```jsx
import { useState } from 'react';
import { useTasksDispatch } from './TasksContext.js';

let nextId = 3;

export default function AddTask() {
  const [text, setText] = useState('');
  const dispatch = useTasksDispatch();

  return (
    <>
      <input
        placeholder="Nouvelle tâche"
        value={text}
        onChange={e => setText(e.target.value)}
      />
      <button
        onClick={() => {
          dispatch({ type: 'added', id: nextId++, text });
          setText('');
        }}
      >
        Ajouter
      </button>
    </>
  );
}
```

***

### ✅ Résultat final

* `TasksContext.js` contient **tout le câblage** : reducer + contexts + provider + hooks.
* `TaskApp.js` est ultra simple : il ne fait que rendre `<TasksProvider>` et les composants enfants.
* Les composants (`TaskList`, `Task`, `AddTask`) utilisent `useTasks` et `useTasksDispatch` au lieu de recevoir des props.

***

### 📝 Récapitulatif

* 🎯 **Reducer** → centralise la logique d’état
* 🎯 **Context** → évite le prop drilling
* 🎯 **TasksProvider** → encapsule le state + dispatch et les fournit à toute l’arborescence
* 🎯 **Custom Hooks** (`useTasks`, `useTasksDispatch`) → rendent l’utilisation très simple dans les composants

👉 Tu obtiens une architecture proche de Redux, mais en natif React, plus légère et maintenable.
