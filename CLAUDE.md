# CLAUDE.md — Python Development Style

This file defines how to write Python code in this project.
Read it fully before writing any code.

---

## North Star

Alan Kay's original vision of OOP: **objects are agents that communicate via messages**.
They are not data containers. They are not structs with functions attached.
Each object is responsible for its own state, its own validity, and its own vocabulary.

Code must read like a short story. Each line should feel like a sentence in natural language.

---

## Core Principle: Tell, Don't Ask

Never extract state from an object to compare it outside. Ask the object the semantic question.

```python
# ❌ Ask style — extracts internals, compares outside
if user._active:
if document.status == "draft":
if len(cart._items) == 0:

# ✅ Tell style — the object answers
if user.is_active():
if document.is_draft():
if cart.is_empty():
```

This applies **everywhere**: in services, in other objects, in tests.
If you find yourself writing `obj.attribute == something`, stop and ask whether that comparison belongs inside `obj`.

---

## Rules

### 1. Objects validate themselves

An object is born valid or not born at all.
Validation lives inside `__init__` via a dedicated private assert method — never in an external service.

Prefer explicit `__init__` over `@dataclass`. Initialization is not boilerplate — it is the moment
the object is born and takes responsibility for its own validity. That deserves to be written explicitly.

```python
class Email:
    def __init__(self, address: str):
        self._address = address
        self._assert_email_is_valid()

    def _assert_email_is_valid(self):
        import re
        pattern = r"^[\w\.-]+@[\w\.-]+\.\w{2,}$"
        if not re.match(pattern, self._address):
            raise ValueError(f"{self._address!r} is not a valid email address.")
```

**Exception — value objects**: pure values with no initialization logic (e.g. `Color`, `Coordinate`)
may use `@dataclass(frozen=True)` to get immutability, `__eq__`, and `__hash__` for free.

```python
@dataclass(frozen=True)
class Color:
    red: int
    green: int
    blue: int
```

### 2. Boolean queries are named predicates

Use prefixes that read naturally: `is_`, `has_`, `can_`, `was_`, `contains_`.

```python
def is_active(self) -> bool: ...
def has_permission(self, action: str) -> bool: ...
def can_be_edited(self) -> bool: ...
def was_created_by(self, user: User) -> bool: ...
def contains_item(self, item: Item) -> bool: ...
```

These predicates are used **everywhere**, including inside the object's own methods.
Do not inline their logic — call the predicate, even from private methods.

```python
# ❌ Inlined logic inside the object
def next_state(self):
    if self.status == "pending":
        self.status = "active"

# ✅ Predicate used even internally
def next_state(self):
    if self.is_pending():
        self.status = "active"
```

### 3. Method names are sentences

Methods are messages. Name them as natural language predicates or commands.

```python
# ❌
def check(self, user): ...
def get_items(self): ...

# ✅
def can_be_accessed_by(self, user: User) -> bool: ...
def active_items_sorted_by_date(self) -> list[Item]: ...
```

### 4. Errors are built by the object

Error construction is a private responsibility of the object that owns the invariant.

```python
# ❌ Raw error inline
raise ValueError(f"User {self.username!r} has no permission for {action}")

# ✅ Named private builder
def _unauthorized_error(self, action: str) -> ValueError:
    return ValueError(
        f"User {self.username!r} is not allowed to perform {action!r}."
    )
```

### 5. Services read as orchestration prose

Application services coordinate objects. No business logic here — just workflow.

```python
def publish(self, article: Article, author: User) -> None:
    if not article.can_be_published():
        raise RuntimeError("Article is not ready for publication.")

    if not author.has_permission("publish"):
        raise author._unauthorized_error("publish")

    article.mark_as_published()
    self._notifier.notify_subscribers(article)
```

### 6. Strategies are first-class objects

Variable behavior is a Strategy object — never a string flag or boolean parameter.

```python
# ❌
report.generate(format="pdf")
sorter.sort(items, mode="alphabetical")

# ✅
report.generate(formatter=PdfFormatter())
sorter.sort(items, strategy=AlphabeticalStrategy())
```

### 7. Value objects are frozen

Objects that represent pure values are frozen dataclasses:
immutable, hashable, equal by value.

```python
@dataclass(frozen=True)
class Coordinate:
    latitude: float
    longitude: float
```

---

## Inheritance vs Composition

Both are valid. Choose based on the problem:

- **Inheritance** when there is a genuine *is-a* relationship and you want polymorphism:
  the same message sent to different objects, each responding in its own way.
- **Composition** when an object *has* collaborators and delegates specific responsibilities to them.

Analyze the domain first. Choose the tool second.
Never bend the problem to fit the tool.

---

## Layer Structure

```
domain/
    *.py                # Pure domain objects — no external dependencies

application/
    *_service.py        # Orchestration only — reads like a workflow

infrastructure/
    repositories.py     # Storage implementations
    external_apis.py    # External service adapters

tests/
    # Uses in-memory repos and simple fakes — zero mocking complexity
```

**Domain never imports from application or infrastructure.**
Infrastructure depends on domain, never the reverse.

---

## Naming Cheatsheet

| Pattern           | Example                                              |
|-------------------|------------------------------------------------------|
| Boolean predicate | `is_active()`, `can_be_edited()`, `has_permission()` |
| Membership check  | `contains_item(i)`, `belongs_to(user)`               |
| Historical fact   | `was_created_by(user)`, `was_confirmed_at(dt)`       |
| Command           | `activate()`, `publish()`, `add_item(i)`             |
| Factory / finder  | `find_by_username(n)`, `_find_handler_for(event)`    |
| Error builder     | `_unauthorized_error(action)`, `_not_found_error()`  |
| Comparison        | `expires_before(other)`, `is_older_than(other)`      |
| Self-validation   | `_assert_email_is_valid()`, `_assert_not_expired()`  |
