# 🛠️ Escape Hatches

👉 La plupart de ton application devrait rester dans le flux normal de React (props + state).\
Mais parfois tu dois **sortir de React** pour :

* contrôler un élément DOM directement (ex. mettre le focus dans un input),
* gérer un lecteur vidéo externe,
* synchroniser avec un système distant (WebSocket, API navigateur, etc.),
* ou stocker des infos qui n’ont **pas besoin de déclencher un re-render**.

C’est là qu’entrent en jeu ces « issues de secours ».

***

### 🔹 Dans ce chapitre, tu verras :

1. **Comment “mémoriser” une valeur sans re-render** → avec `useRef`.
2. **Comment accéder au DOM géré par React** → aussi avec `useRef`.
3. **Comment synchroniser des composants avec des systèmes externes** → avec `useEffect`.
4. **Comment éviter les Effets inutiles** et comprendre leur cycle de vie.
5. **Comment empêcher certaines valeurs de relancer des Effets**.
6. **Comment contrôler la fréquence d’exécution des Effets**.
7. **Comment partager de la logique entre composants** (custom hooks).

***

## 🔑 Référencer des valeurs avec `useRef`

👉 Quand tu veux **stocker une info entre les re-renders**, sans déclencher un nouveau rendu, utilise `useRef`.

```jsx
const ref = useRef(0);
```

* Comme `useState`, un `ref` est **conservé par React entre les re-renders**.
* **Différence** : changer un `state` ⇒ **re-render** le composant.
* Changer un `ref` ⇒ **ne re-render pas**.

Tu accèdes à la valeur via la propriété `.current` :

```jsx
ref.current
```

***

#### Exemple : compteur avec `useRef`

```jsx
import { useRef } from 'react';

export default function Counter() {
  let ref = useRef(0);

  function handleClick() {
    ref.current = ref.current + 1;
    alert('Vous avez cliqué ' + ref.current + ' fois !');
  }

  return (
    <button onClick={handleClick}>
      Cliquez-moi !
    </button>
  );
}
```

✅ Ici :

* `ref.current` stocke le nombre de clics.
* Mais comme ce n’est **pas du state**, le composant ne re-render pas à chaque clic.

***

### 🧩 Métaphore

Un `ref` est comme une **poche secrète dans ton composant**.

* React **ne regarde pas** ce qu’il y a dedans.
* Tu peux y mettre tout ce qui **n’influence pas le rendu** :
  * IDs de `setTimeout`,
  * objets,
  * éléments DOM,
  * états internes que tu ne veux pas afficher.

## 🔹 Manipuler le DOM avec les Refs

👉 En React, **tu n’as presque jamais besoin de manipuler le DOM directement**, car React s’en occupe automatiquement.\
Mais parfois, tu dois accéder à un élément **géré par React** pour :

* mettre le focus sur un champ,
* scroller jusqu’à un élément,
* mesurer sa taille ou sa position.

⚠️ React n’a pas d’API intégrée pour ça → tu utilises une **ref** vers le DOM.

***

#### Exemple : focus sur un input

```jsx
import { useRef } from 'react';

export default function Form() {
  const inputRef = useRef(null);

  function handleClick() {
    inputRef.current.focus();
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

✅ Ici :

* `inputRef` est lié à l’élément `<input />` grâce à `ref={inputRef}`.
* `inputRef.current` pointe vers l’élément DOM réel.
* Quand tu cliques sur le bouton, `focus()` est appelé directement sur le DOM.

📌 Une **ref** est donc ton **pont entre React et le DOM natif**.

***

## 🔹 Synchroniser avec des Effets (`useEffect`)

Certaines fois, tes composants doivent **se synchroniser avec des systèmes externes** :

* contrôler un lecteur vidéo (non-React),
* ouvrir une connexion serveur,
* envoyer un log analytique quand un composant apparaît, etc.

👉 Pour ça, tu utilises un **Effet** (`useEffect`).\
Un Effet permet d’exécuter du code **après le rendu** de ton composant.

***

#### Exemple : lecteur vidéo contrôlé par l’état React

```jsx
import { useState, useRef, useEffect } from 'react';

function VideoPlayer({ src, isPlaying }) {
  const ref = useRef(null);

  useEffect(() => {
    if (isPlaying) {
      ref.current.play();
    } else {
      ref.current.pause();
    }
  }, [isPlaying]); // dépendance : se déclenche à chaque changement de isPlaying

  return <video ref={ref} src={src} loop playsInline />;
}

export default function App() {
  const [isPlaying, setIsPlaying] = useState(false);

  return (
    <>
      <button onClick={() => setIsPlaying(!isPlaying)}>
        {isPlaying ? 'Pause' : 'Lecture'}
      </button>
      <VideoPlayer
        isPlaying={isPlaying}
        src="https://interactive-examples.mdn.mozilla.net/media/cc0-videos/flower.mp4"
      />
    </>
  );
}
```

✅ Ici :

* React gère l’état `isPlaying`.
* L’Effet `useEffect` écoute les changements de `isPlaying`.
* Quand il change → on joue ou on met en pause la vidéo via le DOM.

***

#### Exemple : connexion avec nettoyage

```jsx
import { useEffect } from 'react';
import { createConnection } from './chat.js';

export default function ChatRoom() {
  useEffect(() => {
    const connection = createConnection();
    connection.connect();

    return () => connection.disconnect(); // cleanup au démontage
  }, []);

  return <h1>Bienvenue dans le chat !</h1>;
}
```

✅ Ici :

* Quand le composant s’affiche → on établit la connexion (`connect()`).
* Quand il disparaît ou avant de se recréer → on nettoie (`disconnect()`).

ℹ️ En **mode développement**, React exécute **deux fois** l’Effet + le nettoyage pour vérifier que tu n’oublies pas de gérer la phase de _cleanup_.

***

### 📝 Récap

* **Refs** → accéder et manipuler le DOM (focus, scroll, mesure).
* **useEffect** → synchroniser ton composant avec un système externe (vidéo, serveur, etc.).
* Toujours prévoir un **nettoyage (`return () => {...}`)** si tu crées une ressource externe.

## ⚡ Tu n’as peut-être pas besoin d’un Effet (`useEffect`)

👉 Les **Effets sont une “porte de sortie”** du paradigme React.\
Ils servent uniquement à **synchroniser ton composant avec un système externe** (DOM natif, API, socket, etc.).

❌ Si aucun système externe n’est impliqué → tu n’as **pas besoin d’Effect**.

***

### 🔹 Cas où tu n’as PAS besoin d’un Effet

#### 1. Transformer des données pour l’affichage

Mauvais code (Effet inutile) :

```jsx
function Form() {
  const [firstName, setFirstName] = useState('Taylor');
  const [lastName, setLastName] = useState('Swift');

  // 🔴 Mauvais : état redondant + useEffect inutile
  const [fullName, setFullName] = useState('');
  useEffect(() => {
    setFullName(firstName + ' ' + lastName);
  }, [firstName, lastName]);

  return <h1>{fullName}</h1>;
}
```

Meilleur code (calcul direct pendant le rendu) :

```jsx
function Form() {
  const [firstName, setFirstName] = useState('Taylor');
  const [lastName, setLastName] = useState('Swift');

  // ✅ Calculé directement au rendu
  const fullName = firstName + ' ' + lastName;

  return <h1>{fullName}</h1>;
}
```

***

#### 2. Gérer les événements utilisateur

❌ Mauvais réflexe : utiliser un `useEffect` pour mettre à jour l’état après un clic.\
✅ Meilleur réflexe : gérer directement l’événement dans le handler (`onClick`, `onChange`...).

***

📌 **Règle d’or** :\
👉 Utilise un `useEffect` **seulement** pour synchroniser avec quelque chose **en dehors de React**.

***

## 🔄 Cycle de vie d’un Effet réactif

Un composant peut :

* **monter** (apparaître),
* **se mettre à jour**,
* **se démonter** (disparaître).

Un **Effet** quant à lui peut seulement :

* **commencer une synchronisation** (ex: ouvrir une connexion, lancer un timer),
* **arrêter cette synchronisation** (ex: fermer la connexion, nettoyer le timer).

➡️ Et ce cycle peut se répéter plusieurs fois si les dépendances changent.

***

#### Exemple : synchroniser une connexion de chat

```jsx
import { useState, useEffect } from 'react';
import { createConnection } from './chat.js';

const serverUrl = 'https://localhost:1234';

function ChatRoom({ roomId }) {
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();

    // 🔑 Nettoyage quand le composant se démonte OU roomId change
    return () => connection.disconnect();
  }, [roomId]); // dépendance

  return <h1>Bienvenue dans la salle {roomId} !</h1>;
}

export default function App() {
  const [roomId, setRoomId] = useState('general');
  return (
    <>
      <label>
        Choisis une salle de chat :{' '}
        <select
          value={roomId}
          onChange={e => setRoomId(e.target.value)}
        >
          <option value="general">Général</option>
          <option value="travel">Voyage</option>
          <option value="music">Musique</option>
        </select>
      </label>
      <hr />
      <ChatRoom roomId={roomId} />
    </>
  );
}
```

***

#### 🖥️ Console en action

```
✅ Connexion à la salle "general" sur https://localhost:1234...
❌ Déconnexion de la salle "general" sur https://localhost:1234
✅ Connexion à la salle "travel" sur https://localhost:1234...
```

***

### 📌 Points clés

* Un **Effet est déclenché après chaque rendu** si une dépendance change.
* Si `roomId` change, React :
  1. appelle la fonction de nettoyage (`disconnect()`),
  2. relance la nouvelle connexion (`connect()` avec le nouveau `roomId`).
* React possède une **règle ESLint intégrée** qui te prévient si tu oublies une dépendance (ex: `roomId`).

## 🎯 Problème : quand trop de dépendances redéclenchent un `useEffect`

👉 Rappel :\
Un `useEffect` est **réactif** → il se relance **à chaque fois qu’une dépendance change**.

Exemple avec un chat :

```jsx
function ChatRoom({ roomId, theme }) {
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.on('connected', () => {
      showNotification('Connected!', theme);
    });
    connection.connect();

    return () => connection.disconnect();
  }, [roomId, theme]); // dépendances
}
```

Ici :

* ✅ `roomId` → nécessaire (on doit se reconnecter si on change de salle).
* ❌ `theme` → pas logique ! Changer le thème ne devrait pas relancer toute la connexion.

Résultat : **on se reconnecte inutilement au serveur juste parce qu’un thème change**. 🙃

***

## ⚡ Solution expérimentale : `useEffectEvent`

React propose `experimental_useEffectEvent` → renommé `useEffectEvent`.\
Il permet de **séparer le code réactif (qui dépend des dépendances) et le code non réactif (qui ne doit pas relancer l’Effet)**.

Exemple corrigé :

```jsx
import { experimental_useEffectEvent as useEffectEvent } from 'react';

function ChatRoom({ roomId, theme }) {
  const onConnected = useEffectEvent(() => {
    // Code qui dépend du thème,
    // mais qui ne doit PAS relancer l’Effet
    showNotification('Connected!', theme);
  });

  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.on('connected', () => {
      onConnected();
    });
    connection.connect();

    return () => connection.disconnect();
  }, [roomId]); // ✅ seul roomId est réactif
}
```

***

## 📌 Résultat

* Changer `roomId` ➝ le `useEffect` se relance (logique).
* Changer `theme` ➝ **pas de reconnexion**, mais la callback `onConnected` utilise bien la nouvelle valeur de `theme`.

👉 `useEffectEvent` agit comme une **fonction “fraîche”** (toujours à jour avec les dernières props/états), mais **qui n’est pas une dépendance** du `useEffect`.

***

## 🛑 Attention

* ⚠️ Cette API est **expérimentale** : pas encore dispo dans les versions stables de React.
* Elle est pensée pour des cas **où tu veux du code “à jour” sans relancer l’Effet**.
* Alternative courante aujourd’hui → utiliser des `refs` pour stocker les valeurs, ou découper la logique.

## 🚨 Le problème : dépendances inutiles

Quand tu écris un `useEffect`, l’ESLint de React vérifie que **tous les éléments réactifs (props, state, fonctions, objets, etc.) utilisés dedans** apparaissent bien dans le tableau des dépendances.

👉 Mais attention :

* Si tu crées un **objet** ou une **fonction** à chaque rendu, sa **référence change** à chaque fois.
* Donc React pense que la dépendance a changé → ce qui relance l’Effet inutilement.

Exemple de bug :

```jsx
function ChatRoom({ roomId }) {
  const [message, setMessage] = useState('');

  const options = { // ⚠️ recréé à chaque rendu !
    serverUrl: serverUrl,
    roomId: roomId
  };

  useEffect(() => {
    const connection = createConnection(options);
    connection.connect();
    return () => connection.disconnect();
  }, [options]); // ⚠️ toujours "différent", donc relance à chaque render
```

Résultat : à chaque frappe dans l’input `message`, `options` est recréé → reconnexion au serveur. 🫠

***

## ✅ La bonne approche : changer le code, pas la liste

La **liste des dépendances ne doit pas être manipulée “à la main”**.\
Elle reflète ton code.\
Donc, si tu veux la simplifier → tu dois réorganiser ton code.

Solution 👉 **déplacer la création de `options` dans le `useEffect`** :

```jsx
function ChatRoom({ roomId }) {
  const [message, setMessage] = useState('');

  useEffect(() => {
    const options = { // ✅ recréé uniquement quand roomId change
      serverUrl: serverUrl,
      roomId: roomId
    };

    const connection = createConnection(options);
    connection.connect();
    return () => connection.disconnect();
  }, [roomId]); // ✅ dépend uniquement de roomId
```

***

## 🛠️ Cas plus avancés

1. **Si tu as besoin de réutiliser un objet/fonction en dehors de l’Effet**\
   Utilise `useMemo` ou `useCallback` pour **mémoriser** la valeur :

```jsx
const options = useMemo(() => ({
  serverUrl,
  roomId
}), [roomId]);

useEffect(() => {
  const connection = createConnection(options);
  connection.connect();
  return () => connection.disconnect();
}, [options]); // ✅ options reste stable tant que roomId ne change pas
```

2. **Si tu veux une valeur stable entre les rendus sans déclencher l’Effet**\
   Tu peux utiliser `useRef` :

```jsx
const latestTheme = useRef(theme);
useEffect(() => {
  latestTheme.current = theme; // ✅ mis à jour sans relancer l’Effet
}, [theme]);
```

***

## 📌 Récap

* 🔴 Mauvais : retirer une dépendance du tableau “à la main” → bug garanti.
* ✅ Bon : **réécrire le code** pour ne pas recréer inutilement des objets/fonctions.
* ✅ Astuces : `useMemo`, `useCallback`, ou `useRef` selon le besoin.

## 🎯 Pourquoi créer un custom Hook ?

React fournit déjà des Hooks intégrés (`useState`, `useEffect`, `useContext`, etc.).\
Mais souvent, tu as besoin de logique plus **spécialisée** pour ton application :

* suivre la position de la souris 🖱️
* détecter si l’utilisateur est en ligne 📡
* gérer une API de chat 💬
* appliquer un délai ou une animation ⏳

👉 Plutôt que de dupliquer du code dans plusieurs composants, tu extrais cette logique dans un **Hook personnalisé**.

***

## 🛠 Exemple : traîner un "retard visuel" sur la position du curseur

### `usePointerPosition.js`

Un Hook pour suivre la position du curseur avec un `mousemove` :

```jsx
import { useState, useEffect } from 'react';

export function usePointerPosition() {
  const [position, setPosition] = useState({ x: 0, y: 0 });

  useEffect(() => {
    function handleMove(e) {
      setPosition({ x: e.clientX, y: e.clientY });
    }
    window.addEventListener('pointermove', handleMove);
    return () => window.removeEventListener('pointermove', handleMove);
  }, []);

  return position;
}
```

***

### `useDelayedValue.js`

Un Hook qui garde une valeur avec un **décalage dans le temps** :

```jsx
import { useState, useEffect } from 'react';

export function useDelayedValue(value, delay) {
  const [delayedValue, setDelayedValue] = useState(value);

  useEffect(() => {
    const timeout = setTimeout(() => setDelayedValue(value), delay);
    return () => clearTimeout(timeout);
  }, [value, delay]);

  return delayedValue;
}
```

***

### `App.js`

On compose ces deux Hooks ensemble ✨ :

```jsx
import { usePointerPosition } from './usePointerPosition.js';
import { useDelayedValue } from './useDelayedValue.js';

export default function Canvas() {
  const pos1 = usePointerPosition();
  const pos2 = useDelayedValue(pos1, 100);
  const pos3 = useDelayedValue(pos2, 200);
  const pos4 = useDelayedValue(pos3, 100);
  const pos5 = useDelayedValue(pos4, 50);

  return (
    <>
      <Dot position={pos1} opacity={1} />
      <Dot position={pos2} opacity={0.8} />
      <Dot position={pos3} opacity={0.6} />
      <Dot position={pos4} opacity={0.4} />
      <Dot position={pos5} opacity={0.2} />
    </>
  );
}

function Dot({ position, opacity }) {
  return (
    <div style={{
      position: 'absolute',
      backgroundColor: 'pink',
      borderRadius: '50%',
      opacity,
      transform: `translate(${position.x}px, ${position.y}px)`,
      pointerEvents: 'none',
      left: -20,
      top: -20,
      width: 40,
      height: 40,
    }} />
  );
}
```

***

## 📌 Règles importantes pour les custom Hooks

1. **Le nom doit commencer par `use`** → ex. `useAuth`, `useFetch`.\
   Cela permet à React (et à ESLint) de vérifier les règles des Hooks.
2. Un Hook personnalisé n’est **qu’une fonction** qui :
   * utilise d’autres Hooks (state, effect, etc.),
   * retourne quelque chose (valeur, fonction, objet, etc.).
3. Tu peux **composer** plusieurs Hooks ensemble.\
   Dans l’exemple, `usePointerPosition` → `useDelayedValue` → réutilisés en cascade.

***

## 🔥 Récap

* ✅ Un custom Hook = une fonction qui commence par `use`.
* ✅ Sert à réutiliser et partager de la logique entre composants.
* ✅ Peut combiner plusieurs Hooks intégrés ou personnalisés.
* ✅ Rend le code plus **propre**, plus **facile à tester** et à **maintenir**.

## 📖Réutiliser la logique avec des Hooks personnalisés( Reusing Logic with Custom Hooks)

React fournit déjà des Hooks intégrés comme :

* `useState` → gérer l’état local
* `useEffect` → synchroniser avec un système externe
* `useContext` → lire un contexte

Mais parfois, tu aurais besoin d’un Hook plus **spécifique** :

* récupérer des données depuis une API (`useFetch`)
* suivre si l’utilisateur est en ligne (`useOnlineStatus`)
* se connecter à un chat en temps réel (`useChat`)

👉 Pour cela, tu peux **créer tes propres Hooks personnalisés**.

***

### 🖱 Exemple pratique : suivre la souris + ajouter un délai

Ici, on crée deux Hooks :

#### 1. `usePointerPosition.js`

Sert à suivre la position du pointeur (curseur de la souris) :

```jsx
import { useState, useEffect } from 'react';

export function usePointerPosition() {
  const [position, setPosition] = useState({ x: 0, y: 0 });

  useEffect(() => {
    function handleMove(e) {
      setPosition({ x: e.clientX, y: e.clientY });
    }
    window.addEventListener('pointermove', handleMove);
    return () => window.removeEventListener('pointermove', handleMove);
  }, []);

  return position;
}
```

***

#### 2. `useDelayedValue.js`

Sert à renvoyer une valeur **avec un délai** (lagging value) :

```jsx
import { useState, useEffect } from 'react';

export function useDelayedValue(value, delay) {
  const [delayedValue, setDelayedValue] = useState(value);

  useEffect(() => {
    const timeout = setTimeout(() => setDelayedValue(value), delay);
    return () => clearTimeout(timeout);
  }, [value, delay]);

  return delayedValue;
}
```

***

#### 3. `App.js` (Canvas)

On combine les deux Hooks pour faire une **traînée de points** qui suit le curseur :

```jsx
import { usePointerPosition } from './usePointerPosition.js';
import { useDelayedValue } from './useDelayedValue.js';

export default function Canvas() {
  const pos1 = usePointerPosition();
  const pos2 = useDelayedValue(pos1, 100);
  const pos3 = useDelayedValue(pos2, 200);
  const pos4 = useDelayedValue(pos3, 100);
  const pos5 = useDelayedValue(pos4, 50);

  return (
    <>
      <Dot position={pos1} opacity={1} />
      <Dot position={pos2} opacity={0.8} />
      <Dot position={pos3} opacity={0.6} />
      <Dot position={pos4} opacity={0.4} />
      <Dot position={pos5} opacity={0.2} />
    </>
  );
}

function Dot({ position, opacity }) {
  return (
    <div style={{
      position: 'absolute',
      backgroundColor: 'pink',
      borderRadius: '50%',
      opacity,
      transform: `translate(${position.x}px, ${position.y}px)`,
      pointerEvents: 'none',
      left: -20,
      top: -20,
      width: 40,
      height: 40,
    }} />
  );
}
```

***

## 🔑 Points importants à retenir

1. **Un custom Hook est une fonction** → il commence toujours par `use` (`useAuth`, `useFetch`, etc.).
2. **Un Hook peut utiliser d’autres Hooks** (`useState`, `useEffect`, ou même d’autres custom Hooks).
3. **On peut les composer** : `usePointerPosition` → `useDelayedValue` → résultat combiné.
4. **Ils permettent de factoriser la logique** : éviter la duplication de code et garder les composants plus lisibles.
5. **La communauté en propose déjà plein** : `react-use`, `usehooks-ts`, etc.

***

👉 Résumé :\
Les **custom Hooks** te permettent d’encapsuler de la logique réutilisable, de la partager entre composants et de composer plusieurs comportements.\
Plus ton application grandit, plus tu écriras **moins d’`useEffect` à la main**, car tu réutiliseras des Hooks déjà construits.
