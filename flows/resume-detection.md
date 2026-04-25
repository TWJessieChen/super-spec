# Resume Detection

Run: `openspec status --change <name> --json`. Parse, then route by detected state:

| State | Jump to |
|---|---|
| no `design.md` | Phase 1 |
| `design.md` exists, no `tasks.md` | Phase 3 |
| `tasks.md` exists with `- [ ]` items | Phase 4 |
| all tasks `- [x]`, no `review.md` | Phase 5 |
| `review.md` verdict = `APPROVED`, not yet archived | Phase 6 |
| `review.md` verdict = `CHANGES REQUESTED` | Phase 4 (run revision tasks only) |
| `review.md` verdict = `NEEDS DESIGN UPDATE` | Recover flow (`flows/recover.md`) |

Announce: `Resuming <name> at <phase>.`
