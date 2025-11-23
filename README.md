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

Quand c'est possible, préférer une assertion à un cast plutôt que de faire un cast

- coût négligeable
- mieux maintenue (changement de nom, erreurs de types)
- plus grande confiance

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


## Errers concrètes que j'ai attrapé

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
	if (condition === false) throw new Error(message ?? 'assertion failed')
}

isoAssert(x > y, 'x should be greater than y')
```


## Erreurs attendues (Expected Errors)

![Love means never having to say you're sorry.](https://media.tenor.com/EyurLM2pd6YAAAAC/love-story-sorry.gif)

## Varia

erreur technique vs erreur métier
