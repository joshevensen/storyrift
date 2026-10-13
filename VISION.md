# Vision

Storyrift is a tool that lets an author build a world, and lets a reader experience a
different story in it every time.

Classic interactive fiction is authored end to end: every room, branch and line of text is
written in advance, so the reader can only discover what the author already wrote.
Storyrift moves the author up a level. The author defines the world: its places, people,
objects, rules and secrets. An AI narrator writes the story as it happens, shaped by the
decisions the reader makes.

This is the next step for interactive fiction now that AI can write prose.

## The problem it has to solve

An AI narrator on its own drifts. It forgets that the door was locked, invents a key that
was never there, or spoils a secret early. A story that can't keep its own facts straight
isn't a world, it's improv.

## The idea

The world's facts live in an engine, not in the AI's memory. The narrator proposes what
happens, the engine decides what is allowed, and the narrator tells the story of what
actually occurred. The author's rules and secrets hold no matter what the AI wants to
write.

- **For the author:** you write a world and its rules once, and you don't script the paths.
- **For the reader:** no two playthroughs are the same, and their choices have consequences that stick.

## Where it starts

The first version is a playable prototype where one person is both author and reader, run
through Claude Code. It exists to find out whether the loop works: reader decides,
narrator proposes, engine judges, narrator tells the result. See `DESIGN.md`.
