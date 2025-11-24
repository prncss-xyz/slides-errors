# Quitter le chemin du bonheur: toutes ces choses que nous appellons « erreur »

La gestion des erreurs est souvent le maillon manquant dans la conception des applications. Nous plongerons en profondeur dans les différents concepts qui peuvent se cacher derrière la notion d’erreur et leurs implémentations les plus courantes. Des exemples concrets seront tirés de Node, React et Tanstack Query.

## Qui suis-je

- Poétesse et bachelière es mathématiques devenue cuisinière deveneue développeuse
- Dev Front-End (React et TypeScript) à la Banque Nationale du Canada

## Erreurs inattendues (Unexpected Errors)

![I'm sorry](https://images5.fanpop.com/image/photos/26400000/Love-means-never-having-to-say-you-re-sorry-love-story-the-movie-26452763-500-230.gif)

Un état dans lequel le programme ne devrait jamais se retrouver; l'erreur ne peut réellement se réparer qu'en modifiant le programme.

## Instruction d'assertion (Code Assertion)

```typescript
if (x > y) throw new Error("x should be greater than y");
```

Quand on raisonne sur du code, exprimer les propriétés du code sous forme d'assertions plutôt que de commentaires.

```typescript
if (z === undefined) throw new Error("z should be defined");
// z!
// z as number
```

Quand c'est possible, préférer une assertion à un cast

- coût négligeable
- mieux maintenue (changement de nom, erreurs de types)
- plus grande confiance

Les assertions simplifient le diagnostic d'une erreur:

- Plus on repère un état abhérant tôt, plus c'est facile de trouver la cause de l'erreur.
- Un bug lié à un comportement anormal est plus difficile à reproduire et à comprendre qu'une execption causée par un état abhérant.

## Comment les éviter

### Rendre les états impossibles irreprésentables

```typescript
{ loading: true: error: true }
{ status: 'loading' } // 'loading' | 'error' | 'succes'
```

```typescript
[
  { key: "a", value: "toto" },
  { key: "b", value: "popo" },
  { key: "c", value: "kiki" },
];
{
  a: {
    value: "toto";
  }
  b: {
    value: "kiki";
  }
}
```

Pas toujours possibles:

- les requis évoluent et la refactorisation ne vaudrait pas l'effort
- données dénormalisées

```typescript
{
    a: {
        id: 'a'
        value: 'toto',
    },
    b: {
        id: 'b'
        value: 'kiki',
    }
}

```

## D'autres raisons d'avoir des erreurs inattendues

- le système de types a ses limites
  - typescript: nombres entiers
  - les branded types permettent d'annoter qu'un nombre est entiers, mais ne permettent pas de prouver que la somme de deux entiers est aussi un entier
  - dans des systèmes très critiques, on utilise un assistant de preuve (Lean, Coq) pour prouver des propriétées du code
- il n'est pas toujours souhaitable de pousser le système de types à ses limites
  - tests, application, libraires

## Erreurs concrètes que j'ai attrapé

- une fonction devait avoir la même référence à deux endroits, j'employais une dépendance ordinaire au lieu d'une peer dependancy

## Quoi faire avec une erreur inattendue

- monitoring
- planter l'application ou une partie de l'application (éventuellement la repartir)
- vs essayer de corriger l'erreur et poursuivre
  - mettre des ressources sur quelquechose qui apporte peu de valeur et va causer pleins d'autres erreurs en aval
- si il y a des efforts à mettre (ça dépend de l'application): compartimenter l'application en petites morceaux qui peuvent être redémarrés (cf. erlang, react: error boundaries)

## Incitatif

```typescript
export function isoAssertion(
  condition: unknown,
  message?: string,
): asserts condition {
  if (condition === false) throw new Error(message ?? "assertion failed");
}

isoAssert(x > y, "x should be greater than y");
```

## Erreurs attendues (Expected Errors)

![Love means never having to say you're sorry.](https://media.tenor.com/EyurLM2pd6YAAAAC/love-story-sorry.gif)
Une erreur attendue est une condition nécessaire pour réaliser une opération, qui est testée à même le code de l'opération.

Considérons cette interface (fictive):

```typescript
const str = "ajfskla;;fjklsd;afjkl;dsjfkl;sd";
const target = "fk";

if (str.has(target)) {
  const index = str.findIndex(target);
  // ...
}
```

Cette interface serait:

- désagréable
- inefficace

Donc on préfère:

```typescript
const index = str.findIndex(target);
if (index >= 0) {
  // ...
}
```

En conséquence, il est nécessaire de représenter d'une manière distincte les valeurs de succès et les valeurs d'erreur (sans risquer de les confondre).

### Étude de cas, identifier les requis potentiel d'une gestion d'erreur

En tant qu'utilisateur, je veux afficher une image d'un chat qui n'est accessible que si je suis connecté et que j'ai les droits d'accès.

![cat](https://media2.giphy.com/media/v1.Y2lkPTc5MGI3NjExaXJkMTV6MGVsbDVlYjBxMGsyNDZlYnNmNXh4cTQyZmt1MjBwdnR3ZSZlcD12MV9pbnRlcm5hbF9naWZfYnlfaWQmY3Q9Zw/aE7wJrIV0qeLEKKrjn/giphy.gif)

- erreur de réseau
- erreur de serveur (500)
- je ne suis pas connecté
- je n'ai pas les droits d'accès
- l'image n'existe pas

S'il y a une erreur de réseau, je veux relancer la requête. S'il y a un autre type d'erreur, je veux afficher un message approprié.

**Imbrication**: on veut être capable de représenter de manière distincte:

- une erreur (de premier niveau)
- un succès à l'intérieur d'une erreur

Du point de vue de la requête html, toutes les erreurs autre que l'erreur réseau sont un succès.

Côté serveur, on a une séquence d'acction donc chacune repose sur la précédente. On veut interrombre la séquence à la première erreur.

**Séquence**: On veut être capable de combiner une séquence d'opération avec une potentiel d'erreur sans avoir à gérer manuellement l'échec d'une erreur précédente à chaque fois.

```typescript
a?.b?.c; // optional chaining
```

**Information**: On veut garder une identité distincte pour chaque erreur, et possiblement lui associer d'autre information (par exemble la collection dont fait partie le gif auquel je n'ai pas accès)

**Exhaustivité**: On veut être capable de garantir par le typages que tous les types d'erreurs possibles sont gérés.

**Sérialisabilité**: On veut être capable d'envoyer l'erreur serveur au client sans gestion manuelle.

Ces requis ne sont pas toujours nécessaires.

## Différentes représentations des erreurs

### L'erreur comme une valeur de plus

- valeur impossible: `findIndex`, -1
- valeur spéciale: `undefined` , `null`, `NaN`
- instances de la classe erreur

- les valeurs impossibles sont aussi une stratégie pour rendre les états contradictoires impossible à représenter
- non-sérialisable: la classe d'erreur et `undefined` ne sont pas sérialisables
- les instances de la classe d'erreur permette de distinguer autant de types d'erreur que l'on veut et y ajouter de l'infromation arbitraire
- aucune de ces solution ne permet de représenter un succès qui contient une erreur

```typescript
function sequence(a) {
  const b = f1(a);
  if (b === undefined) return undefined;
  const c = f2(b);
  if (c === undefined) return undefined;
  return f3(c);
}
```

La manière la plus raisonnable de les combiner en séquence est simplement extraire une fonction.

On peut aussi lancer une exception et l'attraper. Possible avec n'importe quelle valuer, surtout utiliser avec les instances d'erreur. Typescript ne suit pas le type des exception, donc l'exhaustivité ne fonctionne pas. (On peu limiter les dégats avec les sous-classes, mais beaucoup de gestion manuelle et jamais de vraie garantie d'exhaustivité.)

On peut juste retourner une erreur comme une grande personne, pas besoin de de la lancer partout.

Pour le code asynchrone, on peut dire des promesses à peu près ce qu'on a déjà dit des exceptions.

## Result type (Either Modad)

```typescript
const s = {
    type: 'success'
    value: 3
}

const e = {
    type: 'error',
    message: 'not happy'
}
```

- on peut imbriquer une erreur dans un succès
- les types gardent la trace des erreurs potentielles, peut porter de l'information arbitraire
- sérialisable ou non (selon l'implémentation)
- les séquences sont plus verbeuses parce qu'il faut toujours déballer la valeur du succès

```typescript
function sequence(a) {
  const b = f1(a);
  if (b.type === "error") return b;
  const c = f2(b.value);
  if (c.type === "error") return c;
  return f3(c.value);
}
```

libraries

## Erreur comme connotation

- couleur rouge
- icones

Plutôt lié au UX et au design. Va souvent être lié à une erreur attendue ou une erreur inattendue dans le code, mais pas nécessairement.
