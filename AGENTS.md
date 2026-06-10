# Anti-OOP Design Instructions

Use these instructions when designing or refactoring business logic in this project.

## Core Model

A program has only two kinds of functions:

- **Pure functions** compute: transform explicit input data into output data.
- **Side-effect functions** store or touch the outside world: read, write, persist, fetch, send, log, schedule, mutate, or depend on external state.

Orchestration is not a third kind. It is only the calling pattern between pure functions and side-effect functions.

## Main Rule

Do not design architecture from nouns such as `User`, `Book`, `Order`, `Manager`, or `Service`.

Instead:

1. Aggregate related business variables, checks, conditions, and operations.
2. Split the result into pure computation and side-effect functions.
3. Name the unit that emerges after aggregation.

The real business unit may be counterintuitive. It may be `BorrowDecision`, `RenewPolicy`, `EligibilityCheck`, or `MembershipMerge`, not `User` or `Book`.

## Design Principles

1. **Aggregate first, separate second**

   Gather all facts related to one business rule before naming, classifying, or abstracting.

2. **Pure computation is the center**

   Put business rules in pure functions. Controllers, services, repositories, transports, and classes only supply inputs or execute outputs.

3. **Use exact growth-independent facts**

   Exact dependencies are not enough. A function can receive only the fields it uses and still be wrong if those fields arrive as a collection that grows with system data. Pass exact, growth-independent facts. If a rule only needs to know whether a sibling node with the same `name` and `type` exists, pass `sameNameAndTypeExists: boolean`, not a `siblings` array containing only `name` and `type`.

4. **Identify growth vectors early**

   Identify data that grows over time: users, books, orders, loans, events, messages, logs, rows, pages, streams. Business computation should not depend on their size.

5. **Project growing data before computation**

   Do not pass a smaller version of growing data into business functions. Selecting only a few fields from a growing set still couples computation to growth. Side-effect functions project growing data into growth-independent facts first: existence checks, counts, totals, selected single records, enum states, timestamps, flags, or fixed-shape summaries.

6. **Name last**

   Name the aggregated unit after its role is visible. Do not force storage nouns to become business objects.

7. **Abstract only after repetition**

   Create shared types, interfaces, base classes, helpers, or folders only when the same shape or rule actually repeats.

8. **Verify one step at a time**

   Make one structural change, run the relevant test or program, then continue. If the structure looks wrong, re-aggregate instead of forcing the planned design.

9. **Make business tests mock-free**

   The most important business logic should be testable without mocks. Test pure business functions with plain input data and expected output data. Use mocks only at side-effect boundaries such as storage, network, time, random values, queues, logs, or external APIs.

10. **Point outer work toward the business core**

   Let UI/app code coordinate runtime. Let runtime and adapters gather or compute the growth-independent facts requested by the business core from storage, network, time, random values, queues, logs, and other external systems. Pass those facts into pure business functions. Use the returned decisions to execute runtime effects. Keep the business core as plain functions with explicit fact inputs and decision outputs.

## Workflow

1. Choose one concrete business operation, not a noun.

   Prefer "decide whether a user can borrow a book" over "create a User class".

2. Gather all related logic into one visible cluster.

   Include checks, fields, reads, writes, mutations, time, random values, and external state used by the operation.

3. Split the cluster.

   Pure computation compares, calculates, validates, transforms, selects, decides, merges, ranks, or formats.

   Side effects query, persist, mutate, call APIs, send messages, read time, generate random values, log, schedule, or access global state.

4. Extract the pure function first.

   Good:

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

   Bad:

   ```ts
   user.borrow(book, database)
   ```

5. Narrow the inputs.

   Remove whole ORM records, mutable domain objects, owner objects, child collections, raw query results, smaller projections of growing collections, and values kept only because they might be useful later.

6. Project growing data in side-effect functions.

   If the rule seems to need a list that grows, ask what growth-independent fact the rule really needs. The business core defines the fact it needs; the repository implements the query that produces that fact. Orchestration only satisfies that data contract.

   Replace:

   ```ts
   evaluateBorrow(user, book, loans)
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

   Also replace:

   ```ts
   decideCreateNode({
     name,
     type,
     siblings: siblings.map(({ name, type }) => ({ name, type })),
   })
   ```

   With:

   ```ts
   decideCreateNode({
     name,
     type,
     sameNameAndTypeExists,
   })
   ```

   The repository may provide `existsSiblingNodeByNameAndType({ parentId, name, type })`. That is not business leakage into orchestration; it is the data specification requested by the business core. The business function still owns the decision.

7. Build I/O around the pure function.

   Side-effect functions should be shaped by what the pure function needs, not by what the database schema happens to expose.

8. Keep orchestration thin.

   Orchestration loads the growth-independent facts requested by the business core, calls pure computation, and executes resulting effects. It should not contain business conditions.

9. Name the aggregated unit.

   Name from the role that emerged, such as `BorrowDecision`, `RenewPolicy`, `MembershipMerge`, or `BorrowInputRepository`.

## Refactoring From Traditional OOP

When existing code mixes data and methods in classes:

1. Pick one method containing a business rule.
2. Copy its conditions into a standalone pure function.
3. Replace `this.field` access with explicit parameters.
4. Replace child arrays or nested growing structures with growth-independent facts computed by storage.
5. Move database writes, mutation, logging, time, and random values outside the pure function.
6. Let the original method become thin orchestration or delete it if it no longer owns behavior.
7. Run tests after each step.

Do not preserve a class just because its name sounds like the domain. If it only stores data, it is a record. If it mutates or persists, it is side-effect code. If it decides from explicit inputs, it is a pure function.

## Red Flags

- Starting by creating nouns: `User`, `Book`, `Loan`, `Manager`, `Service`.
- Pure functions accepting whole records when they use one or two fields.
- Business rules inside services, controllers, repositories, or entity methods.
- Pure functions calling time, random, database, network, filesystem, logger, or global state.
- Arrays, pages, cursors, streams, or growing nested structures in business computation.
- Smaller projections of growing collections passed into business functions.
- Shared types created before actual repetition.
- Repository functions returning class instances with behavior.
- Architecture folders created before concrete rules exist.
- Business-rule tests that require mocks for database, network, clock, random values, or framework objects.
- Business rules requiring runtime objects before they can run.
- Runtime, framework, storage, or network details appearing in pure function signatures.

## Final Check

Before finishing, verify:

- Can every business rule be tested without database, network, clock, random source, or framework?
- Do business-rule tests use plain inputs and outputs instead of mocks?
- Does every pure function receive exact, growth-independent facts?
- Does any business input grow when users, files, orders, loans, messages, or sibling nodes grow?
- Can a repository answer the need as an existence check, count, enum state, selected single record, or fixed-shape summary?
- Are growing datasets projected by side-effect functions before computation?
- Is orchestration free of business conditions?
- Do UI, runtime, storage, network, and framework code gather the facts requested by pure business functions?
- Are business functions still plain input-to-decision functions?
- Did every abstraction appear because of actual repetition?
- Did names come after aggregation exposed the real unit?
