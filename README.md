# Tell Me A Story — A CLAUDE.md file for Objects that Think, Speak, and Own their Truth

This repository contains a `CLAUDE.md` file — a style guide written for AI assistants.
Drop it in the root of any Python project and any AI that reads it will write code the way it should be written.

---

## What is this?

Most style guides tell you where to put your imports.
This one tells you how your objects should *think*.

`CLAUDE.md` is a convention for AI coding assistants. When placed at the root of a project,
tools like Claude Code read it automatically and use it as the law of the land for that codebase.

This particular file encodes a philosophy rooted in **Alan Kay's original vision of OOP**:
objects are not data containers. They are agents. They receive messages, they answer questions
about themselves, and they are responsible for their own validity from the moment they are born.

---

## The Core Idea

Code should read like a short story.

Not like a series of instructions. Not like a data pipeline. A story —
where each object has a voice, each method is a sentence, and the reader
never has to peek inside an object to understand what is happening.

```python
# This is not a story. It is an interrogation.
if user._role == "admin" and len(user._permissions) > 0:
    ...

# This is a story.
if user.is_admin() and user.has_any_permission():
    ...
```

---

## What the CLAUDE.md covers

- **Objects validate themselves** — an object is born valid or not born at all
- **Tell, Don't Ask** — objects answer semantic questions; they never expose raw state
- **Predicates over comparisons** — `is_`, `has_`, `can_`, `was_`, `contains_` everywhere
- **Errors belong to the object** — the one who owns the invariant builds the error
- **Services read as prose** — orchestration looks like a workflow description, not procedural code
- **Strategies are objects** — variable behavior is never a string flag or a boolean
- **Explicit `__init__` over `@dataclass`** — initialization is not boilerplate, it is the birth of the object
- **Inheritance vs Composition** — chosen deliberately based on the problem, never dogmatically

---

## How to use it

1. Copy `CLAUDE.md` to the root of your Python project
2. Open the project with any AI assistant that supports the convention (e.g. Claude Code)
3. The assistant will read the file and write code that follows the style automatically

No configuration needed. No plugins. Just a markdown file that speaks for you.

---

## Philosophy in one sentence

> An object is not a bag of data with functions attached.
> It is a living thing that knows itself, speaks for itself, and answers for itself.

---

## Author

Written by a developer who believes that good code
should be indistinguishable from good writing.
