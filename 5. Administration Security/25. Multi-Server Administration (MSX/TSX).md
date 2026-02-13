# TSX)

Canonical documentation for [TSX)](5. Administration Security/25. Multi-Server Administration (MSX/TSX).md). This document defines concepts, terminology, and standard usage.

## Purpose
TSX (TypeScript XML) is a syntax extension for TypeScript that allows developers to write declarative, XML-like structures directly within TypeScript code. Its primary purpose is to provide a type-safe mechanism for defining hierarchical structures, most commonly used for describing User Interfaces (UI).

By integrating markup directly into the programming language, TSX enables the compiler to validate the relationship between the UI structure and the underlying logic, ensuring that components, attributes (props), and children conform to defined type contracts.

> [!NOTE]
> This documentation is intended to be implementation-agnostic and authoritative. While TSX is frequently associated with specific libraries like React, the specification itself is a general-purpose syntax extension supported by the TypeScript compiler.

## Scope
This document covers the syntactic rules, type-checking mechanisms, and transformation logic of TSX.

> [!IMPORTANT]
> **In scope:**
> * Syntactic grammar of TSX tags and expressions.
> * Type-checking rules for intrinsic and value-based elements.
> * The transformation process from TSX to standard TypeScript/JavaScript.
> * Configuration of the TSX factory and namespace.

> [!WARNING]
> **Out of scope:**
> * Specific vendor implementations (e.g., React, Preact, SolidJS, Vue).
> * Virtual DOM reconciliation algorithms.
> * CSS-in-JS or styling methodologies.

## Definitions
Provide precise definitions for key terms.

| Term | Definition |
|------|------------|
| **JSX** | JavaScript XML; the original specification upon which TSX is based. |
| **Intrinsic Element** | An element that is expected to exist in the target environment (e.g., `<div>` in a browser), typically denoted by lowercase. |
| **Value-based Element** | An element defined by a TypeScript variable or component, typically denoted by an uppercase starting letter. |
| **Pragma** | A compiler directive (e.g., `@jsx`) that specifies which function should be used to transform TSX tags. |
| **Props** | The set of attributes passed to an element, represented as a plain object in the transformed output. |
| **Fragment** | A special syntax (`<>...</>`) used to group multiple elements without adding an extra node to the hierarchy. |

## Core Concepts

### The Transformation Pipeline
TSX is not executed directly by runtimes. It undergoes a transformation process where the XML-like syntax is converted into standard function calls. For example, `<div id="main">Content</div>` is transformed into a factory call such as `h('div', { id: 'main' }, 'Content')`.

### Type Checking
Unlike standard JSX, TSX enforces strict type safety. The TypeScript compiler looks for a specific namespace (usually `JSX`) to determine the validity of an element.
* **Intrinsic Elements** are looked up in `JSX.IntrinsicElements`.
* **Value-based Elements** are checked against the signature of the function or class being referenced.

> [!TIP]
> Think of TSX as a "type-safe template string" that understands the structure of the data it represents, rather than just treating it as a sequence of characters.

### The JSX Namespace
The behavior of TSX is governed by the `JSX` namespace. This namespace defines the available elements, the types of allowed attributes, and the valid types for children. This allows different frameworks to define their own rules for what constitutes a valid "element."

## Standard Model
The standard model for TSX implementation follows the "Factory Pattern."

1.  **Parsing:** The compiler identifies tags within a `.tsx` file.
2.  **Resolution:** The compiler determines if the tag is an intrinsic string or a reference to a value.
3.  **Validation:** The compiler checks the provided attributes against the expected types (Props).
4.  **Emission:** The compiler outputs a JavaScript function call, passing the tag, the props object, and the children as arguments.

> [!IMPORTANT]
> The output of a TSX expression is determined by the `jsxFactory` configuration in `tsconfig.json` or a per-file pragma.

## Common Patterns

### Conditional Rendering
Using TypeScript's logical operators or ternary expressions within curly braces to determine if an element should be rendered.
```tsx
{isLoggedIn && <LogoutButton />}
```

### List Mapping
Transforming arrays of data into arrays of TSX elements using standard array methods like `.map()`.

### Composition via Children
Passing TSX elements as properties to other elements, allowing for nested, reusable structures.

## Anti-Patterns

### Logic-Heavy Templates
Embedding complex business logic or heavy computations directly inside TSX tags. This obscures the declarative nature of the UI and makes debugging difficult.

### Manual DOM Manipulation
Attempting to modify the underlying DOM nodes directly while using a TSX-based declarative framework, which can lead to state desynchronization.

> [!CAUTION]
> Avoid circular dependencies where Component A renders Component B, and Component B renders Component A, as this can lead to infinite recursion during the rendering phase or type-checking errors.

## Edge Cases

### Generic Components
TSX supports generic components, but the syntax can be ambiguous with standard TSX tags.
* *Ambiguity:* `<T>(val: T) => ...` can be mistaken for a TSX tag.
* *Resolution:* Using a trailing comma `<T,>` or extending an object `<T extends {}>` clarifies the intent to the compiler.

### Namespaced Tags
Some environments (like SVG or XML) use namespaced tags (e.g., `<svg:path />`). TSX handles these as string-based intrinsic elements, but the factory function must be capable of parsing the colon.

### Fragment Limitations
While fragments allow grouping, they cannot accept attributes (except for `key` in some specific implementations). Using a fragment where a semantic wrapper is required (like a `section` or `nav`) is considered a misuse of the feature.

## Related Topics
* **TypeScript Type System:** The foundation upon which TSX type-checking is built.
* **ECMAScript (ES6+):** The underlying language standard for the logic within TSX.
* **Virtual DOM:** The architectural pattern often used to efficiently render the output of TSX.

## Change Log

| Version | Date | Description |
|---------|------|-------------|
| 1.0 | 2026-02-13 | Initial AI-generated canonical documentation |