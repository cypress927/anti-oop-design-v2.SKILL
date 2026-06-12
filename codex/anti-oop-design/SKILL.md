---
name: anti-oop-design
description: >-
  Use when starting or refactoring business logic with an anti-OOP,
  function-first architecture: discover structure by aggregating around
  complete business units, split pure computation from side-effect
  functions, derive fixed-size business facts from source data, return complete
  business-computed data, and name abstractions only after they emerge.
---

# Anti-OOP Design

Use this skill to design or refactor business logic without starting from entities, class hierarchies, DTOs, preset layers, or storage nouns. The goal is not to invent an architecture upfront. The goal is to expose the architecture by aggregating related business facts, separating computation from side effects, and naming only what remains.

## Core Model

A program has only two kinds of functions:

- **Pure functions** compute: they transform explicit input data into explicit output data. They do not read time, random values, files, databases, network state, global state, or mutable objects.
- **Side-effect functions** touch the outside world: they read, write, persist, fetch, send, log, schedule, mutate, or otherwise depend on external state. They may return values, but they have effects.

Orchestration is not a third kind of function. It is the calling pattern between these two kinds: use side-effect functions to prepare the facts required by the business core, call pure functions, then execute external effects from the returned data.

## Main Direction

Do not design architecture from storage nouns, entity nouns, framework nouns, or layer nouns.

Instead:

1. Start from one business unit and aggregate the related business facts, constraints, computed results, and external effects around it.
2. Split the result into pure computation and side-effect functions.
3. Name the unit that emerges after aggregation.

A business unit is usually one complete business action, such as transferring money, depositing money, registering a client, renewing a loan, placing an order, or moving a file.

Avoid choosing an internal step as the unit, such as computing a fee, validating an amount, checking a duplicate name, calculating a discount, or formatting output records.

The real business unit may be counterintuitive. Its name should come from the responsibility that appears after aggregation, not from the storage shape seen at the beginning.

## Design Principles

### 1. Aggregate First, Separate Second

First choose one business unit, then bring the facts, checks, reads, writes, computed results, and state changes around that unit into the same view.

Aggregation keeps related business change points visible together: when a rule changes, the facts it reads, the data it computes, and the external effects it drives can be reviewed and updated as one unit.

Feature evolution starts by re-aggregating the affected business unit. When a change affects an existing business operation, gather the new facts, constraints, computed values, result data, and external effects around that unit.

For example, if transfers later require fees, revisit the transfer unit so fee facts, fee calculation, balance changes, and returned ledger data are settled together.

This is discovery, not upfront design. Do not classify, name, or abstract before the related facts are visible together.

### 2. Pure Computation Is The Center

Business rules belong in pure functions.

Controllers, services, repositories, adapters, and framework code exist to prepare inputs or execute outputs.

### 3. Business Facts Are Fixed-Size Projections

When shaping a pure business function, observe the facts the rule actually needs.

A business fact is a fixed-size feature projected from source data. It may come from many records or a complex query, but by the time it enters the business core, it has been shaped into a small stable form.

The business core sees the features needed by the rule, not the source data that produced those features.

### 4. Business Facts Are Quantifiable Data Features

A business fact should be clear enough to compare, calculate, judge, or name.

It is usually shaped as existence, count, total, state, rank, flag, timestamp, threshold relationship, single selected result, or fixed-shape summary.

When a collection or sequence appears as a parameter, observe whether the pure function only traverses it to reach a smaller feature. If the computation is trying to obtain a count, total, existence result, extreme value, unique state, or fixed summary, that smaller feature is closer to the business fact.

These facts can be used directly by pure functions and constructed directly in tests with plain data.

### 5. Input Business Facts, Not Source Data

Source data may contain business facts, but source data itself is usually not the business fact.

If a rule appears to need records, query results, child collections, paginated results, or external objects, keep observing the fixed-size feature the rule is trying to learn.

The business core declares the facts it needs. Repositories, runtime code, and adapters prepare the outside world into those facts.

### 6. Output Complete Business-Computed Data

A pure business function receives business facts prepared for the rule and outputs data formed by applying the business rule.

Inputs are business facts prepared before computation.  
Outputs are new business facts formed after computation.

The returned data should be complete enough for the next external action. If external code must derive another business value before it can persist, update, send, schedule, or present the result, that derivation still belongs to the pure business computation.

External code continues from the business function output instead of reinterpreting business meaning after the pure function returns.

### 7. Observe Data That Can Grow

During design, notice which data can grow over time.

The business core is interested in the fixed-size features derived from that data, not the growing data itself.

If an input shape grows with the amount of source data, the fact has probably not been fully extracted yet.

### 8. Prepare Facts Before Computation

Source data belongs to repositories, network code, runtime code, and adapters.

Before calling a pure function, prepare the source data into the facts required by the business rule.

Keep the direction simple: outer code prepares facts; the business core interprets facts.

The orchestration layer satisfies this data contract. It does not interpret business meaning.

### 9. Separate Mechanical Preparation From Business Judgment

Orchestration may perform mechanical data preparation to satisfy fact inputs already declared by the pure business function.

Mechanical preparation projects external data into facts. It may load, look up, filter, count, sum, extract fields, check membership, or build a fixed-size summary.

These operations prepare the fact; they do not decide what the fact means.

Business judgment begins when meaning is assigned to prepared facts. That judgment belongs in the pure function.

### 10. Orchestration Does Not Carry Business Judgment

Orchestration loads facts, calls the core, and executes results.

Business conditions should be aggregated into pure functions. Orchestration should not reinterpret business meaning before or after the pure function call.

### 11. Name Last

Do not begin by naming business objects.

Write concrete rules first. After aggregation is complete, name the unit that has appeared.

Names come from the responsibility that has emerged, not from preset nouns.

### 12. Abstract After Repetition

Shared types, interfaces, base classes, helpers, and folder structures should be extracted only after real repetition appears.

Do not abstract because something might be useful later.

### 13. Business Tests Should Not Need Mocks

The most important business logic should be testable with plain inputs and plain outputs.

Pure business function tests should not need databases, networks, clocks, randomness, queues, logs, or framework objects.

Mocks belong at side-effect boundaries.

### 14. Outer Code Works Around The Business Core

Application entry points coordinate runtime.

Runtime code and adapters prepare the facts requested by the business core.

The business core uses pure functions to compute decisions and result data.

Runtime code then performs external effects from those result values.

The business core remains ordinary functions with explicit fact inputs and explicit computed outputs.

## Workflow

### 1. Choose One Concrete Business Unit

Choose one concrete business unit, usually a complete business action, not a noun, storage object, validation step, or calculation step.

Boundary check: describe the unit to a non-technical person. If the natural response is "and then what happens?", the chosen unit is probably only a step inside a larger action.

Prefer operations such as:

- "transfer money between accounts"
- "deposit money into an account"
- "withdraw money from an account"
- "register a new client"
- "open a bank account"
- "renew a book loan"
- "place an order"
- "move a file"

Avoid starting from:

- "compute transfer fees"
- "validate deposit amount"
- "calculate available balance"
- "check duplicate file name"
- "create a User class"
- "design the domain layer"
- "make the repository interfaces"
- "define all shared types"

### 2. Gather The Related Logic

Find the checks, fields, reads, writes, mutations, time, randomness, and external state involved in the operation.

Let the dependencies become visible first.

### 3. Split Into Two Piles

Pure computation compares, calculates, validates, transforms, selects, decides, merges, ranks, and produces results.

Side effects query, persist, mutate external state, call APIs, send messages, read current time, generate randomness, log, schedule, and access global state.

If a function both decides and performs I/O, split it.

### 4. Extract The Pure Function First

Write the business computation as plain data in and plain data out.

Good shape:

```ts
evaluateBorrow({
  score,
  availableCopies,
  activeLoanCount,
  overdueCount,
  alreadyBorrowed,
  now,
})
```

Bad shape:

```ts
user.borrow(book, database)
```

A pure function does not read current time, generate randomness, access repositories, call HTTP, or mutate objects.

Those values are passed in from the outside.

### 5. Narrow Inputs To Fixed-Size Facts

After the pure function works, narrow its parameters to the facts actually used by the business rule.

For each parameter, ask whether it is already the fixed-size fact the rule needs. If it must be traversed, searched, counted, summed, filtered, or compared before a smaller value appears, keep converging the parameter toward that smaller value.

Repeat until the inputs are stable as scalars, enums, timestamps, flags, or fixed-field records.

Replace this kind of shape:

```ts
evaluateBorrow(user, book, loans)
```

With this kind of shape:

```ts
evaluateBorrow({
  score,
  availableCopies,
  activeLoanCount,
  overdueCount,
  alreadyBorrowed,
})
```

Another example:

Bad shape:

```ts
decideCreateNode({
  name,
  type,
  siblings: siblings.map(({ name, type }) => ({ name, type })),
})
```

The rule is looking for one fixed-size fact: whether a conflicting sibling already exists.

Good shape:

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

Passing `sameNameAndTypeExists` into the business function is not business leakage into orchestration. It is the data specification declared by the business core. The business function still owns the decision.

### 6. Prepare Facts In Side-Effect Functions

Repositories and adapters project the outside world into the facts requested by the business core.

The business core still owns the decision.  
External code only provides the facts it asked for.

Typical fact-preparation functions are shaped around the business core's requested facts:

```ts
getUserScore(userId)
getAvailableCopies(bookId)
countActiveLoans(userId)
countOverdueLoans(userId)
existsActiveLoan(userId, bookId)
```

### 7. Output Business-Computed Data

Pure business function output should express the data formed by the business rule.

That data comes from the business rule itself, not from another round of derivation in external runtime code.

External code uses the output data to persist, update, send, schedule, or present.

For result types, include the facts that the next external action needs:

```ts
type RenewalDecision = {
  allowed: boolean
  reason: string | null
  nextPlanRank: number
  chargeAmount: number
  quotaDelta: number
  effectiveAt: Date
}
```

On the output side, observe what facts the rule settles. If a create-node rule settles the recorded size or whether storage usage changes, those settled facts should travel out with the result. Runtime continues from those returned facts instead of inspecting the original input again to rediscover business meaning.

### 8. Build I/O Around The Pure Function

Only after the pure function inputs and outputs are clear should side-effect functions be defined.

Side-effect functions are shaped by the facts needed by the business core, not by the storage schema alone.

### 9. Keep Orchestration Thin

Orchestration does three things:

1. Load the facts requested by the business core.
2. Call the pure business function.
3. Execute external effects from the returned result.

For one business unit, orchestration should read like one pass: load prepared facts, call the pure business computation for that unit, then execute the returned results.

When a new feature adds business meaning, return to the affected business unit and re-aggregate it.

Good shape:

```ts
const facts = await loadBorrowDecisionFacts(userId, bookId)
const decision = evaluateBorrow(facts)
await applyBorrowDecision(decision)
```

Business conditions should not be scattered through orchestration.

### 10. Name The Aggregated Unit

After aggregation and separation, name the unit that has appeared.

The name should come from the responsibility that is now visible.

Examples:

- a pure function that settles whether borrowing can proceed may become `BorrowDecision`
- a pure function that settles a loan renewal may become `LoanRenewal`
- a pure function that merges incoming records may become `MembershipMerge`
- repeated side-effect functions that prepare business facts may become `BorrowFactRepository`
- repeated fact input shapes may become shared types, but only after repetition

## Refactoring From Traditional OOP

When existing code mixes data and methods inside classes:

1. Choose one concrete business unit touched by the class method.
2. Move its conditions, computed values, and result data into a standalone pure function.
3. Replace object field access with explicit parameters.
4. Replace collections, nested records, and query results with fixed-size business facts.
5. Move persistence, mutation, logging, time, and randomness outside the pure function.
6. Let the original method become thin orchestration, or remove it if it no longer owns behavior.
7. Run tests after each step.

If a class only stores data, it is a record.  
If it mutates external state, it is side-effect code.  
If it decides from explicit inputs, it should become a pure function.

## Observation Signals

The design is drifting when these signals appear:

- Architecture starts from noun-shaped objects.
- A pure function receives a complete record while using only a few fields.
- Business judgment appears inside controllers, services, repositories, or entity methods.
- A pure function reads time, randomness, databases, network, logs, or global state.
- A business function is shaped around source data instead of the facts required by the rule.
- A top-level business function called by orchestration represents a calculation or validation step instead of the whole business unit.
- Business input size grows with source data size.
- A pure function receives a collection or sequence mainly to derive a count, total, existence result, extreme value, state, or fixed summary.
- Growing data is passed into the core to obtain a fixed-size feature.
- External code receives pure function output and then derives another business value before applying the result.
- A new feature changes an existing operation, but the affected business unit is not re-aggregated.
- Shared types or abstractions appear before real repetition.
- Business tests require mocks for databases, networks, clocks, randomness, or framework objects.
- Runtime objects appear in pure business function signatures.
- Orchestration reinterprets business meaning after the pure function returns.

## Structural Direction

Folder names are not important. What matters is:

Business computation stays pure.  
The outside world is projected into fixed-size business facts before computation.  
Business functions return the data they compute.  
Returned data is complete enough for the external action that follows.  
Names come from the structure that appears after aggregation.  
External code prepares facts according to the business core input requirements and performs side effects from the output data.

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

A typical application composition may look like:

```txt
UI/app -> runtime/adapters -> business facts -> pure computation -> computed result -> runtime effects
```

## Final Check

Before finishing, check:

- Did the design start from a concrete business unit rather than a calculation or validation step?
- Can every business rule be tested without a database, network, clock, random source, or framework?
- Do business tests use only plain inputs and plain outputs?
- Are pure function inputs the business facts actually needed by the rule?
- Do pure function inputs stay fixed-size instead of growing with source data size?
- Are business facts quantifiable, comparable, judgeable, and nameable data features?
- If a parameter is a collection or sequence, what smaller fixed-size feature is the rule trying to obtain?
- Does pure function output express the data formed by the business rule?
- Is pure function output complete enough for the external action that follows?
- If external code derives another business value after the pure function returns, should that value be returned by the pure function instead?
- Can repositories answer the need as fixed-size business facts?
- Is external state prepared into business facts before computation?
- Does orchestration only load facts, call the core, and execute results?
- Does orchestration limit itself to mechanical preparation before calling the core?
- Is business meaning assigned inside pure functions?
- When an existing operation changed, was the affected business unit re-aggregated?
- Is each business function still ordinary fact input to computed data output?
- Did abstraction come from real repetition?
- Did naming come from the unit that appeared after aggregation?
