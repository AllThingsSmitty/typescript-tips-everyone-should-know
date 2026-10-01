# TypeScript Tips Everyone Should Know

Practical TypeScript patterns for writing safer, more maintainable code.

Most of these are small individually. Together, they change how TypeScript code feels day to day.

## Table of Contents

1. [Prefer `unknown` Over `any`](#prefer-unknown-over-any)
2. [Let Type Inference Do the Work](#let-type-inference-do-the-work)
3. [Prefer `satisfies` Over `as`](#prefer-satisfies-over-as)
4. [Derive Types From Values](#derive-types-from-values-instead-of-duplicating-them)
5. [Make Invalid States Impossible to Represent](#make-invalid-states-impossible-to-represent)
6. [Use Exhaustive Checks With `never`](#use-exhaustive-checks-with-never)
7. [Use `as const` for Constants](#use-as-const-for-configuration-and-constants)
8. [Use Type Predicates](#use-type-predicates-for-reusable-narrowing)
9. [Build New Types From Existing Types](#build-new-types-from-existing-types)
10. [Validate External Data at Runtime](#validate-external-data-at-runtime)
11. [Avoid `enum` in Most Cases](#avoid-enum-in-most-cases)
12. [Prefer Inferable Generics](#prefer-generics-that-infer-automatically)
13. [Enable Strict Compiler Options](#turn-on-the-strict-compiler-options)
14. [Learn Template Literal Types](#learn-template-literal-types)
15. [Type Safety ≠ Runtime Safety](#type-safe-does-not-mean-runtime-safe)
16. [Use Branded Types to Model Nominal Types](#use-branded-types-to-model-nominal-types)
17. [Use `const` Type Parameters for Better Literal Inference](#use-const-type-parameters-for-better-literal-inference)
18. [Use `NoInfer<T>` to Control Where TypeScript Infers From](#use-noinferT-to-control-where-typescript-infers-from)

### Prefer `unknown` Over `any`

A lot of type safety starts here.

`unknown` forces you to prove what a value is before using it. `any` skips the type system entirely, allowing unsafe operations to spread through your code.

```ts
function parse(data: unknown) {
  if (typeof data === "string") {
    return data.toUpperCase();
  }
}
```

#### Why it matters

- Forces validation before use
- Preserves type safety
- Prevents unsafe type leakage

<sup>[Table of Contents](#table-of-contents)</sup>

### Let Type Inference Do the Work

The best TypeScript code relies on inference instead of restating what the compiler already knows.

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

Inference scales better.

<sup>[Table of Contents](#table-of-contents)</sup>

### Prefer `satisfies` Over `as`

Added in TS 4.9, and one worth adopting immediately.

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

`satisfies` checks that a value matches a type while preserving its inferred type.

Use `satisfies` when validating object shapes. Reserve `as` for when the compiler can't figure it out on its own.

<sup>[Table of Contents](#table-of-contents)</sup>

### Derive Types From Values Instead of Duplicating Them

One of the biggest TypeScript mindset shifts.

```ts
const roles = ["admin", "user", "guest"] as const;

type Role = (typeof roles)[number];
```

Single source of truth. Change the runtime values and the type updates with them.

<sup>[Table of Contents](#table-of-contents)</sup>

### Make Invalid States Impossible to Represent

Good TypeScript models don't just describe data. They prevent impossible combinations.

Discriminated unions are the cleanest way to enforce these constraints.

```ts
type State =
  | { status: "loading" }
  | { status: "success"; data: User }
  | { status: "error"; error: Error };
```

These models scale much better than loose optional property blobs because invalid states can't be represented.

Future refactors become safer because the compiler ensures every valid state is handled.

<sup>[Table of Contents](#table-of-contents)</sup>

### Use Exhaustive Checks With `never`

Once you've modeled your states as a discriminated union, exhaustiveness checking ensures every case is handled.

```ts
function render(state: State) {
  switch (state.status) {
    case "loading":
      return "Loading...";
    case "success":
      return state.data;
    case "error":
      throw state.error;
    default: {
      const exhaustive: never = state;
      return exhaustive;
    }
  }
}
```

Add a new state to the union, and the compiler immediately points out every place that needs updating.

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

A small change that makes a real difference for config objects and constants.

<sup>[Table of Contents](#table-of-contents)</sup>

### Use Type Predicates for Reusable Narrowing

Type predicates let a runtime check teach the compiler something.

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

Most useful at API and input boundaries.

<sup>[Table of Contents](#table-of-contents)</sup>

### Build New Types From Existing Types

Think in transformations instead of duplication.

```ts
type UserPreview = Pick<User, "id" | "name">;
```

#### Learn these utility types

- `Pick`
- `Omit`
- `Partial`
- `Required`
- Indexed access types

They pay off more as the codebase grows.

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

Every API response, form submission, environment variable, JSON file, and user input is an untrusted boundary.

TypeScript can't validate external data; you need runtime validation for that.

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

In most application code, literal unions are easier to refactor, serialize, and work with than enums.

Enums have their place, but they're usually overkill.

<sup>[Table of Contents](#table-of-contents)</sup>

### Prefer Generics That Infer Automatically

Great TypeScript APIs rarely require manual generic arguments. Design them so the type infers from what callers pass in.

Caller has to specify the type manually:

```ts
getData<User>("/api/user");
```

T infers from the schema, nothing to annotate:

```ts
getData("/api/user", userSchema);
```

If callers are constantly writing `<SomeType>`, that's usually a sign the API could do more of the work.

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

`strict` is the baseline. The other two aren't covered by it, and they catch a real class of bugs that strict alone misses.

<sup>[Table of Contents](#table-of-contents)</sup>

### Learn Template Literal Types

Underused, and worth learning.

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

This compiles:

```ts
const user = (await response.json()) as User;
```

But it may still fail at runtime.

TypeScript improves correctness, but it isn't a runtime safety net.

- It does not validate external data
- It does not guarantee good architecture
- It does not eliminate runtime bugs

Use TypeScript to model your program well. Then validate anything that comes from the outside world.

<sup>[Table of Contents](#table-of-contents)</sup>

### Use Branded Types to Model Nominal Types

TypeScript is structural, so two types with the same shape are interchangeable even when they mean different things.

```ts
type UserId = string;
type OrderId = string;

function getUser(id: UserId) { /* ... */ }

const orderId: OrderId = "order_123";
getUser(orderId); // No error, but almost certainly a bug
```

A brand adds a compile-time-only tag that makes them distinct:

```ts
type UserId = string & { readonly __brand: "UserId" };
type OrderId = string & { readonly __brand: "OrderId" };

function getUser(id: UserId) { /* ... */ }

declare const orderId: OrderId;
getUser(orderId); // Error: OrderId is not assignable to UserId
```

Zero runtime cost, and look-alike primitives can't be swapped by mistake.

<sup>[Table of Contents](#table-of-contents)</sup>

### Use `const` Type Parameters for Better Literal Inference

Without `const`, generic parameters widen to their base type:

```ts
function first<T extends readonly unknown[]>(arr: T) {
  return arr[0];
}

const result = first(["a", "b", "c"]); // string
```

With the `const` modifier, the literal comes through:

```ts
function first<const T extends readonly unknown[]>(arr: T) {
  return arr[0];
}

const result = first(["a", "b", "c"]); // "a"
```

Good for any API where the exact shape matters, without making callers remember `as const`.

<sup>[Table of Contents](#table-of-contents)</sup>

### Use `NoInfer<T>` to Control Where TypeScript Infers From

TypeScript infers `T` from every argument, so a loose argument site can silently widen the type.

```ts
function pick<T>(values: T[], fallback: T): T {
  return values[0] ?? fallback;
}

pick(["a", "b"], 42); // T infers as string | number, no error, but almost certainly a bug
```

`NoInfer<T>` tells the compiler to skip that argument when solving for `T`:

```ts
function pick<T>(values: T[], fallback: NoInfer<T>): T {
  return values[0] ?? fallback;
}

pick(["a", "b"], 42); // Error: Argument of type 'number' is not assignable to 'string'
```

Most common with default values, fallbacks, and validation callbacks.

<sup>[Table of Contents](#table-of-contents)</sup>
