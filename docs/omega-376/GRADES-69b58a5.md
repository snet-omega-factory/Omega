# QA step 4: per-ticket grades, review head 69b58a5 (bundle 1232Z)

snet-omega-qa-engineer, 2026-10-12 ~13:20Z.
- **Venue:** fork github.com/snet-omega-factory/Omega. Each branch runs its own workflows, unchanged (spec §15).
- **Evidence classes:**
  - **CI** = observed in a fork run, at its head sha;
  - **shim** = QA's own re-run in the engineer's shim with named stubs;
  - **read** = static review.
- **The ticket's bar:** *pass* = every T observed green on the stated sha, with output.

## Runs (all checked: head_sha == bundle sha)
| ref | sha | run | result |
|---|---|---|---|
| omega-376-llm-routing-inc1 (review head) | 69b58a5 | 38250418425 | **success**. MeTTa 8/8 files OK, 0 ❌. Unit pytest **171 passed, 1 skipped** (test_embedding_models: no import_knowledge, not ours). Autotests Phase 1 **136 passed**. Phase 2: collection error, see "pre-existing" below |
| omega-376-r2-t3-violation-red-2 | 27a3a90 | 38250421964 | **failure, the right one**. router_default OK. pytest 2 failed, 169 passed, 1 skipped. The 2 items name exactly the 4 violations |
| omega-376-r2-t3-violation-2 | 1091762 | 38250425736 | **success**. Tree == 69b58a5. Same counts as inc1 |
| omega-376-r1-only-2 | 73c4d15 | 38250429034 | **success**. test_lib_llm_ext 3/3. Unit 122 passed, 2 skipped |
| omega-376-b1-only | 8961739 | 38247145144 | **success**. Builds only (B16) |

Superseded and recorded for history: 7b69735 / 6d67d29 / 6c5bbba / c80ae61 runs 38247126417 / 32534 / 37275 / 41587. They were red: the R7 PeTTa type error, and the circular import in the test harness. The diagnostic run qa/ci-split-steps @921773b (38249780255) is never a grading venue.

## Per ticket at 69b58a5
| Ticket | Grade | Evidence |
|---|---|---|
| **R1** | **pass** | test_lib_llm_ext.py 3/3, CI. Red-first on the bug (AttributeError on stop()) is **shim** (QA re-run at e8e8422 and 193e9f58); no fork ref carries the unfixed tree. FR1–FR3 read; FR3 per addendum 1 |
| **R1-only** 73c4d15 | **pass** | 3/3, CI on main's workflows. The B14 condition holds: synthetic `delegate._client` and `getattr(delegate,"_client")` are flagged, plus the docstring. It differs from c80ae61 only by the openai stand-in |
| **R2** | **pass** | test_architecture_rules.py 10/10, CI. **R2-T3's three states are observed in CI:** clean 69b58a5 green; violation 27a3a90 with exactly 2 failing items naming all 4 lines (`routers/default.metta:83 check1-metta-py py-call`, `routers/violation.py:3 check1-py-env environ`, `providers/asione.py:53 check2a-attr request.router`, `providers/lib_llm_ext.py:111 check2b-word premium`); revert 1091762 green with an identical tree. B15 is observed (the message lists every violation). FR5 read |
| **R3** | **pass** | test_providers_effective_model.py 24/24, CI. QA regenerated the R3-T2 snapshot from c193ab3 and it is identical. B1 is green in CI via R3-T4, and its bite is shown in shim (red with 61524d5 reverted). B2 read |
| **B1-only** 8961739 | builds; port read, not run, vs 17cc2223; no coverage on this base | B16 |
| **R4** | **not built** | QA's ticket. Building it is the next sub-task |
| **R5** | T1, T2 **pass**; T3, T4 **not built** (R4) | test_providers_catalogue.py 25/25, CI. QA planted defects in shim and they turned red |
| **R6** | T1, T2 **pass**; T3, T5 **not built** (R4); T4 **read** | src_router.metta 6 ✅ and test_providers_router.py 23/23, both CI. R6-FR1 read: no router name in src/. B6: no_router is gone. B11 read: stock config gives a null router |
| **R7** | T1, T2, T3, T5, T6 **pass**; T4 **not built** (R4); the T3 halt route **not observed** | router_default.metta **10 ✅** (the B12 count), test_router_default.py 7/7 and test_src_providers.py 4/4, all CI. **37e0ad0 (strings → symbols) is accepted by QA**; its premise is listed for R4 below. The T5 bite is in shim |
| **Default-off (invariant 7)** | **run, CI** | The existing Autotests Phase 1 suite, 136 passed, on the stock config (router unset, so the null router) at 69b58a5 |
| **B12** | premises (2), (3) asserted in CI; premise (1) **not asserted** | Premise (1), the variable-head call, needs R4's rendered runs |

**Not built (R4), so no evidence yet:** A2–A5 rendered, A7, R5-T3/T4, R6-T3/T5, R7-T4, B12 premise (1), the R7-T3 halt route. Also nothing yet shows the hook → router → Python round trip with symbols (37e0ad0's premise). No log line names the model the router chose.

## Findings, none blocking a grade
1. **Check 1 has undocumented escapes:** `from os import environ as e`, an aliased getenv, `importlib.import_module`, `__import__`. §8 names only the src/ helpers limit. **Architect: close them or document them.**
2. **R2-FR4 / R2-T2:** the scanned-file list and the "vacuous" line are `print`s, which pytest hides for passing tests. They never appear in a green CI log. The empty-list assertion still enforces non-vacuity.
3. Check 2a misses `Optional[LLMRequest]`, `vars(args)`, `builtins.getattr` and a keyword-only `chat` parameter; R1-T3 misses `builtins.getattr`. Low.
4. **Default router plugin loads at every start:** a malformed `router_escalate_keywords` can halt an agent even with routing off. **Architect: is that intended?**
5. Keywords given as a YAML list would carry quote marks and silently never match. Read only.
6. `router_escalate_items` from env or the command line arrives as a string, a named gap. The env=4 scenario proposal: a QA-owned workflow on a fork-only branch. It's pending the architect's OK and needs R4's harness to observe "never escalates".
7. R3: dead `openai.py` `log_usage_statistics` still logs `self._model_name`. B1's fix sends the computed key when `OPENAI_PROMPT_CACHE_KEY=""`, which used to send `""`; that's unstated. R3-T2's configured models equal the plugin defaults, which is weaker. Base `LLMProvider.model()` caches `model=None`, an ordering hazard.
8. R5-T1's "other providers' `*_model` keys excluded" can't fail at startup; a planted leak stayed green. Ask for one startup case, or rule it a named gap.
9. No CI run names skipped tests: the project's pytest runs without `-rs`. That's a later CI gap, out of scope (architect).

## Pre-existing, not the increment's
- Autotests Phase 2 (`continue-on-error`) errors at collection on `Autotests/unit/test_llm_budget.py`: `NameError: name 'LLMToolCall' is not defined` at providers/lib_llm_ext.py:66. **Upstream's own CI at 193e9f58 shows the same error** (singnet/Omega job 110305200117). As a result **none of Phase 2's 6 items ran**, including `test_git_push_to_remote_mock`. That test is **not run**, not a skip and not a pass.
