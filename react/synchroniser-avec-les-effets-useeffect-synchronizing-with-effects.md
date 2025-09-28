# Synchroniser avec les Effets (useEffect) - (Synchronizing with Effects)

Certains composants doivent se **synchroniser avec des systèmes externes**.\
Par exemple :

* contrôler un composant **hors React** (ex. un lecteur vidéo natif du navigateur),
* établir une **connexion serveur** (ex. WebSocket, chat),
* envoyer un **événement analytique** quand un composant apparaît à l’écran.

👉 Pour cela, React fournit les **Effets** (via le hook `useEffect`).\
Un **Effet** te permet d’exécuter du code **après le rendu**, afin de synchroniser ton composant avec l’extérieur.

***

### 🎯 Ce que tu vas apprendre

1. **Ce que sont les Effets**
2. **En quoi ils sont différents des événements**
3. **Comment déclarer un Effet dans un composant**
4. **Comment éviter de relancer un Effet inutilement**
5. **Pourquoi les Effets s’exécutent deux fois en développement et comment corriger cela**

***

### 🔹 Qu’est-ce qu’un Effet ?

Un **Effet** est un morceau de code qui s’exécute **après que React a mis à jour le DOM**.\
Tu le déclares avec :

```jsx
import { useEffect } from "react";

useEffect(() => {
  // Code à exécuter après chaque rendu
});
```

Exemple : connecter/déconnecter un chat quand le composant est monté ou démonté :

```jsx
import { useEffect } from "react";
import { createConnection } from "./chat.js";

export default function ChatRoom() {
  useEffect(() => {
    const connection = createConnection();
    connection.connect();

    // Fonction de nettoyage (cleanup) exécutée à la désactivation
    return () => connection.disconnect();
  }, []); // [] → exécution uniquement au montage/démontage

  return <h1>Bienvenue dans le chat !</h1>;
}
```

***

### 🔹 Différence entre **Effets** et **Événements**

*   **Événements** : s’exécutent uniquement lorsqu’une interaction se produit (clic, input, submit).

    ```jsx
    <button onClick={() => console.log("Clic!")}>Clique-moi</button>
    ```
* **Effets** : s’exécutent automatiquement **après le rendu** → parfaits pour se synchroniser avec l’extérieur.

***

### 🔹 Contrôler quand l’Effet se relance

L’Effet peut dépendre de **valeurs réactives** (props, state).

```jsx
useEffect(() => {
  console.log("Connexion mise à jour pour roomId :", roomId);

  const connection = createConnection(roomId);
  connection.connect();

  return () => connection.disconnect();
}, [roomId]); // Effet relancé uniquement si roomId change
```

👉 Ici, `roomId` est dans la **liste des dépendances**.\
Si `roomId` change → l’Effet est nettoyé puis ré-exécuté.

***

### 🔹 Pourquoi les Effets s’exécutent **2 fois en développement** ?

En **React Strict Mode**, pour t’aider à détecter les problèmes :

1. React monte le composant → exécute l’Effet.
2. React le démonte immédiatement → exécute le nettoyage.
3. Puis il le remonte → ré-exécute l’Effet.

➡️ En production, l’Effet ne s’exécute **qu’une seule fois**.\
➡️ En dev, cela sert à vérifier que ton **nettoyage est bien écrit**.

***

✅ **Résumé**

* Les **Effets** (useEffect) servent à synchroniser React avec un système externe.
* Ils s’exécutent après chaque rendu (selon leurs dépendances).
* Ils sont différents des **événements**, qui ne s’exécutent que sur interaction utilisateur.
* Les dépendances contrôlent quand l’Effet est relancé.
* En dev, les Effets s’exécutent deux fois (Strict Mode) → normal et utile pour détecter les bugs.

## 🌟 Qu’est-ce que les **Effets** et en quoi diffèrent-ils des **Événements** ?

Avant de comprendre les **Effets** dans React, il faut distinguer deux grands types de logique à l’intérieur des composants :

***

### 1. 🔹 Le **code de rendu**

* Placé **au niveau supérieur** du composant.
* Il utilise **les props et l’état** pour produire du JSX.
* Il doit rester **pur** : comme une fonction mathématique, il calcule uniquement le résultat, **sans effet de bord** (pas de modification d’état, pas d’appel API, pas d’accès DOM direct, etc.).

Exemple :

```jsx
function Welcome({ name }) {
  return <h1>Hello, {name}!</h1>; // rendu pur
}
```

***

### 2. 🔹 Les **gestionnaires d’événements** (_event handlers_)

* Ce sont des fonctions **imbriquées** dans le composant.
* Elles réagissent à une action précise de l’utilisateur (clic, saisie clavier, soumission d’un formulaire, etc.).
* Elles provoquent des **effets de bord** ciblés (modifier l’état, envoyer une requête HTTP, changer de page, etc.).

Exemple :

```jsx
function Form() {
  function handleSubmit() {
    // ⚡ Effet déclenché par un événement utilisateur
    alert("Form submitted!");
  }

  return <button onClick={handleSubmit}>Submit</button>;
}
```

***

### 3. 🔹 Limites de cette approche

Parfois, il faut déclencher une action **sans qu’il y ait un événement spécifique**.\
👉 Exemple : un composant `ChatRoom` doit se connecter au serveur de chat **dès qu’il apparaît à l’écran**.

⚠️ Problème :

* Ce n’est pas un rendu pur (car connecter un serveur ≠ simple calcul).
* Ce n’est pas non plus un événement précis de l’utilisateur (le composant peut apparaître pour différentes raisons).

***

### 4. 🔹 Les **Effets (useEffect)**

Les **Effets** sont faits pour ce type de logique.

* Ils permettent de définir des **effets de bord déclenchés par le rendu lui-même**.
* Ils s’exécutent **après que React a mis à jour l’interface** (fin de la phase _commit_).
* C’est le moment idéal pour **synchroniser ton composant React avec un système externe** :
  * établir une connexion réseau,
  * intégrer une librairie tierce,
  * envoyer une mesure analytique,
  * manipuler un élément DOM.

Exemple avec une connexion à un serveur :

```jsx
import { useEffect } from "react";
import { connectToServer } from "./chat";

function ChatRoom() {
  useEffect(() => {
    const connection = connectToServer();
    connection.connect();

    // Nettoyage quand le composant disparaît
    return () => connection.disconnect();
  }, []); // [] → exécution seulement au montage/démontage

  return <h1>Bienvenue dans le chat</h1>;
}
```

***

### 5. 🔹 Différence **Événements vs Effets**

| **Événements** ⚡                                                                         | **Effets** 🔄                                                                                    |
| ---------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------ |
| Déclenchés par une action **spécifique de l’utilisateur** (clic, saisie, scroll).        | Déclenchés par le **rendu lui-même** (quand un composant apparaît ou se met à jour).             |
| Contiennent des effets de bord **ciblés** (soumettre un formulaire, envoyer un message). | Contiennent des effets de bord **automatiques** (connexion serveur, synchro avec une API, logs). |
| Exemple : envoyer un message en cliquant sur “Envoyer”.                                  | Exemple : établir une connexion au serveur quand `ChatRoom` est monté.                           |

***

✅ **Résumé**

* Le **rendu** : calcul pur, JSX uniquement.
* Les **événements** : effets de bord déclenchés par l’utilisateur.
* Les **Effets** : effets de bord déclenchés par le rendu React lui-même.

## ⚛️ Tu n’as peut-être pas besoin d’un Effet (useEffect)

Il est très tentant d’ajouter un **Effet** (`useEffect`) partout dans ses composants, mais ce n’est pas toujours nécessaire ni recommandé.\
👉 Les **Effets sont surtout utiles pour synchroniser React avec un système externe** (API navigateur, widget tiers, serveur, analytics, etc.).

Si ton **Effet ne fait que mettre à jour un état en fonction d’un autre état**, il y a de fortes chances que tu **n’aies pas besoin d’un useEffect**. Dans ce cas, tu peux souvent résoudre ton problème avec du simple **rendu dérivé** (calculer une valeur à partir des props ou state dans le JSX) ou des **hooks comme useMemo**.

***

### ✅ Comment écrire un Effet correctement

Il existe **3 étapes clés** pour écrire un Effet :

#### 1. Déclarer un Effet

Importer `useEffect` et l’appeler en haut de ton composant :

```jsx
import { useEffect } from "react";

function MyComponent() {
  useEffect(() => {
    // Ce code s’exécute après CHAQUE rendu
    console.log("Effet exécuté !");
  });

  return <div>Bonjour</div>;
}
```

⚠️ Par défaut, un Effet s’exécute **après chaque rendu** (y compris les re-renders).

***

#### 2. Spécifier les dépendances

Dans la majorité des cas, tu ne veux pas que ton Effet s’exécute après chaque rendu.\
Tu veux qu’il **s’exécute uniquement quand c’est nécessaire**.

👉 Exemple : connexion à un chat = seulement au **montage** du composant, et reconnexion si l’ID du chat change :

```jsx
useEffect(() => {
  const connection = createConnection(roomId);
  connection.connect();

  return () => connection.disconnect(); // Nettoyage
}, [roomId]); 
```

Ici :

* `[roomId]` → l’Effet se relance uniquement si `roomId` change.
* Le `return` permet de **nettoyer** (disconnect) avant de relancer l’Effet.

***

#### 3. Ajouter un nettoyage si nécessaire

Certains Effets nécessitent un **nettoyage** pour éviter les fuites mémoire ou les comportements indésirables :

* `connect` → `disconnect`
* `subscribe` → `unsubscribe`
* `setInterval` → `clearInterval`
* `fetch` → `abort` ou ignorer la réponse

Exemple avec un `setInterval` :

```jsx
useEffect(() => {
  const id = setInterval(() => {
    console.log("tick");
  }, 1000);

  return () => clearInterval(id); // Nettoyage
}, []);
```

***

### 🎬 Exemple concret : synchroniser une vidéo avec React

Tu veux contrôler la lecture d’une vidéo via une prop `isPlaying`.\
Le navigateur n’a pas de prop `isPlaying`, donc tu dois utiliser les méthodes DOM `play()` et `pause()`.

❌ Mauvaise approche (pendant le rendu, DOM pas encore dispo) :

```jsx
if (isPlaying) {
  ref.current.play();  // Erreur
} else {
  ref.current.pause(); // Erreur
}
```

✅ Bonne approche avec `useEffect` :

```jsx
import { useRef, useEffect } from "react";

function VideoPlayer({ src, isPlaying }) {
  const ref = useRef(null);

  useEffect(() => {
    if (isPlaying) {
      ref.current.play();
    } else {
      ref.current.pause();
    }
  }, [isPlaying]); // synchronisation quand isPlaying change

  return <video ref={ref} src={src} loop playsInline />;
}
```

Ainsi, React met d’abord à jour le DOM (`<video>` rendu), puis ton Effet exécute `play()` ou `pause()`.

***

### ⚠️ Piège : la boucle infinie

Si tu mets un `setState` direct dans un Effet **sans dépendances**, tu crées une boucle infinie :

```jsx
const [count, setCount] = useState(0);

useEffect(() => {
  setCount(count + 1); // ⚠️ provoque un re-render infini
});
```

Pourquoi ?

* L’Effet dépend du rendu.
* `setCount` déclenche un nouveau rendu.
* Le rendu relance l’Effet.
* Et ainsi de suite... 🔄

👉 Solution : ne fais ça **que si tu synchronises avec un système externe**.\
Sinon, calcule l’état dérivé directement dans le JSX ou avec `useMemo`.

***

### ✅ Récap

* **Effets = synchronisation avec l’extérieur** (API, serveur, DOM, analytics).
* Pas besoin d’Effet pour **calculer un nouvel état à partir d’un autre**.
* 3 étapes :
  1. Déclarer (`useEffect`)
  2. Dépendances (`[roomId]`)
  3. Nettoyage (`return () => …`)
* ⚠️ Attention aux boucles infinies si `setState` est mal placé.

## ⚛️ Étape 2 : Spécifier les dépendances d’un Effet

Par défaut, **un Effet (`useEffect`) s’exécute après chaque rendu**.\
👉 Mais ce comportement n’est pas toujours souhaitable :

* **Parfois, c’est lent** : par ex. se reconnecter à un serveur de chat à chaque frappe clavier.
* **Parfois, c’est faux** : par ex. rejouer une animation de fade-in à chaque re-render, alors qu’elle devrait se déclencher **seulement une fois**.

***

### 🚨 Problème sans dépendances

Prenons l’exemple du `VideoPlayer` :

```jsx
import { useState, useRef, useEffect } from 'react';

function VideoPlayer({ src, isPlaying }) {
  const ref = useRef(null);

  useEffect(() => {
    if (isPlaying) {
      console.log('Appel à video.play()');
      ref.current.play();
    } else {
      console.log('Appel à video.pause()');
      ref.current.pause();
    }
  }); // ⚠️ pas de dépendances

  return <video ref={ref} src={src} loop playsInline />;
}
```

➡️ Ici, **chaque frappe dans un input** provoque un re-render du parent → l’Effet se relance → inutilement `play()` ou `pause()` sont appelés.

***

### ✅ Solution : ajouter le tableau de dépendances

On peut demander à React de **n’exécuter l’Effet que si certaines valeurs changent** :

```jsx
useEffect(() => {
  if (isPlaying) {
    console.log("Appel à video.play()");
    ref.current.play();
  } else {
    console.log("Appel à video.pause()");
    ref.current.pause();
  }
}, [isPlaying]); // 🔑 l’Effet ne dépend que de isPlaying
```

👉 Maintenant, taper dans l’input **ne relance plus l’Effet**.\
Seul un clic sur Play/Pause le relance, car `isPlaying` change.

***

### 📌 Règles importantes

* Le **linter React** t’indiquera si tu oublies une dépendance.\
  Exemple : `isPlaying` est utilisé dans l’Effet, donc il **doit être listé**.
* **Tu ne choisis pas arbitrairement tes dépendances**.\
  → Elles doivent **refléter exactement** les variables utilisées dans ton Effet.\
  Si ton code dépend de `roomId` et `theme`, alors `[roomId, theme]`.
* Pour ne pas ré-exécuter un Effet, **change ton code** pour qu’il n’ait plus besoin de cette dépendance (et non pas supprimer la dépendance de la liste !).

***

### ⚖️ Différences entre les cas

```jsx
useEffect(() => {
  // S’exécute après CHAQUE rendu
});

useEffect(() => {
  // S’exécute seulement au montage du composant
}, []);

useEffect(() => {
  // S’exécute au montage + à chaque changement de a ou b
}, [a, b]);
```

***

### 🔍 Cas particuliers : refs et setState

Pourquoi dans notre exemple on n’ajoute **pas** `ref` dans les dépendances ?\
Parce que :

* `useRef` retourne **toujours le même objet** (identité stable).
* Pareil pour les fonctions `setX` de `useState`.
* Donc, inutile de les mettre, **sauf si le linter l’exige** (ex. si la ref vient d’un parent).

***

### ✅ Récap

* Un Effet **sans tableau** → s’exécute après chaque rendu.
* Un Effet avec `[]` → seulement au montage.
* Un Effet avec `[a, b]` → au montage + si `a` ou `b` changent.
* Les dépendances ne sont **pas au choix**, elles reflètent ton code.
* `ref` et `setState` n’ont pas besoin d’être dans la liste (identité stable).

## ⚛️ Étape 3 : Ajouter un _cleanup_ si nécessaire

Quand on écrit un Effet avec `useEffect`, il ne suffit pas toujours de "faire quelque chose" → il faut aussi **prévoir comment arrêter, annuler ou nettoyer cette action**.\
👉 Sinon, on risque d’empiler des comportements parasites (ex. plusieurs connexions serveur, timers non arrêtés, écouteurs d’événements accumulés…).

***

### 📝 Exemple sans cleanup

On veut qu’un composant `ChatRoom` se connecte à un serveur de chat :

```jsx
import { useEffect } from 'react';
import { createConnection } from './chat.js';

export default function ChatRoom() {
  useEffect(() => {
    const connection = createConnection();
    connection.connect();
  }, []); // 👀 seulement au montage
  return <h1>Bienvenue dans le chat !</h1>;
}
```

#### 🔎 Problème

* En dev, on voit **"✅ Connecting..."** affiché **deux fois**.
* Pourquoi ? Parce que React (en mode Strict) **monte → démonte → remonte** immédiatement le composant pour vérifier si ton code est robuste.

➡️ Or ici, tu connectes bien, mais tu **n’as jamais déconnecté**.\
Résultat : chaque montage ajoute une connexion supplémentaire.

***

### ✅ Ajout du cleanup

Pour corriger : retourne une fonction de nettoyage dans `useEffect`.

```jsx
import { useEffect } from 'react';
import { createConnection } from './chat.js';

export default function ChatRoom() {
  useEffect(() => {
    const connection = createConnection();
    connection.connect();

    return () => {
      connection.disconnect(); // 🔑 nettoyage
    };
  }, []); // 👌 toujours seulement au montage/démontage

  return <h1>Bienvenue dans le chat !</h1>;
}
```

#### 🖥️ Console en développement

```
✅ Connecting...
❌ Disconnected.
✅ Connecting...
```

***

### ⚙️ Fonctionnement de `cleanup`

* **Avant chaque nouvel appel de l’Effet**, React exécute d’abord la fonction de cleanup.
* **Au démontage du composant**, React exécute aussi la fonction de cleanup une dernière fois.

C’est ce qui garantit que ton composant :

* 🔄 ne laisse pas traîner de connexions,
* 🚫 ne crée pas de fuites mémoire,
* ✅ se comporte correctement même après navigation ou rechargement.

***

### 📌 À retenir

1. **Toujours nettoyer** ce qui peut être annulé :
   * `setInterval` → `clearInterval`
   * `addEventListener` → `removeEventListener`
   * `connect` → `disconnect`
   * `fetch` → ignorer/abandonner
2. Le **double appel en dev est normal** (Strict Mode).\
   👉 En prod, ton Effet s’exécutera **une seule fois** au montage.
3. Un Effet correctement écrit doit donner **le même résultat visible**, qu’il soit exécuté une fois ou plusieurs fois avec cleanup.

## ⚛️ Comment gérer le double déclenchement des Effets en développement ?

En **mode développement**, React remonte volontairement (_remount_) vos composants pour détecter des bugs cachés.\
👉 La vraie question n’est pas _« Comment exécuter un Effet une seule fois ? »_, mais bien :\
➡️ _« Comment écrire mon Effet pour qu’il fonctionne correctement même après un cycle montage → nettoyage → remontage ? »_

***

### ✅ La bonne pratique : toujours prévoir un cleanup

La règle d’or :

> L’utilisateur **ne doit pas voir de différence** entre un Effet exécuté **une seule fois** (prod) et un cycle complet **setup → cleanup → setup** (dev).

***

### 🚩 Mauvaise solution : bloquer avec un `ref`

Exemple de **mauvaise approche** :

```jsx
const connectionRef = useRef(null);

useEffect(() => {
  // 🚩 Ceci cache juste le problème en dev
  if (!connectionRef.current) {
    connectionRef.current = createConnection();
    connectionRef.current.connect();
  }
}, []);
```

* En dev, ça affiche bien `"✅ Connecting..."` **une seule fois**.
* Mais ⚠️ au démontage, **la connexion n’est jamais fermée** !
* Résultat : si l’utilisateur navigue et revient, les connexions s’accumulent → fuite mémoire, comportements imprévus.

***

### ✅ Bonne solution : cleanup systématique

On doit **toujours nettoyer** dans le return de `useEffect` :

```jsx
useEffect(() => {
  const connection = createConnection();
  connection.connect();

  return () => {
    connection.disconnect(); // 🔑 nettoyage obligatoire
  };
}, []);
```

Avec ce code :

* En dev → `"✅ Connecting..." → "❌ Disconnected." → "✅ Connecting..."`
* En prod → `"✅ Connecting..."` (une seule fois)

Dans les **deux cas**, ton app reste saine : pas de connexions fantômes.

***

### 🔄 Cas fréquents d’Effets + cleanup

| Effet              | Setup (dans `useEffect`)                | Cleanup (dans `return`)                    |
| ------------------ | --------------------------------------- | ------------------------------------------ |
| ⏱️ Timer           | `setInterval(fn, 1000)`                 | `clearInterval(id)`                        |
| 🖱️ Événement DOM  | `window.addEventListener('resize', fn)` | `window.removeEventListener('resize', fn)` |
| 🌐 WebSocket       | `socket.connect()`                      | `socket.disconnect()`                      |
| 📡 Fetch abortable | `fetch(url, { signal })`                | `controller.abort()`                       |
| 🎬 Animation API   | `startAnimation()`                      | `stopAnimation()`                          |

***

### 📌 À retenir

* Le **double déclenchement en dev est normal et voulu** (Strict Mode).
* Ne cherchez pas à “empêcher” un Effet de se lancer deux fois.
* La bonne pratique est de **rendre vos Effets idempotents** (capables de supporter plusieurs cycles grâce au cleanup).
* En prod, l’Effet s’exécute **une seule fois** au montage (ou selon vos dépendances).

## ⚛️ Contrôler des widgets non-React avec les Effets

Il arrive que tu doives utiliser des **composants externes** qui ne sont pas écrits en React (par exemple : une carte interactive, un widget jQuery, une librairie externe, etc.). Ces éléments possèdent souvent des **APIs impératives** (méthodes comme `setZoomLevel()`, `open()`, `showModal()`, etc.) qu’il faut synchroniser avec l’état React.

C’est exactement ce pour quoi les **Effets (`useEffect`)** sont utiles : faire le lien entre React et le monde extérieur.

***

### 🎯 Exemple 1 : synchroniser un niveau de zoom

Supposons que tu utilises un widget carte avec une méthode `setZoomLevel()`.\
Tu veux que la valeur de `zoomLevel` (stockée dans ton state React) reste synchronisée avec le widget.

```jsx
useEffect(() => {
  const map = mapRef.current;
  map.setZoomLevel(zoomLevel);
}, [zoomLevel]);
```

* Ici, **pas besoin de cleanup**.
* Si React appelle l’Effet deux fois en développement (Strict Mode), le widget recevra deux fois la même valeur → pas d’impact visible.
* En production, l’Effet ne sera appelé qu’une seule fois par changement de `zoomLevel`.

***

### 🎯 Exemple 2 : un widget qui n’accepte pas les appels répétés

Certains widgets ou APIs du navigateur **n’autorisent pas d’appel répété**.\
Exemple : l’élément natif `<dialog>` → sa méthode `.showModal()` déclenche une erreur si on l’appelle deux fois.

👉 Solution : utiliser un cleanup (`return () => …`) pour fermer le widget avant chaque nouveau montage.

```jsx
useEffect(() => {
  const dialog = dialogRef.current;
  dialog.showModal(); // ouverture

  return () => dialog.close(); // cleanup : fermeture
}, []);
```

En développement (Strict Mode), le cycle sera :

1. `showModal()` (ouverture)
2. `close()` (fermeture)
3. `showModal()` à nouveau (réouverture)

➡️ Résultat final identique à la prod : le modal est ouvert une seule fois à l’écran.\
➡️ Le code reste robuste et ne plante pas.

***

### ✅ Bonnes pratiques

* Utilise `useEffect` pour **synchroniser l’état React** avec un widget externe.
* Si l’API tolère les appels répétés → pas besoin de cleanup.
* Si l’API **ne tolère pas** les doublons (ex. `showModal`) → ajoute un cleanup.
* Toujours tester en **Strict Mode** pour vérifier que tes Effets sont idempotents (supportent le cycle `setup → cleanup → setup`).

## ⚛️ S’abonner à des événements avec les Effets

Un cas très courant d’utilisation des **Effets (`useEffect`)** est la gestion des **abonnements** (events, sockets, listeners, etc.).\
L’idée est simple :

* Quand ton composant s’affiche → tu **t’abonnes** (setup).
* Quand ton composant disparaît ou est mis à jour → tu **te désabonnes** (cleanup).

***

### 🎯 Exemple : écouter le scroll de la fenêtre

```jsx
useEffect(() => {
  function handleScroll(e) {
    console.log(window.scrollX, window.scrollY);
  }

  // ✅ On s'abonne à l'événement
  window.addEventListener('scroll', handleScroll);

  // ✅ Cleanup : on se désabonne
  return () => window.removeEventListener('scroll', handleScroll);
}, []);
```

#### 🔎 Explication

* `useEffect` s’exécute après le rendu → ajoute le listener `scroll`.
* Le `return` dans l’Effet définit le **cleanup** → supprime le listener quand le composant est démonté ou re-monté.
* En **mode développement (Strict Mode)** :
  * React monte le composant → `addEventListener()`
  * React démonte immédiatement → `removeEventListener()`
  * React remonte à nouveau → `addEventListener()`\
    ➝ Résultat : **toujours un seul abonnement actif** à la fois, comme en production.

***

### ✅ Bonnes pratiques

1. Toujours retourner une fonction de **cleanup** quand tu utilises `addEventListener`, `setInterval`, WebSocket `.subscribe()`, etc.
2. Évite les effets sans cleanup → sinon tu risques des **fuites de mémoire** ou des abonnements doublés.
3. Ton cleanup doit **inverser précisément ce que tu as fait** dans le setup (ex. `unsubscribe`, `disconnect`, `clearInterval`).

## ⚛️ Déclencher des animations avec les Effets

Les **Effets (`useEffect`)** sont aussi utilisés pour **déclencher des animations** quand un composant apparaît.\
Mais comme toujours avec React, il faut penser à la **fonction de nettoyage (cleanup)** pour garantir un comportement cohérent en développement **et** en production.

***

### 🎯 Exemple simple avec style inline

```jsx
useEffect(() => {
  const node = ref.current;
  node.style.opacity = 1; // ✅ Déclenche l’animation (fade-in)

  return () => {
    node.style.opacity = 0; // ✅ Reset à l’état initial
  };
}, []);
```

#### 🔎 Explication

* **Montage du composant** → `opacity = 1` → animation visible.
* **Cleanup (démontage ou remount en dev)** → `opacity = 0` → l’état est remis à zéro.
* **Mode développement (Strict Mode)** :
  * `opacity = 1`
  * immédiatement reset → `opacity = 0`
  * puis de nouveau → `opacity = 1`\
    ➝ Résultat : pour l’utilisateur, c’est **comme si on avait juste fait `opacity = 1`** dès le début (comme en prod).

***

### ⚡ Exemple avec une librairie d’animation (ex. GSAP ou Anime.js)

Si tu utilises une lib de tweening (animations complexes), le **cleanup doit réinitialiser la timeline** :

```jsx
useEffect(() => {
  const animation = gsap.to(ref.current, {
    opacity: 1,
    duration: 1,
  });

  return () => {
    animation.kill(); // ✅ Stop et reset l’animation
    gsap.set(ref.current, { opacity: 0 }); // ✅ Retour état initial
  };
}, []);
```

***

### ✅ Bonnes pratiques pour les animations avec `useEffect`

1. Toujours **réinitialiser à l’état initial** dans le cleanup.
2. Évite de manipuler directement le DOM si tu peux utiliser **CSS transitions** ou **Framer Motion** (plus déclaratif).
3. En dev, ne cherche pas à éviter le double déclenchement → au contraire, le cleanup permet de tester la **robustesse de ton animation**.

## ⚛️ Récupérer des données avec les Effets (`useEffect`)

L’un des cas d’usage les plus fréquents des **Effets (`useEffect`)** est le **fetch de données**.\
Mais il faut gérer correctement le cycle de vie pour éviter des **problèmes de concurrence** (race conditions).

***

### 🚀 Exemple : fetch avec nettoyage

```jsx
useEffect(() => {
  let ignore = false;

  async function startFetching() {
    const json = await fetchTodos(userId);
    if (!ignore) {
      setTodos(json); // ✅ On ne met à jour que si la requête est encore valide
    }
  }

  startFetching();

  return () => {
    ignore = true; // ✅ On ignore le résultat si l’Effect est "nettoyé"
  };
}, [userId]);
```

#### 🔎 Explication

* Lors du montage ou quand `userId` change → on déclenche le fetch.
* Si l’utilisateur change vite d’utilisateur (`Alice → Bob`), l’ancienne requête (`Alice`) peut toujours répondre plus tard.
* Grâce à `ignore = true` dans le cleanup, on s’assure que ce résultat **n’écrase pas l’état de `Bob`**.

***

### 🧑‍💻 En mode développement

* React exécute l’Effect **deux fois** (Strict Mode).
* Tu verras donc **deux requêtes dans le Network tab**.
* Mais : le premier Effect est immédiatement nettoyé (`ignore = true`), donc son résultat est **ignoré**.\
  ➡️ Aucun impact négatif, seulement un peu plus de bruit en dev.

### 🏗️ En production

* L’Effect ne s’exécute qu’une seule fois.
* Donc **une seule requête** part réellement.

***

### ⚠️ Limites du fetch direct dans `useEffect`

1. **Pas exécuté côté serveur** → ton HTML initial n’a pas les données (seulement un loader).
2. **Waterfall réseau** → si un parent fetch, puis un enfant fetch, tu attends en série → plus lent.
3. **Pas de cache** → si le composant se démonte/remonte, il refait la requête.
4. **Beaucoup de boilerplate** → gestion des erreurs, annulation, état loading…

***

### ✅ Alternatives recommandées

👉 **Selon ton setup React, préfère une de ces approches :**

* **Framework (Next.js, Remix, etc.)**\
  Utilise leur **système intégré de data fetching** (`getServerSideProps`, `loader()`, `server actions`…) → plus efficace (server-side rendering, pas de waterfall).
* **Bibliothèque de cache côté client**
  * TanStack Query (React Query)
  * SWR
  * React Router Data APIs (6.4+)\
    Ces libs gèrent :
  * la **déduplication des requêtes**,
  * le **cache**,
  * la **synchronisation automatique**,
  * et évitent les **race conditions**.
* **Solution maison**\
  Tu peux bâtir ton propre mini-cache avec `useContext` ou `zustand`, en utilisant `useEffect` en interne mais en ajoutant la logique manquante (cache, abort controller, etc.).

***

✅ En résumé :

* Tu peux fetch avec `useEffect`, mais il faut gérer le **cleanup** pour éviter les bugs.
* En **production**, une seule requête partira.
* Pour des applis modernes, préfère une **solution de cache** (React Query, SWR, data APIs de ton framework).

## 📊 Envoi d’analytics avec les **Effets (`useEffect`)**

Prenons un exemple simple où tu veux envoyer un événement d’analytics lorsqu’un utilisateur visite une page :

```jsx
useEffect(() => {
  logVisit(url); // Envoie une requête POST à ton service d’analytics
}, [url]);
```

***

### 🔎 Comportement en développement

* En mode **Strict Mode** (par défaut dans React 18), l’Effect s’exécute **deux fois**.
* Donc `logVisit(url)` sera appelé **deux fois pour chaque URL**.
* C’est normal et voulu → React **remonte puis démonte le composant** pour vérifier que ton code gère bien le montage/démontage proprement.

⚠️ Mais en pratique :

* Tu **ne veux pas envoyer d’analytics réelles en dev** (cela fausserait tes métriques).
* Ton composant se remonte de toute façon à chaque fois que tu sauvegardes le fichier → encore plus de visites fantômes.

***

### ✅ Comportement en production

* Pas de double appel.
* `logVisit(url)` est exécuté **une seule fois par changement d’URL**.\
  ➡️ Tes logs seront **fiables** et non dupliqués.

***

### 🛠️ Bonnes pratiques

1. **Ne corrige pas artificiellement le “double appel” en dev** → ce n’est pas un bug, et ce n’impacte pas la prod.
2. **Désactive l’envoi d’analytics en local** → configure ton `logVisit` pour ne rien faire en `NODE_ENV=development`.
3. **Debugge en staging** → déploie sur un environnement de préprod qui tourne en mode production pour tester les envois réels.
4. **Alternatives** :
   * Tu peux aussi envoyer les analytics **au niveau du routeur** (par exemple, sur chaque `onRouteChange`).
   * Pour un suivi plus fin, utilise un **IntersectionObserver** → utile pour savoir quand un composant est visible à l’écran et combien de temps.

***

✅ **Résumé** :

* En dev → `logVisit` s’exécute deux fois, c’est attendu et sans conséquence réelle.
* En prod → une seule exécution par page visitée.
* La bonne pratique est **d’ignorer les analytics en développement** et de tester en staging.

## 🚀 Pas un _Effect_ : Initialiser l’application

Certaines logiques doivent s’exécuter **une seule fois au démarrage de l’application**, et **pas à chaque rendu de composant**.\
Dans ce cas, il ne faut pas utiliser un `useEffect`, mais mettre ce code **en dehors de tes composants**.

***

### ✅ Exemple

```jsx
// S’exécute une seule fois au chargement de l’app (côté client)
if (typeof window !== 'undefined') {  
  checkAuthToken();              // Vérifie si un token d’auth existe
  loadDataFromLocalStorage();    // Charge des données persistées
}

function App() {
  return (
    <div>
      <h1>Mon application React</h1>
    </div>
  );
}
```

***

### 🔎 Pourquoi en dehors des composants ?

* 🔁 Si tu mets ce code **dans un composant avec `useEffect([])`**, il sera exécuté à chaque fois que ce composant est **monté**.\
  → Si le composant se démonte/remonte, ton code serait relancé inutilement.
* 📦 En le plaçant **en dehors**, tu garantis qu’il ne s’exécute **qu’une fois** au lancement de l’application (au premier chargement du bundle JS).
* 🌍 Le `if (typeof window !== 'undefined')` évite les erreurs lors du rendu côté serveur (**SSR**), car `window` n’existe pas dans Node.js.

***

### 🛠️ Cas d’usage typiques

* Vérifier si un **token JWT** ou une session existe déjà.
* Charger des données depuis **`localStorage`** ou **`sessionStorage`**.
* Initialiser un **service tiers** (par exemple Firebase, Sentry, Google Analytics).
* Configurer un **cache global** ou une **connexion WebSocket** partagée.

## 🛑 Pas un _Effect_ : Acheter un produit

Certains effets ne doivent **jamais** être placés dans un `useEffect`, même avec un cleanup, car ils entraînent des conséquences visibles pour l’utilisateur s’ils sont exécutés deux fois.

***

### 🚨 Exemple incorrect

```jsx
useEffect(() => {
  // ❌ Mauvais : cet effet s’exécute deux fois en dev,
  // et pourrait acheter deux fois le produit !
  fetch('/api/buy', { method: 'POST' });
}, []);
```

➡️ Ici, **l’achat se déclenche dès que la page est rendue**.

* En mode développement, React **remonte** les composants deux fois → donc 2 achats.
* Même en production, si l’utilisateur va sur une autre page puis fait **Retour**, ton effet se relancerait → achat répété.

Ce comportement est **buggé**, car acheter un produit n’est pas lié à l’affichage de la page.

***

### ✅ La bonne approche : via un événement utilisateur

L’achat doit être déclenché par une interaction (ex : clic sur un bouton), **pas par un rendu**.

```jsx
export default function ProductPage() {
  function handleClick() {
    // ✅ Acheter est un événement utilisateur
    fetch('/api/buy', { method: 'POST' });
  }

  return (
    <div>
      <h1>Produit</h1>
      <button onClick={handleClick}>
        Acheter
      </button>
    </div>
  );
}
```

***

### 🔑 À retenir

* **Un effet (`useEffect`) = causé par un rendu** (ex : connecter un chat quand il apparaît).
* **Un événement (`onClick`, `onSubmit`) = causé par une interaction** (ex : acheter un produit, envoyer un formulaire).
* Si le fait de **remonter un composant** casse ta logique → ça veut dire que ton code ne devrait **pas** être dans un `useEffect`.

## Tout mettre ensemble — Playground des Effets

Ce **playground** permet de vraiment _ressentir_ comment les **Effets (`useEffect`)** fonctionnent en pratique.

***

#### Exemple avec un `setTimeout`

```jsx
import { useState, useEffect } from 'react';

function Playground() {
  const [text, setText] = useState('a');

  useEffect(() => {
    function onTimeout() {
      console.log('⏰ ' + text);
    }

    console.log('🔵 Planifier "' + text + '"');
    const timeoutId = setTimeout(onTimeout, 3000);

    return () => {
      console.log('🟡 Annuler "' + text + '"');
      clearTimeout(timeoutId);
    };
  }, [text]);

  return (
    <>
      <label>
        Texte à afficher :{' '}
        <input
          value={text}
          onChange={e => setText(e.target.value)}
        />
      </label>
      <h1>{text}</h1>
    </>
  );
}

export default function App() {
  const [show, setShow] = useState(false);
  return (
    <>
      <button onClick={() => setShow(s => !s)}>
        {show ? 'Démonter' : 'Monter'} le composant
      </button>
      <hr />
      {show && <Playground />}
    </>
  );
}
```

***

### Ce qui se passe

*   **Au premier montage (en mode développement avec Strict Mode)**\
    Tu verras dans la console :

    * `🔵 Planifier "a"`
    * `🟡 Annuler "a"`
    * `🔵 Planifier "a"`

    👉 React monte, démonte puis remonte immédiatement ton composant une fois en dev pour vérifier que ton **cleanup** (nettoyage) est correct.\
    En production, tu n’auras qu’une seule planification.

***

*   **Quand tu tapes dans l’input**\
    Chaque frappe :

    * Annule le timeout précédent (`🟡 Annuler "ancien texte"`)
    * Planifie un nouveau timeout (`🔵 Planifier "nouveau texte"`)

    👉 Il n’y a **jamais plus d’un timeout actif en même temps**.

***

* **Quand tu démontes le composant (bouton “Démonter”)**\
  React appelle une dernière fois la fonction de nettoyage, ce qui annule le dernier timeout.\
  👉 Aucun log “perdu” n’apparaît après démontage.

***

*   **Si tu commentes le cleanup (`return () => {...}`)**\
    Les timeouts ne seront pas annulés.\
    Si tu tapes vite `abcde`, trois secondes plus tard tu verras :

    * `a`
    * `ab`
    * `abc`
    * `abcd`
    * `abcde`

    👉 Pas cinq fois `abcde`, car **chaque rendu capture sa propre valeur de `text`** (c’est le concept de **closure**).\
    Chaque Effet est indépendant et isolé.

***

### Exemple : `ChatRoom` avec dépendances

```jsx
export default function ChatRoom({ roomId }) {
  useEffect(() => {
    const connection = createConnection(roomId);
    connection.connect();
    return () => connection.disconnect();
  }, [roomId]);

  return <h1>Bienvenue dans {roomId} !</h1>;
}
```

* **Montage avec `roomId = "general"`** → connexion à _general_.
* **Re-render avec `roomId = "general"`** → pas de changement, l’Effet est ignoré.
* **Re-render avec `roomId = "travel"`** → React exécute le cleanup de _general_ (déconnexion) puis connecte à _travel_.
* **Démontage** → cleanup final, déconnexion de _travel_.

***

### Points clés à retenir

✅ Contrairement aux événements, les **Effets** sont déclenchés par le **rendu**.\
✅ Ils permettent de **synchroniser un composant avec un système externe** (API, abonnement, widget, etc.).\
✅ Par défaut, un Effet tourne **après chaque rendu**.\
✅ React **ignore** un Effet si ses dépendances n’ont pas changé.\
✅ Tu ne choisis pas tes dépendances : elles sont déterminées par ton code.\
✅ `[]` = exécution uniquement au **montage**.\
✅ En **Strict Mode (dev seulement)**, React monte deux fois pour tester ton code.\
✅ Si ton Effet casse au remontage, ajoute un **cleanup**.\
✅ React appelle le cleanup avant le prochain Effet et lors du démontage.
