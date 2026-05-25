# TypeScript Tips Everyone Should Know

A curated collection of practical TypeScript patterns that improve safety, readability, maintainability, and developer experience.

Most of these are small individually. Together, they dramatically change how TypeScript code feels to work in.

## Table of Contents

1. [Prefer `unknown` Over `any`](#prefer-unknown-over-any)
2. [Let Type Inference Do the Work](#let-type-inference-do-the-work)
3. [Prefer `satisfies` Over `as`](#prefer-satisfies-over-as)
4. [Derive Types From Values](#derive-types-from-values-instead-of-duplicating-them)
5. [Use Discriminated Unions](#model-impossible-states-with-discriminated-unions)
6. [Use Exhaustive Checks With `never`](#use-exhaustive-checks-with-never)
7. [Use `as const` for Constants](#use-as-const-for-configuration-and-constants)
8. [Use Type Predicates](#use-type-predicates-for-reusable-narrowing)
9. [Build Types From Existing Types](#build-new-types-from-existing-types)
10. [Validate External Data at Runtime](#validate-external-data-at-runtime)
11. [Avoid `enum` in Most Cases](#avoid-enum-in-most-cases)
12. [Prefer Inferable Generics](#prefer-generics-that-infer-automatically)
13. [Enable Strict Compiler Options](#turn-on-the-strict-compiler-options)
14. [Learn Template Literal Types](#learn-template-literal-types)
15. [Type Safety ≠ Runtime Safety](#type-safe-does-not-mean-runtime-safe)

### Prefer `unknown` Over `any`

A lot of type safety starts here.

Most TypeScript problems start when `any` enters the system.

```ts
function parse(data: unknown) {
  if (typeof data === "string") {
    return data.toUpperCase();
  }
}
```

#### Why it matters

- Forces validation
- Preserves safety
- Prevents type leakage

<sup>[Table of Contents](#table-of-contents)</sup>

### Let Type Inference Do the Work

The best TypeScript code often has fewer explicit types than beginners expect.

```ts
const name = "Ada";
```

Instead of:

```ts
const name: string = "Ada";
```

#### Over-annotation

- Widens types
- Hurts inference
- Creates maintenance overhead

Inference tends to scale better than annotation.

<sup>[Table of Contents](#table-of-contents)</sup>

### Prefer `satisfies` Over `as`

One of the most important modern TypeScript features.

```ts
const routes = {
  home: "/",
  about: "/about",
} satisfies Record<string, string>;
```

Instead of:

```ts
const routes = {
  home: "/",
  about: "/about",
} as Record<string, string>;
```

`satisfies` validates without losing inference.

<sup>[Table of Contents](#table-of-contents)</sup>

### Derive Types From Values Instead of Duplicating Them

One of the biggest TypeScript mindset shifts.

```ts
const roles = ["admin", "user", "guest"] as const;

type Role = (typeof roles)[number];
```

Keeping runtime values and types in sync manually almost always drifts over time.

<sup>[Table of Contents](#table-of-contents)</sup>

### Model Impossible States With Discriminated Unions

This is where TypeScript starts becoming architectural.

```ts
type State =
  | { status: "loading" }
  | { status: "success"; data: User }
  | { status: "error"; error: Error };
```

These models tend to scale much better than loose optional-property blobs.

<sup>[Table of Contents](#table-of-contents)</sup>

### Use Exhaustive Checks With `never`

Discriminated unions become much more powerful with exhaustiveness checking.

```ts
default: {
  const exhaustive: never = state;
  return exhaustive;
}
```

Future refactors become compiler errors instead of runtime bugs.

<sup>[Table of Contents](#table-of-contents)</sup>

### Use `as const` for Configuration and Constants

Without `as const`:

```ts
const theme = {
  mode: "dark",
};
```

`mode` becomes `string`.

With `as const`:

```ts
const theme = {
  mode: "dark",
} as const;
```

Now it becomes `'dark'`.

Tiny feature, huge usefulness.

<sup>[Table of Contents](#table-of-contents)</sup>

### Use Type Predicates for Reusable Narrowing

Connect runtime checks to compile-time intelligence.

```ts
function isUser(value: unknown): value is User {
  return typeof value === "object" && value !== null && "id" in value;
}
```

Then:

```ts
if (isUser(data)) {
  data.id;
}
```

This becomes especially useful around APIs and external input boundaries.

<sup>[Table of Contents](#table-of-contents)</sup>

### Build New Types From Existing Types

Think in transformations instead of duplication.

```ts
type UserPreview = Pick<User, "id" | "name">;
```

### Learn these utility types

- `Pick`
- `Omit`
- `Partial`
- `Required`
- Indexed access types

These utilities become much more valuable as applications grow.

<sup>[Table of Contents](#table-of-contents)</sup>

### Validate External Data at Runtime

TypeScript does **not** validate API responses.

This is one of the most misunderstood parts of TypeScript.

```ts
const UserSchema = z.object({
  id: z.string(),
  name: z.string(),
});
```

Type safety ends at runtime boundaries unless you validate.

<sup>[Table of Contents](#table-of-contents)</sup>

### Avoid `enum` in Most Cases

Usually simpler:

```ts
const roles = ["admin", "user"] as const;
```

Than:

```ts
enum Role {
  Admin,
  User,
}
```

In most applications, literal unions end up simpler to refactor, easier to serialize, and less surprising at runtime than enums.

<sup>[Table of Contents](#table-of-contents)</sup>

### Prefer Generics That Infer Automatically

Great TypeScript APIs rarely require manual generic arguments.

Less ideal:

```ts
getData<User>();
```

Better:

```ts
getData(userSchema);
```

Inference usually scales better than annotation-heavy APIs.

<sup>[Table of Contents](#table-of-contents)</sup>

### Turn On the Strict Compiler Options

Many teams use TypeScript in "autocomplete mode."

Strict mode is where TypeScript really starts paying off.

```json
{
  "strict": true,
  "noUncheckedIndexedAccess": true,
  "exactOptionalPropertyTypes": true
}
```

These flags dramatically improve correctness.

<sup>[Table of Contents](#table-of-contents)</sup>

### Learn Template Literal Types

One of the most powerful modern TypeScript features.

```ts
type Route = `/api/${string}`;
```

Excellent for:

- Routes
- Event names
- CSS utilities
- Design systems
- Query keys

Once you start using them, they show up everywhere.

<sup>[Table of Contents](#table-of-contents)</sup>

### "Type-Safe" Does Not Mean "Runtime Safe"

A perfect final tip because it reframes everything.

This compiles:

```ts
const user = (await response.json()) as User;
```

But may still explode at runtime.

TypeScript improves correctness:

- It does not replace validation
- It does not guarantee good architecture
- It does not eliminate runtime bugs

This distinction becomes increasingly important in larger systems.

<sup>[Table of Contents](#table-of-contents)</sup>
