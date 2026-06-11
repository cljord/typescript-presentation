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

# Was erwartet euch heute?

- Strukturelles vs. nominales Typsystem
- Typen als Mengen denken
- Der Typ-Verband: `unknown`, `never`, und alles dazwischen
- Generics als Funktionen auf Typebene
- Das Modell in der Praxis: Discriminated Unions, Branded Types, Template Literal Types
- Wo das Modell an Grenzen stößt

---
layout: section
---

# Strukturelles vs. nominales Typsystem

---

# Nominales Typsystem: C

In C identifiziert der Compiler Typen anhand ihres **Namens** — nicht ihrer Form.

```c
typedef struct { char* name; } Dog;
typedef struct { char* name; } Cat;

void greet(Dog d) {
    printf("Hello, %s\n", d.name);
}

int main() {
    Cat c = { "Whiskers" };
    greet(c); // ❌ error: incompatible type for argument
}
```

`Dog` und `Cat` haben exakt dieselbe Struktur — aber unterschiedliche Namen.  
Für C ist das ein Typfehler.

---

# Strukturelles Typsystem: TypeScript

TypeScript fragt nur: **Hat dieser Wert die richtige Form?**

```typescript
type Dog = { name: string }
type Cat = { name: string }

function greet(d: Dog) {
    console.log(`Hello, ${d.name}`)
}

const c: Cat = { name: "Whiskers" }
greet(c) // ✅ fine
```

Kein gemeinsamer Ancestor, keine explizite Deklaration — gleiche Form reicht.

**Die entscheidende Frage:** Warum funktioniert das so — und was hat das mit Mengen zu tun?

---
layout: section
---

# Typen als Mengen

---

# Jeder Typ ist eine Menge von Werten

| Typ | Menge |
|---|---|
| `number` | alle Zahlen: `0, 1, -1, 3.14, ...` |
| `string` | alle Strings: `"", "hello", "abc", ...` |
| `"hello"` | genau ein Wert: `{ "hello" }` |
| `true` | genau ein Wert: `{ true }` |
| `{ name: string }` | alle Objekte mit mindestens `name: string` |

Ein **Literal-Typ** ist eine einelementige Menge.  
Ein **primitiver Typ** ist eine unendliche Menge.

---

# Union und Intersection

```typescript
type A = { x: number }
type B = { y: number }
```

**Union `A | B`** — Mengenvereiningung: Werte in A, oder B, oder beiden

```typescript
type AorB = A | B
// alle Objekte mit x, oder alle mit y
```

**Intersection `A & B`** — Mengenschnitt: Werte, die **beide** Constraints erfüllen

```typescript
type AandB = A & B
// { x: number, y: number }
// alle Objekte mit x UND y
```

Nicht als "zwei Objekte zusammenfügen" denken — als "die Menge, die beide Bedingungen erfüllt."

---

# Subtypen sind Teilmengen

```typescript
type Shape   = { kind: string }
type Circle  = { kind: 'circle'; radius: number }
```

`Circle` ist ein Subtyp von `Shape` — weil jeder `Circle`-Wert auch ein gültiger `Shape`-Wert ist.

In Mengensprache: `Circle ⊆ Shape`

Das erklärt auch, warum strukturelles Typing funktioniert:  
`Cat` mit `{ name: string }` ist eine Teilmenge der Werte, die `Dog` mit `{ name: string }` beschreibt —  
dieselbe Menge, sogar.

---
layout: section
---

# Der Typ-Verband

---

# unknown, never, und alles dazwischen

TypeScript hat zwei besondere Typen:

**`unknown`** — die universelle Menge: *alle* möglichen Werte  
Jeder Typ ist ein Subtyp von `unknown`. Es ist der allgemeinste Typ.

**`never`** — die leere Menge: *keine* Werte  
`never` ist ein Subtyp von jedem Typ. Kein Wert hat diesen Typ.

```typescript
type A = string & number  // never — kein Wert ist gleichzeitig string und number

function fail(msg: string): never {
    throw new Error(msg)  // diese Funktion gibt nie einen Wert zurück
}
```

---

# Der Verband (Lattice)

```
          unknown        ← größte Menge (alle Werte)
          /     \
       string  number
       /    \
   "hello" "world"       ← einelementige Mengen
          \     /
          never          ← leere Menge (keine Werte)
```

**Die kontraintuitive Regel:** mehr Properties = **kleinere** Menge

```typescript
type A = { x: number }                       // viele Objekte passen
type B = { x: number; y: number }            // weniger Objekte passen
type C = { x: number; y: number; z: string } // noch weniger
```

`C ⊆ B ⊆ A` — mehr Constraints, kleinere Menge, spezifischerer Typ.

`any` ist der Ausreißer: es ist gleichzeitig oben und unten. Es verlässt das System.

---
layout: section
---

# Generics als Funktionen auf Typebene

---

# Generics sind Typ-Funktionen

Eine Funktion nimmt Werte und gibt Werte zurück.  
Ein Generic nimmt **Typen** und gibt **Typen** zurück.

```typescript
// Wert-Ebene
function identity(x: number): number { return x }

// Typ-Ebene
type Identity<T> = T
```

`Array<T>` ist eine Funktion: nimmt `T`, gibt "Menge aller Arrays von T" zurück.

```typescript
Array<string>  // Menge aller String-Arrays
Array<number>  // Menge aller Number-Arrays
```

---

# Utility Types als Mengentransformationen

```typescript
type Partial<T>  // T → größere Menge (optionale Properties = mehr Objekte passen)
type Required<T> // T → kleinere Menge
type Readonly<T> // T → gleich große Menge, andere Constraints
```

`Exclude` ist **direkt** Mengendifferenz:

```typescript
type A = string | number | boolean
type B = Exclude<A, boolean>  // string | number

// A \ boolean
```

Conditional Types sind if-then-else auf Typebene:

```typescript
type IsString<T> = T extends string ? true : false
//                 ^^^^^^^^^^^^^^^^^^
//                 "ist T eine Teilmenge von string?"
```

---
layout: section
---

# Das Modell in der Praxis

---

# Discriminated Unions

```typescript
type Shape =
  | { kind: 'circle';   radius: number }
  | { kind: 'square';   side: number   }
  | { kind: 'triangle'; base: number; height: number }
```

`kind` ist ein Literal-Typ — eine einelementige Menge.  
Jede Branch ist eine **disjunkte Teilmenge** der Union.

Narrowing ist Mengenschnitt:

```typescript
if (shape.kind === 'circle') {
    // TypeScript schneidet Shape mit { kind: 'circle' }
    // nur die Circle-Branch bleibt übrig
    shape.radius  // ✅
}
```

TypeScript kann Exhaustiveness-Checking machen, weil es weiß:  
die drei Branches partitionieren die gesamte Union vollständig.

---

# Branded Types

Problem: strukturelles Typing bedeutet, jede `number` ist eine gültige `number`.

```typescript
function transferMoney(amount: number, from: Account, to: Account) { ... }

const euros = 42
const dollars = 42
transferMoney(euros, ...)   // ✅ — aber ist das richtig?
transferMoney(dollars, ...) // ✅ — TypeScript sieht keinen Unterschied
```

Lösung: durch Intersection eine spezifischere Teilmenge schaffen.

```typescript
type Brand<T, B> = T & { readonly _brand: B }

type EUR = Brand<number, 'EUR'>
type USD = Brand<number, 'USD'>

function transferEUR(amount: EUR) { ... }

transferEUR(42 as EUR)        // ✅
transferEUR(42 as USD)        // ❌ Type 'USD' is not assignable to type 'EUR'
transferEUR(42)               // ❌
```

Nominales Typing durch Mengenschnitt simuliert — der Kreis schließt sich.

---

# Template Literal Types

```typescript
type EventName = `on${Capitalize<string>}`
// die Menge aller Strings der Form "on..." mit Großbuchstaben danach
// 'onClick', 'onChange', 'onSubmit', ...
```

Nicht eine Aufzählung von Werten — eine Beschreibung einer **unendlichen Menge** durch ein Muster.

Kombiniert mit Mapped Types:

```typescript
type EventMap<T extends string> = {
    [K in `on${Capitalize<T>}`]: () => void
}

type MouseEvents = EventMap<'click' | 'move' | 'down'>
// { onClick: () => void; onMove: () => void; onDown: () => void }
```

---
layout: section
---

# Wo das Modell an Grenzen stößt

---

# Excess Property Checking

Das Mengenmodell sagt: `{ x, y, z }` ist eine gültige `Point`-Instanz — denn jedes Objekt mit x, y, z erfüllt auch die Point-Constraints.

```typescript
type Point = { x: number; y: number }

const obj = { x: 1, y: 2, z: 3 }
const p: Point = obj  // ✅ — korrekt laut Mengenmodell
```

Aber bei direkter Zuweisung eines Object Literals:

```typescript
const p: Point = { x: 1, y: 2, z: 3 }  // ❌ Object literal may only specify known properties
```

TypeScript fügt hier eine **pragmatische Ausnahme** hinzu: bei frischen Object Literals wird angenommen, dass extra Properties ein Tippfehler sind.

Das Mengenmodell gilt — TypeScript entscheidet sich nur, in diesem Fall strenger zu sein.

---

# Zusammenfassung

- **Strukturelles Typing** beschreibt Formen, nicht Namen — darum mappen Typen natürlich auf Mengen
- Jeder Typ ist eine Menge von Werten; `|` ist Vereinigung, `&` ist Schnitt
- `unknown` = universelle Menge, `never` = leere Menge; mehr Properties = kleinere Menge
- Generics sind Funktionen auf Typebene — Mengentransformationen
- Discriminated Unions, Branded Types, Template Literals werden alle klarer durch die Mengenbrille
- Das Modell hat Grenzen — Excess Property Checking ist eine bewusste pragmatische Ausnahme

**Das Ziel:** TypeScript nicht als Regelwerk auswendig lernen —  
sondern ein Modell haben, aus dem man das Regelwerk ableiten kann.

---
layout: center
class: text-center
---

# Danke!

Fragen?

<div class="text-gray-400 mt-8">
Claude Jordan · claude.jordan@scopevisio.com<br>
github.com/... · LinkedIn: ...
</div>
