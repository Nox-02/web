# Contributing

## Workflow
1. Pick an issue from the Sprint Board (status: Ready) and assign yourself.
2. Create a branch from an up-to-date `main`:

   ```
   git checkout main
   git pull
   git checkout -b feature/123-short-description
   ```

3. Commit small and often. Commit style: `type: summary (#123)`
   - Types: feat, fix, docs, style, refactor, test, chore
4. Push and open a pull request. Put `Closes #123` in the description.
5. Get at least 1 approval and a green CI, then squash and merge.
6. The branch is deleted automatically after merge.

## Branch names
- `feature/<issue>-<slug>`
- `fix/<issue>-<slug>`
- `chore/<issue>-<slug>`

## Rules
- Never push directly to `main`.
- Keep PRs small (aim for under ~400 changed lines).
- Answer review requests within one working day.

## Scrum events
| Event | Where |
|---|---|
| Refinement | Issues labeled `needs-refinement`, then move to Ready |
| Sprint planning | Drag Ready issues into the current Sprint iteration |
| Daily standup | Sprint Board view |
| Review | Merged PRs + a tagged release (v0.1.0) |
| Retrospective | A `retro` issue or Discussion per sprint |