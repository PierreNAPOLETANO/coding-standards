# Standards de code — [Projet]

**Version 1.0 — 20/10/2026**
**Cible :** PHP 8.3 / Laravel 11 / TypeScript / React

Ce document est normatif.

- **DOIT** indique une règle obligatoire.
- **NE DOIT PAS** indique une interdiction.
- **PEUT** indique une option autorisée.
- Une règle s'applique à tout le code nouveau et modifié.
- Une exception doit être écrite dans la revue de code.

## Table des matières

1. [Principes généraux](#1-principes-généraux)
2. [Formatage](#2-formatage)
3. [Nommage](#3-nommage)
4. [Structure du code et organisation des fichiers](#4-structure-du-code-et-organisation-des-fichiers)
5. [Commentaires et documentation](#5-commentaires-et-documentation)
6. [Gestion des erreurs](#6-gestion-des-erreurs)
7. [Performance](#7-performance)
8. [Style inspiré de Bisqwit](#8-style-inspiré-de-bisqwit)
9. [Tests](#9-tests)
10. [Git et revue de code](#10-git-et-revue-de-code)
11. [Sécurité](#11-sécurité)
12. [Exceptions et processus de modification du standard](#12-exceptions-et-processus-de-modification-du-standard)
13. [Checklist finale](#13-checklist-finale)

## 1. Principes généraux

1. Le code DOIT être lisible avant d'être astucieux.
2. Une fonction DOIT avoir une seule responsabilité.
3. Une fonction NE DOIT PAS dépasser environ 20 lignes hors lignes vides et commentaires.
4. Le code mort DOIT être supprimé.
5. Le code NE DOIT PAS être commenté pour être conservé.
6. Le code DOIT appliquer DRY : une règle métier ne doit exister qu'à un endroit.
7. Le code DOIT appliquer YAGNI : une capacité non requise maintenant ne doit pas être ajoutée.
8. Le code DOIT appliquer KISS : la solution la plus simple qui respecte le besoin est choisie.
9. Chaque modification DOIT suivre la Boy Scout rule : le code touché doit être au moins aussi propre qu'avant.
10. Les modules DOIVENT être orthogonaux : une modification locale ne doit pas créer de régression ailleurs.
11. Les entrées invalides DOIVENT échouer tôt avec une erreur explicite.
12. Le code DOIT s'inspirer de _Clean Code_ de Robert C. Martin et de _The Pragmatic Programmer_.
13. Une règle locale plus spécifique prime sur une règle générale de ce document.

✅ Bon : une fonction courte avec une intention visible.

```php
<?php

final class PriceCalculator
{
    public function calculate_total(int $unitPrice, int $quantity): int
    {
        $this->assert_positive_quantity($quantity);

        return $unitPrice * $quantity;
    }

    private function assert_positive_quantity(int $quantity): void
    {
        if ($quantity < 1) {
            throw new InvalidArgumentException('La quantité doit être positive.');
        }
    }
}
```

❌ Mauvais : une fonction mélange validation, accès réseau, calcul et affichage.

```php
function process_order(array $order): void
{
    // 80 lignes non testables : SQL, calcul, email, HTML et logs mélangés.
}
```

## 2. Formatage

1. PHP DOIT être formaté par Laravel Pint avec le profil PSR-12.
2. TypeScript et TSX DOIVENT être formatés par Prettier.
3. TypeScript et TSX DOIVENT être contrôlés par ESLint.
4. Le formatage DOIT être exécuté en local et en CI.
5. L'indentation DOIT utiliser quatre espaces en PHP.
6. L'indentation DOIT utiliser deux espaces en TypeScript et TSX.
7. Une ligne DOIT contenir au maximum 120 caractères.
8. Les accolades DOIVENT être présentes pour tout bloc conditionnel et toute boucle.
9. Une expression binaire DOIT avoir un espace autour de son opérateur.
10. Les virgules DOIVENT être suivies d'un espace.
11. Les imports inutilisés DOIVENT être supprimés.
12. Le code généré NE DOIT PAS être reformatté manuellement.
13. Les noms de fichiers de configuration imposés (`pint.json`, `.prettierrc`, `eslint.config.js`) sont des exceptions aux noms de fichiers PascalCase.

### Configuration minimale

`pint.json` :

```json
{
  "preset": "psr12",
  "rules": {
    "array_indentation": true,
    "no_unused_imports": true,
    "ordered_imports": { "sort_algorithm": "alpha" }
  }
}
```

`.prettierrc` :

```json
{
  "singleQuote": true,
  "semi": true,
  "trailingComma": "all",
  "printWidth": 120,
  "tabWidth": 2,
  "useTabs": false
}
```

`eslint.config.js` :

```js
import tseslint from 'typescript-eslint'
import react from 'eslint-plugin-react'
import namingConvention from '@typescript-eslint/eslint-plugin'

export default tseslint.config(...tseslint.configs.recommended, {
  plugins: { react, '@typescript-eslint': namingConvention },
  rules: {
    '@typescript-eslint/naming-convention': [
      'error',
      { selector: 'variable', format: ['camelCase', 'UPPER_CASE'] },
      {
        selector: 'function',
        format: ['snake_case'],
        leadingUnderscore: 'allow'
      },
      {
        selector: 'parameter',
        format: ['camelCase'],
        leadingUnderscore: 'allow'
      },
      { selector: 'typeLike', format: ['PascalCase'] },
      { selector: 'interface', format: ['PascalCase'] },
      { selector: 'property', format: null }
    ]
  }
})
```

La règle ESLint ci-dessus s'applique aux fonctions ordinaires. Les composants React et les hooks suivent l'exception obligatoire de la section 3.

✅ Bon :

```tsx
const MAX_VISIBLE_ITEMS = 50

function format_label(labelText: string): string {
  return labelText.trim()
}
```

❌ Mauvais :

```tsx
function formatLabel(labelText: string) {
  return labelText.trim()
}
```

## 3. Nommage

### 3.1 Tableau des règles

| Élément                        | Format              | Exemple                       |
| ------------------------------ | ------------------- | ----------------------------- |
| Variable (PHP et TS)           | camelCase           | `$totalPrice`, `totalPrice`   |
| Méthode / fonction (PHP et TS) | snake_case          | `calculate_total()`           |
| Classe, interface, type        | PascalCase          | `InvoiceService`, `UserProps` |
| Fichier de classe PHP          | PascalCase (PSR-4)  | `InvoiceService.php`          |
| Composant React                | PascalCase          | `UserCard.tsx`                |
| Hook React                     | `use_` + snake_case | `use_user_list.ts` (voir 3.4) |
| Fichier utilitaire TS          | snake_case          | `format_date.ts`              |
| Constante                      | UPPER_SNAKE_CASE    | `MAX_RETRY_COUNT`             |
| Table SQL                      | snake_case pluriel  | `order_items`                 |
| Colonne SQL                    | snake_case          | `created_at`                  |
| Route (URL)                    | kebab-case          | `/user-profiles`              |
| Variable `.env`                | UPPER_SNAKE_CASE    | `STRIPE_SECRET`               |

### 3.2 Choix volontaire : méthodes en snake_case

> **Décision** : les méthodes et fonctions sont en `snake_case`, contrairement à PSR-1 (PHP) et aux conventions JavaScript/TypeScript.
>
> **Raison** : le caractère `_` sépare visuellement les mots et réduit l'effort de lecture pour les personnes dyslexiques. C'est un choix d'accessibilité, pas un oubli.

### 3.3 Exemples

```php
// ❌ Incorrect
public function calculateTotal(Cart $cart): int {}
public function CalculateTotal(Cart $cart): int {}
public function calc(Cart $cart): int {}

// ✅ Correct
public function calculate_total(Cart $cart): int {}
public function send_welcome_email(User $user): void {}
public function is_expired(): bool {}
```

```ts
// ❌ Incorrect
function formatDate(date: Date): string {}
function fd(date: Date): string {}

// ✅ Correct
function format_date(date: Date): string {}
function parse_user_input(raw: string): UserInput {}
function is_valid_email(email: string): boolean {}
```

```ts
// ✅ Les variables restent en camelCase : on distingue
// visuellement une action (snake_case) d'une donnée (camelCase)
const totalPrice = calculate_total(cart)
const userList = fetch_users()
```

Règles de nommage complémentaires :

| Règle                                   | ❌ Incorrect                          | ✅ Correct                |
| --------------------------------------- | ------------------------------------- | ------------------------- |
| Verbe pour une action                   | `user_data()`                         | `get_user_data()`         |
| `is_` / `has_` / `can_` pour un booléen | `valid()`                             | `is_valid()`              |
| Pas d'abréviation obscure               | `calc_ttl_prc()`                      | `calculate_total_price()` |
| Un nom = un sens                        | `handle()`, `process()`, `do_stuff()` | `archive_invoice()`       |

### 3.4 Exceptions imposées par les frameworks

Certains noms ne peuvent pas être changés : le framework les appelle lui-même. Les renommer casse l'application.

**PHP / Laravel : noms à garder tels quels**

| Catégorie                      | Exemples à ne PAS renommer                                                           |
| ------------------------------ | ------------------------------------------------------------------------------------ |
| Méthodes magiques PHP          | `__construct`, `__toString`, `__invoke`, `__get`                                     |
| Cycle de vie Laravel           | `handle`, `boot`, `register`, `up`, `down`, `rules`, `authorize`, `casts`, `toArray` |
| Contrôleurs resource           | `index`, `show`, `create`, `store`, `edit`, `update`, `destroy`                      |
| Relations Eloquent             | `hasMany`, `belongsTo` (appels), mais nos propres relations : `order_items()`        |
| Scopes Eloquent                | `scopeActive` (préfixe `scope` obligatoire)                                          |
| Accesseurs / mutateurs         | `getNameAttribute`, `setNameAttribute`                                               |
| Méthodes de tests Pest/PHPUnit | `setUp`, `tearDown`                                                                  |

```php
// ✅ Les méthodes de Laravel gardent leur nom ; les nôtres sont en snake_case
class InvoiceController extends Controller
{
    public function store(StoreInvoiceRequest $request): JsonResponse   // imposé par Laravel
    {
        $invoice =  $this->invoice_service->create_from_request( $ request); // notre code

        return response()->json($invoice, 201);
    }
}
```

```php
// ✅ Relation Eloquent : notre propre méthode, donc snake_case
public function order_items(): HasMany
{
    return $this->hasMany(OrderItem::class);   // hasMany = méthode de Laravel, inchangée
}
```

**TypeScript / React : noms à garder tels quels**

| Catégorie                       | Règle                                                                           | Exemple                                 |
| ------------------------------- | ------------------------------------------------------------------------------- | --------------------------------------- |
| Composants React                | PascalCase obligatoire, sinon React les traite comme du HTML                    | `UserCard`                              |
| Hooks                           | Le préfixe `use` est obligatoire, sinon le linter de React ne les reconnaît pas | `use_user_list`                         |
| API du navigateur et librairies | Appels inchangés                                                                | `addEventListener`, `useState`, `fetch` |
| Props React standard            | Inchangées                                                                      | `onClick`, `className`                  |
| Méthodes JS natives             | Inchangées                                                                      | `.map()`, `.forEach()`, `.toString()`   |

```tsx
// ✅ Hook personnalisé : préfixe "use_" + snake_case
//    (React exige que le nom commence par "use")
function use_user_list(): User[] {
  const [userList, setUserList] = useState<User[]>([]) // useState : API React, inchangée

  useEffect(() => {
    fetch_users().then(setUserList) // notre fonction : snake_case
  }, [])

  return userList
}
```

```tsx
// ✅ Composant : PascalCase obligatoire ; ses fonctions internes en snake_case
function UserCard({ user }: UserCardProps) {
  function handle_delete_click(): void {
    delete_user(user.id)
  }

  return <button onClick={handle_delete_click}>Supprimer</button>
}
```

> **Attention pour les hooks** : `use_user_list` commence bien par `use`, donc `eslint-plugin-react-hooks` le reconnaît. Un nom comme `user_list_hook` ne serait pas reconnu et désactiverait les vérifications.

### 3.5 Précautions à connaître

| Situation                                               | Effet                               | Solution                                                                                                   |
| ------------------------------------------------------- | ----------------------------------- | ---------------------------------------------------------------------------------------------------------- |
| Un contributeur utilise un IDE qui suggère du camelCase | Incohérences                        | Vérification à la relecture de code (section 7)                                                            |
| Une librairie externe expose du camelCase               | Mélange de styles dans le code      | Acceptable : on n'enveloppe pas chaque appel de librairie                                                  |
| Réponses JSON de l'API                                  | Convention à choisir                | **snake_case** pour les clés JSON (Laravel le fait par défaut), converti en camelCase côté React si besoin |
| `README` du dépôt                                       | Les contributeurs doivent le savoir | Ajouter une ligne « Méthodes en snake_case : voir STANDARDS section 3.2 »                                  |

```ts
// Exemple : clés JSON snake_case reçues de Laravel
interface UserResponse {
  id: number
  first_name: string // clé JSON : snake_case (venant de Laravel)
  created_at: string
}

// Variable locale : camelCase
const firstName = response.first_name
```

## 4. Structure du code et organisation des fichiers

1. Un fichier DOIT contenir une classe principale ou un module cohérent.
2. Un fichier DOIT rester sous environ 300 lignes.
3. Une classe DOIT rester sous environ 200 lignes.
4. Une fonction DOIT rester sous environ 20 lignes.
5. Une Action DOIT représenter un cas d'usage unique.
6. Un Service DOIT contenir une logique métier partagée par plusieurs cas d'usage.
7. Une Form Request DOIT valider et autoriser une requête entrante.
8. Un contrôleur DOIT orchestrer et déléguer ; il ne doit pas contenir la logique métier.
9. Les modèles Eloquent DOIVENT rester centrés sur les données et leurs relations.
10. La structure Laravel DOIT suivre cette base :

```text
app/
├── Actions/
│   └── CreateOrderAction.php
├── Http/Requests/
│   └── StoreOrderRequest.php
├── Models/
│   └── Order.php
├── Services/
│   └── OrderPricingService.php
└── Exceptions/
    └── OrderCannotBeCreatedException.php
```

11. La structure React DOIT être organisée par fonctionnalité :

```text
resources/js/
├── features/orders/
│   ├── components/OrderCard.tsx
│   ├── hooks/useOrder.ts
│   ├── services/OrderApi.ts
│   ├── types/Order.ts
│   └── pages/OrdersPage.tsx
├── shared/components/Button.tsx
└── app/App.tsx
```

12. Les éléments partagés DOIVENT être placés dans `shared` uniquement après un second usage réel.
13. Une abstraction créée pour un seul appel PEUT rester locale.

✅ Bon :

```php
final class CreateOrderAction
{
    public function __construct(private OrderPricingService $pricingService) {}

    public function execute(StoreOrderRequest $request): Order
    {
        $validatedData = $request->validated();
        $totalAmount = $this->pricingService->calculate_total($validatedData['items']);

        return Order::create([
            'user_id' => $request->user()->id,
            'total_amount' => $totalAmount,
        ]);
    }
}
```

✅ Bon côté React :

```tsx
export function OrdersPage() {
  const orders = useOrderList()
  return <OrderList orders={orders} />
}
```

## 5. Commentaires et documentation

1. Un commentaire DOIT expliquer pourquoi une décision existe.
2. Un commentaire NE DOIT PAS répéter ce que le code dit déjà.
3. Un commentaire obsolète DOIT être supprimé lors de la modification du code.
4. PHPDoc DOIT documenter les types complexes, les effets et les exceptions publiques.
5. TSDoc DOIT documenter une API publique, une contrainte ou un comportement surprenant.
6. Les commentaires TODO DOIVENT contenir un ticket ou une date d'expiration.
7. Un README DOIT exister à la racine.

✅ Bon :

```php
/**
 * Retourne le total en centimes pour éviter les erreurs d'arrondi flottant.
 *
 * @param list<array{unit_price:int, quantity:int}> $items
 */
public function calculate_total(array $items): int
{
    return array_sum(array_map(
        static fn (array $item): int => $item['unit_price'] * $item['quantity'],
        $items,
    ));
}
```

✅ Bon TSDoc :

```tsx
/** Charge une page seulement après l'ouverture du panneau. */
function useOrderPanel(): void {
  // Le chargement différé réduit le coût initial de la page.
}
```

### Modèle de README obligatoire

```markdown
# [Projet]

## Objectif

Une phrase qui décrit le problème résolu.

## Prérequis

- PHP 8.3
- Node.js LTS
- Base de données configurée

## Installation

1. Copier `.env.example` vers `.env`.
2. Exécuter `composer install`.
3. Exécuter `npm ci`.
4. Exécuter `php artisan migrate`.

## Développement

- `composer run dev`
- `npm run lint`
- `php artisan test`

## Déploiement

Décrire les étapes et les variables nécessaires.

## Sécurité

Indiquer le canal de signalement d'une vulnérabilité.
```

## 6. Gestion des erreurs

1. Une erreur métier DOIT utiliser une exception dédiée.
2. Les exceptions DOIVENT être préférées aux codes de retour ambigus.
3. Une exception DOIT contenir un message exploitable par un développeur.
4. Un `catch` vide est interdit.
5. Un `catch` DOIT soit traiter l'erreur, soit l'enrichir et la relancer.
6. Les détails internes NE DOIVENT PAS être exposés à l'utilisateur.
7. Laravel DOIT journaliser les erreurs avec un contexte utile.
8. React DOIT utiliser un Error Boundary autour des zones récupérables.
9. Un type `Result` PEUT être utilisé quand l'échec est une réponse métier normale.

✅ Bon Laravel :

```php
final class OrderCannotBeCreatedException extends RuntimeException
{
    public function __construct(string $reason)
    {
        parent::__construct("La commande ne peut pas être créée : {$reason}");
    }
}

try {
    $order = $action->execute($request);
} catch (OrderCannotBeCreatedException $exception) {
    Log::warning('Création de commande refusée', [
        'userId' => $request->user()->id,
        'reason' => $exception->getMessage(),
    ]);

    throw $exception;
}
```

✅ Bon TypeScript :

```ts
type Result<T> = { ok: true; value: T } | { ok: false; error: string }

function parse_order_id(rawValue: string): Result<string> {
  const orderId = rawValue.trim()
  return orderId === '' ?
      { ok: false, error: 'Identifiant vide' }
    : { ok: true, value: orderId }
}
```

✅ Bon Error Boundary :

```tsx
class OrderErrorBoundary extends React.Component<
  React.PropsWithChildren,
  { hasError: boolean }
> {
  state = { hasError: false }

  static getDerivedStateFromError(): { hasError: boolean } {
    return { hasError: true }
  }

  render() {
    return this.state.hasError ?
        <p>Impossible d'afficher les commandes.</p>
      : this.props.children
  }
}
```

## 7. Performance

### 7.1 Principe

> **Mesurer d'abord. Optimiser ensuite.**
> Dans une application Laravel + React, les lenteurs viennent dans cet ordre :
>
> 1. La base de données (requêtes en trop, index manquants)
> 2. Le réseau (trop d'appels API, réponses trop lourdes)
> 3. La taille du JavaScript envoyé au navigateur
> 4. La mémoire (gros volumes chargés d'un coup)
> 5. Le CPU (rarement)

### 7.2 Outils de mesure

| Côté            | Outil                               |
| --------------- | ----------------------------------- |
| Laravel         | Laravel Debugbar, Laravel Telescope |
| Base de données | `EXPLAIN` sur les requêtes lentes   |
| React           | React DevTools Profiler             |
| Navigateur      | Lighthouse, onglet Network          |

### 7.3 Base de données — éviter le N+1

```php
// ❌ Incorrect — 1 requête + 1 requête par commande
$orders = Order::all();
foreach ($orders as $order) {
    echo $order->customer->name;
}

// ✅ Correct — 2 requêtes au total
$orders = Order::with('customer')->get();
foreach ($orders as $order) {
    echo $order->customer->name;
}
```

Activer la détection automatique en développement :

```php
// app/Providers/AppServiceProvider.php
public function boot(): void
{
    Model::preventLazyLoading(! app()->isProduction());
}
```

### 7.4 Mémoire — traiter les gros volumes par morceaux

```php
// ❌ Incorrect — charge 1 million de lignes en mémoire
User::all()->each(fn (User $user) =>  $this->export($user));

// ✅ Correct — charge les lignes une par une
User::lazy()->each(fn (User $user) =>  $this->export($user));
```

### 7.5 Frontend — réduire le JavaScript chargé

```tsx
// ✅ Correct — la page n'est chargée que si on la visite
import { lazy, Suspense } from 'react'

const ReportsPage = lazy(() => import('./pages/ReportsPage'))

export function App() {
  return (
    <Suspense fallback={<Spinner />}>
      <ReportsPage />
    </Suspense>
  )
}
```

### 7.6 Frontend — éviter les rendus inutiles (seulement si mesuré)

```tsx
// ✅ Utiliser useMemo seulement si le Profiler montre un calcul lent
const sortedUsers = useMemo(
  () => [...users].sort((a, b) => a.lastName.localeCompare(b.lastName)),
  [users]
)
```

> ❌ Ne pas mettre `useMemo` et `useCallback` partout « par précaution ».
> Ça rend le code plus difficile à lire, sans gain mesurable.

## 8. Sobriété du code (inspirée de Bisqwit)

### 8.1 Ce qu'on garde de Bisqwit

Bisqwit est un programmeur connu pour son code efficace et sans superflu.
On garde **son état d'esprit**, pas sa mise en forme :

| On garde ✅                           | On ne garde pas ❌                                            |
| ------------------------------------- | ------------------------------------------------------------- |
| Comprendre ce que fait la machine     | Le code compressé sur une seule ligne                         |
| Pas d'abstraction inutile             | Les astuces difficiles à lire                                 |
| Pas de dépendance inutile             | L'alignement vertical manuel (Pint et Prettier le suppriment) |
| Choisir la bonne structure de données | Les noms de variables d'une lettre                            |

> En cas de conflit, **la lisibilité gagne toujours** (voir section 1).

### 8.2 Pas d'abstraction inutile

```php
// ❌ Incorrect — une interface + une fabrique pour une seule implémentation
interface PriceCalculatorInterface { public function calculate(Cart $cart): int; }
class PriceCalculatorFactory { public function make(): PriceCalculatorInterface { /* ... */ } }

// ✅ Correct — une seule classe, tant qu'il n'y a pas de 2e implémentation
final class PriceCalculator
{
    public function calculate(Cart $cart): int
    {
        return $cart->items->sum(fn (CartItem $item) => $item->price * $item->quantity);
    }
}
```

### 8.3 Pas de dépendance inutile

```ts
// ❌ Incorrect — installer une librairie pour une seule ligne
import isEmpty from 'lodash/isEmpty'
if (isEmpty(items)) {
  /* ... */
}

// ✅ Correct — le langage le fait déjà
if (items.length === 0) {
  /* ... */
}
```

### 8.4 Choisir la bonne structure de données

```ts
// ❌ Incorrect — recherche lente dans un tableau (parcourt tout)
const allowedIds = [3, 8, 15, 42]
if (allowedIds.includes(userId)) {
  /* ... */
}

// ✅ Correct — recherche instantanée dans un Set
const allowedIds = new Set([3, 8, 15, 42])
if (allowedIds.has(userId)) {
  /* ... */
}
```

> Ce dernier exemple compte surtout pour les grandes listes (plusieurs milliers d'éléments).

## 9. Tests

1. Le backend DOIT utiliser Pest ou PHPUnit.
2. Le frontend DOIT utiliser Vitest et React Testing Library.
3. Chaque test DOIT suivre Arrange, Act, Assert (AAA).
4. Le nom d'un test DOIT décrire le comportement attendu.
5. Un test NE DOIT PAS dépendre de l'ordre d'exécution d'un autre test.
6. Les tests unitaires DOIVENT être rapides et isolés.
7. Les tests d'intégration DOIVENT vérifier les contrats entre composants.
8. Chaque bug corrigé DOIT recevoir un test de régression.
9. La couverture minimale DOIT être de 80 % sur le code métier.
10. Une baisse sous 80 % DOIT bloquer la CI sauf exception documentée.
11. Un test DOIT vérifier le cas nominal et au moins un cas d'erreur pertinent.

✅ Bon Pest :

```php
it('calcule le total en centimes', function () {
    // Arrange
    $calculator = new PriceCalculator();

    // Act
    $totalAmount = $calculator->calculate_total(1250, 2);

    // Assert
    expect($totalAmount)->toBe(2500);
});
```

✅ Bon Vitest :

```tsx
test('affiche le numéro de commande', () => {
  // Arrange
  render(<OrderCard orderId='A-1' />)

  // Act
  const orderNumber = screen.getByRole('link', { name: 'Commande A-1' })

  // Assert
  expect(orderNumber).toBeVisible()
})
```

## 10. Git et revue de code

1. La branche `main` DOIT être protégée.
2. `develop` PEUT regrouper les changements avant une livraison.
3. Une fonctionnalité DOIT partir de `feature/<nom-court>`.
4. Une correction DOIT partir de `fix/<nom-court>`.
5. Un commit DOIT suivre Conventional Commits au format Angular.
6. Le format DOIT être `<type>(<scope>): <description>`.
7. Les types autorisés sont `feat`, `fix`, `docs`, `style`, `refactor`, `perf`, `test`, `build`, `ci`, `chore`, `revert`.
8. Un breaking change DOIT utiliser `!` et une section `BREAKING CHANGE:`.
9. Une PR DOIT rester sous 400 lignes ajoutées ou supprimées.
10. Une PR au-dessus de 400 lignes DOIT être découpée ou justifiée.
11. Une PR DOIT avoir un objectif, un test, un risque et une procédure de retour arrière.
12. Une PR DOIT avoir au moins une revue avant fusion.

✅ Bons commits :

```text
feat(orders): add order cancellation
fix(auth): reject expired session
refactor(pricing)!: replace float amounts with cents
```

```text
feat(api)!: change order response

BREAKING CHANGE: `total` devient un entier en centimes.
```

### Checklist de revue

- [ ] Le besoin est compris.
- [ ] Les noms suivent ce standard.
- [ ] La fonction reste courte.
- [ ] Les erreurs sont traitées.
- [ ] Les entrées sont validées.
- [ ] Les tests couvrent le cas nominal et l'erreur.
- [ ] Aucun secret n'est commité.
- [ ] La performance a été mesurée si nécessaire.
- [ ] La documentation est à jour.
- [ ] La PR contient moins de 400 lignes ou une justification.

## 11. Sécurité (OWASP Top 10 : 2025)

### 11.0 Règles de base

| Règle                                | Détail                                                 |
| ------------------------------------ | ------------------------------------------------------ |
| Aucun secret dans le code            | Clés API, mots de passe, tokens : toujours dans `.env` |
| `.env` jamais dans Git               | Fichier `.env.example` sans valeurs réelles            |
| Toute entrée utilisateur est hostile | Validée côté serveur, toujours                         |
| Dépendances à jour                   | `composer audit` et `npm audit` avant chaque release   |
| Principe du moindre privilège        | Chaque accès est refusé par défaut                     |

---

### A01 — Contrôle d'accès défaillant

**Risque** : un utilisateur accède aux données d'un autre (IDOR), ou à une action réservée.

```php
// ❌ Ne pas faire — n'importe quel utilisateur connecté voit n'importe quelle facture
public function show(int $id)
{
    return Invoice::findOrFail($id);
}

// ✅ Faire — une Policy vérifie que la facture appartient à l'utilisateur
public function show(Invoice $invoice)
{
    $this->authorize('view', $invoice);

    return $invoice;
}

// app/Policies/InvoicePolicy.php
public function view(User $user, Invoice $invoice): bool
{
    return $user->id === $invoice->user_id;
}
```

```php
// ❌ Ne pas faire — route sans protection
Route::delete('/users/{user}', [UserController::class, 'destroy']);

// ✅ Faire — authentification + autorisation
Route::delete('/users/{user}', [UserController::class, 'destroy'])
    ->middleware(['auth', 'can:delete,user']);
```

```tsx
// ❌ Ne pas faire — cacher un bouton ne protège rien
{
  user.isAdmin && <DeleteButton />
} // seul contrôle = le frontend

// ✅ Faire — le frontend cache le bouton, le BACKEND refuse l'action
{
  user.isAdmin && <DeleteButton />
} // confort d'affichage uniquement
// + Policy côté Laravel (voir ci-dessus) = vraie protection
```

---

### A02 — Mauvaise configuration de sécurité

**Risque** : mode debug en production, erreurs détaillées visibles, configuration par défaut.

```dotenv
# ❌ Ne pas faire — en production
APP_ENV=local
APP_DEBUG=true

# ✅ Faire — en production
APP_ENV=production
APP_DEBUG=false
```

```php
// ❌ Ne pas faire — CORS ouvert à tout le monde
// config/cors.php
'allowed_origins' => ['*'],

// ✅ Faire — origines explicitement listées
'allowed_origins' => [env('FRONTEND_URL')],
```

```php
// ✅ Faire — en-têtes de sécurité (middleware)
public function handle(Request $request, Closure $next): Response
{
    $response =  $next($request);

    $response->headers->set('X-Content-Type-Options', 'nosniff');
    $response->headers->set('X-Frame-Options', 'DENY');
    $response->headers->set('Referrer-Policy', 'strict-origin-when-cross-origin');
    $response->headers->set('Strict-Transport-Security', 'max-age=31536000; includeSubDomains');

    return $response;
}
```

---

### A03 — Défaillances de la chaîne d'approvisionnement logicielle

**Risque** : une dépendance compromise ou vulnérable infecte l'application.

```bash
# ❌ Ne pas faire — installer sans regarder, ne jamais auditer
npm install paquet-inconnu-4-telechargements

# ✅ Faire — vérifier avant d'installer, auditer régulièrement
npm audit
composer audit
```

```jsonc
// ❌ Ne pas faire — version flottante "latest"
"dependencies": { "some-lib": "latest" }

// ✅ Faire — version précise + fichier de verrouillage commité
"dependencies": { "some-lib": "4.2.1" }
// package-lock.json et composer.lock toujours dans Git
```

| Avant d'ajouter une dépendance | Vérifier                                     |
| ------------------------------ | -------------------------------------------- |
| Maintenance                    | Dernière mise à jour récente                 |
| Popularité                     | Nombre de téléchargements, étoiles           |
| Nécessité                      | Le langage ou le framework le fait-il déjà ? |
| Licence                        | Compatible avec le projet                    |

---

### A04 — Défaillances cryptographiques

**Risque** : mots de passe lisibles, données sensibles en clair, algorithmes faibles.

```php
// ❌ Ne pas faire — hash faible, réversible par force brute en secondes
$user->password = md5($password);
$user->password = sha1($password);

// ✅ Faire — hash adapté aux mots de passe (bcrypt/argon2 via Laravel)
$user->password = Hash::make($password);

// Vérification
if (Hash::check($password, $user->password)) { /* ... */ }
```

```php
// ❌ Ne pas faire — donnée sensible stockée en clair
$user->national_id = $request->national_id;

// ✅ Faire — chiffrement via le cast Laravel
// app/Models/User.php
protected function casts(): array
{
    return [
        'national_id' => 'encrypted',
        'password'    => 'hashed',
    ];
}
```

```php
// ❌ Ne pas faire — jeton prévisible
$token = md5((string) time());
$token = rand();

// ✅ Faire — aléatoire cryptographiquement sûr
$token = Str::random(64);
// ou
$token = bin2hex(random_bytes(32));
```

---

### A05 — Injection

**Risque** : SQL, commande système, XSS. Le code de l'attaquant est exécuté.

**Injection SQL**

```php
// ❌ Ne pas faire — concaténation dans la requête
$users = DB::select("SELECT * FROM users WHERE email = ' $email'");
// Avec $email = "' OR '1'='1" → toute la table est renvoyée

// ✅ Faire — requête préparée (liaison de paramètres)
$users = DB::select('SELECT * FROM users WHERE email = ?', [ $email]);

// ✅ Mieux — Eloquent / Query Builder
$users = User::where('email', $email)->get();
```

```php
// ❌ Ne pas faire — nom de colonne venant de l'utilisateur, sans contrôle
$users = User::orderBy($request->input('sort'))->get();

// ✅ Faire — liste blanche
$allowedColumns = ['name', 'created_at'];
$sort = in_array($request->input('sort'), $allowedColumns, true)
    ? $request->input('sort')
    : 'created_at';

$users = User::orderBy($sort)->get();
```

**Injection de commande**

```php
// ❌ Ne pas faire — entrée utilisateur dans une commande shell
exec("convert " . $request->input('file') . " output.png");

// ✅ Faire — arguments séparés + validation
use Illuminate\Support\Facades\Process;

$result = Process::run(['convert', $validatedFilePath, 'output.png']);
```

**XSS (Cross-Site Scripting)**

```php
{{-- ❌ Ne pas faire — Blade, affichage brut --}}
{!! $comment->body !!}

{{-- ✅ Faire — Blade échappe automatiquement --}}
{{ $comment->body }}
```

```tsx
// ❌ Ne pas faire — React, injection de HTML brut
<div dangerouslySetInnerHTML={{ __html: comment.body }} />

// ✅ Faire — React échappe automatiquement
<div>{comment.body}</div>

// ✅ Si le HTML est vraiment nécessaire : nettoyer avec DOMPurify
import DOMPurify from 'dompurify';

<div dangerouslySetInnerHTML={{ __html: DOMPurify.sanitize(comment.body) }} />
```

```tsx
// ❌ Ne pas faire — URL venant de l'utilisateur utilisée sans contrôle
;<a href={user.website}>Site web</a>
// Avec website = "javascript:alert(1)"

// ✅ Faire — n'accepter que http(s)
function isSafeUrl(url: string): boolean {
  try {
    const { protocol } = new URL(url)
    return protocol === 'https:' || protocol === 'http:'
  } catch {
    return false
  }
}

{
  isSafeUrl(user.website) && (
    <a href={user.website} rel='noopener noreferrer'>
      Site web
    </a>
  )
}
```

---

### A06 — Conception non sécurisée

**Risque** : la faille est dans la logique, pas dans une ligne de code.

```php
// ❌ Ne pas faire — aucune limite sur la connexion (force brute possible)
Route::post('/login', [AuthController::class, 'login']);

// ✅ Faire — limitation du nombre de tentatives
Route::post('/login', [AuthController::class, 'login'])
    ->middleware('throttle:5,1');   // 5 tentatives par minute
```

```php
// ❌ Ne pas faire — le client décide du prix
$order->total = $request->input('total');

// ✅ Faire — le serveur recalcule toujours
$order->total =  $this->priceCalculator->calculate($cart);
```

```php
// ❌ Ne pas faire — attribuer en masse tout ce que l'utilisateur envoie
 $user->update($request->all());
// Avec "is_admin": true dans la requête → élévation de privilèges

// ✅ Faire — seulement les champs validés
 $user->update($request->validated());
```

| Question à se poser avant de coder                              |     |
| --------------------------------------------------------------- | --- |
| Que peut faire un utilisateur malveillant avec cette fonction ? |     |
| Qu'est-ce qui se passe s'il l'appelle 10 000 fois ?             |     |
| Qu'est-ce qui se passe s'il modifie la requête ?                |     |

---

### A07 — Défaillances d'authentification

**Risque** : comptes volés, sessions détournées, mots de passe faibles.

```php
// ❌ Ne pas faire — aucune règle sur le mot de passe
'password' => ['required'],

// ✅ Faire — règles solides
use Illuminate\Validation\Rules\Password;

'password' => [
    'required',
    'confirmed',
    Password::min(12)->letters()->mixedCase()->numbers()->uncompromised(),
],
```

```php
// ❌ Ne pas faire — message qui révèle si l'email existe
if (! $user) {
    return back()->withErrors(['email' => 'Cet email n\'existe pas.']);
}
return back()->withErrors(['password' => 'Mot de passe incorrect.']);

// ✅ Faire — même message dans tous les cas
return back()->withErrors(['email' => 'Identifiants incorrects.']);
```

```php
// ❌ Ne pas faire — session conservée après connexion
Auth::login($user);

// ✅ Faire — régénérer la session (empêche la fixation de session)
Auth::login($user);
$request->session()->regenerate();

// À la déconnexion
Auth::logout();
$request->session()->invalidate();
$request->session()->regenerateToken();
```

```tsx
// ❌ Ne pas faire — token stocké dans localStorage (lisible par n'importe quel script XSS)
localStorage.setItem('token', response.token)

// ✅ Faire — cookie HttpOnly posé par le serveur (illisible par JavaScript)
// Avec Laravel Sanctum en mode SPA : le cookie est géré automatiquement
await axios.get('/sanctum/csrf-cookie')
await axios.post('/login', credentials) // withCredentials: true
```

---

### A08 — Défaillances d'intégrité des logiciels et des données

**Risque** : données ou code modifiés sans contrôle (désérialisation, mises à jour non vérifiées).

```php
// ❌ Ne pas faire — désérialiser une donnée venant de l'utilisateur
$data = unserialize($request->input('payload'));
// Peut exécuter du code arbitraire

// ✅ Faire — JSON, avec classes autorisées explicitement si unserialize est inévitable
$data = json_decode($request->input('payload'), true, 512, JSON_THROW_ON_ERROR);
```

```php
// ❌ Ne pas faire — webhook accepté sans vérifier l'expéditeur
Route::post('/webhook/payment', function (Request  $request) {
    Order::find($request->order_id)->markAsPaid();
});

// ✅ Faire — vérifier la signature
Route::post('/webhook/payment', function (Request $request) {
    $expected = hash_hmac('sha256',  $request->getContent(), config('services.payment.secret'));

    abort_unless(hash_equals($expected,  $request->header('X-Signature', '')), 403);

    Order::findOrFail($request->order_id)->markAsPaid();
});
```

```php
// ❌ Ne pas faire — upload accepté sans contrôle
$request->file('document')->store('uploads');

// ✅ Faire — type, taille et extension validés
$request->validate([
    'document' => ['required', 'file', 'mimes:pdf,png,jpg', 'max:2048'],
]);

$request->file('document')->store('uploads');   // hors du dossier public/
```

---

### A09 — Défaillances de journalisation et d'alerte

**Risque** : une attaque a lieu et personne ne le voit. Ou les logs contiennent des secrets.

```php
// ❌ Ne pas faire — rien n'est journalisé
if (! Hash::check($password,  $user->password)) {
    return back();
}

// ✅ Faire — journaliser les événements de sécurité
if (! Hash::check($password, $user->password)) {
    Log::warning('Échec de connexion', [
        'email' => $email,
        'ip'    => $request->ip(),
    ]);

    return back()->withErrors(['email' => 'Identifiants incorrects.']);
}
```

```php
// ❌ Ne pas faire — secrets dans les logs
Log::info('Connexion', ['email' => $email, 'password' => $password]);
Log::info('Paiement', ['card_number' => $cardNumber]);

// ✅ Faire — jamais de mot de passe, de token ni de numéro de carte
Log::info('Connexion réussie', ['user_id' => $user->id]);
```

```php
// ❌ Ne pas faire — erreur avalée sans trace
try {
     $this->chargeCard($order);
} catch (Throwable) {
}

// ✅ Faire — log avec contexte, puis relancer ou gérer
try {
     $this->chargeCard($order);
} catch (PaymentFailedException $exception) {
    Log::error('Paiement échoué', [
        'order_id'  => $order->id,
        'exception' => $exception->getMessage(),
    ]);

    throw $exception;
}
```

| À journaliser                   | À ne jamais journaliser        |
| ------------------------------- | ------------------------------ |
| Connexions réussies et échouées | Mots de passe                  |
| Changements de droits           | Tokens et clés API             |
| Refus d'accès (403)             | Numéros de carte               |
| Actions d'administration        | Données personnelles complètes |

---

### A10 — Mauvaise gestion des conditions exceptionnelles

**Risque** : en cas d'erreur, l'application s'ouvre au lieu de se fermer, ou révèle des détails internes.

```php
// ❌ Ne pas faire — en cas d'erreur, l'accès est accordé
try {
    $allowed =  $this->permissionService->check($user, $resource);
} catch (Throwable) {
    $allowed = true;
}

// ✅ Faire — en cas d'erreur, l'accès est refusé (fail closed)
try {
    $allowed =  $this->permissionService->check($user, $resource);
} catch (Throwable $exception) {
    Log::error('Vérification des droits impossible', ['exception' => $exception]);
    $allowed = false;
}
```

```php
// ❌ Ne pas faire — message d'erreur technique envoyé à l'utilisateur
return response()->json(['error' => $exception->getMessage()], 500);
// Peut révéler : "SQLSTATE[42S02]: Table 'shop.users' not found"

// ✅ Faire — message générique, détail dans les logs
Log::error('Erreur création commande', ['exception' => $exception]);

return response()->json(['message' => 'Une erreur est survenue.'], 500);
```

```php
// ❌ Ne pas faire — opérations liées sans transaction
$order->save();
$this->reserveStock($order);    // si ça échoue : commande sans stock
$this->chargeCard($order);

// ✅ Faire — tout réussit ou tout est annulé
DB::transaction(function () use ($order) {
    $order->save();
    $this->reserveStock($order);
    $this->chargeCard($order);
});
```

```tsx
// ❌ Ne pas faire — l'erreur est ignorée, l'écran reste blanc
const data = await fetch('/api/orders').then(response => response.json())

// ✅ Faire — gérer l'erreur et informer l'utilisateur
async function loadOrders(): Promise<Order[]> {
  const response = await fetch('/api/orders')

  if (!response.ok) {
    throw new Error(`Chargement impossible (${response.status})`)
  }

  return response.json()
}
```

---

### 11.1 Validation des entrées (règle transversale)

```php
// ❌ Ne pas faire — validation dans le contrôleur, ou aucune
public function store(Request  $request)
{
    User::create($request->all());
}

// ✅ Faire — FormRequest dédiée
// app/Http/Requests/StoreUserRequest.php
public function rules(): array
{
    return [
        'name'  => ['required', 'string', 'max:255'],
        'email' => ['required', 'email:rfc,dns', 'max:255', 'unique:users,email'],
    ];
}

// Contrôleur
public function store(StoreUserRequest  $request)
{
    return User::create($request->validated());
}
```

```ts
// ✅ Côté frontend : validation pour l'expérience utilisateur seulement (Zod)
import { z } from 'zod'

const userSchema = z.object({
  name: z.string().min(1).max(255),
  email: z.string().email()
})

// ⚠️ Le frontend ne remplace JAMAIS la validation du backend
```

---

### 11.2 Secrets

```php
// ❌ Ne pas faire — secret écrit dans le code
$stripe = new StripeClient('sk_live_4eC39HqLyjWDarjtT1zdp7dc');

// ✅ Faire — secret dans .env, lu via la configuration
// .env
STRIPE_SECRET=sk_live_xxxxx

// config/services.php
'stripe' => ['secret' => env('STRIPE_SECRET')],

// Utilisation
$stripe = new StripeClient(config('services.stripe.secret'));
```

> ⚠️ Ne jamais appeler `env()` en dehors des fichiers de `config/`.
> Avec `php artisan config:cache`, `env()` renvoie `null` ailleurs.

```tsx
// ❌ Ne pas faire — secret dans le frontend (visible par tout le monde)
const API_SECRET = 'sk_live_xxxxx'

// ✅ Faire — le frontend n'a JAMAIS de secret
// Seules les clés publiques, préfixées VITE_, sont acceptables
const publicKey = import.meta.env.VITE_STRIPE_PUBLIC_KEY
```

Si un secret a été commité par erreur : le **révoquer immédiatement**, puis en générer un nouveau. Le supprimer du dépôt ne suffit pas, il reste dans l'historique Git.

---

### 11.3 Checklist sécurité avant chaque release

- [ ] `APP_DEBUG=false` et `APP_ENV=production`
- [ ] `composer audit` et `npm audit` sans faille critique
- [ ] Aucun secret dans le dépôt (`git log -p | grep -i "secret\|password\|key"`)
- [ ] Toutes les routes sensibles ont `auth` + une Policy
- [ ] Toutes les entrées passent par une `FormRequest`
- [ ] Aucun `{!! !!}` ni `dangerouslySetInnerHTML` non justifié
- [ ] Limitation de débit (`throttle`) sur login, inscription, réinitialisation
- [ ] Logs : événements de sécurité présents, aucun secret

## 12. Exceptions et processus de modification du standard

1. Une exception à ce standard DOIT être écrite dans la PR.
2. L'exception DOIT indiquer la règle, la raison, la durée et le plan de retour.
3. Une exception de sécurité DOIT être approuvée par le responsable technique.
4. Le standard suit le versionnage sémantique.
5. Une modification incompatible augmente le numéro majeur.
6. Une nouvelle règle compatible augmente le numéro mineur.
7. Une correction éditoriale augmente le numéro de patch.
8. Chaque version DOIT être inscrite dans `CHANGELOG.md`.
9. Le standard DOIT être relu au moins une fois par an.

### Format d'exception

```markdown
## Exception STD-2026-001

- Règle : 3.3
- Motif : dépendance externe imposant un nom différent.
- Portée : `VendorAdapter.php`, méthode `fromJson`.
- Expiration : 20/01/2027.
- Validation : responsable technique.
```

### Format de changelog

```markdown
# Changelog

## [1.0.0] - 2026-10-20

- Ajout du standard PHP, Laravel, TypeScript et React.
- Ajout des règles OWASP Top 10 2025.
```

## 13. Checklist finale

Avant de fusionner, l'auteur DOIT cocher chaque ligne applicable.

- [ ] Le code est lisible avant d'être astucieux.
- [ ] Chaque fonction a une responsabilité claire.
- [ ] Le code mort a été supprimé.
- [ ] Les variables et paramètres sont en `camelCase`.
- [ ] Les méthodes, fonctions et procédures sont en `snake_case`.
- [ ] Les classes, composants et fichiers de code sont en `PascalCase`.
- [ ] Les hooks React utilisent `useXxx`.
- [ ] Les constantes sont en `UPPER_SNAKE_CASE`.
- [ ] Pint, Prettier et ESLint passent sans erreur.
- [ ] Les lignes restent sous 120 caractères.
- [ ] Les contrôleurs délèguent la logique métier.
- [ ] Les fonctions restent autour de 20 lignes maximum.
- [ ] Les commentaires expliquent le pourquoi.
- [ ] Le README est à jour.
- [ ] Les exceptions sont dédiées et journalisées.
- [ ] Aucun `catch` n'est vide.
- [ ] Les requêtes évitent le N+1.
- [ ] Les optimisations sont basées sur une mesure.
- [ ] Les tests suivent AAA.
- [ ] Un test de régression existe pour chaque bug corrigé.
- [ ] La couverture métier atteint au moins 80 %.
- [ ] Le commit suit Conventional Commits.
- [ ] La PR contient au plus 400 lignes ou une justification.
- [ ] Aucun secret n'est présent dans le diff.
- [ ] Les dix contrôles OWASP applicables sont passés.
- [ ] `composer audit` et `npm audit` ont été exécutés.
- [ ] Le changelog est à jour si le standard ou le comportement public change.

## Références

- Robert C. Martin, _Clean Code_, Prentice Hall, 2008 — ISBN 978-0132350884
- David Thomas et Andrew Hunt, _The Pragmatic Programmer_ (20e anniversaire), Addison-Wesley, 2019 — ISBN 978-0135957059
- PSR-1 et PSR-12 : https://www.php-fig.org/psr/
- Conventions Laravel : https://laravel.com/docs
- Règles des hooks React : https://react.dev/reference/rules
- OWASP Top 10 2025 : https://owasp.org/Top10/ et https://top10.owasp.org/2025/
- Conventional Commits (format Angular) : https://www.conventionalcommits.org/
