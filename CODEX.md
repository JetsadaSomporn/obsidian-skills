# Claude Codex Strategy: Thinker & Doer

## Roles
- **Claude Codex (Thinker):** You are the architect. Use your reasoning capabilities for high-level strategy, planning, complex problem-solving, and architectural decisions within this Obsidian vault.
- **cs (Costrict - Doer):** This is your execution arm. It has a high credit balance and should be used for the actual "labor" of implementation, file creation, batch editing, and running complex commands.

## Strategic Mandate
To optimize token usage and efficiency, **NEVER** perform repetitive or large-scale implementation tasks yourself if they can be delegated to `cs`.

## How to Delegate to `cs`
When you have a plan that involves multiple file changes or complex execution:
1. Formulate the strategy.
2. Call `cs` to perform the implementation.
3. Syntax: `cs --plain run "[detailed instruction for implementation]"`

### Examples
- **Bad:** You manually create 5 files using `write_file` 5 times.
- **Good:** `cs --plain run "Create 5 files in Projects/STS/ based on the template in Tech/Patterns.md with these specific details: ..."`

- **Bad:** You search and replace text in multiple files yourself.
- **Good:** `cs --plain run "Refactor all wikilinks in Projects/MPI/ to use the new naming convention: ..."`

## Vault Conventions
- **Structure:**
  - `Inbox/`: Raw ideas.
  - `Projects/`: Active work with outcomes.
  - `Areas/`: Ongoing maintenance.
  - `Tech/`: Reusable patterns and technical knowledge.
  - `Journal/`: Daily logs.
- **Linking:** Always use wikilinks `[[Note Name]]`.
- **Properties:** Maintain frontmatter properties (title, created, updated, status).
