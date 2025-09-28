# 🔄 Cycle de vie des Effets réactifs (useEffect) - (Lifecycle of Reactive Effects)

Les **Effets** (`useEffect`) n’ont pas le même cycle de vie que les **composants React**.\
Un **composant** peut :

* se **monter** (mount),
* se **mettre à jour** (update),
* se **démonter** (unmount).

👉 Un **Effet**, en revanche, ne fait que deux choses :

1. **Démarrer** une synchronisation (ex. ouvrir une connexion, lancer un timer, ajouter un listener).
2. **Arrêter** cette synchronisation (cleanup : fermer la connexion, nettoyer le timer, retirer le listener).

Et ce cycle peut se répéter plusieurs fois en fonction des **props** et du **state** dont l’Effet dépend.

***

### ⚡ Différence fondamentale

* Le composant vit **globalement** (montage ➝ mises à jour ➝ démontage).
* Chaque **Effet vit isolément**. Il peut être exécuté, nettoyé, puis ré-exécuté indépendamment, selon ses **dépendances**.

***

### 📌 Points clés que tu vas apprendre

1. **Cycle de vie spécifique des Effets**\
   ➝ Comment ils s’exécutent à chaque rendu dépendant des props/state.
2. **Penser chaque Effet en isolation**\
   ➝ Chaque `useEffect` est une petite “machine” qui démarre/arrête une tâche précise.
3. **Quand resynchroniser un Effet et pourquoi**\
   ➝ Ex. quand `roomId` change, il faut couper la connexion au chat précédent et ouvrir la nouvelle.
4. **Dépendances déterminées automatiquement**\
   ➝ Ce que tu mets dans ton Effet détermine ses dépendances. Tu ne “choisis” pas arbitrairement.
5. **Valeurs réactives**\
   ➝ Toute prop, state ou valeur dérivée qui change avec le rendu est **réactive** et doit être déclarée comme dépendance.
6. **Tableau vide `[]`**\
   ➝ Signifie que l’Effet ne dépend de rien de réactif → il s’exécute uniquement **au montage/démontage**.
7. **Vérification par linter (ESLint React Hooks)**\
   ➝ Il t’avertit si une dépendance est manquante pour éviter des bugs liés à des valeurs périmées.
8. **Que faire si tu n’es pas d’accord avec le linter ?**\
   ➝ La règle est là pour garantir la cohérence. Mais si tu veux intentionnellement ignorer une dépendance (rare cas), tu peux utiliser des techniques comme `useRef` ou extraire une logique ailleurs.

***

👉 En résumé :

* Un composant a une **vie complète** (mount → update → unmount).
* Un Effet est **atomique** : il commence une tâche puis la nettoie. Et ce cycle peut se répéter à chaque fois que ses dépendances changent.

## 🔄 Cycle de vie d’un **Effet** React

Chaque **composant React** suit un cycle bien défini :

1. **Montage** ➝ ajouté à l’écran.
2. **Mise à jour** ➝ reçoit de nouvelles props ou un nouvel état (souvent après une interaction).
3. **Démontage** ➝ retiré de l’écran.

👉 Cette vision fonctionne très bien pour les composants, **mais pas pour les Effets**.\
Un **Effet** doit être pensé comme une unité indépendante de synchronisation avec un système externe.

***

### ⚡ Exemple : connexion à un serveur de chat

```jsx
const serverUrl = 'https://localhost:1234';

function ChatRoom({ roomId }) {
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();

    return () => {
      connection.disconnect();
    };
  }, [roomId]);
}
```

***

### 🟢 Ce que fait l’Effet

1.  **Démarrage de la synchronisation**

    ```js
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    ```

    ➝ Ici, on ouvre la connexion au serveur de chat pour la salle `roomId`.
2.  **Arrêt de la synchronisation (cleanup)**

    ```js
    return () => {
      connection.disconnect();
    };
    ```

    ➝ Lorsqu’il faut nettoyer, on coupe la connexion.

***

### 🧩 Important à comprendre

Tu pourrais penser que :

* L’Effet démarre **au montage** du composant.
* L’Effet s’arrête **au démontage** du composant.

➡️ C’est vrai, **mais incomplet** !\
En réalité, un Effet peut démarrer **et s’arrêter plusieurs fois** alors que le composant reste monté.

***

### 📌 Quand un Effet est rejoué ?

* Quand ses **dépendances changent** (`roomId` dans l’exemple).
  * Si l’utilisateur passe de `roomId="general"` à `roomId="travel"`, React :
    1. Exécute le **cleanup** (déconnexion de "general").
    2. Relance l’Effet (connexion à "travel").
* En **mode développement** avec `StrictMode`.
  * React monte, démonte puis remonte le composant **exprès** pour tester que ton Effet sait bien se nettoyer.

***

### ⚠️ Cas particuliers

* Si ton Effet **ne retourne pas de fonction de cleanup**, React agit comme si tu avais retourné une fonction vide `() => {}`.
* Certains Effets n’ont pas besoin de cleanup (ex. mettre à jour un titre de page ou synchroniser une valeur de state avec une animation).

***

👉 En résumé :

* Un composant a un cycle global (mount → update → unmount).
* Un Effet est **réactif** et peut être exécuté/nettoyé plusieurs fois au cours de ce cycle.
* Pense chaque Effet comme une **boîte noire de synchronisation** avec un système externe.

## 🔄 Pourquoi la synchronisation doit parfois se refaire plusieurs fois

Prenons un exemple concret avec un composant **ChatRoom** qui reçoit une prop `roomId`.\
L’utilisateur peut changer de salle via un menu déroulant.

***

### 1️⃣ Premier rendu : salle "general"

```jsx
const serverUrl = 'https://localhost:1234';

function ChatRoom({ roomId /* "general" */ }) {
  return <h1>Welcome to the {roomId} room!</h1>;
}
```

➡️ L’UI affiche **"Welcome to the general room!"**.\
➡️ Après le rendu, l’Effet démarre la **connexion au serveur** pour la salle "general" :

```jsx
useEffect(() => {
  const connection = createConnection(serverUrl, roomId); // Connecte à "general"
  connection.connect();

  return () => {
    connection.disconnect(); // Déconnexion de "general"
  };
}, [roomId]);
```

***

### 2️⃣ L’utilisateur change de salle → "travel"

React met d’abord à jour le rendu de l’UI :

```jsx
function ChatRoom({ roomId /* "travel" */ }) {
  return <h1>Welcome to the {roomId} room!</h1>;
}
```

➡️ L’utilisateur **voit "Welcome to the travel room!"**.\
➡️ Mais attention : **l’Effet précédent est toujours connecté à la salle "general"**.\
Cela crée une incohérence : l’UI affiche "travel", mais techniquement l’app écoute encore "general".

***

### 3️⃣ Que doit faire React maintenant ?

À ce moment, il faut :

1. **Arrêter** la synchronisation avec l’ancienne salle (déconnexion de "general").
2. **Démarrer** la synchronisation avec la nouvelle salle (connexion à "travel").

👉 Bonne nouvelle : tu as déjà expliqué à React **comment faire ces deux étapes** :

* Le corps de l’Effet dit **comment commencer la synchro**.
* La fonction de **cleanup** dit **comment l’arrêter**.

***

### 4️⃣ L’ordre exact exécuté par React

Quand `roomId` change de `"general"` à `"travel"` :

1. React exécute le **cleanup** de l’ancien Effet ➝ `disconnect("general")`.
2. Ensuite, React exécute à nouveau le corps de l’Effet ➝ `connect("travel")`.

Résultat :\
✅ L’app affiche la bonne salle **et** est connectée au bon serveur.

***

### ⚡ Résumé

* Un **Effet n’est pas lié seulement au cycle de vie du composant**, mais aussi aux **changements de props/état**.
* À chaque changement de dépendances, React :
  1. Nettoie l’ancienne synchro.
  2. Relance l’Effet avec les nouvelles valeurs.

## 🔄 Comment React re-synchronise ton Effet

Reprenons ton composant **ChatRoom** avec la prop `roomId`.\
Supposons que l’utilisateur change de salle dans le menu déroulant :

***

### 1️⃣ Avant : `roomId = "general"`

```jsx
function ChatRoom({ roomId /* "general" */ }) {
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId); // ✅ Connexion à "general"
    connection.connect();
    return () => {
      connection.disconnect(); // ❌ Déconnexion de "general"
    };
  }, [roomId]);
}
```

➡️ L’Effet connecte l’utilisateur à la salle **general**.\
➡️ La fonction de nettoyage (`cleanup`) est prête à déconnecter de cette salle quand nécessaire.

***

### 2️⃣ Changement : `roomId = "travel"`

Quand `roomId` change, React :

1. **Appelle le cleanup de l’ancien Effet** → déconnexion de `"general"`.
2. **Exécute le corps du nouvel Effet** → connexion à `"travel"`.

```jsx
function ChatRoom({ roomId /* "travel" */ }) {
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId); // ✅ Connexion à "travel"
    connection.connect();
    return () => {
      connection.disconnect(); // ❌ Déconnexion de "travel" (lors du prochain changement ou unmount)
    };
  }, [roomId]);
}
```

➡️ Résultat : tu es **connecté à la bonne salle affichée dans l’UI**.

***

### 3️⃣ Autre changement : `"travel"` → `"music"`

🔁 Le même cycle recommence :

* Cleanup : déconnexion de `"travel"`.
* Nouvel Effet : connexion à `"music"`.

***

### 4️⃣ Unmount du composant

Quand `ChatRoom` est retiré de l’écran :

* React appelle une **dernière fois le cleanup**.
* Tu es déconnecté de `"music"`.
* Plus aucune synchro n’est nécessaire.

***

✅ Grâce à ce mécanisme, **ton UI (ce que voit l’utilisateur) et ton état externe (connexion serveur)** restent toujours cohérents.

## 🔄 Penser un Effet comme un cycle _start → stop_

Quand on raisonne depuis le **composant**, on a tendance à imaginer des “événements du cycle de vie” :

* _après un render_,
* _avant un unmount_,
* etc.

👉 Ce raisonnement devient vite complexe.

Mais si on bascule la perspective sur **l’Effect lui-même**, tout s’éclaire :

* Chaque Effet **démarre une synchronisation** (`connect()`),
* Puis, tôt ou tard, **l’arrête** (`disconnect()`).
* Ce cycle peut se répéter autant de fois que nécessaire selon l’évolution des props ou du state.

Ainsi, peu importe que le composant soit en _mount_, _update_ ou _unmount_ :\
➡️ **Ton job est juste de décrire comment commencer et comment arrêter la synchro.**\
➡️ React se charge d’appeler ça au bon moment.

***

## 🧠 Exemple : ChatRoom

```jsx
function ChatRoom({ roomId }) {
  useEffect(() => {
    // 🚀 Démarrage de la synchro
    const connection = createConnection(serverUrl, roomId);
    connection.connect();

    // 🛑 Arrêt de la synchro
    return () => {
      connection.disconnect();
    };
  }, [roomId]);

  return <h1>Welcome to the {roomId} room!</h1>;
}
```

Cycle réel :

1. Connexion à `"general"` → Déconnexion de `"general"`
2. Connexion à `"travel"` → Déconnexion de `"travel"`
3. Connexion à `"music"` → Déconnexion de `"music"`
4. Déconnexion finale lors de l’unmount

***

## 🧪 Pourquoi React _force_ ce cycle en dev ?

En mode **StrictMode (dev)**, React fait exprès :

* Monte le composant → appelle l’Effect
* Démonte immédiatement → appelle le cleanup
* Remonte encore → ré-exécute l’Effect

➡️ C’est comme **ouvrir/fermer une porte une fois de plus** pour vérifier que la serrure fonctionne 🔐.\
➡️ Cela détecte les Effects mal nettoyés (qui laisseraient des connexions, timers, events en fuite).

En prod ✅ : ce double cycle disparaît.

***

## 📌 À retenir

* Un Effet n’est **pas un callback “post-render”**, c’est une **synchro externe**.
* Chaque Effet suit un cycle **start → stop**, répété autant que nécessaire.
* React vérifie en dev que ton `cleanup` est bien robuste.
* Si ton Effet est bien écrit, peu importe combien de fois il est monté/démonté : le comportement sera toujours correct.

## 🧩 Comment React sait qu’il doit re-synchroniser un Effet ?

Prenons l’exemple :

```jsx
function ChatRoom({ roomId }) {
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();

    return () => {
      connection.disconnect();
    };
  }, [roomId]); // 👈 Dépendance déclarée
}
```

***

### ⚡ Le principe

1. Tu sais que **`roomId` peut changer** (c’est une prop).
2. Ton Effet **lit `roomId`** pour décider à quelle salle se connecter.
3. Donc tu déclares `roomId` dans le tableau de dépendances `[roomId]`.

👉 Cela dit à React : _“si `roomId` change entre deux renders, cet Effet doit être re-synchronisé.”_

***

### 🔍 Comment React compare ?

* Lors du **premier render**, React stocke la liste des dépendances (par ex. `["general"]`).
* Lors du **render suivant**, il compare avec la nouvelle liste (par ex. `["travel"]`).
* Si **au moins une valeur est différente** (via `Object.is`), alors :
  * React appelle le `cleanup` (déconnexion de `"general"`),
  * Puis exécute à nouveau l’Effet (connexion à `"travel"`).

👉 Si les dépendances n’ont **pas changé**, l’Effet n’est pas rejoué et la synchro continue telle quelle.

***

### ✅ Exemple concret

* Initial : `[ "general" ]` → connexion à _general_.
* Nouveau render : `[ "travel" ]` → différent → déconnexion de _general_, connexion à _travel_.
* Nouveau render : `[ "travel" ]` → identique → rien ne change, la connexion reste active.

***

### 📌 À retenir

* Le **tableau de dépendances** est la _checklist_ de React pour savoir quand relancer un Effet.
* Chaque valeur est comparée avec sa version précédente via `Object.is`.
* Si au moins une change ➝ React **re-synchronise l’Effect** (cleanup + ré-exécution).
* Si rien ne change ➝ l’Effect reste tel quel.

## 🔄 Chaque Effet = un processus de synchronisation indépendant

Un **Effet** ne doit gérer **qu’un seul processus de synchronisation**.\
Si tu mélanges plusieurs logiques différentes dans un même `useEffect`, tu risques d’introduire des comportements inattendus.

***

### ❌ Mauvaise pratique : mélanger des logiques

Ici on fait **deux choses différentes** dans un seul effet :

1. Envoyer un événement analytics (`logVisit`).
2. Se connecter à un serveur de chat (`createConnection`).

```jsx
function ChatRoom({ roomId }) {
  useEffect(() => {
    logVisit(roomId); // Analytics
    const connection = createConnection(serverUrl, roomId); // Connexion chat
    connection.connect();
    return () => {
      connection.disconnect();
    };
  }, [roomId]);
}
```

👉 Problème :

* Si un changement de dépendance provoque une re-synchro (par ex. `roomId` ou autre), **les analytics seront renvoyés plusieurs fois** pour la même salle ➝ bug.

***

### ✅ Bonne pratique : séparer les responsabilités

On sépare en **deux Effets indépendants** :

```jsx
function ChatRoom({ roomId }) {
  // Effet 1 : Analytics (traque la visite d'une salle)
  useEffect(() => {
    logVisit(roomId);
  }, [roomId]);

  // Effet 2 : Connexion réseau (chat)
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();

    return () => {
      connection.disconnect();
    };
  }, [roomId]);
}
```

👉 Résultat :

* Les **analytics** ne dépendent que de `roomId`.
* La **connexion chat** est un autre processus, aussi basé sur `roomId`.
* Supprimer l’un des deux n’affecte pas l’autre ➝ séparation claire.

***

### 📌 Règle à retenir

* Un **Effet = un processus de synchronisation** (DOM, API, socket, analytics, etc.).
* Si supprimer un Effet **ne casse pas** l’autre, alors tu avais raison de les séparer.
* À l’inverse, si tu sépares artificiellement une logique cohérente en plusieurs Effets, ton code semblera plus “propre” mais sera en fait plus fragile.

## ⚡ Les Effets réagissent aux valeurs réactives

Un **Effet** (`useEffect`) ne se déclenche **que lorsque ses dépendances changent**.\
Ces dépendances doivent être **des valeurs réactives**, c’est-à-dire celles qui **peuvent évoluer entre deux re-renders** :

* ✅ **Props** (`roomId`) ➝ elles changent quand le parent re-render.
* ✅ **State local** (`serverUrl` si on utilise `useState`) ➝ il peut changer suite à une interaction.
* ✅ **Variables créées dans le composant** (dérivées de props/state) ➝ recalculées à chaque render.

En revanche, les valeurs **statiques** ou définies **hors du cycle de rendu** ne changent pas entre deux renders, donc inutile de les mettre en dépendances.

***

### ❌ Exemple : dépendance inutile

```jsx
const serverUrl = 'https://localhost:1234'; // Constante, ne change jamais

function ChatRoom({ roomId }) {
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => connection.disconnect();
  }, [roomId]); // ✅ Seule roomId est réactive
}
```

👉 Ici, `serverUrl` ne dépend d’aucune interaction ni re-render, donc pas besoin de l’ajouter.

***

### ✅ Exemple : dépendance réactive

```jsx
function ChatRoom({ roomId }) {
  const [serverUrl, setServerUrl] = useState('https://localhost:1234');

  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => connection.disconnect();
  }, [roomId, serverUrl]); // ✅ Props + State = valeurs réactives
}
```

👉 Ici, si l’utilisateur change le champ **Server URL**, `serverUrl` se met à jour et l’Effet doit se re-synchroniser ➝ donc on l’ajoute aux dépendances.

***

### 📌 Règle d’or

* **Inclure toutes les valeurs réactives** (props, state, variables internes calculées).
* **Ne pas inclure** ce qui est **constant** (const hors composant, fonctions pures importées).
* React (et son linter) te rappellera toujours : “si tu utilises une valeur réactive dans un `useEffect`, ajoute-la à la liste des dépendances”.

## ⚡ Que signifie un `useEffect` avec `[]` (dépendances vides) ?

👉 Lorsque tu passes un tableau **vide** à `useEffect` :

```jsx
useEffect(() => {
  // logique
  return () => {
    // nettoyage
  };
}, []); // dépendances vides
```

Cela veut dire :

* **Le code de démarrage** (connecter, initialiser, écouter…) s’exécute **une seule fois au montage**.
* **Le code de nettoyage** (déconnecter, arrêter, désabonner…) s’exécute **une seule fois au démontage**.
* ⚠️ En mode développement **StrictMode**, React va quand même **monter/démonter une fois supplémentaire** pour tester ton cleanup.

***

### Exemple concret

```jsx
const serverUrl = 'https://localhost:1234';
const roomId = 'general';

function ChatRoom() {
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();

    return () => connection.disconnect();
  }, []); // ✅ aucune dépendance réactive

  return <h1>Welcome to the {roomId} room!</h1>;
}
```

👉 Ici, `serverUrl` et `roomId` sont **des constantes hors composant** → elles ne changent jamais pendant le cycle de vie du composant → pas besoin de les mettre en dépendances.

***

### 📌 Deux manières de voir

#### 1. **Perspective du composant**

* `[]` = “exécuter l’effet **au montage** et **nettoyer au démontage**”.

#### 2. **Perspective de l’Effet**

* L’Effet n’utilise **aucune valeur réactive** (props/state).
* Donc il n’a pas besoin d’être relancé après un re-render.
* Si un jour tu rends `roomId` ou `serverUrl` réactifs (state ou props), tu n’auras qu’à les ajouter dans la dépendance **sans changer la logique de l’effet**.

***

✅ En résumé :\
Un `useEffect` avec `[]` sert uniquement pour les effets **qui n’ont pas de dépendances réactives** et doivent s’exécuter **une fois** (ex. initialisation, config globale, log analytics au premier affichage, connexion à un service fixe).

## ⚡ Toutes les variables déclarées dans un composant sont réactives

En React, **toute valeur définie dans le corps du composant est considérée comme réactive**.

* **Props** ➝ changent si le parent les met à jour.
* **State** ➝ change avec `setState`.
* **Variables calculées à partir des props/state** ➝ changent aussi puisqu’elles sont recalculées à chaque re-render.

Donc si ton `useEffect` utilise une valeur définie dans le composant (prop, state ou variable dérivée), **elle doit être déclarée dans la liste des dépendances**.

***

### Exemple concret

```jsx
function ChatRoom({ roomId, selectedServerUrl }) {
  const settings = useContext(SettingsContext); // réactif (peut changer)
  const serverUrl = selectedServerUrl ?? settings.defaultServerUrl; // réactif (dépend des props/context)

  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();

    return () => connection.disconnect();
  }, [roomId, serverUrl]); // ✅ toutes les dépendances déclarées
}
```

👉 Ici :

* `roomId` (prop) est réactif.
* `settings` (context) est réactif.
* `serverUrl` est calculé à partir de `props` + `context`, donc réactif.

Tous doivent être inclus dans le tableau de dépendances.

***

### 🚨 Exemple de bug fréquent

```jsx
function ChatRoom({ roomId }) {
  const [serverUrl, setServerUrl] = useState('https://localhost:1234');

  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => connection.disconnect();
  }, []); // ❌ Mauvais : dépendances manquantes
}
```

⚠️ Ici, React affiche une erreur de linter :

> `React Hook useEffect has missing dependencies: 'roomId' and 'serverUrl'.`

Pourquoi ?\
Parce que si l’utilisateur change de salle (`roomId`) ou d’URL (`serverUrl`), ton effet **ne se relancera pas**, et tu resteras connecté à l’ancienne valeur.

***

### ✅ Correction

```jsx
useEffect(() => {
  const connection = createConnection(serverUrl, roomId);
  connection.connect();
  return () => connection.disconnect();
}, [roomId, serverUrl]); // ✅ tout est déclaré
```

***

### 📌 Exceptions : valeurs stables

Certaines valeurs **ne changent jamais** même si elles sont créées dans le composant :

* La fonction `setX` de `useState`.
* L’objet `ref` de `useRef`.

👉 Elles ne sont **pas réactives**, donc tu peux les omettre du tableau des dépendances. (Les inclure ne pose pas de problème, mais c’est inutile.)

***

✅ En résumé :

* **Tout ce qui peut changer au re-render (props, state, variables calculées)** doit être dans les dépendances.
* **Ce qui est garanti stable (`setState`, `ref`)** peut être ignoré.
* Le linter est ton allié : s’il signale une dépendance manquante, c’est généralement un vrai bug.

## 🚫 Que faire quand on ne veut pas re-synchroniser un `useEffect`

***

### 1. ✅ Déplacer les valeurs en dehors du composant

Si une valeur **n’est pas réactive** (elle ne dépend pas du rendu, ni des props/state), tu peux la déclarer **hors du composant**.

```jsx
const serverUrl = 'https://localhost:1234'; // non réactif
const roomId = 'general'; // non réactif

function ChatRoom() {
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => connection.disconnect();
  }, []); // ✅ aucun problème
}
```

***

### 2. ✅ Déplacer les valeurs dans le corps de l’effet

Autre solution : les déclarer **directement dans l’effet**. Elles ne font pas partie du cycle de rendu → donc pas réactives.

```jsx
function ChatRoom() {
  useEffect(() => {
    const serverUrl = 'https://localhost:1234'; // non réactif
    const roomId = 'general'; // non réactif
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => connection.disconnect();
  }, []); // ✅ pas de dépendances manquantes
}
```

***

### 3. ⚡ Règles d’or des dépendances

* **Tu ne choisis pas tes dépendances** : elles sont imposées par le code que tu écris dans l’effet.
* **Toute valeur réactive (props, state, variables dérivées)** doit être listée.
* Le linter (`react-hooks/exhaustive-deps`) t’indique si tu en oublies une → **il a toujours raison**.

⚠️ Ne jamais “casser” le linter avec :

```jsx
// eslint-disable-next-line react-hooks/exhaustive-deps
```

C’est un **mauvais signal** : le problème est dans ton code, pas dans le linter.

***

### 4. ✅ Comment corriger si ça boucle ?

Si ajouter une dépendance provoque une **boucle infinie** (re-render/re-sync permanent), il faut revoir ton code :

* **Séparer la logique** :\
  Si ton effet fait plusieurs choses indépendantes → divise-les en plusieurs `useEffect`.
* **Stabiliser une valeur** :
  * Utilise `useMemo` pour les objets/valeurs recalculées.
  * Utilise `useCallback` pour les fonctions créées dans le composant.
* **Éviter les dépendances inutiles** :\
  Si tu passes un objet `{}` ou une fonction inline dans un `useEffect`, il sera **recréé à chaque render** → donc l’effet se relancera.\
  Corrige en mémorisant avec `useMemo` ou `useCallback`.

***

### 5. ✅ Quand tu veux lire une valeur **sans re-synchroniser**

Parfois, tu veux lire une prop/state dans ton effet **sans que cela en déclenche la re-synchro** (ex: tu veux juste loguer sa valeur actuelle).\
👉 Dans ce cas, React propose le pattern **Effect Event** (séparer la partie réactive de la partie “lecture”).

***

### 🔑 Récap

* Les **valeurs réactives** doivent être dans le tableau des dépendances.
* Si une valeur n’est **pas réactive** → déclare-la **hors du composant** ou **dans l’effet**.
* Si un effet re-synchronise trop souvent → stabilise tes fonctions/objets avec `useMemo` ou `useCallback`.
* 🚫 Ne jamais désactiver le linter.
