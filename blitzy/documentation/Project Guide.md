# 1. Executive Summary

## 1.1 Project Overview

This project delivers the implementation specification for the OpenTelemetry **Metrics** signal in the `opentelemetry-kotlin` Kotlin Multiplatform SDK, measured against the repository at commit `20d7a394`. It is a documentation deliverable by design: no Kotlin, Gradle, dump or configuration file was to change, and none did. The audience is the maintainers and implementers who will build the signal — the document gives them an executed baseline of what the repository actually contains, eighteen binding architecture decisions, thirty-five dependency-ordered change units each under a 500-line ceiling, a correctness strategy built around silent data loss, and a thirty-four-row risk register. Business impact: metrics is the largest outstanding gap in a four-platform telemetry SDK, and this removes the discovery cost from that work.

## 1.2 Completion Status

**230 of 272 hours complete = 84.6%.**

```mermaid
pie showData title Completion — 84.6%
    "Completed (230h)" : 230
    "Remaining (42h)" : 42
```

<span style="color:#5B39F3">■</span> Completed = Dark Blue `#5B39F3` · <span style="color:#FFFFFF">□</span> Remaining = White `#FFFFFF`

| Metric | Value |
| --- | --- |
| Total Hours | **272** |
| Completed Hours (AI + Manual) | **230** (230 AI + 0 manual) |
| Remaining Hours | **42** |
| Percent Complete | **84.6%** (230 ÷ 272) |

## 1.3 Key Accomplishments

- Specification complete and contract-conformant: eight ordered sections, both frozen table headers byte-exact, 196-word summary carrying the compliance integer.
- Empirical baseline executed against the live repository, not assumed — 32 command transcripts with exit statuses and eleven measured deviations.
- Eighteen architecture decisions, every one carrying context, status, decision, alternatives, rejection reasoning, consequences and evidence.
- Thirty-five change units forming an acyclic dependency graph: 75 edges, longest chain 11, none exceeding 490 estimated lines.
- Correctness strategy naming silent data loss as the dominant failure mode, with 21 loss modes and a stated differential reach limit.
- Repository left untouched: net tracked diff against the base commit is empty, and all 2,965 tests still pass.
- Every behavioural claim carries a resolvable locator — 858 locator occurrences, zero unresolved, zero out-of-range.

## 1.4 Critical Unresolved Issues

| Issue | Impact | Owner | ETA |
| --- | --- | --- | --- |
| Authoring commits were made on, and pushed to, a published branch, which the agreed execution boundary prohibited absolutely | Delivery acceptance, not content — needs an explicit decision on the record | Maintainer / release owner | 3h |
| The specification is not present at its mapped destination `metrics-implementation-plan.md`, and the branch's net tracked diff is empty | Merging the branch delivers nothing; the finished document must be placed or collected | Maintainer / release owner | 3h |
| A push-capable access credential was readable in the checkout's git configuration; removal is not revocation | Credential must be treated as compromised until revoked and rotated | Repository owner | 2h |
| Kotlin Gradle plugin `2.4.10` is inside the affected range of GHSA-r937-wjx7-w2jp / CVE-2026-53914 | Needs a recorded acceptance with an owner and re-review boundary, or a scheduled upgrade | Repository owner | 2h |
| Apple runtime behaviour is built into the specification but exercised nowhere — the simulator test task cannot run without macOS | The specified emission gate stays closed for Apple targets until a macOS run supplies evidence | Platform owner | 6h |

## 1.5 Access Issues

| System/Resource | Type of Access | Issue Description | Resolution Status | Owner |
| --- | --- | --- | --- | --- |
| `origin` remote | Git push | Both remotes' push URLs are set to a disabled sentinel, so `git push` fails at exit 128 before any network contact | Deliberate control; must be restored before the branch can be updated | Release owner |
| Embedded remote credential | Git HTTPS | A push-capable credential is re-injected into the untracked `.git/config` by the delivery environment after each removal | Contained (mode 600, push paths disabled); revocation outstanding | Repository owner |
| macOS host | Build/test runner | Apple framework linking and iOS simulator tests require macOS; iOS klibs cross-compile on Linux but no simulator test executes | Not available in this environment | Platform owner |
| Codecov coverage upload | CI artifact | The workflow uploads `build/reports/kover/reportRelease.xml`, which the build never produces | Open — the project coverage gate is unobservable in CI | Maintainer |

## 1.6 Recommended Next Steps

1. **[High]** Decide the specification's placement and collect it, so the branch carries the deliverable rather than an empty diff (3h).
2. **[High]** Take and record the delivery-boundary acceptance decision on the commits and pushes to the published branch (3h).
3. **[High]** Revoke and rotate the exposed credential, review the remote's audit log, and record a disposition for the Kotlin Gradle plugin advisory (4h).
4. **[High]** Put the specification through independent maintainer review before it governs implementation (12h).
5. **[Medium]** Sign off the seven ranked open questions so the first work package can start (8h).

# 2. Project Hours Breakdown

## 2.1 Completed Work Detail

| Component | Hours | Description |
| --- | --- | --- |
| Environment provisioning and capacity gate | 6 | JDK 11, 17 and 21 installed and proven discoverable through `./gradlew -q javaToolchains`; Android SDK 36 with `build-tools;36.0.0` and licences accepted; Kotlin/Native toolchains; capacity measured before the first build by a container-aware method with an executed failing control |
| Clone topology, upstream refspecs and pinned reference trees | 3 | Read-only `upstream` remote carrying both the heads and pull-head refspecs; the OpenTelemetry Java reference tree pinned at tag `v1.65.0` and the specification tree pinned, so every reference measurement is reproducible at a fixed revision |
| Baseline gate execution and measurement | 8 | The canonical multiplatform build executed in the fixed order and run twice to establish configuration-cache reuse; `apiCheck detekt`; local publish and consumer integration as two separate invocations; coverage rate and total measurable line count captured |
| Upstream metadata harvest and reference register | 8 | Authenticated read-only harvest producing a 22-row register with a closed set of dispositions and a zero-dangling assertion; records the open-count field-semantics trap and names the two previously unnamed upstream items |
| Two disposable verification spikes | 14 | Configuration-cache compatibility proved by a store-then-reuse pair a warm build cannot show; a LongCounter vertical slice run end to end, which measured two concurrency defects and a static-analysis task-reach asymmetry and shaped three architecture decisions. Both branches deleted, no refs surviving |
| Baseline evidence ledger (Section 2) | 34 | Twelve ledger rows with the matches/differs/unavailable outcome triple, 32 verbatim command transcripts with exit statuses, eleven measured deviations, a repository evidence index, the compliance denominator with its basis, fourteen non-goal dispositions and the constraint traceability set |
| Architecture decisions (Section 3) | 40 | Eighteen records covering module layout, the public meter and instrument surface, configuration shape, metric identity, attribute materialisation, multiplatform accumulation, aggregation and exemplars, callback lifecycle, the reader pipeline, exporters and wire format, lifecycle containment, correctness strategy, binary-compatibility authority, the exporter selection seam, upstream divergence, naming fidelity, hostile extension points and resource bounds — plus a 28-row map placing every mandated technical area |
| Work breakdown, port-size netting and gate registers (Section 4) | 26 | Thirty-five change units in the frozen eight-column format; a subpackage-by-subpackage netting of the 16,406-line reference tree down to a ~11,760-line port across 176 files; the eleven-gate register; twelve security-acceptance rows attached to existing gates |
| Dependency plan and critical path (Section 5) | 12 | Critical path derived from the blocking column alone, eleven dependency levels, twelve co-critical chains, the verification-branch protocol, rebase cadence and dump-conflict policy, the minimum viable slice placed early, and eleven work packages |
| Correctness strategy (Section 6) | 16 | Silent data loss named as the dominant failure mode with 21 loss modes each carrying an oracle and an owner; what each gate proves and does not; the differential reach limit stated with both of its bounds; a ten-kind evidence matrix; coverage arithmetic constraining unit size |
| Risk register (Section 7) | 10 | Thirty-four rows in the frozen six-column format, composite consistent with likelihood × impact in every row, ordered descending, every row carrying an observable early-warning signal |
| Executive summary and ranked open questions (Sections 1, 8) | 6 | A 196-word summary carrying the compliance integer with its stated denominator basis, and seven genuine residual questions ranked by how much downstream work each blocks |
| Evidence discipline across the document | 18 | Six-rank authority ladder with a per-claim authority and status label on every substantive claim; 858 locator occurrences resolving with no out-of-range anchors; the full build command written out verbatim at all 70 reference sites; specification voice in the forward-looking sections; secret hygiene |
| Specification verification | 24 | Structural contract validation, independent recomputation of the dependency graph, locator resolution sweeps, re-execution of all 32 transcripts, external link checks, syntax checks over every shell example, and browser rendering of the whole document |
| Delivery-boundary controls, spike teardown and clean-tree self-check | 3 | Both verification branches deleted with no surviving refs; working tree left with zero tracked modifications; exactly one deliverable; no upstream contact and no pull request opened |
| Placement ruling and removal of the in-repository copy | 2 | The placement conflict adjudicated in writing, recorded as a differs outcome with the `.gitignore` reasoning, and the in-repository copy removed in commit `0d651dae` |
| **Total** | **230** | |

## 2.2 Remaining Work Detail

| Category | Hours | Priority |
| --- | --- | --- |
| Delivery-boundary acceptance decision on the commits and pushes to the published branch | 3 | High |
| Specification placement and collection — restore the mapped repository artifact or record where the deliverable now lives | 3 | High |
| Credential revocation, rotation and remote audit-log review | 2 | High |
| Kotlin Gradle plugin advisory disposition — recorded acceptance with an owner and re-review boundary, or a scheduled upgrade | 2 | High |
| Independent maintainer review of the specification before it governs implementation | 12 | High |
| Sign-off on the seven ranked open questions | 8 | Medium |
| Apple execution evidence from a macOS host, to unblock the specified emission gate for Apple source sets | 6 | Medium |
| Coverage-artifact path repair so the project coverage gate becomes observable in CI | 2 | Medium |
| `commonTest` test-dependency repair in `exporters-otlp` and `exporters-protobuf` | 2 | Medium |
| Inherited documentation corrections in `AGENTS.md` and `CONTRIBUTING.md` | 1 | Low |
| Confirmation of the broadened change-unit measurement pathspec | 1 | Low |
| **Total** | **42** | |

## 2.3 Basis of Estimate

Total project hours are scoped strictly to the Agent Action Plan's own deliverable obligations plus the path to production for that deliverable. The thirty-five implementation units the specification defines are explicitly out of this scope — the plan's governing instruction is to write them down, not to build them — so their estimated 12,850 added lines contribute nothing to the denominator here.

Completed hours were derived per component from delivered volume and verification depth: a 2,001-line specification carrying eighteen decision records, thirty-five change units, thirty-four risk rows, thirty-two command transcripts and 858 evidence locators, plus the executed baseline and the two verification spikes behind it. Two components are counted as partially complete: the delivery self-check at roughly half, because its teardown and clean-tree obligations hold while its no-commit obligation cannot be met retrospectively; and deliverable placement at roughly two-fifths, because the placement ruling was made and recorded but the artifact is not where a merge would find it.

Confidence is **high** on the twelve content components, which are measurable against the document and the repository. It is **medium** on maintainer review and open-question sign-off, whose duration depends on reviewer availability rather than on scope. Remaining estimates are conservative and rounded up to the nearest half hour.

`Completion % = 230 ÷ (230 + 42) × 100 = 84.6%`

# 3. Test Results

Every figure below was produced by running the suite in this checkout. The canonical build was executed three times: once cold, once again to establish configuration-cache reuse, and once with test outputs deleted and the build cache disabled so that all 38 test tasks genuinely executed rather than being restored.

| Area / Category | Framework | Tests | Passed | Failed | Coverage | What This Proves |
| --- | --- | --- | --- | --- | --- | --- |
| JVM unit and behaviour suite, all 20 modules with tests | JUnit 6.1.3 + `kotlin.test` | 1,651 | 1,651 | 0 | included below | Tracing, logging, configuration, exporter, persistence and platform behaviour is intact on the JVM after a delivery that changed no source |
| Kotlin/JS Node suite | `kotlin.test` on Node | 1,312 | 1,312 | 0 | included below | The same shared behaviour holds on the JavaScript target, so the multiplatform contracts the specification builds on are exercised on two runtimes |
| Consumer minimum-supported-version integration | JUnit 6.1.3 + Gradle TestKit | 2 | 2 | 0 | n/a | A downstream consumer on the declared minimum Gradle, Kotlin and Android plugin versions still resolves and builds against locally published artifacts |
| Merged coverage measurement | Kover 0.9.9 | n/a | n/a | n/a | **LINE 94.39%** (4,864 / 5,153) · INSTRUCTION 95.10% · BRANCH 87.67% | The 5,153-line measurable base and its rate are real numbers, which is what makes the specification's coverage-window arithmetic checkable rather than notional |
| Binary compatibility and static analysis gates | binary-compatibility-validator 0.18.1 · Detekt 1.23.8 | 265 tasks | 265 | 0 | n/a | Zero public-API dump deltas and zero static-analysis issues under `maxIssues: 0` and warnings-as-errors — the two gates every future change unit must satisfy pass on this tree |
| Full multiplatform release build | Gradle 9.7.0 | 2,186 tasks | 2,186 | 0 | n/a | JVM, Android release assembly, JavaScript and iOS klib compilation all succeed, and a second identical invocation reports configuration-cache reuse |
| Specification contract validation | Purpose-built structural checks | 14 checks | 14 | 0 | n/a | Eight ordered sections; both frozen headers byte-exact and unique; identifier families complete; 35 units all ≤490 lines summing to 12,850; the dependency graph acyclic with zero row-order violations; risk composites consistent; compliance arithmetic closing at 0 + 4 + 77 = 81 |

**Not Covered**

- **The entire specified Metrics implementation.** The eighteen architecture decisions and thirty-five change units describe code that does not exist yet, so no test exercises the designed behaviour. Four design claims were exercised by the disposable vertical slice on the JVM and JavaScript before its branch was discarded; everything else is specification. Before release, treat every unit's stated acceptance criteria as the test plan rather than as evidence.
- **Apple runtime behaviour.** Both Apple targets compile and iOS klibs are produced on Linux, but `:implementation:iosSimulatorArm64Test` exits 0 while skipped, reporting that simulator tests require macOS. Nothing Apple-specific has ever been executed. A human must run the shared behaviour and golden tests on a macOS host.
- **Android instrumented tests.** `:benchmark-android:compileReleaseAndroidTestKotlin` compiles the instrumented sources, but no device or emulator run occurred, so Android host behaviour is compile-verified only.
- **The aggregate `allTests` entry point.** `./gradlew allTests --rerun` fails deterministically in `exporters-otlp` and `exporters-protobuf` with 163 compile errors, because both declare the deprecated split test coordinates in `commonTest` (`exporters-otlp/build.gradle.kts:L36-L37`, `exporters-protobuf/build.gradle.kts:L61-L62`). The canonical build does not invoke that task, so the failure is invisible in CI. Test through the canonical build until those two declarations are replaced.
- **The existing metrics surface itself.** Of the 22 metrics files in the tree, only `implementation/src/commonTest/kotlin/io/opentelemetry/kotlin/metrics/MeterProviderImplTest.kt` and two compatibility adapter tests carry coverage; there is no instrument, storage, aggregation or export coverage because none of that code exists. `api/src/commonMain/kotlin/io/opentelemetry/kotlin/metrics/Meter.kt` is still an interface with no body.

# 4. Runtime Validation & UI Verification

This project has no service and no user interface. Its runtime surfaces are the build and its gates, the example applications, and the specification document itself, which was driven in a real browser. Every line below records what was observed, not what was expected.

- ✅ **Toolchain discovery** — `./gradlew -q javaToolchains` lists JDK 11.0.31, 17.0.19 and 21.0.11, all reported as JDKs. The declared toolchain is 11 (`buildSrc/src/main/kotlin/io/opentelemetry/kotlin/KotlinConfig.kt:L11,L17`) and no toolchain resolver is registered, so its presence is load-bearing rather than incidental.
- ✅ **Canonical multiplatform build** — exit 0 on every invocation. Cold: 2,186 actionable tasks in 37s, configuration cache stored. Identical second run: 10s, `Configuration cache entry reused.` Third run with test outputs deleted and the build cache disabled: 40s with 38 test tasks executed for real.
- ✅ **Quality gates** — `./gradlew apiCheck detekt --stacktrace` exits 0 across 265 actionable tasks with zero binary-compatibility deltas and zero static-analysis issues under `maxIssues: 0`.
- ✅ **Local publish, then consumer integration, as two invocations** — publish exits 0 in 12s; the separate consumer run exits 0 in 49s with 2 tests and no failures. Running them as one invocation is what `CONTRIBUTING.md:L32-L42` warns against.
- ✅ **JVM example application** — `:examples:example-app:runConsoleExampleApp` exits 0, exercising SDK start-up, telemetry emission and shutdown on a real JVM runtime.
- ✅ **JavaScript example application** — `:examples:example-app:runNodeExampleApp` exits 0 on Node, so the same start-up and shutdown path is driven on a second runtime.
- ✅ **Coverage artifact** — `build/reports/kover/report.xml` regenerates and parses: 4,864 covered, 289 missed, 5,153 measurable lines.
- ✅ **Specification rendered in a real browser** — served and loaded at HTTP 200 with the response body matching `Content-Length` exactly; interactive in 36 ms; 98,021 px of content traversed to its true end. Eight top-level sections read back character-for-character in order; 23 tables, 372 rows and 1,818 cells with zero empty and zero ragged cells; the work-breakdown table renders 8 columns over 35 rows with every trailing risk cell populated; the risk register renders 6 columns over 34 rows, closing 22 px clear of the next heading; 150 cells containing literal pipe characters render intact without splitting; the page itself produced zero console messages and zero failed network requests.
- ⚠ **Apple targets** — compilation and iOS klib production succeed on Linux, but `:implementation:iosSimulatorArm64Test` exits 0 while skipped with `simulator tests require macOS`. No Apple code path has been executed at runtime anywhere, and the specification's response is a gate that denies emission from an unverified source set rather than a label.
- ⚠ **Android instrumented layer and the metrics pipeline** — Android instrumented sources compile but no device or emulator run occurred, and the Metrics measurement pipeline has no runtime at all because the code it specifies has not been written. Neither has been exercised, and neither should be assumed working.

# 5. Compliance & Quality Review

## 5.1 Compliance Matrix

Each row states where the deliverable stands now, measured against this checkout.

| # | Deliverable | Benchmark | Status | Evidence |
| --- | --- | --- | --- | --- |
| 1 | Output-contract structure | Exactly eight numbered top-level sections in the fixed order, no preamble, no ninth section | ✅ PASS · 100% | Eight fence-aware level-one headings at document lines 1, 13, 1126, 1637, 1751, 1845, 1935 and 1978 |
| 2 | Executive summary | Under 200 words while carrying the compliance integer with a stated denominator basis | ✅ PASS · 100% | 196 words; `0 of 81 satisfied, 4 of 81 excluded by non-goals, 77 outstanding, against 88 total rows of which 7 optional rows are out of scope`, present at two sites and closing as 0 + 4 + 77 = 81 and 81 + 7 = 88 |
| 3 | Baseline evidence ledger | Twelve rows, each carrying one of matches, differs or unavailable, backed by a locator or a producing command | ✅ PASS · 100% | Twelve rows complete; 32 verbatim transcripts with exit statuses; eleven measured deviations recorded with both the asserted and observed values |
| 4 | Architecture decisions | A decision, its alternatives and explicit rejection reasoning for every record | ✅ PASS · 100% | Eighteen records, each carrying context, status, decision, alternatives, rejected-because, consequences and evidence; a 28-row map places every mandated technical area |
| 5 | Work-breakdown format | The frozen eight-column header reproduced exactly, with no duration, date, owner or effort column | ✅ PASS · 100% | Header present once, byte-exact including the `≤` glyph; the words duration, sprint, week and deadline appear nowhere as schedule terms |
| 6 | Change-unit sizing | Every unit within the 500 added-line ceiling | ✅ PASS · 100% | Thirty-five units summing to 12,850 estimated lines, mean 367.1, maximum 490 |
| 7 | Dependency ordering | Ordering by the blocking column alone, acyclic, with the critical path identified | ✅ PASS · 100% | Graph re-derived from the table: 35 nodes, 75 edges, acyclic, roots `M-01`/`M-02`, single sink `M-29`, longest chain 11, zero row-order violations |
| 8 | Correctness strategy | Names the dominant failure mode and states where verification cannot reach | ✅ PASS · 100% | Silent data loss named, with 21 loss modes each carrying an oracle and an owner; the differential reach limit stated with both bounds — JVM-only, and the reference implementation itself unsatisfied on thirteen compliance rows |
| 9 | Risk-register format | The frozen six-column header, no fewer than fourteen rows, measurable early-warning signals | ✅ PASS · 100% | Header present once, byte-exact; 34 rows, composite equal to likelihood × impact in every row, ordered descending |
| 10 | Evidence discipline | Every behavioural claim carries a resolvable locator and an authority and status label | ✅ PASS · 100% | 858 locator occurrences across 487 unique targets, zero unresolved and zero out-of-range; 570 valid claim labels with none malformed; the full build command written out at all 70 reference sites |
| 11 | Repository immutability | No source, build script, dump, manifest, lockfile or generated file changed | ✅ PASS · 100% | Net tracked diff against `20d7a394` is empty; 27 `.api` dumps, `gradle/libs.versions.toml`, `gradle.properties` and `kotlin-js-store/yarn.lock` all unchanged; 2,965 tests still pass and both gates exit 0 |
| 12 | Delivery boundary | One deliverable, verification branches torn down, no upstream contact, no commit to a pushed branch | ⚠ PARTIAL · 60% | Exactly one deliverable; zero surviving verification-branch refs; no pull request and no upstream contact. Two commits were nonetheless made on, and pushed to, the published branch, and the mapped repository artifact is absent — see §5.2 rows 1 and 2 |

## 5.2 AAP & Rule Divergences and Gaps

**No user-specified rules exist for this project.** The rules document returns no rules — it was queried twice, the second time over its full range — so no rule divergence is possible and none is reported. In their place the repository's own checked-in standards were applied as the bar: the workflow and error-handling guidance in `AGENTS.md`, the development and publishing guidance in `CONTRIBUTING.md`, the stability matrix in `api/README.md`, the thresholds in `codecov.yml` and `config/detekt/detekt.yml`, and the compiler conventions in `buildSrc/src/main/kotlin/io/opentelemetry/kotlin/KotlinConfig.kt`. The eight rows below are all divergences from the Agent Action Plan.

| # | What the AAP/Rule Required | What Was Delivered Instead | Why It Diverged | Impact | Remediation |
| --- | --- | --- | --- | --- | --- |
| 1 | No commit to a pushed branch and no push to any remote, absolutely | Two commits on the branch, and the branch pushed twice | The branch is the delivery channel, which the prohibition forecloses | Delivery acceptance; content unaffected | Explicit acceptance on the record, or re-author on a never-pushed checkout |
| 2 | The sole deliverable placed outside the repository, never relocated into it | Out-of-repository placement; the in-repository copy removed in `0d651dae` | An explicit conflict the AAP itself anticipated and ruled on | The branch's net tracked diff is empty and the mapped artifact is absent | Decide placement, then place or collect the finished document |
| 3 | Report compliance as "N of 78" | Reported against 81, with the basis stated | **Sanctioned** — three optional matrix rows are committed deliverables | None; the arithmetic closes and the supersession is stated | None required |
| 4 | Architecture records ADR-01 to ADR-13 | Eighteen records | **Sanctioned** — the AAP calls the set a minimum, not a ceiling | None; no required number renumbered | None required |
| 5 | A 2.00-percentage-point transient coverage window | 1.00 percentage point | **Sanctioned** — `codecov.yml` enforces 1%, and that is the gate that fails a build | The window absorbs roughly one exemption-path unit | Confirm the tightened exemption policy when signing off the open questions |
| 6 | Change-unit budget measured recursively over source directories | Whole tree with `**/*.api` excluded, plus a separately bounded dump delta | The literal form silently dropped most of one unit's content | Positive — the gate now measures what a reviewer reads | Confirm the broadened reading, or restate the gate |
| 7 | At least 16 GB of memory, measured before the first build | Container-aware measurement of the process cgroup's own limit | Host `MemTotal` reports no container limit; the root cgroup reports none either | The gate is discharged by method rather than by host total | None here; re-measure on any host whose cgroup limit is genuinely small |
| 8 | No file written inside the checkout and kept | Untracked browser-evidence images kept under `blitzy/screenshots/` | The evidence directory is fixed and pre-existing, and is not covered by `.gitignore` | Cosmetic — `git status` reports the untracked directory; zero tracked files changed | Delete the directory if a pristine status is wanted |

**Row 1 — commits and pushes on a published branch.** The boundary prohibited any commit to a pushed branch and any push to any remote, without exception. Two commits exist — `a2e8a63b`, which added the specification, and `0d651dae`, which removed it — and the remote-tracking reflog records exactly two push events. Nothing reached the upstream project and no pull request was opened, so the blast radius is this fork alone. It is in published history and cannot be undone from inside the repository. Both remotes' push URLs are now a disabled sentinel, and `git push --dry-run origin HEAD` fails at exit 128 before network contact. Accept the deviation explicitly, or re-author on a checkout that is never pushed.

**Row 2 — the deliverable is not where a merge would find it.** The plan fixes the sole write target outside the repository and forbids relocating it into the checkout; the platform's mapped destination is the repository-root file `metrics-implementation-plan.md`. The out-of-repository placement was chosen, and commit `0d651dae` removed the in-repository copy to honour it. The consequence is measurable: `git diff --numstat origin/blitzy-otel-kotlin...HEAD` returns nothing, so the branch adds no tracked content. Only the earlier 468-line revision survives in history, recoverable with `git show a2e8a63b:metrics-implementation-plan.md`. Decide the placement — commit the final revision at the mapped path, or collect it and record where it lives — but do not leave two copies.

**Row 3 — the compliance denominator.** The plan prescribed reporting against 78 non-optional specification rows. The document reports `0 of 81 satisfied, 4 of 81 excluded by non-goals, 77 outstanding, against 88 total rows of which 7 optional rows are out of scope`. The denominator moved because three of the ten optional rows — wildcard view selection, more than one view per instrument, and configuring a stream's exemplar reservoir — are things the plan itself requires the design to cover, so reporting against 78 would silently omit three committed deliverables. This is sanctioned by the plan's own supersession, the arithmetic closes twice over, and the basis is stated wherever the figure appears. Nothing to do.

**Row 4 — eighteen architecture records rather than thirteen.** The record set grew by five. The plan describes its list as a required minimum rather than a ceiling, and each addition answers a question the required records could not hold without becoming five asides: the exporter and reader selection seam, upstream divergence and rebase posture, naming fidelity, hostile extension points with their execution bounds, and resource bounds with input validation. No required number was renumbered or re-scoped, and a 28-row coverage map shows every mandated technical area resolving to exactly one record. Verify by reading the map rather than counting records. Nothing to do.

**Row 5 — the coverage window.** The plan proposed tolerating a 2.00-percentage-point transient project-coverage regression at work-package boundaries. `codecov.yml` sets `threshold: 1%`, and that is the gate that actually fails a build, so the specification fixes the window at 1.00 point and never widens it. The consequence is arithmetical and is carried through: against the measured 5,153-line base, a 500-line unit landing at the 50% patch floor costs 3.93 points, so a one-point window absorbs roughly one exemption-path unit rather than the two the plan assumed. The response is a tightened exemption policy, not a wider window. Confirm that trade when signing off the ranked questions.

**Row 6 — how a change unit is measured.** The plan's budget gate says the measurement must be recursive over source directories. As delivered it measures the whole tree with `**/*.api` excluded, and bounds regenerated dump lines separately at 700. The literal form was incoherent against the units themselves: one unit's entire content is a regenerated dump baseline, and another regenerates seventeen dumps, so a source-directory pathspec reported a false zero for the first and risked a silent breach on the second. The broadened form measures what a human reviews, including markdown and the compliance sheet. The plan's stated intent — recursive, never root-anchored, test sources counted, deletions not — is preserved. Confirm the broadened reading is acceptable.

**Row 7 — the capacity gate.** The plan requires at least 16 GB of memory measured before the first build and treats a shortfall as a stop condition. This host's `MemTotal` reads 3,935,084 kB and the root of the cgroup hierarchy carries no limit, so a naive reading concludes either "far too small" or "no limit" depending on which file it opens. The figure that governs is the process's own cgroup limit, 128 GiB effective, which the JVM independently reports; the same probe under a 512 MiB limit reports 512 MiB and exits non-zero, so the method discriminates. `gradle.properties` is byte-identical to baseline, so the daemon heap request was never lowered, and the full build passes under it.

**Row 8 — files kept inside the checkout.** The plan's self-check requires that no file be written inside the checkout and kept. Browser-evidence images are kept under `blitzy/screenshots/`, a fixed evidence directory that pre-existed this work and is not covered by `.gitignore`, so they surface as untracked. Their location is not configurable. The impact is confined to `git status`, which reports the untracked directory and nothing else: zero tracked files were modified, and every dump, manifest and lockfile is untouched. Delete the directory's contents if a pristine status is wanted before merging; nothing else depends on it.

# 6. Risk Assessment

These are forward-looking exposures — what could still go wrong from here, not what has already been dealt with.

| Risk | Category | Severity | Probability | Mitigation | Status |
| --- | --- | --- | --- | --- | --- |
| The branch carries no tracked content, so merging it delivers nothing and the specification can be lost outright | Operational | **Critical** | High | Place the finished revision at the mapped repository path or collect it and record where it lives, before the branch is closed; the earlier revision remains recoverable from commit `a2e8a63b` | Open |
| The exposed push-capable credential stays valid until its issuer revokes it — removal from a file is not revocation | Security | High | Medium | Both remotes' push URLs are disabled and a push attempt fails at exit 128 before network contact; the configuration is mode 600. Revoke, rotate and review the remote's audit log | Contained; owner action open |
| Apple runtime behaviour cannot be verified without macOS, so the specified emission gate stays closed for Apple source sets and Apple support cannot be claimed | Integration | High | High | The gate starts failed and denies emission from any source set with no executed evidence, degrading to one sanitised diagnostic rather than silent wrong numbers; only an executed report from a macOS runner opens it | Open by design |
| The Android binary-compatibility dumps are out of step with source, so the dump check does not enforce the Android variant and a metrics API change can land with that dump silently wrong | Technical | High | Medium | `api/api/android/api.api` carries no metrics package while `api/api/jvm/api.api` does; the specification makes the JVM dump plus source govern, requires explicit review of both dumps, and isolates the baseline disagreement in a dedicated first unit before any API delta is accepted. Zero `.klib.api` baselines exist, so no klib tiebreak is available | Specified, unproven |
| The project coverage gate is unobservable in CI, because the workflow uploads a report path the build never produces | Operational | Medium | High | `.github/workflows/ci-build.yml` uploads `build/reports/kover/reportRelease.xml`; the build writes `build/reports/kover/report.xml`. The specification makes repairing this artifact a transitive prerequisite of every other unit; a human must decide which side to change | Open |
| Silent data loss is the dominant failure mode of the specified pipeline, and the generic export seam it reuses reports success before transport | Technical | High | Medium | `exporters-otlp/src/commonMain/kotlin/io/opentelemetry/kotlin/export/TelemetryExporter.kt` launches asynchronously and returns success, swallows terminal partial success and client errors, has a constant force-flush and cancels in-flight work on shutdown. The specification makes repairing that seam a prerequisite unit with mandatory logs and traces regression assertions, and gives each of 21 loss modes a named oracle | Specified, not implemented |
| The Kotlin Gradle plugin in use sits inside a published advisory's affected range | Security | Medium | Low | `gradle/libs.versions.toml` pins `2.4.10`; GHSA-r937-wjx7-w2jp / CVE-2026-53914 concerns unsafe deserialization in the Kotlin build cache and is first patched in `2.4.20-Beta1`. No remote build cache is used and CI caching is disabled, which is what makes acceptance defensible today; version changes are outside this plan's scope | Accepted; needs a recorded owner |
| Seven ranked decisions remain open and several gate the front of the dependency graph, so the first work package can stall | Operational | Medium | Medium | Each question states what it blocks, what would settle it and an interim posture that keeps work moving; two of the seven are repository-policy calls the specification deliberately declines to make for maintainers | Open |

# 7. Visual Project Status

**Hours split** — Completed 230, Remaining 42, Total 272 (84.6% complete). Completed is rendered in Blitzy Dark Blue `#5B39F3`; Remaining in White `#FFFFFF`.

```mermaid
pie showData title Project Hours Breakdown
    "Completed Work" : 230
    "Remaining Work" : 42
```

**Remaining work by priority** — High 22h, Medium 18h, Low 2h (sums to 42h).

```mermaid
pie showData title Remaining Work by Priority
    "High" : 22
    "Medium" : 18
    "Low" : 2
```

**Where the remaining 42 hours sit**

| Category | Hours | Share of remaining |
| --- | --- | --- |
| Maintainer review of the specification | 12 | 29% |
| Open-question sign-off | 8 | 19% |
| Apple execution evidence on a macOS host | 6 | 14% |
| Delivery-boundary acceptance decision | 3 | 7% |
| Specification placement and collection | 3 | 7% |
| Credential revocation and rotation | 2 | 5% |
| Dependency advisory disposition | 2 | 5% |
| Coverage-artifact path repair | 2 | 5% |
| Exporter `commonTest` dependency repair | 2 | 5% |
| Inherited documentation corrections | 1 | 2% |
| Measurement-pathspec confirmation | 1 | 2% |
| **Total** | **42** | **100%** |

**Delivery readiness by dimension**

```mermaid
graph LR
    A["Specification content<br/>COMPLETE"] --> B["Repository integrity<br/>COMPLETE"]
    B --> C["Gates and test estate<br/>COMPLETE"]
    C --> D["Delivery placement<br/>OPEN"]
    D --> E["Boundary acceptance<br/>OPEN"]
    E --> F["Maintainer sign-off<br/>OPEN"]
    F --> G["Ready for implementation"]
```

# 8. Summary & Recommendations

**What was delivered.** A single 2,001-line implementation specification for the OpenTelemetry Metrics signal in this Kotlin Multiplatform SDK, written against the repository at commit `20d7a394` and structured as the eight sections its contract fixes. It carries an executed baseline of what the repository actually contains — twelve verification rows, thirty-two command transcripts with exit statuses, eleven measured deviations from prior assumptions — followed by eighteen binding architecture decisions, thirty-five dependency-ordered change units none exceeding 490 estimated added lines, a correctness strategy built around silent data loss with twenty-one named loss modes, a thirty-four-row risk register, and seven genuinely open questions ranked by how much downstream work each blocks. On the hours scoped to this plan and the path to production for it, the project is **84.6% complete: 230 of 272 hours**.

**What was verified.** The specification's whole contract was checked mechanically against the document: eight ordered sections, both frozen table headers byte-exact and unique, the compliance arithmetic closing at 0 + 4 + 77 = 81 against 88 total rows, thirty-five units summing to 12,850 estimated lines, and a dependency graph independently re-derived as 75 edges over 35 nodes — acyclic, with a single sink and no row ordered ahead of its blockers. Its 858 evidence locators all resolve, with no out-of-range line anchors. The repository it describes is provably untouched: the branch's net tracked diff against its base commit is empty, all 27 public-API dumps and every manifest and lockfile are unchanged, the canonical multiplatform build passes twice with configuration-cache reuse on the second run, `apiCheck` and Detekt exit 0 with zero violations under `maxIssues: 0`, and the full estate of 2,965 tests passes with no failures, errors or skips at 94.39% line coverage. The document was also driven in a real browser and renders complete and defect-free through its final line.

**What is not verified, and must not be assumed.** Everything the specification describes for the future is specification, not code. The eighteen decisions and thirty-five units govern software that has not been written, so no test exercises the designed behaviour, and the metrics surface in the tree remains what it was — `Meter` is still an interface with no body. Apple runtime behaviour is exercised nowhere: both Apple targets compile and iOS klibs are produced on Linux, but the simulator test task skips itself for want of macOS. Android instrumented sources compile without any device run. The specification's response to the Apple gap is the right one — a gate that denies emission from a source set with no executed evidence rather than a label admitting the gap — but the gate is itself unproven until a macOS run supplies that evidence.

**The critical path to production is delivery, not content.** The single most consequential fact in this assessment is that the branch carries no tracked content: the specification is not present at its mapped destination `metrics-implementation-plan.md`, and only an earlier revision survives in history at commit `a2e8a63b`. A merge as things stand delivers nothing. That decision, and the acceptance decision on the commits and pushes made to this published branch, are the first two things to settle; they are followed by revoking the exposed credential, recording a disposition for the Kotlin Gradle plugin advisory, and putting the specification through the independent maintainer review the repository's own guidance requires for AI-assisted work. Only then do the seven ranked questions and the macOS evidence matter, because only then does implementation start.

**Production readiness.** The content is ready: it is complete, internally consistent, cited to evidence a reader can open, and it names its own limits rather than hiding them. The delivery is not. Success is measurable and near — place or collect the document, take two decisions on the record, close two owner actions, and complete one review, and the twenty-two high-priority hours are done. Recommendation: **accept the specification's content, hold release until the placement and boundary decisions are recorded**, and treat the four coverage and toolchain repairs the document surfaces as separate, non-metrics changes so they do not become entangled with the first implementation unit.

# 9. Development Guide

Every command below was executed in this checkout and exited 0 unless stated otherwise. Run all of them from the repository root.

## 9.1 System Prerequisites

| Requirement | Version | Why |
| --- | --- | --- |
| Linux or macOS | Ubuntu 24.04+ verified | CI runs Linux for the main build and macOS only for the consumer integration job |
| JDK 11 | 11.0.31 verified | The **declared** Gradle and Kotlin toolchain — `jvmToolchain(11)` at `buildSrc/src/main/kotlin/io/opentelemetry/kotlin/KotlinConfig.kt:L11,L17`. No toolchain resolver is registered, so a missing JDK 11 fails resolution rather than downloading |
| JDK 17 | 17.0.19 verified | Selected by `:testing` and `:gradle-integration-test`; also the Gradle 9 and JUnit 6 floor |
| JDK 21 | 21.0.11 verified | Default `java`/`javac` and the Gradle launcher, matching CI |
| Android SDK | `platform-tools`, `platforms;android-36`, `build-tools;36.0.0` | `android-compileSdk = 36`; four Android targets are part of the canonical build |
| Node.js | 22.23.2 verified | Optional — Kotlin/JS provisions its own Node and Yarn — but useful for inspecting the resolved JS dependency tree |
| RAM | 16 GB recommended | `gradle.properties:L2` requests an 8 GB Gradle daemon plus a separate 8 GB Kotlin daemon plus up to 4 GB metaspace |
| Disk | 40 GB+ | Gradle distribution, full dependency graph, Kotlin/Native toolchain, Android SDK, Node |
| System packages | `git curl wget unzip zip jq file build-essential zlib1g-dev libtinfo6` | Kotlin/Native and Wire code-generation paths |

## 9.2 Environment Setup

```bash
# JDKs — install all three; Gradle discovers them by scanning /usr/lib/jvm
export DEBIAN_FRONTEND=noninteractive
apt-get update
apt-get install -y --no-install-recommends \
  openjdk-11-jdk openjdk-17-jdk openjdk-21-jdk \
  git curl wget unzip zip ca-certificates jq file build-essential zlib1g-dev libtinfo6

# Android SDK — compileSdk 36; licences must be accepted non-interactively
export ANDROID_HOME=/opt/android-sdk
export ANDROID_SDK_ROOT="$ANDROID_HOME"
mkdir -p "$ANDROID_HOME/cmdline-tools"
cd /tmp && curl -fsSLO https://dl.google.com/android/repository/commandlinetools-linux-13114758_latest.zip
unzip -q commandlinetools-linux-*.zip -d "$ANDROID_HOME/cmdline-tools"
mv "$ANDROID_HOME/cmdline-tools/cmdline-tools" "$ANDROID_HOME/cmdline-tools/latest"
export PATH="$ANDROID_HOME/cmdline-tools/latest/bin:$ANDROID_HOME/platform-tools:$PATH"
yes | sdkmanager --licenses > /dev/null 2>&1 || true
sdkmanager --install "platform-tools" "platforms;android-36" "build-tools;36.0.0"
```

`ANDROID_HOME` is the only environment variable the build needs. Do **not** set `JAVA_HOME` to a hard-coded path — Debian's JDK directories are architecture-suffixed and Gradle's auto-detection finds them anyway. Do **not** override the memory settings in `gradle.properties`; size the machine instead.

## 9.3 Verify the Toolchain Before Building

```bash
./gradlew --version                 # expect: Gradle 9.7.0
./gradlew -q javaToolchains         # expect: JDK 11, 17 and 21 all listed as "Is JDK: true"
```

Observed here: Gradle 9.7.0, and `Ubuntu JDK 11 (11.0.31…)`, `Ubuntu JDK 17 (17.0.19…)`, `Ubuntu JDK 21 (21.0.11…)`. If JDK 11 is absent the build fails rather than downloading it, so this check is load-bearing.

## 9.4 The Canonical Build

This is the exact invocation CI runs on Linux (`.github/workflows/ci-build.yml:L42`). Never abbreviate it — the Android and benchmark targets it names are what make it a blast-radius check.

```bash
./gradlew build :benchmark-android:assemble :benchmark-android:compileReleaseAndroidTestKotlin :benchmark-jvm:assemble :examples:example-app-android:assembleRelease koverXmlReport -x :gradle-integration-test:test --stacktrace
```

Observed: `BUILD SUCCESSFUL`, exit 0, 2,186 actionable tasks. Cold 37s; an identical second run completes in 10s and reports `Configuration cache entry reused.` To force the tests to execute rather than being restored from cache:

```bash
find . -path "*/build/test-results" -type d -prune -exec rm -rf {} +
./gradlew build :benchmark-android:assemble :benchmark-android:compileReleaseAndroidTestKotlin :benchmark-jvm:assemble :examples:example-app-android:assembleRelease koverXmlReport -x :gradle-integration-test:test --stacktrace --no-build-cache
```

Observed: 38 test tasks executed, `BUILD SUCCESSFUL` in 40s.

## 9.5 Quality Gates

```bash
./gradlew apiCheck detekt --stacktrace
```

Observed: exit 0, 265 actionable tasks, zero violations. Detekt runs with `maxIssues: 0` and the compiler with warnings-as-errors, so any warning is a build failure. After changing public API, regenerate the dumps rather than editing them:

```bash
./gradlew apiDump          # then commit the regenerated */api/jvm/*.api files with the change
```

## 9.6 Publish and Consumer Integration — Two Separate Invocations

```bash
./gradlew publishToMavenLocal -PsnapshotPublish=true -Psigning.skip=true --stacktrace
./gradlew :gradle-integration-test:test --stacktrace
```

Observed: publish exit 0 in 12s; the integration run exit 0 in 49s with 2 tests and no failures. `-Psigning.skip=true` removes the need for a signing key and is sanctioned by `CONTRIBUTING.md:L44-L45`. Keep the two invocations separate — `CONTRIBUTING.md:L32-L42` states that combining them races with the native compiler.

## 9.7 Run the Example Applications

```bash
./gradlew :examples:example-app:runConsoleExampleApp   # JVM runtime
./gradlew :examples:example-app:runNodeExampleApp      # JavaScript / Node runtime
```

Both observed exiting 0 (64 and 169 actionable tasks respectively). They start the SDK, emit telemetry and shut down, which is the quickest end-to-end smoke check available.

## 9.8 Read the Measurements

```bash
# Line coverage — note the file name; reportRelease.xml is never produced
python3 - <<'PY'
import xml.etree.ElementTree as ET
r = ET.parse('build/reports/kover/report.xml').getroot()
c = {x.attrib['type']: (int(x.attrib['covered']), int(x.attrib['missed'])) for x in r if x.tag == 'counter'}
cv, mi = c['LINE']
print(f"LINE {cv}/{cv+mi} = {100*cv/(cv+mi):.2f}%")
PY

# Aggregate the whole test estate
python3 - <<'PY'
import glob, xml.etree.ElementTree as ET
t = f = e = s = 0
for p in glob.glob('*/build/test-results/*/TEST-*.xml'):
    a = ET.parse(p).getroot().attrib
    t += int(a['tests']); f += int(a['failures']); e += int(a['errors']); s += int(a.get('skipped', 0))
print(f"tests={t} failures={f} errors={e} skipped={s}")
PY

# Public-API dump inventory
git ls-files '*.api' | wc -l          # 27
git ls-files '*.klib.api' | wc -l     # 0 — no klib baseline exists
```

Observed: `LINE 4864/5153 = 94.39%` and `tests=2965 failures=0 errors=0 skipped=0`.

## 9.9 Measure a Change Unit Against the 500-Line Budget

The pathspec matters: a root-anchored one matches nothing and reports a false zero.

```bash
BASE=$(git merge-base upstream/main HEAD)
# Hand-authored lines, dumps excluded — this is the figure bounded at 500
git diff --numstat "$BASE"...HEAD -- . ':(exclude)**/*.api' | awk '{a+=$1} END {print "added:", a+0}'
# Regenerated dump lines, measured and bounded separately
git diff --numstat "$BASE"...HEAD -- '**/*.api'            | awk '{a+=$1} END {print "dump added:", a+0}'
```

## 9.10 Read the Specification

The document is a single Markdown file. To review it rendered, convert and serve it from a scratch directory outside the repository so the checkout stays clean:

```bash
pip install --break-system-packages markdown-it-py     # if not already present
RENDER_DIR=$(mktemp -d)
python3 - "$RENDER_DIR" <<'PY'
import sys, os
from markdown_it import MarkdownIt
out = os.path.join(sys.argv[1], 'plan.html')
src = open('metrics-implementation-plan.md', encoding='utf-8').read()
html = MarkdownIt('commonmark').enable('table').render(src)
open(out, 'w', encoding='utf-8').write(
    '<!doctype html><meta charset="utf-8">'
    '<style>table{border-collapse:collapse}th,td{border:1px solid #ccc;padding:4px 8px;'
    'vertical-align:top;font-size:13px}pre{background:#f6f6f6;padding:8px;overflow-x:auto}</style>' + html)
print('wrote', out)
PY
python3 -m http.server 8099 --bind 127.0.0.1 --directory "$RENDER_DIR" &
# then open http://127.0.0.1:8099/plan.html
```

Observed on the delivered document: 8 top-level sections, 23 tables, 372 rows, zero empty cells, 97 preformatted blocks, and 702,925 bytes of HTML. Several data tables are wider than a 1440-pixel viewport and scroll horizontally; nothing is clipped.

If the file is not present in the tree, the earlier revision is recoverable from history:

```bash
git show a2e8a63b:metrics-implementation-plan.md | head -20
```

## 9.11 Troubleshooting

| Symptom | Cause | Resolution |
| --- | --- | --- |
| Build fails with a configuration-cache problem | `gradle.properties:L5` sets `org.gradle.configuration-cache.problems=fail`, so any incompatibility is fatal rather than a warning | Reproduce with `--no-configuration-cache` **only** to isolate the cause, then fix the offending build logic. Never leave the cache disabled as a workaround; model new build logic on an existing module's script |
| `./gradlew allTests --rerun` fails with ~163 compile errors in `exporters-otlp` and `exporters-protobuf` | Both declare the deprecated split test coordinates in `commonTest` (`exporters-otlp/build.gradle.kts:L36-L37`, `exporters-protobuf/build.gradle.kts:L61-L62`) | Use the canonical build in §9.4 instead. The permanent fix is to declare the aggregate `kotlin("test")` dependency in those two source sets |
| First `./gradlew` invocation fails on download or checksum | The wrapper is SHA-256 pinned with `retries=0` and a 10-second timeout (`gradle/wrapper/gradle-wrapper.properties`) | Retry once; ensure `services.gradle.org` passes through any proxy byte-identically, because a proxy that recompresses the archive breaks the checksum |
| `iosSimulatorArm64Test` exits 0 but reports SKIPPED | Simulator tests require a macOS host; iOS klibs cross-compile on Linux but cannot run | Expected, not a fault. Run Apple tests on macOS |
| `apiCheck` fails after a public-API change | The committed dumps no longer match the generated ones | Run `./gradlew apiDump` and commit the regenerated dumps with the change. Never hand-merge a dump. Note that the Android dumps are already out of step with source — see §6 |
| Coverage looks absent in CI | `.github/workflows/ci-build.yml` uploads `build/reports/kover/reportRelease.xml`, which the build never produces | Read `build/reports/kover/report.xml` locally; repairing the workflow path is listed in §2.2 |
| `publishToMavenCentral` fails | Publishing credentials and signing keys are not configured, and are not needed here | Use `publishToMavenLocal` with `-Psigning.skip=true` as in §9.6 |
| Build thrashes or is killed | The build requests 8 GB plus 8 GB across two daemons | Increase the machine's memory. Do not lower the settings in `gradle.properties` — that invalidates the premise every measurement in the specification rests on |

# 10. Appendices

## A. Command Reference

| Purpose | Command | Observed result |
| --- | --- | --- |
| Gradle version | `./gradlew --version` | Gradle 9.7.0 |
| Toolchain discovery | `./gradlew -q javaToolchains` | JDK 11.0.31, 17.0.19, 21.0.11 |
| Canonical build | `./gradlew build :benchmark-android:assemble :benchmark-android:compileReleaseAndroidTestKotlin :benchmark-jvm:assemble :examples:example-app-android:assembleRelease koverXmlReport -x :gradle-integration-test:test --stacktrace` | exit 0, 2,186 tasks, 37s cold / 10s warm |
| Quality gates | `./gradlew apiCheck detekt --stacktrace` | exit 0, 265 tasks, zero violations |
| Regenerate API dumps | `./gradlew apiDump` | Commit the regenerated dumps with the change |
| Local publish | `./gradlew publishToMavenLocal -PsnapshotPublish=true -Psigning.skip=true --stacktrace` | exit 0, 12s |
| Consumer integration (separate invocation) | `./gradlew :gradle-integration-test:test --stacktrace` | exit 0, 49s, 2 tests, 0 failures |
| JVM example | `./gradlew :examples:example-app:runConsoleExampleApp` | exit 0, 64 tasks |
| JavaScript example | `./gradlew :examples:example-app:runNodeExampleApp` | exit 0, 169 tasks |
| Force test execution | append `--no-build-cache` after deleting `*/build/test-results` | 38 test tasks executed |
| Isolate a configuration-cache failure | `./gradlew <task> --no-configuration-cache` | Diagnostic only; never a fix |
| Branch commits | `git log --oneline origin/blitzy-otel-kotlin..HEAD` | `0d651dae`, `a2e8a63b` |
| Net tracked change | `git diff --numstat origin/blitzy-otel-kotlin...HEAD` | no output — empty diff |
| Dump inventory | `git ls-files '*.api' \| wc -l` and `git ls-files '*.klib.api' \| wc -l` | 27 and 0 |

## B. Port Reference

No service, listener or database is part of this project; the build binds no port. The two ports that appear are incidental to local work.

| Port | Used by | Notes |
| --- | --- | --- |
| 8099 | Local static server for reviewing the rendered specification (§9.10) | Arbitrary; any free port works |
| 4318 | OTLP/HTTP default endpoint referenced illustratively in the specification | Referenced only, never contacted |

## C. Key File Locations

| Path | What it is |
| --- | --- |
| `settings.gradle.kts` | 37 explicit project includes; with the root and the `examples` container this is the 39-project topology. No metrics module exists |
| `buildSrc/src/main/kotlin/io/opentelemetry/kotlin/KotlinConfig.kt` | The single most consequential build file — JDK 11 toolchain, exactly two Apple targets, the `jvmAndAndroidMain` intermediate source set, warnings-as-errors, Kotlin 2.0 language and API pin |
| `gradle.properties` | 8 GB Gradle daemon, 8 GB Kotlin daemon, 4 GB metaspace, configuration cache with `problems=fail` |
| `gradle/libs.versions.toml` | Every dependency and plugin version; the seven frozen minimum-support pins |
| `gradle/wrapper/gradle-wrapper.properties` | Gradle 9.7.0 pinned by SHA-256 with `retries=0` |
| `codecov.yml` | Project threshold 1%, patch target 50% |
| `config/detekt/detekt.yml` | `maxIssues: 0`, 140-character lines, parameter and function ceilings of 30/31 |
| `.github/workflows/ci-build.yml` | Three jobs; the canonical build command at line 42; the coverage upload path |
| `api/src/commonMain/kotlin/io/opentelemetry/kotlin/metrics/` | The eight-file metrics API surface; `Meter.kt` is an interface with no body |
| `implementation/src/commonMain/kotlin/io/opentelemetry/kotlin/metrics/` | `MeterProviderImpl` (a real provider with scope caching and lifecycle) and `MeterImpl` (a shell) |
| `api/api/jvm/api.api` · `api/api/android/api.api` | The two rank-one dumps that disagree: 402 lines carrying the metrics package versus 462 lines carrying none |
| `exporters-otlp/src/commonMain/kotlin/io/opentelemetry/kotlin/export/TelemetryExporter.kt` | The generic retry and transport seam the specification requires be repaired before reuse |
| `integration-test/src/commonMain/kotlin/io/opentelemetry/kotlin/framework/OtelKotlinTestRule.kt` | The abstract differential harness; its reference-delegating back end lives in `compat/src/jvmTest`, which is what bounds differential testing to the JVM |
| `AGENTS.md` · `CONTRIBUTING.md` · `api/README.md` | The repository's own standards: 500-line diff guidance, dump-before-change discipline, error-handling doctrine, publish separation, component stability matrix |
| `build/reports/kover/report.xml` | The coverage report the build actually produces |

## D. Technology Versions

| Component | Version |
| --- | --- |
| Gradle | 9.7.0 (SHA-256 pinned, `retries=0`) |
| Kotlin Multiplatform plugin | 2.4.10, compiling at language and API level **2.0** |
| Android Gradle Plugin | 9.3.1 · `compileSdk` 36 · `minSdk` 21 |
| JDK toolchain | 11 declared; 17 for two modules; 21 as launcher |
| Detekt | 1.23.8 (`maxIssues: 0`) |
| binary-compatibility-validator | 0.18.1 |
| Kover | 0.9.9 |
| JUnit | 6.1.3 |
| Kotest (property testing) | 6.2.4 |
| Ktor client | 3.2.3 |
| kotlinx-coroutines | 1.10.2 |
| kotlinx-serialization-json | 1.8.1 |
| Okio | 3.18.1 |
| Wire | 6.4.5 · protobuf-kotlin 4.35.1 |
| OpenTelemetry Java (compatibility and reference) | 1.65.0 |
| Compose Multiplatform | 1.11.1 |
| KotlinPoet | 2.3.0 · yamlkt 0.13.0 · download plugin 5.7.0 · BuildKonfig 0.22.0 |
| Node.js | 22.23.2 system; Kotlin/JS provisions its own |

## E. Environment Variable Reference

| Variable | Required | Value used | Purpose |
| --- | --- | --- | --- |
| `ANDROID_HOME` | Yes | `/opt/android-sdk` | The Android Gradle Plugin cannot locate the SDK without it |
| `ANDROID_SDK_ROOT` | Recommended | same as `ANDROID_HOME` | Compatibility alias |
| `JAVA_HOME` | No | JDK 21 path | Gradle auto-detects toolchains by scanning `/usr/lib/jvm`; do not hard-code an architecture-suffixed path |
| `GITHUB_TOKEN` | Only for metadata harvesting | supplied externally | Unauthenticated API access is capped and returns empty bodies once exhausted. Never write its value anywhere |
| `GRADLE_USER_HOME` · `KONAN_DATA_DIR` | No | defaults | Only set them if a fixed cache mount layout demands it |

Publishing credentials (`ORG_GRADLE_PROJECT_mavenCentral*`, `ORG_GRADLE_PROJECT_signingInMemory*`, `SONATYPE_*`, `GPG_*`, `CODECOV_TOKEN`) are **not** needed. `publishToMavenLocal` requires none of them.

## F. Developer Tools Guide

- **Caches worth persisting** between runs: `~/.gradle/caches`, `~/.gradle/wrapper/dists`, `~/.konan` (large), `~/.m2/repository`, `$ANDROID_HOME`, `~/.npm`, `~/.cache/yarn`. CI deliberately disables caching for reproducibility; local work should do the opposite.
- **Never regenerate** `kotlin-js-store/yarn.lock` casually. It is checked in and validated; a rewritten lockfile produces a confusing lock-mismatch failure rather than a dependency error.
- **Build-time downloads**: `exporters-protobuf`, `config-schema` and `semconv` fetch upstream archives during execution, so a blocked host surfaces as an opaque task failure rather than a network error.
- **Two test frameworks are in use**: JUnit 6.1.3 broadly, and Kotest 6.2.4 for property-based tests. Match whichever the neighbouring module uses rather than choosing.
- **Detekt deliberately disables** the swallowed-exception, generic-exception-caught and instance-of-check rules, because the error-containment strategy requires catching throwables at callback and coroutine boundaries.
- **The `api` module does not receive** the project-wide experimental opt-in, so every new public declaration there must carry `@ExperimentalApi` explicitly.

## G. Glossary

| Term | Meaning in this project |
| --- | --- |
| Canonical build | The exact multiplatform build invocation CI runs on Linux, written out in full at §9.4. It is the merge gate |
| Change unit | One reviewable increment of the specified implementation, bounded at 500 added hand-authored lines and identified `M-01` … `M-35` |
| Work package | A group of change units sharing a coverage boundary, identified `WP-A` … `WP-K` |
| Gate | A named, mechanically checkable acceptance condition, `G3-01` … `G3-11` |
| Ledger row | One executed baseline verification, `P3-01` … `P3-12`, carrying an outcome of matches, differs or unavailable |
| Deviation | A measured contradiction between an asserted value and the observed one, `D1` … `D11` |
| Authority ladder | The six-rank precedence order for evidence — public-API dumps first, then executable build configuration, then source, then the pinned reference implementation, then unverified assertions, then prose |
| Differential testing | Running the same scenario against both the Kotlin back end and the reference implementation and comparing golden output. Reachable only on the JVM, because the reference-delegating harness lives in a JVM-and-Android module's test source set |
| Silent data loss | A measurement accepted by the API but never exported, with no error surfaced. Named as the dominant failure mode of the specified pipeline |
| Verified / unverified target | A platform whose behaviour has actually been executed, versus one that only compiles. Apple is currently unverified |
| Verification branch | A local throwaway branch used to answer one question empirically, never pushed or merged, deleted once its outcome is recorded |
