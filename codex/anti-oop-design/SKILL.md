---
name: anti-oop-design
description: >-
  Use when starting or refactoring business logic with an anti-OOP,
  function-first architecture: discover structure by aggregating related
  business variables and rules, split pure computation from side-effect
  functions, derive business facts from source data, and name abstractions only
  after they emerge.
---

# Anti-OOP Design

Use this skill to design or refactor business logic without starting from entities, class hierarchies, DTOs, or preset layers. The goal is not to invent an architecture upfront. The goal is to expose the architecture by aggregating related business facts, separating computation from side effects, and naming only what remains.

## Core Model

A program has only two kinds of functions:

- **Pure functions** compute: they transform explicit input data into output data. They do not read time, random values, files, databases, network state, global state, or mutable objects.
- **Side-effect functions** store or touch the outside world: they read, write, persist, fetch, send, log, schedule, mutate, or otherwise depend on external state. They may return values, but they have effects.

Orchestration is not a third kind. It is the calling pattern between these two kinds: use side-effect functions to prepare the business facts requested by pure functions, call those pure functions, then execute the resulting side effects.

## Design Principles

1. **Aggregate first, separate second**

   Start by gathering all variables, checks, conditions, and operations related to one business rule into one visible place. This is discovery, not design. Do not classify, name, or abstract before the related facts are visible together.

2. **Pure computation is the center**

   Business rules belong in pure functions. Services, controllers, repositories, classes, and transports exist only to supply inputs to those functions or execute their outputs.

3. **Ask for business facts, not source data**

   When shaping a pure business function, look for the fact the rule needs. The core should receive that fact directly: whether something exists, how many, which state, what value, which single selected item, which timestamp, or another small decision input. If a rule needs to know whether a sibling node with the same `name` and `type` exists, shape the input around `sameNameAndTypeExists: boolean`.

4. **Identify growth vectors early**

   During design, identify which data can grow over time: users, books, orders, loans, events, messages, logs, rows, pages, streams. Business computation should be independent of their size.

5. **Derive facts before computation**

   Source data belongs to storage, network, runtime, and adapters. Before calling pure computation, derive the business fact the rule asked for: an existence check, count, total, selected single record, enum state, timestamp, flag, string, number, or fixed-shape summary. A collection can contain the fact, but the business function should be shaped around the fact itself.

6. **Name last**

   Do not begin by naming `User`, `Book`, `Order`, `Manager`, `Service`, or shared types. Write concrete rules first. After aggregation, name the unit that actually emerged. It may be a surprising business unit such as `BorrowDecision`, `RenewPolicy`, `EligibilityCheck`, or `MembershipMerge`, not the obvious storage nouns. A name should come from the completed aggregation, not from an architectural guess.

7. **Extract abstraction only after repetition**

   Do not create shared types, base classes, interfaces, or folders because they seem likely to be useful. Extract them only when the same shape or rule actually repeats.

8. **Verify one step at a time**

   Make one structural change, run the relevant test or program, then continue. If the structure looks wrong, step back and re-aggregate instead of forcing the planned design.

9. **Make business tests mock-free**

   The architecture should make the most important business logic easy to test without mocks. Pure business functions should be tested with plain input data and expected output data. Mocks belong only at side-effect boundaries such as storage, network, time, random values, queues, logs, or external APIs.

10. **Point outer work toward the business core**

   Let UI/app code coordinate runtime. Let runtime and adapters prepare the facts requested by the business core from storage, network, time, random values, queues, logs, and other external systems. Pass those facts into pure business functions. Use the returned decisions to execute runtime effects. Keep the business core as plain functions with explicit fact inputs and decision outputs.

## Workflow

### 1. Choose One Business Rule

Pick a concrete operation, not a noun. Prefer:

- "decide whether a user can borrow a book"
- "decide whether an order can be cancelled"
- "calculate renewal eligibility"
- "merge incoming membership state"

Avoid starting with:

- "create a User class"
- "design the domain layer"
- "make the repository interfaces"
- "define all shared types"

### 2. Gather the Rule Into One Cluster

Find all scattered checks, fields, mutations, reads, and writes involved in that operation. Temporarily bring the related logic together so the real dependencies are visible.

Look for:

- business conditions
- values compared by those conditions
- data fetched only to support those conditions
- writes triggered by the decision
- time, randomness, I/O, or external state used during the decision

### 3. Split the Cluster Into Two Piles

Separate each operation by what it fundamentally is:

- **Pure computation**: compare, calculate, validate, transform, select, decide, merge, rank, format a result.
- **Side effect**: query, persist, mutate, call APIs, send messages, read time, generate random values, log, schedule, access global state.

If a function both decides and performs I/O, split it.

### 4. Extract the Pure Function First

Write the business decision as plain data in and plain data out.

Good:

```ts
evaluateBorrow(
  user: { score: number; unpaidFine: number } | null,
  book: { availableCopies: number } | null,
  status: { activeLoanCount: number; overdueCount: number },
  alreadyBorrowed: boolean,
  now: Date,
): BorrowDecision
```

Bad:

```ts
user.borrow(book, database)
```

The pure function must not call `new Date()`, `Date.now()`, `Math.random()`, a repository, an HTTP client, the filesystem, or mutate objects. Pass those values in from the outside.

### 5. Narrow the Inputs

After the pure function works, reduce its parameters to the facts the rule actually uses.

Do not pass:

- whole ORM records
- mutable domain objects
- large owner objects
- child collections used only to discover one fact
- database query results
- field-only projections of source data, such as `siblings.map(({ name, type }) => ...)`, when the rule is asking for one fact
- values kept only because they "might be needed later"

Prefer:

- `score: number`
- `hasUnpaidFine: boolean`
- `activeLoanCount: number`
- `alreadyBorrowed: boolean`
- `sameNameAndTypeExists: boolean`
- `availableCopies: number`
- `now: Date`

### 6. Prepare Business Facts in Side-Effect Functions

If the rule seems to need source data, ask what fact the rule is trying to learn from that data. The business core defines the fact it needs; the repository implements the query that produces that fact. Orchestration only satisfies that data contract.

Replace:

```ts
evaluateBorrow(user, book, loans: Loan[])
```

With:

```ts
evaluateBorrow({
  score,
  availableCopies,
  activeLoanCount,
  overdueCount,
  alreadyBorrowed,
})
```

The repository or storage function should perform filtering, counting, existence checks, pagination, and aggregation. Computation receives the business fact requested by the core.

Another example:

Bad:

```ts
decideCreateNode({
  name,
  type,
  siblings: siblings.map(({ name, type }) => ({ name, type })),
})
```

The rule is looking for one fact: whether a conflicting sibling already exists.

Good:

```ts
decideCreateNode({
  name,
  type,
  sameNameAndTypeExists,
})
```

The side-effect function prepares that fact from storage:

```ts
existsSiblingNodeByNameAndType({
  parentId,
  name,
  type,
})
```

Passing `sameNameAndTypeExists` into the business function is not business leakage into orchestration. It is the data specification declared by the business core. The business function still owns the decision: today it may reject duplicates; later it may allow duplicates under other conditions.

### 7. Build Side Effects Around the Pure Function

Only after the pure function's input and output are clear, define the I/O required to support it.

Typical side-effect functions:

```ts
getUserScore(userId)
getAvailableCopies(bookId)
countActiveLoans(userId)
countOverdueLoans(userId)
existsActiveLoan(userId, bookId)
saveLoan(record)
updateBookCopies(bookId, delta)
```

These functions are shaped by what the pure function needs. The pure function is not shaped by the database schema.

### 8. Keep Orchestration Thin

Orchestration should load the business facts requested by the core, call pure computation, and execute the resulting effects. It should not contain business conditions.

Good shape:

```ts
const input = await loadBorrowDecisionFacts(userId, bookId)
const decision = evaluateBorrow(input)
if (!decision.allowed) return decision
await persistBorrow(decision.effects)
```

Red flag:

```ts
if (user.score < 60) ...
if (loans.length > 5) ...
if (book.availableCopies <= 0) ...
```

Those checks belong in pure computation.

### 9. Name the Aggregated Unit

After aggregation and separation, name the unit that is now visible. Do not name from the nouns you expected at the beginning. Name from the role the aggregated unit actually plays.

Examples:

- a pure function that decides whether borrowing is allowed may become `BorrowDecision`
- a pure function that calculates renewal eligibility may become `RenewPolicy`
- a pure function that merges incoming cluster records may become `MembershipMerge`
- repeated side-effect functions that prepare business facts may become `BorrowFactRepository`
- repeated fact input shapes may become shared types, but only after repetition

Do not move from `User` or `Book` toward a business object just because those nouns exist in storage. Storage records such as `User`, `Book`, or `Loan` may be only persistence containers. The true business unit can be counterintuitive and should be named only after aggregation exposes it.

## Refactoring From Traditional OOP

When existing code uses classes with data and methods mixed together:

1. Pick one method that contains a business rule.
2. Copy its conditions into a standalone pure function.
3. Replace `this.field` access with explicit parameters.
4. Replace child arrays or nested source data with business facts prepared by storage.
5. Move database writes, mutation, logging, time, and random values outside the pure function.
6. Let the original method become thin orchestration or delete it if it no longer owns behavior.
7. Run tests after each step.

Do not preserve a class just because its name sounds like the business domain. If it only stores data, it is a record. If it mutates or persists, it is side-effect code. If it decides from explicit inputs, it is a pure function.

## Red Flags

Watch for these signs that the design is drifting back into premature OOP or layered architecture:

- Starting by creating nouns: `User`, `Book`, `Loan`, `Manager`, `Service`.
- Pure functions accepting whole records when they use one or two fields.
- Business rules inside services, controllers, repositories, or entity methods.
- A pure function calling time, random, database, network, filesystem, logger, or global state.
- Business functions shaped around source data instead of the fact the rule needs.
- Field-only projections of source data where the rule is asking for an existence, count, state, value, or selected item.
- Shared types created before actual repetition.
- Repository functions returning class instances with behavior.
- Architecture folders created before concrete rules exist.
- Business-rule tests that require mocks for database, network, clock, random values, or framework objects.
- Business rules requiring runtime objects before they can run.
- Runtime, framework, storage, or network details appearing in pure function signatures.

## Expected Shape

The final project may look like this, but do not force this shape upfront:

```txt
src/
  domain/
    decisions/        # pure business computation
    utils/            # shared pure helpers, only after repetition
  repository/         # side-effect functions for storage and external state
  service/            # thin orchestration between side effects and pure functions
  transport/          # HTTP, CLI, queue, RPC, UI adapters
```

The folder names are less important than the invariant: business computation stays pure, growing data stays behind side-effect functions, names follow the structure that emerges, and outer layers point inward toward the business core.

A typical application composition may look like:

```txt
UI/app -> runtime/adapters -> business facts -> pure computation -> decisions -> runtime effects
```

Runtime can provide network, storage, clock, random, queue, logging, and scheduling support. It depends on the business core for business guidance. The business core remains independent and testable with plain values and no mocks.

## Final Check

Before finishing, verify:

- Can every business rule be tested without a database, network, clock, random source, or framework?
- Do business-rule tests use plain inputs and outputs instead of mocks?
- Does every pure function receive the business facts the rule actually needs?
- If source data appears in a business input, what fact is the rule trying to learn from it?
- Can a repository answer the need as an existence check, count, enum state, selected single record, or fixed-shape summary?
- Are source data and external state prepared into business facts before computation?
- Is orchestration free of business conditions?
- Do UI, runtime, storage, network, and framework code gather the facts requested by pure business functions?
- Are business functions still plain input-to-decision functions?
- Did every shared type or abstraction appear because of actual repetition?
- Did the names come after observing the clusters?
