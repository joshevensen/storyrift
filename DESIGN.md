# Design

Working design for the first prototype. Short on purpose: update it when a decision
changes. The schema lives in `schema.prisma`, not here. See `VISION.md` for the why.

## Principle and flow

Claude never changes world facts directly. A script owns state.

Per turn: player input -> Claude proposes changes -> engine judges -> Claude narrates the accepted result.

Claude's only route to state is MCP tool calls to the engine. This reopens the earlier
idea that "the AI never proposes state changes": Claude proposes, the engine decides.

## Decisions made

| Decision | Reason |
| --- | --- |
| TypeScript, MCP server via the official MCP TypeScript SDK | Tool calls are the only route to state |
| Runs through Claude Code on a subscription, no direct API calls | Project constraint |
| SQLite via Prisma 7.10.0 + `@prisma/adapter-better-sqlite3`, with migrations | Chosen stack. Pin 7.10.0: npm's `latest` for `prisma` was 8.0.0-rc.22 when checked |
| Fixed tables: entities, properties, events. Author content is data inside them | No tables generated from the author's world |
| `Json` columns are used for property values and event payloads | Tested on 7.10.0: migrates as JSONB, round-trips objects and scalars, `equals` and `path` filters work |
| zod validates proposals; proposals come from a typed set (move, set property, create entity) | Claude can't propose arbitrary changes |
| The judge is a pure function `(state, proposal) -> accepted or rejected(reason)`, separate from MCP and the DB | Rules are testable without Claude |
| No framework (not NestJS, not Mastra) | NestJS adds ceremony for about five tools. Mastra runs its own LLM loop, which is the API route we're not using |
| One repo. `engine/` is the build workspace, `play/` holds play-session config | Build and play are separate sessions |
| DB and author seed content live outside the repo, e.g. `~/.storyrift/` | Secrets must not live in files Claude reads |

## Open questions

Not decided. The lean is a lean, not a decision.

- **Event log.** Options: (A) append-only is the source of truth and state is derived by replay; (B) `properties` holds current state and `events` is an append-only audit trail written in the same transaction; (C) mutable, no log. Lean: B.
- **State vs. prose.** Options: (A) Claude must declare anything a rule reads or whose later contradiction would be a bug, and atmosphere stays free prose; (B) Claude declares everything that changes. Lean: A.
- **Secrets.** Tool calls show in the player's terminal, so the protection is that Claude doesn't receive a secret before it is relevant. Options: (A) the engine resolves hidden checks itself and returns only outcomes; (B) secrets are released on a trigger as narratable text; (C) Claude gets everything and spoilers are accepted. Lean: A plus B. Cost: Claude can't foreshadow a secret it hasn't been given, so the engine supplies "narrator may hint" text.
- **Claude Chat / Claude Desktop as a play surface.** Not verified. Only third-party sources were found, and they disagree on plan requirements. Build for Claude Code first.

## Security model

Restricting Claude's access to the state directory (from the Claude Code permissions and
sandboxing docs, as of this writing):

- `Read(...)` deny rules cover the built-in file tools. Grep and Glob coverage is described as best-effort.
- `Read` deny rules do not cover Bash. `sandbox.filesystem.denyRead` does, with the sandbox enabled. The docs say Bash deny rules are not a security boundary. Use both.
- Local MCP servers run outside the sandbox with full access, so the engine can reach the DB while Claude can't.
- In user-level settings, use `~/` or `//absolute` path forms. A bare `/path` is project-relative.
- **Unverified:** whether Claude can edit its own settings file to remove the deny rules. Check before relying on this.

## First slice

One room, a few entities, one rule, and the full propose -> judge -> narrate loop.

- **Tables:** `Entity`, `Property` (with visibility), `Event` (append-only, with status and reason).
- **Tools:** `look` (visible state of the current room), `propose` (typed change set, returns accepted or rejected with reasons), `reveal` (returns a secret only when its trigger condition is met).
- **Rule:** a locked door opens only if the player holds the key, and the key is hidden in the room.

Non-goals: extra features, a UI, a rules DSL, multiple rooms or worlds, Claude Chat support.
