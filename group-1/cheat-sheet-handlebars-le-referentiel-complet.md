# 📘 Cheat-Sheet Handlebars — Le Référentiel Complet

### 🔶 1. Expressions de base

#### Interpolation simple

`{{variable}}`\
Affiche la valeur échappée (sécurisée).

#### Interpolation NON échappée

`{{{variable}}}`\
Insère du HTML brut — à utiliser avec prudence.

#### Chemin d’accès (dot-notation)

`{{user.name}}`\
Accès aux propriétés imbriquées.

#### Appel d’helper inline

`{{formatDate user.birth}}`

***

### 🔶 2. Contexte

#### Contexte courant

À l’intérieur d’un template, `this` représente l’objet sur lequel on est positionné.\
Dans une itération (`each`), `this` = l’élément courant.

#### Accéder à la racine

`{{@root.title}}`

***

### 🔶 3. Helpers intégrés (built-in)

#### Condition

```
{{#if condition}}
  …
{{else}}
  …
{{/if}}
```

#### Condition inverse

```
{{#unless condition}}
  …
{{/unless}}
```

#### Boucle (tableaux OU objets)

```
{{#each items}}
  {{@index}} : {{this}}
{{/each}}
```

Variables spéciales dans `each` :\
`@index`, `@first`, `@last`, `@key`

#### Changer le contexte

```
{{#with user}}
  {{name}} – {{email}}
{{/with}}
```

`else` fonctionne aussi dans `each` et `with`.

***

### 🔶 4. Helpers personnalisés

#### Inline helper

```js
Handlebars.registerHelper("upper", function(str) {
  return str.toUpperCase();
});
```

Usage : `{{upper name}}`

#### Block helper

```js
Handlebars.registerHelper("wrap", function(options) {
  return new Handlebars.SafeString(
    "<div>" + options.fn(this) + "</div>"
  );
});
```

Usage :

```
{{#wrap}}
  texte
{{/wrap}}
```

**Dans un block-helper :**

* `options.fn(context)` → exécute le block normal
* `options.inverse(context)` → exécute le `{{else}}`

***

### 🔶 5. Partials (fragments réutilisables)

#### Déclaration JS

```js
Handlebars.registerPartial("header", "<h1>{{title}}</h1>");
```

#### Utilisation

`{{> header}}`

#### Avec contexte personnalisé

`{{> header user}}`

#### Partials dynamiques

```
{{> (whichPartial type) }}
```

#### Partial inline

```
{{#*inline "card"}}
  <div class="card">{{title}}</div>
{{/inline}}

{{> card}}
```

***

### 🔶 6. Options & hash parameters

#### Passer des options à un helper

```
{{helperName value color="red" size=3}}
```

Dans le helper, récupérés via :\
`options.hash.color`, `options.hash.size`

***

### 🔶 7. Sub-expressions

Utilisées quand un helper doit être le résultat d’un autre.\
`{{upper (concat firstName lastName)}}`

***

### 🔶 8. Échappement & sécurité

* `{{variable}}` → échappé
* `{{{variable}}}` → non échappé
* Pour retourner du HTML dans un helper :\
  `return new Handlebars.SafeString(html);`

⚠️ Attention au contenu utilisateur non filtré.

***

### 🔶 9. Compilation & exécution

#### Compilation

```js
const template = Handlebars.compile("Hello {{name}}");
```

#### Exécution

```js
template({ name: "Alice" });
```

#### Pré-compilation (production)

Permet de compiler les templates avant utilisation pour plus de performance.

***

### 🔶 10. Variables spéciales importantes

* `@index` — dans each
* `@first` / `@last`
* `@key` — si iteration sur objet
* `@root` — contexte global
* `this` — élément courant
* `..` — remonter d’un niveau de contexte

***

### 🔶 11. Éléments subtils à connaître

#### Accéder à un parent dans un nested block

`{{../title}}`

#### Helpers “logiques” personnalisés

Tendance courante pour réaliser :\
`{{#eq x y}} … {{/eq}}`\
ou\
`{{#and a b}} … {{/and}}`

(Handlebars volontairement minimaliste, la logique complexe doit être dans les helpers.)

#### Strict mode

Empêche l’accès à des propriétés non définies.

***

### 🔶 12. Modèles classiques de structure

#### Template composés (header + zones)

```
{{> header}}
<section>
  {{> content}}
</section>
{{> footer}}
```

#### Boucler avec else

```
{{#each users}}
  {{name}}
{{else}}
  Aucun utilisateur
{{/each}}
```

#### Template de composant

```
{{#with product}}
  <h3>{{title}}</h3>
  <p>{{description}}</p>
{{/with}}
```

***

### 🔶 13. Mammouth final : l’ensemble des syntaxes en un coup d’œil

```
{{variable}}
{{{html}}}
{{helper arg}}
{{helper arg key=value}}
{{#blockHelper arg}}…{{/blockHelper}}
{{#blockHelper}}…{{else}}…{{/blockHelper}}
{{> partial}}
{{> partial context}}
{{> (dynamicPartial)}}
{{lookup object key}}
{{@index}} {{@key}} {{@root}}
{{../parentVar}}
{{#*inline "name"}}…{{/inline}}
```

Ce bloc contient littéralement toutes les formes syntaxiques de Handlebars.
