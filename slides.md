---
theme: default
title: "Thinking in Types: Ein mentales Modell für TypeScript"
info: enterJS 2026 — Claude Jordan
highlighter: shiki
transition: slide-left
---

# Thinking in Types

Ein mentales Modell für TypeScript

<div class="text-gray-400 mt-4">Claude Jordan · enterJS 2026 · Mannheim</div>

---

# Heute

- Strukturelles vs. nominales Typsystem
- Typen als Mengen
- Generics als Funktionen auf Typebene
- In der Praxis: Discriminated Unions, Branded Types, Template Literals
- Wo das Modell an Grenzen stößt

---
layout: section
---

# Strukturelles vs. nominales Typsystem

---
layout: two-cols-header
---

# C prüft den Namen

::left::

```c
typedef struct {
  char* name;
} Dog;

typedef struct {
  char* name;
} Cat;
```

::right::

```c
void greet(Dog d) { printf("Hello, %s\n", d.name); }

Dog d = { "Beethoven" };
greet(c); // ✅

Cat c = { "Jennifer" };
greet(c); // ❌
```

<style>
.two-cols-header {
  grid-template-rows: 100px 1fr;
}
</style>

---

# TypeScript prüft die Form

```typescript
type Dog = { name: string }
type Cat = { name: string }

function greet(d: Dog) { console.log("Hello, " + d.name }

const d: Dog = { name: "Beethoven" }
greet(c) // ✅

const c: Cat = { name: "Jennifer" }
greet(c) // ✅
```

Gleiche Form reicht. **Was ist der Unterschied im Typsystem?**

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
# Subtypen sind Teilmengen

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
# Generics als Funktionen auf Typebene

-->

<!--
# Generics sind Typ-Funktionen

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
layout: section
---

# TypeScript ist eine funktionale Programmiersprache, die auf Mengen arbeitet

---

# Was brauchen Funktionen?

- Generics
- Branching
- Looping
- (Unbounded memory access) (anpassen)

---

# Generics Types

```typescript
type MyArray<T> = T[]
type MyNumberArray = MyArray<number>
```

<img src="/GenericsAsFunctionsExample.svg" class="h-80 mx-auto mt-6" />

---

# Conditional Types & infer

```typescript
type ElementType<T> = T extends (infer U)[] ? U : T

type A = ElementType<string[]>  // string
type B = ElementType<number>    // number
```

- `extends` = "ist T eine **Teilmenge**?"
- `infer` = "welcher Typ würde hier passen?"

---

# Mapped Types

```typescript
type Optional<T> = {
  [K in keyof T]?: T[K]
}
```

Iteriere über die Members einer Menge, transformiere jeden.

---

# Rekursion

```typescript
type Reverse<T extends any[]> =
  T extends [infer Head, ...infer Tail]
    ? [...Reverse<Tail>, Head]
    : []

type R = Reverse<[1, 2, 3]>  // [3, 2, 1]
```

Das Typsystem ist **Turing-vollständig** — ganze Parser auf Typebene (SQL, GraphQL).

---
layout: section
---

# In der Praxis

---

# Discriminated Unions

```typescript
type Shape =
  | { kind: 'circle'; radius: number }
  | { kind: 'square'; side: number }
```

Jede Branch ist eine disjunkte Teilmenge. Narrowing ist Mengenschnitt:

```typescript
if (shape.kind === 'circle') {
  shape.radius // ✅  Shape ∩ { kind: 'circle' }
}
```

---

# Branded Types

```typescript
type Brand<T, B> = T & { readonly _brand: B }

type EUR = Brand<number, 'EUR'>
type USD = Brand<number, 'USD'>

declare function transfer(amount: EUR): void

transfer(42 as EUR)  // ✅
transfer(42 as USD)  // ❌
transfer(42)         // ❌
```

Nominales Typing per Mengenschnitt — der Kreis schließt sich.

---

# Template Literal Types

```typescript
type EventName = `on${Capitalize<string>}`
// 'onClick' | 'onChange' | 'onSubmit' | ...
```

Eine unendliche Menge, beschrieben durch ein **Muster** statt durch Aufzählung.

---
layout: section
---

# Wo das Modell an Grenzen stößt

---

# Excess Property Checking

```typescript
type Point = { x: number; y: number }

const obj = { x: 1, y: 2, z: 3 }
const p: Point = obj                  // ✅

const q: Point = { x: 1, y: 2, z: 3 } // ❌ excess property 'z'
```

Laut Mengenmodell sind beide gültig. TypeScript ist bei frischen Object Literals bewusst strenger.

---

# Zusammenfassung

- Strukturelles Typing beschreibt Formen → Typen sind Mengen
- `|` Vereinigung, `&` Schnitt, `never` ∅, `unknown` universell
- Generics sind Funktionen auf Mengen — bis hin zu Rekursion
- Discriminated Unions, Branded Types, Template Literals: alles Mengen

**Das Ziel:** kein Regelwerk auswendig lernen — ein Modell haben, aus dem die Regeln folgen.

---
layout: center
class: text-center
---

# Danke!

Fragen?

<div class="text-gray-400 mt-8">
Claude Jordan · claude.jordan@scopevisio.com
</div>
