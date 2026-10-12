# OMEGA-376: LLM routing, increment 1. Tickets, grades and code

Published 2026-10-12 at the operator's request. These are the QA tickets behind the second comment on [singnet/Omega#373](https://github.com/singnet/Omega/pull/373).

- **Code:** [`omega-376-llm-routing-inc1`](https://github.com/snet-omega-factory/Omega/tree/omega-376-llm-routing-inc1) @ 69b58a5, on top of #373's head 193e9f58.
- **Review PR (in this fork):** [#1](https://github.com/snet-omega-factory/Omega/pull/1).
- **CI:** [run 38250418425](https://github.com/snet-omega-factory/Omega/actions/runs/38250418425), green.

Each ticket file has QA's functional requirements (FR), tests (T) and expected values (E), followed by QA's grade at 69b58a5, the evidence, and links to its commits and CI run.

| Ticket | Title | QA grade at 69b58a5 |
|---|---|---|
| [R1](R1.md) | Guard the lazy client in `AIProvider.stop()` | pass (CI) |
| [R2](R2.md) | The two architecture checks, and the proof they bite | pass (CI); three states observed |
| [R3](R3.md) | The envelope carries the provider and the model; every provider honours it | pass (CI) |
| [R4](R4.md) | The mock reports the model it received, plus the routing behaviour tests | **not built yet** |
| [R5](R5.md) | The catalogue: an allow-list, enumerated at startup | T1, T2 pass (unit); T3, T4 wait on R4 |
| [R6](R6.md) | The router registry, the null router and the hook | T1, T2 pass (unit); T3, T5 wait on R4; T4 read |
| [R7](R7.md) | The default router: #373's keyword policy, returning an envelope | T1, T2, T3, T5, T6 pass; T4 waits on R4 |

QA's full grading record: [GRADES-69b58a5.md](GRADES-69b58a5.md).

## Mapping from the #373 comment's table to tickets

| Comment row | Tickets |
|---|---|
| 1. Router returns `(provider, model)` | [R3](R3.md) (request fields), [R7](R7.md) (the default router's answer) |
| 2. Providers send `request.model` | [R3](R3.md) |
| 3. Routers load as plugins | [R6](R6.md) |
| 4. Off = null router; unknown name halts | [R6](R6.md) |
| 5. Tests: MeTTa and Python unit tests | [R7](R7.md) (router tests), plus each ticket's own unit tests |
| 5. Mock reports the model, plus autotests | [R4](R4.md), not built yet |
| 6. Small fixes | [R7](R7.md) (keywords, `< 2` rule), [R3](R3.md) (`__repr__`, unused imports), [R6](R6.md) (docs) |
| `stop()` fix, for `main` | [R1](R1.md) |
| Catalogue (beyond the PR) | [R5](R5.md) |
| Architecture tests (beyond the PR) | [R2](R2.md) |
| `openai.py` cache-key fix (beyond the PR) | [R3](R3.md) (evidence via R3-T4) |

**"Unit only"** (R5, R6, R7): those tickets' rendered, end-to-end tests run the real agent loop on the mock provider. They need R4's mock verb, which isn't built yet.
