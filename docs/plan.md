# Plan

Target: JCGT, with an arXiv preprint first. SGP 2027 is a stretch.

| # | Phase | Output | Status |
|---|---|---|---|
| 1 | Topic | Chosen after adversarial vetting of 6 candidates | done |
| 2 | Literature | `docs/related-work.md`, `paper/refs.bib` | done |
| 3 | Repo and harness | Sandboxed runner, 5 libraries building in Docker | next (spec first) |
| 4 | Pilot | 500 meshes × 2 libraries × 2 operations; are failures real? | |
| 5 | Validator, minimizer, sanitizer | The three contributions | |
| 6 | Full run | All tables and figures from one script | |
| 7 | Paper | arXiv-ready PDF | |
| 8 | Endorsement and feedback | arXiv endorser, external reader | in parallel from now |
| 9 | Release | Project page, tagged release | |
| 10 | Promotion | Thread, posts, upstream bug reports | |

**Kill criterion:** if the pilot finds under about 1% interesting failures (beyond documented precondition violations), stop and rethink before Phase 5.
