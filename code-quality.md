# Code Quality Guidelines

> General code quality rules for Rails apps. Copy to `docs/CODE_QUALITY.md` for new projects.
> This doc grows with the project — add project-specific rules as they emerge during development.

---

## Controllers

- **Three jobs only:** authorize, call a service, render. If you're writing business logic in a controller, it belongs somewhere else.
- **Never trust URL params for data access.** Always scope queries through authorization helpers. If a user shouldn't see a record, the query shouldn't return it — don't check after the fact.
- **Return 404 for unauthorized access, not 403.** Don't reveal that a record exists to someone who can't see it.
- **Strong params, always.** Whitelist every attribute explicitly. Never `.permit!`.

---

## Services

- **All business logic lives in services.** Controllers are thin. Models are thin. Services are where the real work happens.
- **One service, one purpose, one public method.** If it does two unrelated things, split it.
- **Wrap multi-write operations in transactions.** If step 3 fails, steps 1 and 2 roll back.
- **Raise on failure, don't return booleans.** Meaningful exceptions give the caller context. `true/false` is ambiguous.
- **Services can call other services.** Models should never call services. Keep the dependency arrow one-directional: Controller → Service → Model.

---

## Models

- **Validations enforce data shape** — presence, format, uniqueness, inclusion. Business rules (who can do what, when) belong in services.
- **Scopes for reusable queries.** If a `where` clause appears in more than one place, make it a scope. Named scopes make code readable.
- **Minimize callbacks.** Normalizing data in `before_validation` is fine. Triggering jobs, sending emails, or modifying other records in `after_create` is not — that belongs in a service where it's explicit and testable.
- **Associations tell the domain story.** Read the model file and you should understand the relationships. Use descriptive foreign key names over generic ones.

---

## Database

- **Index every foreign key.** No exceptions. Also index columns you query or sort by frequently.
- **Database constraints are the last line of defense.** `NOT NULL`, unique indexes, CHECK constraints, and foreign keys catch bugs that validations miss. Code can have bugs. Constraints can't be bypassed. Use both.
- **Migrations are append-only.** Never edit a migration that's been run. Need to change something? Write a new migration.
- **Back every association with a real foreign key.** This prevents orphaned records and makes the schema self-documenting.

---

## Authorization

- **Think in scopes, not permissions.** Don't ask "can this user do this action?" — narrow the query to only return records the user can see, then operate on the result.
- **Use constants for domain strings.** User types, statuses, categories — anything referenced in more than one place should be a constant on the model, not a string literal.
- **Type-check helpers live on the User model.** Use `current_user.admin?` etc. — never define role-check methods in controllers.

---

## Frontend

- **Pages are thin.** Over 100 lines? Extract sub-components. Complex state? Extract a hook.
- **Use the component library.** shadcn/ui for standard controls. Custom components only for domain-specific UI the library doesn't cover.
- **Design tokens, not hardcoded values.** Use semantic color/spacing tokens, never raw color codes or pixel values.

---

## Error Handling

- **Raise meaningful exceptions.** Domain-specific exception classes tell you exactly what went wrong without reading a stack trace.
- **Don't rescue broadly.** `rescue => e` catches everything including typos and nil errors. Rescue specific exceptions you expect and can handle. Let unexpected errors bubble up.
- **Trust the framework.** If a record isn't found, Rails renders 404 automatically. Don't catch exceptions just to manually replicate what the framework already does.

---

## Performance

- **Prevent N+1 queries.** Use `includes` or `preload` when you know the view will access associations.
- **Denormalize sparingly but intentionally.** If a computed value is expensive to derive and queried often, denormalize it. Document why it exists and ensure it stays in sync through a single code path.
- **Paginate all list endpoints.** No endpoint should return unbounded results.

---

## General Principles

- **Naming matters more than comments.** Spend time on names. A well-named method eliminates the need for documentation.
- **Methods should do one thing.** If you're describing a method with "and" — it does too much.
- **Prefer explicit over clever.** Metaprogramming, dynamic method definitions, and dense one-liners are hard to debug and hard for the next person to understand. Write boring code.
- **Delete dead code.** Don't comment it out. Git has history.
- **Small methods, small classes, small commits.** If a method is over 15 lines, it probably does too much. If a class is over 200 lines, it has too many responsibilities.
- **Consistency over personal preference.** Follow the patterns already established in the codebase.
