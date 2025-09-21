# 📚 Gestion de l’État (niveau intermédiaire)

À mesure que ton application grandit, il devient crucial d’être **intentionnel** sur la manière dont tu organises ton état et dont les données circulent entre tes composants.\
👉 Un état redondant ou dupliqué est l’une des **sources les plus fréquentes de bugs**.

Dans ce chapitre, tu vas apprendre :

***

### 🎯 Objectifs

1. **Comment réfléchir aux changements de l’UI comme des changements d’état**
   * Tout ce qui change visuellement dans ton interface découle d’un changement d’état.
   * Exemple : une modale qui s’ouvre/ferme, une liste qui s’allonge, un champ de saisie qui se met à jour.
2. **Comment bien structurer ton état**
   * Décider **quoi mettre en état** et quoi calculer à la volée.
   * Éviter la duplication inutile (exemple : ne stocke pas à la fois `firstName` et `fullName`, calcule l’un à partir de l’autre).
3. **Comment “remonter l’état” (lift state up)** pour le partager entre plusieurs composants
   * Lorsqu’un état doit être utilisé par deux composants frères, il faut le déplacer dans leur parent commun.
4. **Comment contrôler si l’état est conservé ou réinitialisé**
   * Comprendre comment React monte/démonte les composants et quand leur état est remis à zéro.
5. **Comment regrouper une logique complexe de mise à jour d’état dans une fonction**
   * Utiliser des fonctions pures ou des reducers (`useReducer`) pour rendre la logique plus lisible et testable.
6. **Comment passer de l’information sans “prop drilling”**
   * Utiliser le **Context** pour éviter de passer des props à chaque niveau inutilement.
7. **Comment faire évoluer ta gestion d’état quand ton app grossit**
   * Savoir quand utiliser uniquement `useState`/`useReducer`, et quand introduire des bibliothèques plus avancées (ex. Redux, Zustand, Recoil, Jotai…).

***

👉 Bref : cette partie va t’apprendre à **penser ton UI en termes d’état**, à **structurer ton état intelligemment**, et à **éviter les pièges classiques** de duplication, de props trop profondes, ou de logique d’état trop dispersée.

## Réagir aux entrées avec l’état

Avec React, vous ne modifiez pas l’UI directement depuis le code.\
Par exemple, vous n’écrirez pas de commandes comme :

* « désactiver le bouton »,
* « activer le bouton »,
* « afficher le message de succès », etc.

👉 À la place, vous **décrivez l’UI que vous souhaitez voir** pour les différents états visuels de votre composant :

* **état initial**,
* **état de saisie**,
* **état de succès**,

puis vous déclenchez les changements d’état en réponse aux entrées de l’utilisateur.

C’est très proche de la façon dont les designers conçoivent une interface.

***

Voici un formulaire de quiz construit avec React.

Notez qu’il utilise la variable d’état `status` pour déterminer :

* si le bouton « Envoyer » doit être activé ou désactivé,
* et si le message de succès doit être affiché à la place du formulaire.

***

#### Exemple : `App.js`

```jsx
import { useState } from 'react';

export default function Form() {
  const [answer, setAnswer] = useState('');
  const [error, setError] = useState(null);
  const [status, setStatus] = useState('typing');

  async function handleSubmit(e) {
    e.preventDefault();
    setStatus('submitting');
    try {
      await submitForm(answer);
      setStatus('success');
    } catch (err) {
      setStatus('typing');
      setError(err);
    }
  }

  function handleTextareaChange(e) {
    setAnswer(e.target.value);
  }

  if (status === 'success') {
    return <h1>Bravo, c’est la bonne réponse !</h1>
  }

  return (
    <>
      <h2>Quiz : les villes</h2>
      <p>
        Dans quelle ville existe-t-il un panneau publicitaire 
        qui transforme l’air en eau potable ?
      </p>
      <form onSubmit={handleSubmit}>
        <textarea
          value={answer}
          onChange={handleTextareaChange}
          disabled={status === 'submitting'}
        />
        <br />
        <button
          disabled={answer.length === 0 || status === 'submitting'}
        >
          Envoyer
        </button>
        {error !== null &&
          <p className="Error">
            {error.message}
          </p>
        }
      </form>
    </>
  );
}

function submitForm(answer) {
  // On fait semblant d’envoyer la réponse au serveur.
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      let shouldError = answer.toLowerCase() !== 'lima';
      if (shouldError) {
        reject(new Error('Bonne tentative mais mauvaise réponse. Réessayez !'));
      } else {
        resolve();
      }
    }, 1500);
  });
}
```

***

✅ Dans cet exemple, React ne manipule pas directement le DOM pour cacher/montrer des éléments ou activer/désactiver des boutons.\
C’est **l’état** qui pilote tout, et React met automatiquement l’UI à jour en fonction de cet état.

## Gérer l’état (Managing State)

Niveau : **Intermédiaire**

À mesure que ton application grandit, il devient essentiel d’être intentionnel sur la façon dont tu organises ton état et sur la manière dont les données circulent entre tes composants.\
⚠️ Un état redondant ou dupliqué est une source fréquente de bugs.

Dans ce chapitre, tu vas apprendre :

* À penser les changements d’UI comme des changements d’état.
* À bien structurer ton état.
* À “remonter l’état” (_lift state up_) pour le partager entre composants.
* À contrôler si l’état doit être préservé ou réinitialisé.
* À consolider une logique d’état complexe dans une fonction.
* À passer de l’information sans “prop drilling”.
* À faire évoluer la gestion de l’état au fur et à mesure que ton app grandit.

***

### Choisir la bonne structure d’état

Un état bien structuré = un composant facile à modifier et déboguer.\
**Principe clé : l’état ne doit pas contenir d’informations redondantes ou dupliquées.**

#### Mauvais exemple : état redondant

```jsx
const [firstName, setFirstName] = useState('');
const [lastName, setLastName] = useState('');
const [fullName, setFullName] = useState(''); // Redondant ❌
```

#### Solution : calculer `fullName` à l’affichage

```jsx
const fullName = firstName + ' ' + lastName;
```

➡️ Beaucoup de bugs disparaissent simplement en supprimant la redondance.

***

### Partager l’état entre composants

Parfois, deux composants doivent rester synchronisés.\
👉 Dans ce cas, enlève l’état de chacun et **remonte-le dans leur parent commun**, puis passe-le en `props`.

C’est le fameux pattern **“lifting state up”**.

Exemple : un accordéon où un seul panneau peut être actif.\
Le parent conserve l’`activeIndex`, et chaque `Panel` reçoit `isActive` + `onShow`.

***

### Préserver et réinitialiser l’état

Quand un composant est re-rendu, React doit décider :

* quels morceaux de l’arbre garder,
* lesquels recréer.

Par défaut, React préserve ce qui “correspond”.\
Mais tu peux forcer la réinitialisation avec une **clé unique** (`key`).

Exemple : une app de chat → changer de destinataire doit réinitialiser le champ texte.\
Solution :

```jsx
<Chat key={to.email} contact={to} />
```

***

### Extraire la logique d’état dans un reducer

Si un composant a beaucoup de mises à jour d’état réparties dans différents handlers, cela devient vite illisible.

👉 Solution : centraliser la logique dans un **reducer**.

* Les handlers déclenchent des **actions** (`dispatch`).
* Le `reducer` décrit comment l’état évolue en fonction de chaque action.

C’est la base de **useReducer**.

***

### Passer des données profondément avec Context

Normalement, les données se transmettent par `props`.\
Mais si tu dois passer une info à travers de nombreux niveaux (ou à plusieurs composants), ça devient lourd.

👉 **Context** permet à un composant parent de “fournir” des données accessibles à tout le sous-arbre, sans prop drilling.

Exemple : une hiérarchie de titres (`Heading`) qui récupèrent leur niveau via un `LevelContext`.

***

### Monter en échelle avec Reducer + Context

Tu peux combiner les deux :

* **Reducer** → gérer une logique d’état complexe.
* **Context** → partager cet état profondément dans l’arbre.

Exemple : une Todo App

* `TasksProvider` gère l’état avec un reducer.
* `AddTask` et `TaskList` accèdent et modifient cet état via Context.

***

### 🚀 Récapitulatif

* **Évite la redondance** dans l’état → calcule plutôt que stocker.
* **Lifting state up** : pour synchroniser plusieurs composants.
* **Clés (key)** : pour contrôler si l’état est préservé ou réinitialisé.
* **Reducers** : pour centraliser et simplifier la logique d’état.
* **Context** : pour partager des infos profondément dans l’arbre.
* **Reducer + Context** : scalable pour les applis complexes.
