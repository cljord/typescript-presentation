---
theme: default
title: "Thinking in Types: Ein mentales Modell für TypeScript"
info: enterJS 2026 — Claude Jordan
highlighter: shiki
transition: slide-left
---

# Thinking in Types

Ein mentales Modell für TypeScript

<div class="text-gray-400 mt-4">Claude Jordan</div>

---
layout: section
---

# Typen sind Mengen
# TypeScript ist eine funktionale Programmiersprache
# TypeScript arbeitet auf Mengen

<!--
 Heute

- Was macht TypeScript anders?
- Typen als Mengen
- Was sind eigentlich Generics?
- Programmieren auf Typebene
- Anwendung!
-->

---

# C prüft den Namen

```c
typedef struct {
  char* name;
} Dog;

typedef struct {
  char* name;
} Cat;

void greet(Dog d) { printf("Hello, %s\n", d.name); }

Dog d = { "Beethoven" };
greet(d); // ✅

Cat c = { "Jennifer" };
greet(c); // ❌
```

<!--
<style>
.two-cols-header {
  grid-template-rows: 100px 1fr;
}
</style>
-->

---

# TypeScript prüft die Form

```typescript
type Dog = { name: string }
type Cat = { name: string }

function greet(d: Dog) { console.log("Hello, " + d.name }

const d: Dog = { name: "Beethoven" }
greet(d) // ✅

const c: Cat = { name: "Jennifer" }
greet(c) // ✅
```

---

# Nominal vs. strukturell

In C: ein Typ wird durch seinen Namen beschrieben

In TypeScript: ein Typ wird durch seine Struktur beschrieben

<img src="/CvsTypeScript.svg" class="h-60 mx-auto mt-6" />

---

# Jeder Typ ist eine Menge von Werten

<!--
| Typ | Menge |
|---|---|
| `number` | alle Zahlen |
| `string` | alle Strings |
| `"hello"` | genau ein Wert |
| `{ name: string }` | alle Objekte mit `name: string` |
-->

<img src="/TSSetExamples.svg" class="h-100 mx-auto mt-6" />

<!-- image: number/string als große Kreise, "hello" als Punkt darin -->

---

# Jeder Typ ist eine Menge von Werten

<img src="/TSObjectIsSetWithAtLeastProperty.svg" class="h-100 mx-auto mt-6" />

---
layout: two-cols-header
---

# Intersection und Union

<!--
```typescript
type A = { x: number }
type B = { y: number }

type U = A | B   // Vereinigung: x ODER y
type I = A & B   // Schnitt: x UND y  →  { x: number, y: number }
```

`&` ist kein "Objekte zusammenfügen" — es ist die Menge, die **beide** Constraints erfüllt.
-->

<!-- image: zwei Venn-Diagramme, Union vs. Intersection -->
::left::
<img src="/IntersectionExample.svg" class="h-100 mx-auto mt-6" />
::right::
<img src="/UnionExample.svg" class="h-100 mx-auto mt-6" />

<style>
.two-cols-header {
  grid-template-rows: 25px 1fr;
}
.two-cols-header :deep(.col-right) {
  border-left: 3px solid #8884;
  padding-left: 1.5rem;
}
</style>
---
layout: two-cols-header
---

# never und unknown

<!--
```typescript
type Nothing = string & number   // never  — leere Menge ∅
```

- `never` — die leere Menge: kein Wert
- `unknown` — die universelle Menge: jeder Wert
-->

<!-- image: ∅ als leerer Kreis, unknown als alles umschließender Kreis -->
::left::
<img src="/Never.svg" class="h-100 mx-auto mt-6" />
::right::
<img src="/Unknown.svg" class="h-100 mx-auto mt-6" />

<style>
.two-cols-header {
  grid-template-rows: 50px 1fr;
}

.two-cols-header :deep(.col-right) {
  border-left: 3px solid #8884;
  padding-left: 1.5rem;
}
</style>

---

<!--

```typescript
type Shape  = { kind: string }
type Circle = { kind: 'circle'; radius: number }
//  Circle ⊆ Shape
```

Mehr Properties = **kleinere** Menge = spezifischerer Typ.
-->

<!-- image: Circle als Teilmenge innerhalb von Shape -->

<!--

section layout

-->

<!--

```typescript
// Werte → Werte
function identity(x: number): number { return x }

// Typen → Typen
type Box<T> = { value: T }
```

`Array<T>`, `Promise<T>` — eine Funktion von Mengen auf Mengen.
-->

# Funktionen sind Abbildungen auf Mengen

<img src="/FunctionExample.svg" class="h-100 mx-auto mt-6" />

---

# Generics

```typescript
type Array<T> = T[]
type MyNumberArray = Array<number> // number[]

function toArray(...numbers) {
  return numbers
}
```

<img src="/GenericsAsFunctionsExample.svg" class="h-60 mx-auto mt-6" />



---

# Branching: Conditional Types & infer

```typescript
type ElementType<T> = T extends (infer U)[] ? U : T

type A = ElementType<number>    // number
type B = ElementType<string[]>  // string
type C = ElementType<string[][]> // string[]
```

```typescript
function elementType(value) {
  return Array.isArray(value) ? value[0] : value;
}

elementType(42);          // 42
elementType(["a", "b"]);  // "a"
elementType([["nested", "array"], "outside"]) // ["nested", "array"]
```

- `T extends X` = "ist T eine **Teilmenge** von X?"
- `infer` = frag TypeScript, welcher Typ passt


<!--
Distribution

```typescript
type ElementTypeDistributive<T> = T extends (infer U)[] ? U : T
type ElementTypeNonDistributive<T> = [T] extends [(infer U)[]] ? U : T

type A = ElementTypeDistributive<number | string[]> // number | string
type B = ElementTypeNonDistributive<number | string[]> // number | string[]
```
-->

---

# Verschachtelung auflösen...?

```typescript
type NestedElementType<T> = T extends (infer U)[]
  ? U extends (infer V)[]
    ? V
    : U
  : T

type T = NestedElementType<string[][]>  // string
```

**Geht das eleganter...?**

---

# Rekursion

```typescript
type DeepElementType<T> =
  T extends (infer U)[]
    ? DeepElementType<U>
    : T;

type A = DeepElementType<number>    // number
type B = DeepElementType<string[]>  // string
type C = DeepElementType<string[][]> // string
```

```typescript
function deepElementType(value) {
  return Array.isArray(value) ? deepElementType(value[0]) : value;
}

DeepElementType(42);          // 42
DeepElementType(["a", "b"]);  // "a"
DeepElementType([["nested", "array"], "outside"]) // "nested"
```

---

# Mapped Types

```typescript
type OnlyStringsAndBoolsAllowed = { [key: string]: string | boolean };
// {a: "hello", myBoolean: true}
```

Mit Teilmengen + Generics:
```typescript
type FamousComposers = "Beethoven" | "Mozart" | "Bach";
type MappedComposers = { [K in Lowercase<FamousComposers>]: K };

// {beethoven: "beethoven", mozart: "mozart", bach: "bach"}
```

---
layout: section
---

# Ziel: kleinste Menge, die Typen vollständig beschreibt

---

# Discriminated Unions

```typescript
type ApiResponse =
  | { status: 'loading' }
  | { status: 'success'; data: User[] }
  | { status: 'error';   message: string }
```

Jede Branch ist eine disjunkte Teilmenge — unterschieden durch `status`.

---

# Narrowing ist Mengen-Verkleinerung

```typescript
function render(res: ApiResponse) {
  // res: loading | success | error   — die ganze Menge
  if (res.status === 'success') {
    res.data    // ✅ nur im success-Branch sichtbar
  }
}
```

Jeder Type Guard verkleinert die Menge: `===` · `typeof` · `instanceof` · `in` · Truthiness

---

# Exhaustiveness mit `never`

```typescript
function render(res: ApiResponse): string {
  switch (res.status) {
    case 'loading': return 'Lädt…'
    case 'success': return `${res.data.length} Einträge`
    case 'error':   return res.message
    default:
      const _exhaustive: never = res // ❌ nicht alle Fälle abgedeckt
  }
}
```

---

# Branded Types

```typescript
type Brand<T, B> = T & { readonly _brand: B }
type Email = Brand<string, 'Email'>

function isEmail(value: string): value is Email {
  return value.includes('@')
}

function sendWelcome(to: Email) {
  ...
}

const input = 'foo@bar.com'
sendWelcome(input)        // ❌ string ist keine Email
if (isEmail(input)) {
  sendWelcome(input)      // ✅ verengt auf die Email-Teilmenge
}
```

---

# Template Literal Types

```typescript
type EventName = `on${Capitalize<string>}`
// 'onClick' | 'onChange' | 'onSubmit' | ...
```

---

<!--
# as const + satisfies

```typescript
type Route = { path: string; auth: boolean }

const routes = {
  home:    { path: '/',   auth: false },
  profile: { path: '/me', auth: true  },
} as const satisfies Record<string, Route>

routes.home.path  // '/'   — exakter Literal-Typ bleibt erhalten
routes.home.auth  // false
```

- `as const` → kleinste Menge: Singletons statt `string` / `boolean`
- `satisfies` → Mitgliedschaft prüfen, **ohne** zu verbreitern
-->

# Excess Property Checking

```typescript
type Point = { x: number; y: number }

const obj = { x: 1, y: 2, z: 3 }
const p: Point = obj                  // ✅

const q: Point = { x: 1, y: 2, z: 3 } // ❌ excess property 'z'
```

Laut Mengenmodell sind beide gültig. TypeScript ist bei frischen Object Literals bewusst strenger.

---

# Routen-Parameter Parser

```typescript
type Params<T extends string> =
  T extends `${string}:${infer Param}/${infer Rest}`
    ? { [K in Param]: string } & Params<`/${Rest}`>
    : T extends `${string}:${infer Param}`
    ? { [K in Param]: string }
    : {}

type R = Params<"/users/:id/posts/:postId">
// { id: string; postId: string }
```

---

# Zusammenfassung

**Strukturell statt nominal** → Typen sind Mengen von Werten
- Intersection & Union · `never` (∅) & `unknown`

**TypeScript ist eine funktionale Sprache auf Mengen**
- Generics = Funktionen · Conditional Types & `infer` · Rekursion · Mapped Types

**In der Praxis**
- Discriminated Unions · Narrowing · Exhaustiveness mit `never`
- Branded Types · Template Literal Types · Excess Property Checking
- Routen-Parser — alles zusammen

**Das Ziel:** kein Regelwerk auswendig lernen — ein Modell haben, aus dem die Regeln folgen.

---
layout: center
class: text-center
---

# Danke!

Fragen?
