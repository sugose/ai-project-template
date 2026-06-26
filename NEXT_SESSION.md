# Next Session

## Ping-pong (Clead → Crog autonomous)
We found that one Routine can call another via curl in its prompt — Crog only receives the `text` payload, never Clead's standing prompt or the token. This is the path to full autonomy without waiting for Anthropic secrets support. Need to update Clead's standing prompt to include the hardcoded curl call to Crog's fire endpoint, then test.

## Generic clone type in `ai-project-template`
Add a "generic" non-programming clone type with:
- Skeleton docs: Vision, Strategy, Spec
- Three-role workflow scaffolding (Adam, Clead, Crog)
- Bootstrap setup

## Clone type hierarchy agreed
- Generic (base)
  - Documentation
  - Product Development
    - Programmatic (Python, Node, TypeScript)
    - Non-programmatic (Training)

## Branch deletion issue
GitHub MCP connector has no delete-branch tool. Git CLI `push --delete` gets 403 in both Routines and interactive sessions. Root cause unclear — investigate or raise with Anthropic.

## Tokens
Crog and Clead tokens stored in password manager — retrieve before ping-pong work.
