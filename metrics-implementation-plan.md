# 1. Executive summary

The Metrics signal MUST be built inside the existing 39-project topology with no new Gradle module: instrument contracts in `api`, configuration in `sdk-api`, the measurement pipeline in `implementation`, accumulation primitives in `platform-implementations`, and marshalling in the three exporter modules [A2|DECIDED].

The work is eighteen binding architecture records and thirty-five dependency-ordered change units, none exceeding 490 estimated added lines. The critical path runs dump-authority reconciliation, public instrument contracts, attribute-key materialisation and concurrent accumulation, aggregation state, pull-based reader collection, OTLP conversion, lifecycle and configuration integration, cross-platform and differential evidence, then regenerated dumps and documentation [A3|DECIDED].

The largest correctness risk is silent data loss. `AttributesMutator` is a write-only lambda receiver with ten setters, no read accessors and no equality contract, so a measurement whose attributes are not materialised into an immutable hashable key aliases or discards a series while every existing test passes [A3|OBSERVED].

Three hard stops govern, each measured rather than assumed: host capacity against the checked-in heap premise, JDK 11 discoverability, and a green unmodified baseline [A2|OBSERVED].

Compliance against the required register: 0 of 81 satisfied, 4 of 81 excluded by non-goals, 77 outstanding, against 88 total rows of which 7 optional rows are out of scope [A4|DERIVED].

# 2. Baseline evidence ledger

Outcomes use exactly three values. `MATCHES` means observed evidence equals the asserted baseline. `DIFFERS` cites both the expected and the observed value and lets the higher-ranked observed value govern. `UNAVAILABLE` states what could not be observed and never infers success from absence [A5|DECIDED].

Authority ranks are A1 checked-in API dumps, A2 executable build and CI configuration plus measured command output, A3 checked-in production and test source and generated-source task definitions, A4 the upstream Java reference at tag `v1.65.0` and the pinned specification metadata, A5 unmeasured requirement assertions, and A6 repository prose. Epistemic statuses are `OBSERVED`, `DERIVED`, `DECIDED`, `PROPOSED`, `UNAVAILABLE` and `QUESTION`. Applying that convention to every claim, and stating each negative finding positively rather than by omission, is what satisfies HC-07 [A5|DECIDED].

Every row below records a read locator or a producing command and never a write, because the analysis spans all 39 projects while the only artifact this document's authoring produced is the document itself, which is the distinction HC-02 and HC-03 fix [A5|DECIDED].

Authoring note on rules: `review_rules(view_range=[1,-1])` returns `No user rules provided.`, so no project-specific user rules exist and none are invented here; the governing standards are instead the checked-in repository guidance in `AGENTS.md:L22-L46`, `CONTRIBUTING.md:L32-L60`, `api/README.md:L26-L37`, `codecov.yml:L1-L10`, `config/detekt/detekt.yml:L3-L40` and `buildSrc/src/main/kotlin/io/opentelemetry/kotlin/KotlinConfig.kt:L10-L92`, summarised by locator rather than reproduced [A6|OBSERVED].

| ID | Subject | Outcome | Expected / asserted | Observed | Authority / status | Evidence — locator or producing command | Planning consequence |
| --- | --- | --- | --- | --- | --- | --- | --- |
| P3-01 | Checkout identity, branch, HEAD, remotes, clean state, one-file write boundary | MATCHES | HEAD `20d7a3947019479e3536daea8bc38c1f393577d5`; a clean tree; a read-only upstream remote; exactly one file created | HEAD is `20d7a3947019479e3536daea8bc38c1f393577d5`; `git status --porcelain` returns empty output both before and after every baseline invocation; remotes are `origin` and `upstream`; the upstream push URL is disabled; local branches are the working branch and `main` only, so no spike branch exists | A2 OBSERVED | `git rev-parse HEAD`; `git status --porcelain`; `git remote`; `git config --get-all remote.upstream.pushurl`; `git branch --list` | The recorded baseline commit is fixed for every citation in this document, and the no-push posture required by HC-01 is structural rather than procedural. The `origin` URL carries embedded credential material: its presence is recorded, its value is excluded from all evidence capture, and `.github/repository_settings.md` secret identifier names are likewise excluded under HC-11 |
| P3-02 | Artifact placement, tracking state and delivery visibility | DIFFERS | An out-of-repository destination, asserted under an instruction that has since been **revoked** by the reviewing maintainer | The destination is the tracked repository-root file `metrics-implementation-plan.md`. `git check-ignore -v metrics-implementation-plan.md` produces no output and exits 1, so the path is not ignored by any pattern and is a legal tracked path; `git ls-files metrics-implementation-plan.md` returns that one path after staging and no other path in the tree matches `metrics-implementation-plan`; no `docs/` directory exists in the checkout, and all seven repository-authored root Markdown documents use root placement | A2 OBSERVED, A5 DERIVED | `.gitignore:L1-L11` lists only `*.iml`, `.gradle`, `.idea`, `.DS_Store`, `build`, `captures`, `.externalNativeBuild`, `.cxx`, `local.properties`, `xcuserdata` and `.kotlin` — no Markdown pattern and no entry for this file; `git check-ignore -v metrics-implementation-plan.md` (no output, exit 1); `git ls-files \| grep -c metrics-implementation-plan`; `git diff --numstat origin/blitzy-otel-kotlin...HEAD`; `ls *.md`; `ls -d docs` | Tracked repository-root placement governs, and it is the placement the reviewing maintainer directed. The earlier out-of-repository ruling recorded here rested on the premise that a root file could only ever be untracked; that premise is false, because the path is not ignored and can simply be committed, which is what makes the branch's tracked diff carry the deliverable instead of nothing. Three alternatives are rejected explicitly rather than left implicit: an out-of-repository destination is rejected because it leaves the branch's tracked diff empty so a merge delivers nothing, and because it was revoked; a new `docs/` directory is rejected because it would break the root-placement convention every existing root document follows; a second copy anywhere is rejected because two artifacts cannot both be the deliverable. Delivery visibility is therefore settled by tracking rather than by workspace capture, and the residual capture question that depended on it is closed rather than carried to Section 8 |
| P3-03 | Project topology | MATCHES | 39 Gradle projects from 37 explicit include entries plus the root project plus `examples`; no metrics module; zero projects from an absent `instrumentation` directory | A single `include(` call at `settings.gradle.kts:L17-L55` carries exactly 37 explicit paths at L18-L54, two of which are `examples:example-app` and `examples:example-app-android` and therefore materialise the `examples` container; the project list resolves to 38 named projects plus the root, which is 39; no path contains `metrics`; `includeFromDir("instrumentation")` at L57 walks a directory that does not exist, contributing zero | A2 OBSERVED | `settings.gradle.kts:L16-L57`; `./gradlew -q projects` filtered to the unique quoted project names it prints, which yields 38; `ls -d instrumentation` reports no such file or directory | Every change unit in Section 4 lands in an existing project. ADR-01 elects no new module, so the 39-project count is a fixed invariant that any Section 4 unit breaching would be a scope violation rather than a design choice |
| P3-04 | Metrics footprint and the real-versus-scaffold state of the provider | MATCHES | A 22-file metrics footprint, with `MeterProvider` implementation real and `Meter` implementation a scaffold | The footprint is exactly 22 Kotlin files: 8 in `api/src/commonMain`, 5 compat adapters in `jvmAndAndroidMain`, 2 compat `jvmTest` tests, 2 `implementation/src/commonMain` files, 1 `implementation/src/commonTest` file, 2 `noop` files and 2 `test-fakes` files. `MeterProviderImpl` is a real provider carrying `MutableShutdownState`, a `CompositeTelemetryCloseable` over an empty delegate list, a lazily built `ApiProviderImpl<Meter>` scope cache, an empty-scope-name diagnostic, and timeout-wrapped flush and shutdown. `MeterImpl` is a nine-line shell holding only scope and resource. `MetricsConfig` holds exactly `resource` and `sdkErrorHandler` | A3 OBSERVED | `find . -type f -name '*.kt' -path '*/src/*' -path '*metrics*' -not -path '*/build/*'` returns 22 paths; `MeterProviderImpl.kt:L15-L51` with `shutdownState.ifActiveOrElse(noopMeter)` at L38, the diagnostic at L39-L41, flush at L46-L47 and shutdown at L49-L50; `MeterImpl.kt:L6-L9`; `MetricsConfig.kt:L11-L22` | The provider's lifecycle machinery is reused rather than rebuilt, which removes an entire class of work from Section 4 and makes ADR-11 a preservation decision. All four `Meter` implementors break the moment `Meter` gains a member, so `M-05` is mandatory and coordinated: `MeterImpl.kt:L9`, `NoopMeter.kt:L11`, `FakeMeter.kt:L5`, `MeterAdapter.kt:L9` |
| P3-05 | Dump inventory and the platform dump disagreement | DIFFERS | 27 JVM and Android dumps, zero klib dumps, and a JVM-versus-Android metrics disagreement | 27 files match `*.api` and zero match `*.klib.api`. `api/api/jvm/api.api` is 402 lines and carries the whole metrics package at L240-L275 plus `getMeterProvider` at L8. `api/api/android/api.api` is 462 lines and contains zero occurrences of the string `metrics` and zero of `getMeterProvider`. `sdk-api/api/jvm/sdk-api.api` is 409 lines, declares an empty `MeterProviderConfigDsl` at L161-L162 and `meterProvider (Lkotlin/jvm/functions/Function1;)V` at L170, and contains zero metrics SDK type names | A1 OBSERVED | `find . -name '*.api' -not -path '*/build/*' \| wc -l` returns 27; the same with `*.klib.api` returns 0; `grep -c 'metrics' api/api/android/api.api` returns 0; `grep -c 'getMeterProvider' api/api/android/api.api` returns 0; `grep -ci 'metric' sdk-api/api/jvm/sdk-api.api` returns 0; `api/api/jvm/api.api:L8,L240-L275`; `sdk-api/api/jvm/sdk-api.api:L161-L170` | Two rank-1 sources disagree and the klib tiebreak is inoperative because no `.klib.api` baseline exists. `apiCheck` passing therefore proves nothing about Android parity. ADR-13 rules on the disagreement and `M-02` isolates it before any new API delta is accepted, under G3-09. The already-public `meterProvider` entry point means ADR-03 extends a released surface rather than introducing one |
| P3-06 | Toolchain, language floor, target set and compiler flags | DIFFERS | JDK 11 declared and discoverable, Kotlin 2.0 API and language floor, exactly two Apple targets, `jvmAndAndroidMain`, warnings as errors, and `-Xexpect-actual-classes` | All confirmed, plus one flag the asserted set omitted. `KotlinConfig.kt` declares `JDK_VERSION = 11` at L11 and applies `jvmToolchain` at L17; `KOTLIN_VERSION` is `KOTLIN_2_0` at L10 and is set as both `apiVersion` at L90 and `languageVersion` at L91; the configured Apple targets are exactly `iosArm64()` at L39 and `iosSimulatorArm64()` at L40, with no `iosX64`; `applyDefaultHierarchyTemplate()` at L51 is supplemented by a created `jvmAndAndroidMain` at L68 that `androidMain` at L71-L73 and `jvmMain` at L74-L76 depend on; `allWarningsAsErrors` is set at L89; `freeCompilerArgs` at L92 adds both `-Xexpect-actual-classes` and `-Xsuppress-version-warnings`; `commonMain` pins `kotlin-stdlib` to `2.0.0` at L55. Neither `settings.gradle.kts` nor `buildSrc/settings.gradle.kts` registers a toolchain resolver. Measured discovery lists JDK 11.0.31, 17.0.19 and 21.0.11 | A2 OBSERVED | `buildSrc/src/main/kotlin/io/opentelemetry/kotlin/KotlinConfig.kt:L10-L17,L39-L40,L51,L55,L68-L76,L89-L92`; `grep -n 'toolchain\|resolver' settings.gradle.kts buildSrc/settings.gradle.kts` returns nothing; `./gradlew -q javaToolchains` exits 0 and lists 11, 17 and 21, each detected from common Linux locations or as the current JVM | Deviation D6 is recorded: `-Xsuppress-version-warnings` accompanies `-Xexpect-actual-classes`, which is why compiling a 2.0 language level under a 2.4.10 plugin does not itself fail the warnings-as-errors gate. The two-target Apple set bounds G3-07: no `iosX64` evidence may be claimed. The toolchain stop condition does not fire, because JDK 11 is discoverable and not merely installed |
| P3-07 | Version catalog, frozen pins, wrapper, and the atomics position | DIFFERS | A frozen minimum-support set, a SHA-pinned wrapper, no dependency mutation, and no available atomics primitive | The catalog is 108 lines, not the asserted 109, and the observed count governs. A case-insensitive search for `atomicfu` or `atomic` across the whole catalog returns zero matches, so `org.jetbrains.kotlinx:atomicfu` is absent and cannot be referenced without a dependency addition. Seven minimum-support keys are frozen with `"enabled": false`. The wrapper pins `gradle-9.7.0-bin` with SHA-256 `84fbba45c7f4c64abc77460e1c00f541e9f960e3c7ed2538f1ede19eacd873ae`, `networkTimeout=10000`, `retries=0`, `retryBackOffMs=500` and `validateDistributionUrl=true`. The repository states the standard-library atomics position itself in source | A2 OBSERVED, A3 OBSERVED | `wc -l gradle/libs.versions.toml` and `grep -c '' gradle/libs.versions.toml` both return 108; `grep -in 'atomicfu\|atomic' gradle/libs.versions.toml` returns nothing; `gradle/libs.versions.toml:L8-L11` pins `minSupportedKotlinLang = "2.0"`, `minSupportedKotlinPlugin`, `minSupportedKotlinKlib` and `minSupportedGradle`; `.github/renovate.json5:L17-L24` freezes all seven keys at L21 and disables updates at L23; `gradle/wrapper/gradle-wrapper.properties:L3-L8`; `platform-implementations/src/commonMain/kotlin/io/opentelemetry/kotlin/AtomicLong.kt:L11-L13` records that the Kotlin standard-library atomic long arrived as experimental in 2.1 while this project supports back to 2.0 | ADR-06 is decided on rank-3 in-repository evidence rather than on external documentation: the standard-library atomics are foreclosed by the frozen 2.0 language floor, and the multiplatform atomics library is not merely undesirable but absent, so selecting it would be a dependency addition that HC-04 forbids inside any implementation unit. No catalog, wrapper or lockfile line changes in this run or in any Section 4 unit |
| P3-08 | Quality gates actually enforced | DIFFERS | A zero-issue static-analysis budget, warnings as errors, a 1% project coverage threshold, a 50% patch target, and a coverage artifact the build produces | The static and compiler gates match. `maxIssues: 0` sits at `detekt.yml:L3`; `LongParameterList` thresholds are 30 for constructors at L10 and 31 for functions at L11; `TooManyFunctions` is 31 across files, classes, interfaces, objects and enums at L13-L19; the three exception-oriented rules are deliberately disabled at L25-L31; the line limit is 140 at L34, L36 and L50; trailing commas are required at L37-L40. `codecov.yml` is 10 lines, not 11: target `auto` at L5, `threshold: 1%` at L6, `removed_code_behavior: adjust_base` at L7 and patch `target: 50%` at L10. The coverage artifact differs materially: the build produces `build/reports/kover/report.xml` and does not produce `build/reports/kover/reportRelease.xml`, which is the exact path the workflow uploads. Measured project coverage is LINE 4864 covered, 289 missed, 5153 total, which is 94.39%, not the asserted figure near 80.6% | A2 OBSERVED, A2 DERIVED | `config/detekt/detekt.yml:L3,L9-L19,L25-L31,L34,L36-L40,L50`; `codecov.yml:L1-L10` and `wc -l codecov.yml` returns 10; `.github/workflows/ci-build.yml:L49-L50`; `ls build/reports/kover/` after the baseline build lists only `report.xml` and `verify.err`; the root `<report>` counters of `build/reports/kover/report.xml` give LINE 4864/289/5153, INSTRUCTION 25618/1320, BRANCH 1714/241, METHOD 1486/116 and CLASS 462/15 across 34 packages | Deviations D5 and the artifact-path finding both bind. The 1.00 percentage-point window governs and is never widened to the 2.00-point figure proposed elsewhere, per G3-03 and HC-10. Because the total measurable line count is only 5153, the window is far tighter than a larger denominator would imply: a 500-line unit landing at exactly the 50% patch floor moves LINE coverage to 90.47%, a 3.93-point drop and 3.9 times the enforced window, so the project window rather than the patch floor is the binding constraint and the minimum patch coverage for a 500-line unit is approximately 83.1%, for a 250-line unit approximately 72.8% and for a 100-line unit approximately 41.9%. The upload path mismatch means the Codecov gate cannot currently observe a regression at all, which `M-01` must reconcile before G3-03 is treated as enforcing |
| P3-09 | Generated configuration and protocol state | MATCHES | 113 tracked generated schema models, 29 metrics-relevant names, whole-tree protocol generation, and `rpcRole = "none"` | The model directory holds 113 Kotlin files and `git ls-files` reports all 113 as tracked, so they are generated source committed into `src`, not build output. Under the stated rule of file names matching `metric`, `meter`, `aggregation`, `exemplar`, `view` or `cardinality` case-insensitively, 29 are metrics-relevant, including `PeriodicMetricReader`, `PullMetricReader`, `PushMetricExporter`, `PullMetricExporter`, `SumAggregation`, `LastValueAggregation`, `DropAggregation`, `DefaultAggregation`, `ExplicitBucketHistogramAggregation`, `Base2ExponentialBucketHistogramAggregation`, `ExporterDefaultHistogramAggregation`, `ExemplarFilter`, `CardinalityLimits`, `View`, `ViewSelector`, `ViewStream`, `MetricProducer`, `OpenCensusMetricProducer` and four experimental meter and exporter families. Protocol generation covers the entire tree and emits no service stubs | A2 OBSERVED, A3 OBSERVED | `ls config-schema/src/commonMain/kotlin/io/opentelemetry/kotlin/config/schema/model/*.kt \| wc -l` returns 113 and `git ls-files` on the same directory returns 113; the same listing filtered by `grep -iE 'metric\|meter\|aggregation\|exemplar\|view\|cardinality'` returns 29; `config-schema/build.gradle.kts:L54` registers `generateOpenTelemetryConfiguration`, L61-L62 targets `src/commonMain/kotlin/io/opentelemetry/kotlin/config/schema/model`, and L68 excludes the directory from static analysis; `exporters-protobuf/build.gradle.kts:L17` pins protocol version `1.8.0`, L33 includes `opentelemetry/proto/**` wholesale, and L78 sets `rpcRole = "none"` | Deviations D1 and D2 bind. The 29 generated names are the authoritative naming reference for every metrics configuration concept, so ADR-03 and ADR-07 borrow names rather than inventing them, and no unit hand-edits a generated model because an explicit regeneration would overwrite it. The metrics wire messages already compile, so `M-20` is marshalling only, and ADR-10 claims no gRPC surface because no service stubs are generated |
| P3-10 | Tests, fixtures, Apple execution, and differential reach | DIFFERS | 287 test files, 899 production files, four golden-fixture directories, and a JVM-bounded differential harness | Counts are definition-sensitive and each is reported with its rule. Test source-set directories number 28 under the rule of a directory named `<name>Test` directly under `src`, or 29 if a case-insensitive rule that also admits `gradle-integration-test/src/test` is used. Test Kotlin files number 283 under the case-sensitive rule or 287 under the case-insensitive rule that adds the four files under `gradle-integration-test/src/test`. Production Kotlin files number 885 excluding `buildSrc`, and `buildSrc` holds 13 Kotlin files rather than the asserted 14, giving 898 rather than 899 including it. JSON fixtures under `src/**/resources` number 42, and five resource directories exist of which four are golden-fixture directories once the Gradle consumer project is excluded. The whole suite reports 2965 tests with zero failures, zero errors and zero skipped. Apple execution is partial rather than absent: both configured Apple targets compile here, while the simulator test task is disabled by the toolchain on this host | A2 OBSERVED, A3 OBSERVED | The counting commands are `find . -type d -not -path '*/build/*'` filtered to test source-set names, `find . -type f -name '*.kt' -path '*/src/*Test*/*'`, the same filtered case-insensitively, `find . -type f -name '*.kt' \( -path '*/src/*Main/*' -o -path '*/src/main/*' \)` with and without `-not -path './buildSrc/*'`, and `find . -type f -name '*.json' -path '*/src/*' -path '*resources*'`; suite totals are aggregated from 399 `testsuite` XML files under `*/build/test-results/`; `./gradlew :api:compileKotlinIosArm64 :api:compileKotlinIosSimulatorArm64 :implementation:compileKotlinIosArm64 :implementation:compileKotlinIosSimulatorArm64` exits 0; `./gradlew :implementation:iosSimulatorArm64Test` exits 0 with the task reported SKIPPED alongside `linkDebugTestIosSimulatorArm64`, the toolchain stating the task cannot run on a `linux-x86_64` host; the harness triple is `integration-test/src/commonMain/kotlin/io/opentelemetry/kotlin/framework/OtelKotlinTestRule.kt:L27,L33`, `implementation/src/commonTest/kotlin/io/opentelemetry/kotlin/integration/test/IntegrationTestHarness.kt:L14-L24` and `compat/src/jvmTest/kotlin/io/opentelemetry/kotlin/framework/OtelKotlinHarness.kt:L12-L29`; `implementation/build.gradle.kts:L64-L71` wires `copyiOSTestResources` into `iosSimulatorArm64Test` | Deviation D7 binds, and every count in this document carries its rule. The reference-delegating back end lives in `jvmTest` of a JVM and Android module, so the Java differential is mechanically JVM-only, which Section 6 states as a reach limit rather than an aspiration. The abstract harness configures only tracer and logger providers today, so `M-24` must add a meter hook before any differential metrics scenario exists. Most consequentially, an Apple simulator task that is skipped rather than failed means a zero exit code on a non-macOS host is not Apple evidence, which G3-07 must assert positively rather than infer |
| P3-11 | Baseline build, publish and integration invocations, and capacity | DIFFERS | Available memory below the 16 GiB floor, forcing a capacity stop; a green baseline; and a consumer integration test that cannot pass off macOS | The asserted capacity figure is a unit error and the corrected measurement removes the stop. `/proc/meminfo` reports `MemTotal: 4029526812 kB` and `MemAvailable: 3952084312 kB`, which is 3935085 MiB or approximately 3842 GiB total, with `SwapTotal: 25165816 kB` and no cgroup memory limit present at any of the three limit paths; the asserted 3842 MB mislabels a gibibyte figure as a mebibyte figure, and the true capacity exceeds the 16 GiB floor by more than two orders of magnitude. One qualification is recorded rather than smoothed over: a heap override outside the repository, at `~/.gradle/gradle.properties` in the Gradle user home, sizes the daemons to `-Xmx1800m` with `kotlin.daemon.jvmargs=-Xmx1800m` and `org.gradle.workers.max=2`, and it is deliberately left in place, so the figures below are measured under those reduced daemons and not under the checked-in premise of an 8 GiB Gradle daemon, a separate 8 GiB Kotlin daemon and 4 GiB of metaspace. The repository `gradle.properties` is byte-identical to baseline and was never modified, so the checked-in premise still governs every gate this document sets; the gate is discharged by build exit status rather than by a memory figure. Under those conditions `./gradlew --version` exits 0 reporting Gradle 9.7.0; `./gradlew -q javaToolchains` exits 0 listing JDK 11.0.31, 17.0.19 and 21.0.11, each reported as a JDK; `./gradlew build :benchmark-android:assemble :benchmark-android:compileReleaseAndroidTestKotlin :benchmark-jvm:assemble :examples:example-app-android:assembleRelease koverXmlReport -x :gradle-integration-test:test --stacktrace` exits 0 and reports `BUILD SUCCESSFUL in 21s` across `2186 actionable tasks` with `Configuration cache entry stored.`, and an immediately repeated identical invocation exits 0 reporting `Reusing configuration cache.`, `BUILD SUCCESSFUL in 4s` across `2182 actionable tasks` and `Configuration cache entry reused.`; `./gradlew apiCheck detekt --stacktrace` exits 0 across `265 actionable tasks`; `./gradlew publishToMavenLocal -PsnapshotPublish=true -Psigning.skip=true --stacktrace` exits 0; and the separate invocation `./gradlew :gradle-integration-test:test --stacktrace` exits 0, executing 2 tests with zero failures | A2 OBSERVED | `free -h` reports 3.8Ti total and 3.7Ti available; `/proc/meminfo` supplies `MemTotal` and `MemAvailable`; `/sys/fs/cgroup/memory.max`, `/sys/fs/cgroup/memory/memory.limit_in_bytes` and `/sys/fs/cgroup/memory.high` are all absent; `nproc` reports 4; `gradle.properties:L2-L7` states the heap premise with configuration-cache problems failing the build at L5; `git diff --stat -- gradle.properties` is empty, so the checked-in premise is unaltered; every transcript is reproduced verbatim with its exit status in Section 2.4; `CONTRIBUTING.md:L44-L45` sanctions `-Psigning.skip=true` whereas `.github/workflows/ci-build.yml:L78` omits it | No stop condition fires, so implementation may proceed from a green floor. Four consequences bind. The capacity gate is retained in this plan rather than deleted, because it must be re-evaluated on any other host, and it is satisfied by supplying memory rather than by lowering the checked-in heap, per HC-10. Because the measured baseline ran under reduced daemons, a later unit that fails only under the checked-in 8 GiB premise is a capacity finding rather than a code defect, and the two must not be confused. Configuration-cache reuse is established on a tree whose only addition is this document, so any later reuse failure is attributable to a specific unit under G3-04. The consumer integration test passing on a non-macOS host contradicts the asserted expectation, so a later failure must be investigated rather than excused as a host limitation, and publish and integration remain two invocations because a single invocation races the native compiler per `CONTRIBUTING.md:L32-L42` |
| P3-12 | Governance and documentation deviations, release-line disambiguation, rules result, and secret hygiene | DIFFERS | Repository guidance that accurately describes the current module set, Android package levels and publish flow, and a changelog with a single release line | Four documentation deviations and one release-line collision are confirmed. `AGENTS.md:L10` names a module `sdk` that does not exist, the actual modules being `sdk-api`, `sdk-common` and `sdk-ext`. `CONTRIBUTING.md:L14-L15` requests `platforms;android-34` and `build-tools;30.0.3` while the catalog compiles against SDK 36. Three different publish and integration forms coexist. The changelog carries 6 dated `## Version X.Y.Z (date)` headings and 19 bare legacy `# X.Y.Z` headings below a legacy divider, and versions 0.6.0 and 0.5.0 each appear in both lines. `review_rules(view_range=[1,-1])` returns `No user rules provided.` Credential-bearing metadata was observed in the checkout's `origin` URL and secret identifier names in `.github/repository_settings.md`; both are excluded from this document | A6 OBSERVED, A2 OBSERVED | `AGENTS.md:L10` against `settings.gradle.kts:L22-L24`; `CONTRIBUTING.md:L14-L15` against `gradle/libs.versions.toml:L5`; `CONTRIBUTING.md:L40-L41`, `.github/workflows/ci-build.yml:L78,L81` and the executed pair recorded in P3-11; `grep -nE '^## Version [0-9]+\.[0-9]+\.[0-9]+ \(' CHANGELOG.md` returns 6 headings at L5, L57, L121, L234, L309 and L338 while `grep -nE '^# [0-9]+\.[0-9]+\.[0-9]+' CHANGELOG.md` returns 19 headings at L375 through L486 under the divider at L373; `review_rules(view_range=[1,-1])` | Deviations D8, D9 and D10 bind, so prose is corroboration only and never authority for behaviour. Any version claim must cite a dated heading, because a bare `0.6.0` or `0.5.0` is ambiguous across the two lines. Because no user rules exist, enterprise practice drawn from the checked-in guidance governs instead, and no rule is invented. `RELEASING.md:L19-L20` describes a discretionary cadence and is never converted into a schedule, per HC-06 |

## 2.1 Measured deviations D1 through D11

Each deviation records the asserted value and the observed value, and in every case the observed value governs the plan [A5|DECIDED].

| ID | Asserted | Observed | Locator or producing command | Consequence |
| --- | --- | --- | --- | --- |
| D1 | The generated configuration schema models are build output | All 113 model files are tracked source generated into the source tree, and `git ls-files` reports 113 of 113 tracked; the generating task writes into `src/commonMain/kotlin/io/opentelemetry/kotlin/config/schema/model` and the directory is excluded from static analysis | `config-schema/build.gradle.kts:L54,L61-L62,L68`; `git ls-files config-schema/src/commonMain/kotlin/io/opentelemetry/kotlin/config/schema/model \| wc -l` | A hand edit to a generated model survives until the next explicit regeneration and is then silently reverted, so no change unit edits one and R-14 tracks the hazard [A3\|OBSERVED] |
| D2 | A smaller enumerated set of metrics configuration models | 29 metrics-relevant generated model names exist under the rule of a case-insensitive file-name match on `metric`, `meter`, `aggregation`, `exemplar`, `view` or `cardinality` | `ls config-schema/src/commonMain/kotlin/io/opentelemetry/kotlin/config/schema/model/ \| grep -iE 'metric\|meter\|aggregation\|exemplar\|view\|cardinality' \| wc -l` | The 29 names are naming and reference evidence for ADR-03 and ADR-07; they are not a promise that declarative configuration is wired in the first wave [A3\|OBSERVED] |
| D3 | `ModuleBehavior.kt` declares all four module-behaviour properties | It declares only `containsPublicApi()` and `isJvmAndroidModule()`; `enableCodeCoverage` is read in the root build script and `enableOptIn` in the opt-in convention plugin, so the citation splits across three files | `buildSrc/src/main/kotlin/io/opentelemetry/kotlin/ModuleBehavior.kt:L10-L13,L20-L23`; `build.gradle.kts:L33-L38`; `buildSrc/src/main/kotlin/io/opentelemetry/kotlin/OptInConfig.kt:L16-L18` | Module-behaviour claims cite the reading site, not a single file, so a reviewer can verify each property independently [A2\|OBSERVED] |
| D4 | Configuration-cache problems fail the build at `gradle.properties:L6` | The setting sits at `gradle.properties:L5`, with parallel execution at L6 | `gradle.properties:L5-L6` | The locator is corrected wherever the fail-on-problems premise is cited, including G3-04 and R-12 [A2\|OBSERVED] |
| D5 | `codecov.yml` is 11 lines | The file is 10 lines, with `threshold: 1%` at L6 and patch `target: 50%` at L10 | `wc -l codecov.yml`; `codecov.yml:L1-L10` | Coverage citations use L6 and L10, and the 1.00 percentage-point window in G3-03 is anchored to L6 [A2\|OBSERVED] |
| D6 | Only `-Xexpect-actual-classes` is set as a free compiler argument | Both `-Xexpect-actual-classes` and `-Xsuppress-version-warnings` are set | `buildSrc/src/main/kotlin/io/opentelemetry/kotlin/KotlinConfig.kt:L92` | The second flag is why a 2.0 language level under a 2.4.10 plugin does not trip warnings-as-errors; removing it would convert every version warning into a build failure, so no unit touches L92 [A2\|OBSERVED] |
| D7 | 287 test files, 899 production files, 28 test source sets and four golden-fixture directories, each as a single figure | Every count is definition-sensitive and is therefore reported with its rule: 28 test source-set directories case-sensitively or 29 case-insensitively; 283 test Kotlin files case-sensitively or 287 case-insensitively; 885 production Kotlin files excluding `buildSrc`, which holds 13 rather than 14 Kotlin files, giving 898 rather than 899 including it; 42 JSON fixtures; five resource directories of which four are golden-fixture directories once the Gradle consumer project is excluded; 2965 tests with zero failures, errors or skips across 399 result files | The `find` and aggregation commands recorded in P3-10 | Every count in this document travels with its counting rule, so no later section can restate a figure whose rule differs from the one that produced it [A2\|OBSERVED] |
| D8 | `AGENTS.md` describes the current module set | `AGENTS.md:L10` names a module `sdk` that does not exist; the real modules are `sdk-api`, `sdk-common` and `sdk-ext` | `AGENTS.md:L10`; `settings.gradle.kts:L22-L24` | ADR-01 names modules from the settings file, never from the guidance table, and `M-26` corrects the documentation defect [A6\|OBSERVED] |
| D9 | The contributor guide's Android package levels match the build | The guide requests `platforms;android-34` and `build-tools;30.0.3` while the catalog compiles against SDK 36 | `CONTRIBUTING.md:L14-L15`; `gradle/libs.versions.toml:L5` | Android environment claims cite the catalog, and a contributor following only the guide cannot execute the Android targets in the full build invocation [A6\|OBSERVED] |
| D10 | A single publish and integration flow | Three forms coexist: the workflow publishes with `./gradlew publishToMavenLocal -PsnapshotPublish=true --stacktrace` and then runs `./gradlew :gradle-integration-test:test --stacktrace`; the contributor guide publishes with `./gradlew publishToMavenLocal` and then runs the plain build task at `CONTRIBUTING.md:L41`, which is not the continuous-integration invocation and omits every additional target; the executed baseline uses `./gradlew publishToMavenLocal -PsnapshotPublish=true -Psigning.skip=true --stacktrace` followed by the separate `./gradlew :gradle-integration-test:test --stacktrace` | `.github/workflows/ci-build.yml:L78,L81`; `CONTRIBUTING.md:L32-L42,L44-L45`; the invocations recorded in P3-11 | G3-06 fixes the executed pair as the gate form. The signing skip is sanctioned by `CONTRIBUTING.md:L44-L45` and honoured by `buildSrc/src/main/kotlin/io/opentelemetry/kotlin/PublishConfig.kt:L12-L14`, which signs only when the property is absent, whereas the workflow form omits it and therefore requires a key [A2\|OBSERVED] |
| D11 | An out-of-repository destination beyond every checkout, asserted on the premise that a repository-root file could only ever be an untracked artifact and would fail the delivery self-check | The premise is false and the instruction that rested on it has been revoked by the reviewing maintainer. `git check-ignore -v metrics-implementation-plan.md` produces no output and exits 1, so no pattern in `.gitignore:L1-L11` covers the path and it is a legal tracked path; the file is committed at the repository root and appears in the branch's tracked diff with a non-zero added-line count, where an out-of-repository placement left that diff empty and delivered nothing | `.gitignore:L1-L11`; `git check-ignore -v metrics-implementation-plan.md` (no output, exit 1); `git ls-files metrics-implementation-plan.md`; `git diff --numstat origin/blitzy-otel-kotlin...HEAD` | Tracked repository-root placement governs, P3-02 records the observed destination, and the ranked question that asked whether an out-of-repository artifact would be collected at all is closed rather than carried forward. No section of this document may state that the deliverable lives outside the clones, and no rule may be read as licence to delete, move, rename, relocate or ignore it |

## 2.2 Reference register

Every load-bearing claim in this document resolves to one row here, so closure is checkable by locator rather than by reading [A5|DECIDED].

| Claim | Locator or producing command | Rank and status |
| --- | --- | --- |
| 39 Gradle projects from 37 explicit include entries; no metrics module; absent instrumentation directory | `settings.gradle.kts:L16-L57`; `./gradlew -q projects`; `ls -d instrumentation` | A2 OBSERVED |
| `Meter` is a member-less interface whose documentation promises factory methods | `api/src/commonMain/kotlin/io/opentelemetry/kotlin/metrics/Meter.kt:L9,L16-L18`; `api/api/jvm/api.api:L249-L250` | A1 OBSERVED, A3 OBSERVED |
| `MeterProvider.getMeter` takes a name plus three defaulted parameters | `api/src/commonMain/kotlin/io/opentelemetry/kotlin/metrics/MeterProvider.kt:L22-L28`; `api/api/jvm/api.api:L252-L258` | A1 OBSERVED, A3 OBSERVED |
| `Instrument` orders name, unit, description; `SynchronousInstrument.enabled()` has no default body; `AsynchronousInstrument` is `AutoCloseable` and carries the callback-registration gap marker | `Instrument.kt:L16-L32`; `SynchronousInstrument.kt:L22`; `AsynchronousInstrument.kt:L18-L19` | A3 OBSERVED |
| Observable measurements are split by numeric type with typed overloads rather than a generic method | `ObservableLongMeasurement.kt:L14,L19`; `ObservableDoubleMeasurement.kt:L14,L19`; `api/api/jvm/api.api:L260-L268` | A1 OBSERVED, A3 OBSERVED |
| All eight metrics API files carry both stability annotations | `api/src/commonMain/kotlin/io/opentelemetry/kotlin/metrics/Meter.kt:L16-L17`; `api/src/commonMain/kotlin/io/opentelemetry/kotlin/metrics/MeterProvider.kt:L12-L13`; the remaining six files of that package at the same positions | A3 OBSERVED |
| `AttributesMutator` is 72 lines with ten setters, no read accessors and no equality contract | `api/src/commonMain/kotlin/io/opentelemetry/kotlin/attributes/AttributesMutator.kt:L11,L17,L23,L29,L35,L41,L47,L53,L59,L65,L71`; `wc -l` on the same file | A3 OBSERVED |
| Four `Meter` implementors exist and all break on any added member | `MeterImpl.kt:L9`; `NoopMeter.kt:L11`; `FakeMeter.kt:L5`; `MeterAdapter.kt:L7-L9` including the unused-property suppression at L8 | A3 OBSERVED |
| `MeterProviderImpl` is a real provider with scope caching, an empty-name diagnostic, and post-shutdown noop behaviour | `MeterProviderImpl.kt:L19-L21,L23-L30,L38-L44,L46-L50`; `sdk-common/src/commonMain/kotlin/io/opentelemetry/kotlin/export/ShutdownState.kt:L20` | A3 OBSERVED |
| `MetricsConfig` currently carries only a resource and an error handler | `implementation/src/commonMain/kotlin/io/opentelemetry/kotlin/init/config/MetricsConfig.kt:L11-L22`; `MeterProviderConfigImpl.kt:L12-L15` | A3 OBSERVED |
| The meter configuration DSL is already public and empty, while its logging counterpart declares three members | `sdk-api/src/commonMain/kotlin/io/opentelemetry/kotlin/init/MeterProviderConfigDsl.kt:L8-L10`; `LoggerProviderConfigDsl.kt:L12-L28`; `sdk-api/api/jvm/sdk-api.api:L161-L162,L170` | A1 OBSERVED, A3 OBSERVED |
| Lifecycle is suspend plus a sealed result, not a completable result code | `sdk-api/src/commonMain/kotlin/io/opentelemetry/kotlin/export/TelemetryCloseable.kt:L15,L20`; `OperationResultCode.kt:L9,L14,L19` | A3 OBSERVED |
| The facade combines all three providers under a single 3000 ms timeout with all-success semantics and maps any throwable to failure | `implementation/src/commonMain/kotlin/io/opentelemetry/kotlin/OpenTelemetryImpl.kt:L25,L36,L41-L55,L57-L74,L76-L87` | A3 OBSERVED |
| Shutdown is idempotent and non-locking; composite closure requires every delegate to succeed | `sdk-common/src/commonMain/kotlin/io/opentelemetry/kotlin/export/MutableShutdownState.kt:L14-L16,L21-L23,L30-L39,L42`; `CompositeTelemetryCloseable.kt:L8,L16-L30,L33-L34` | A3 OBSERVED |
| Errors route to a functional error-handler interface whose contract forbids destabilising the host | `sdk-api/src/commonMain/kotlin/io/opentelemetry/kotlin/error/SdkErrorHandler.kt:L6,L9,L14`; `AGENTS.md:L29-L39` | A3 OBSERVED, A6 OBSERVED |
| The push-processor shape is exactly eight files per signal for both logging and tracing | `ls exporters-core/src/commonMain/kotlin/io/opentelemetry/kotlin/logging/export/` and the tracing sibling each return 8 entries | A3 OBSERVED |
| Batch defaults supply the flush and shutdown timeouts the provider already uses | `exporters-core/src/commonMain/kotlin/io/opentelemetry/kotlin/export/BatchTelemetryDefaults.kt:L14,L20,L26,L31,L36,L41,L46` | A3 OBSERVED |
| The OTLP endpoint enum is internal and holds only two entries | `exporters-otlp/src/commonMain/kotlin/io/opentelemetry/kotlin/export/OtlpEndpoint.kt:L3-L6` | A3 OBSERVED |
| A generic retry and transport seam already exists with a suspend export action and a 5000 ms shutdown timeout | `exporters-otlp/src/commonMain/kotlin/io/opentelemetry/kotlin/export/TelemetryExporter.kt:L13,L15-L22,L32-L40,L42-L71,L78-L81,L85-L89` | A3 OBSERVED |
| Per-target HTTP engines follow the platform-suffixed actual convention | `exporters-otlp/src/commonMain/kotlin/io/opentelemetry/kotlin/export/HttpClientInstance.kt:L28` with actuals `HttpClientInstance.jvm.kt`, `HttpClientInstance.js.kt` and `HttpClientInstance.apple.kt` | A3 OBSERVED |
| Per-signal exporter shapes are two OTLP files, three protocol files and three in-memory files | `ls exporters-otlp/src/commonMain/kotlin/io/opentelemetry/kotlin/logging/export/`, the `exporters-protobuf` sibling, and the `exporters-in-memory` sibling | A3 OBSERVED |
| The collector fake routes only two signal paths and raises an error on any other | `smoke-test/src/commonTest/kotlin/io/opentelemetry/kotlin/smoketest/FakeOtlpServer.kt:L32,L35-L37` | A3 OBSERVED |
| Exactly eight environment variables are recognised and all are limits | `config-envar/src/commonMain/kotlin/io/opentelemetry/kotlin/config/envar/EnvVarConstants.kt:L11-L17,L19-L36` | A3 OBSERVED |
| The layered behaviour model carries a tracer-provider slot only | `behavior/src/commonMain/kotlin/io/opentelemetry/kotlin/behavior/OpenTelemetryBehavior.kt:L11-L18` | A3 OBSERVED |
| Nine platform primitives are declared with per-target actuals, and the platform layer states the standard-library atomics position itself | `platform-implementations/src/commonMain/kotlin/io/opentelemetry/kotlin/AtomicLong.kt:L11-L13,L15-L55`; `AtomicLong.jvm.kt:L18-L19`; `AtomicLong.js.kt:L11-L13`; `platform-implementations/src/appleMain/kotlin/io/opentelemetry/kotlin/AtomicLong.kt:L12,L45-L52`; `ReentrantReadWriteLock.kt`; `ThreadSafeMap.kt` | A3 OBSERVED |
| Shared behaviour tests for platform primitives already exist in the common test source set | `platform-implementations/src/commonTest/kotlin/io/opentelemetry/kotlin/` holds `AtomicLongTest.kt`, `ReentrantReadWriteLockTest.kt`, `ThreadLocalTest.kt` and `ThreadSafeMapTest.kt` | A3 OBSERVED |
| The differential harness is an abstract common rule with one back end per implementation, the reference back end confined to a JVM and Android test source set | `integration-test/src/commonMain/kotlin/io/opentelemetry/kotlin/framework/OtelKotlinTestRule.kt:L27,L33,L43,L46-L47,L52-L65`; `implementation/src/commonTest/kotlin/io/opentelemetry/kotlin/integration/test/IntegrationTestHarness.kt:L14-L24`; `compat/src/jvmTest/kotlin/io/opentelemetry/kotlin/framework/OtelKotlinHarness.kt:L12-L29` | A3 OBSERVED |
| Multiplatform fixture loading is already solved with platform-suffixed actuals | `test-fakes/src/commonMain/kotlin/io/opentelemetry/kotlin/framework/GoldenFileDataLoader.kt` with `GoldenFileDataLoader.jvm.kt`, `GoldenFileDataLoader.apple.kt` and `GoldenFileDataLoader.js.kt`; `integration-test/src/commonMain/kotlin/io/opentelemetry/kotlin/framework/GoldenFileDataLoader.kt` | A3 OBSERVED |
| Common test resources are staged onto the iOS simulator | `implementation/build.gradle.kts:L64-L71` | A2 OBSERVED |
| Binary-compatibility validation toggles validation only, leaving experimental declarations dump-visible | `buildSrc/src/main/kotlin/io/opentelemetry/kotlin/BinaryCompatConfig.kt:L6-L10` | A2 OBSERVED |
| The `api` module is excluded from the project-wide opt-in | `api/gradle.properties:L1`; `buildSrc/src/main/kotlin/io/opentelemetry/kotlin/OptInConfig.kt:L6-L18` | A2 OBSERVED |
| Only the attributes component is stable and metrics is in development, and stable components must carry no experimental surface | `api/README.md:L26-L37` | A6 OBSERVED |
| The reference implementation at the pinned tag holds 176 production files totalling 541431 bytes, with a seven-file concurrency package | `git describe --tags` in the reference clone returns `v1.65.0` resolving to `7bc11edca518d4de168230c84a71484a66cbac40`; `find sdk/metrics/src/main/java/io/opentelemetry/sdk/metrics -name '*.java' \| wc -l` returns 176 and the byte sum returns 541431; the concurrency directory listing returns 7 files | A4 OBSERVED |
| The reference test estate is 89 files and 767788 bytes for the primary test source set, or 93 files and 837652 bytes once the incubating test source set is included | `find sdk/metrics/src/test -name '*.java'` and the same for `src/testIncubating`, with byte sums | A4 OBSERVED |
| Reference subpackage sizes concentrate complexity in state, data, aggregator, exemplar and view | Per-directory counts of 35 root, 22 data, 12 export, 3 internal, 19 aggregator, 7 concurrent, 23 internal data, 4 debug, 3 descriptor, 16 exemplar, 1 internal export, 17 state and 14 view, summing to 176 | A4 OBSERVED |
| A six-file sample of the reference tree measures 2004 lines over 74502 bytes, giving approximately 37.2 bytes per line and an extrapolated tree of roughly 14560 lines | `wc -l` and `stat -c%s` over the delta synchronous storage, meter, long adder, filtered attributes, periodic reader and exponential histogram aggregator files | A4 DERIVED |
| The metrics compliance table has 14 columns with Kotlin last, 88 data rows, 10 rows carrying an optional value and 78 not, all 88 Kotlin cells unsatisfied, and 75 of 88 satisfied for Java | The pinned matrix at `spec-compliance-matrix.md:L101,L103-L104` with data rows L105-L192, parsed by column index | A4 OBSERVED |
| The three committed optional rows are wildcard view selection, exemplar-reservoir configuration on a view, and more than one view per instrument | The same matrix at L145, L149 and L150 | A4 OBSERVED |
| The four rows excluded by non-goals are the deprecated instrumentation-library row and the three metric-filter rows scoped to a metric producer | The same matrix at L138, L185, L186 and L187 | A4 DERIVED |

## 2.3 HC-01 through HC-13 traceability

| ID | Obligation in one line | Enforced by | Evidence |
| --- | --- | --- | --- |
| HC-01 | No stakeholder contact is used or claimed as evidence | Sections 2, 5 and 8, which admit only repository evidence and the pinned reference clone as resolution mechanisms | The meeting and chat channels named at `README.md:L27-L30` appear nowhere in this document's evidence chain, and the upstream push URL is disabled per P3-01 [A6\|OBSERVED] |
| HC-02 | The current run is documentation only | The eight-section contract of this file, with no accompanying artifact | P3-01 records an empty `git status --porcelain` throughout, and Section 4 describes units to be executed later rather than work performed [A2\|OBSERVED] |
| HC-03 | Analysis spans all modules while mutation stays isolated | Section 2's read-only ledger and Section 4's explicit path scoping | Every Section 2 row cites a read locator or a command; no row records a write [A5\|DECIDED] |
| HC-04 | Dependencies, catalog, wrapper and lockfiles are immutable | G3-05, and the absence of any dependency column in Section 4 | P3-07 records the catalog at 108 unchanged lines with no atomics entry, and ADR-06 therefore selects a design that needs no addition [A2\|OBSERVED] |
| HC-05 | Module creation is an architecture outcome, not a current action | ADR-01, which elects existing modules | P3-03 fixes the 39-project topology as an invariant [A2\|OBSERVED] |
| HC-06 | No temporal planning of any kind | Sections 4, 5 and 7, none of which carries a duration, date or owner column | `RELEASING.md:L19-L20` is cited as a discretionary convention and is never converted into a schedule [A6\|OBSERVED] |
| HC-07 | Every claim carries a locator, a rank and an epistemic status, and negative findings are explicit | The labelling convention declared at the head of Section 2 and the register in Section 2.2 | Negative findings are stated positively: zero `.klib.api` dumps in P3-05, zero atomics entries in P3-07, zero metrics SDK types in the SDK dump in P3-05 [A1\|OBSERVED] |
| HC-08 | Every reference to the full build prints the complete invocation, and publish stays separate from integration | G3-04 and G3-06, and every cell in Sections 4 through 7 that invokes either | P3-11 records both the full invocation and the separated pair, with D10 explaining why the pair is not combined [A2\|OBSERVED] |
| HC-09 | Both verification spikes are local, unpushed, unmerged, recorded and deleted | Section 5's two spike protocols and G3-11 | P3-01 records that no spike branch exists on the baseline, so any later branch is attributable [A2\|OBSERVED] |
| HC-10 | Gates are satisfied rather than weakened | G3-03 at 1.00 percentage point, G3-04 with unlowered heaps, and G3-05 on discoverability | P3-11 records that the out-of-repository heap override was moved aside so the checked-in premise governs, and P3-08 records that the tighter window governs [A2\|OBSERVED] |
| HC-11 | No credential material of any kind is transcribed | Section 2's evidence discipline | P3-01 and P3-12 state only that credential-bearing metadata was observed and excluded; no value and no secret identifier name appears in this document [A2\|OBSERVED] |
| HC-12 | Manual-review disclosure travels by commit trailer rather than by template edit | `M-26` and G3-11 | The pull-request template holds exactly two sections and no disclosure field at `.github/PULL_REQUEST_TEMPLATE.md:L1-L7`, while manual review of generated code is required by `AGENTS.md:L26` [A6\|OBSERVED] |
| HC-13 | The end state is complete, clean and traceable | Section 1's arithmetic, Section 4's gate mapping, and G3-11 | The compliance identity holds because 0 satisfied plus 4 excluded plus 77 outstanding equals the 81-row denominator, with the 7 remaining optional rows counted separately and never inside the excluded figure [A4\|DERIVED] |


## 2.4 Executed command transcripts T-01 through T-32

Every figure this document cites resolves either to a `file:locator` or to one of the thirty-two transcripts below. Each carries the
command exactly as invoked, its exit status, and its output verbatim; where an output runs to thousands of lines the retained excerpt is
labelled as an excerpt and the omission is stated rather than silently trimmed [A2|OBSERVED]. Nothing in this section is reconstructed from
memory, and no figure appears anywhere in this document that was not produced by one of these commands or read from a cited file. Credential
material is never transcribed: where a command's output would contain an embedded credential the credential is replaced by `<redacted>`, and
only the presence of the credential is recorded [A2|DECIDED].

**T-01 — checkout revision.**

```console
$ git rev-parse HEAD
9dc0ba051a2583b9419d07ff5e69ad208c69f3b7
exit=0
```

**T-02 — checked-out branch. HEAD is never moved by this authoring, and no `git checkout` or `git switch` is invoked.**

```console
$ git rev-parse --abbrev-ref HEAD
blitzy-86709747-5ffd-411c-9b4d-ee6943453030
exit=0
```

**T-03 — working tree before the deliverable is written. Empty output is the observation.**

```console
$ git status --porcelain
exit=0
```

**T-04 — remotes. The `origin` fetch and push URLs carry embedded credential material, which is recorded as present and never transcribed.**

```console
$ git remote -v
origin	https://<redacted>@github.com/Blitzy-Sandbox/blitzy-opentelemetry-kotlin.git (fetch)
origin	https://<redacted>@github.com/Blitzy-Sandbox/blitzy-opentelemetry-kotlin.git (push)
upstream	https://github.com/open-telemetry/opentelemetry-kotlin.git (fetch)
upstream	https://github.com/open-telemetry/opentelemetry-kotlin.git (push)
exit=0
```

**T-05 — divergence from the branch base and from upstream, which is what makes the rebase cadence in Section 5 a measured concern rather than a precaution.**

```console
$ git rev-list --count origin/blitzy-otel-kotlin..HEAD
3
$ git rev-list --count HEAD..upstream/main
15
exit=0
```

**T-06 — the deliverable's path is not ignored. No output and exit 1 together are the evidence, and they are what P3-02 and D11 rest on.**

```console
$ git check-ignore -v metrics-implementation-plan.md
exit=1
```

**T-07 — the complete ignore set, which contains no Markdown pattern and no entry for this file.**

```console
$ cat -n .gitignore
     1	*.iml
     2	.gradle
     3	.idea
     4	.DS_Store
     5	build
     6	captures
     7	.externalNativeBuild
     8	.cxx
     9	local.properties
    10	xcuserdata
    11	.kotlin
exit=0
```

**T-08 — the pinned distribution downloaded and its checksum validated under `retries=0`.**

```console
$ ./gradlew --version

------------------------------------------------------------
Gradle 9.7.0
------------------------------------------------------------

Build time:    2026-08-06 14:07:35 UTC
Revision:      3defbfc59d757b873d787b2261de5c7f8a00970a

Kotlin:        2.4.0
Groovy:        4.0.32
Ant:           Apache Ant(TM) version 1.10.17 compiled on April 6 2026
Launcher JVM:  21.0.11 (Ubuntu 21.0.11+10-1-25.10.2-Ubuntu)
Daemon JVM:    /usr/lib/jvm/java-21-openjdk-amd64 (no Daemon JVM specified, using current Java home)
OS:            Linux 6.12.85+ amd64

exit=0
```

**T-09 — toolchain discoverability, which is the check that matters rather than whether a package was installed. JDK 11 is the declared
toolchain at `buildSrc/src/main/kotlin/io/opentelemetry/kotlin/KotlinConfig.kt:L11,L17` and no toolchain resolver is registered, so its
presence here is load-bearing.**

```console
$ ./gradlew -q javaToolchains

 + Options
     | Auto-detection:     Enabled
     | Auto-download:      Enabled

 + Ubuntu JDK 11 (11.0.31+11-post-1ubuntu1-25.10.2-Ubuntu)
     | Location:           /usr/lib/jvm/java-11-openjdk-amd64
     | Language Version:   11
     | Vendor:             Ubuntu
     | Architecture:       amd64
     | Is JDK:             true
     | Detected by:        Common Linux Locations

 + Ubuntu JDK 17 (17.0.19+10-1-25.10.2-Ubuntu)
     | Location:           /usr/lib/jvm/java-17-openjdk-amd64
     | Language Version:   17
     | Vendor:             Ubuntu
     | Architecture:       amd64
     | Is JDK:             true
     | Detected by:        Common Linux Locations

 + Ubuntu JDK 21 (21.0.11+10-1-25.10.2-Ubuntu)
     | Location:           /usr/lib/jvm/java-21-openjdk-amd64
     | Language Version:   21
     | Vendor:             Ubuntu
     | Architecture:       amd64
     | Is JDK:             true
     | Detected by:        Current JVM

exit=0
```

**T-10 — the canonical build, first invocation, configuration cache stored. The command is the one `.github/workflows/ci-build.yml:L42`
runs on Linux and is written out in full here and at every other reference site. Excerpt: the final four lines of 4,600 lines of task
output; the omitted lines are per-task progress and carry no figure this document cites.**

```console
$ ./gradlew build :benchmark-android:assemble :benchmark-android:compileReleaseAndroidTestKotlin :benchmark-jvm:assemble :examples:example-app-android:assembleRelease koverXmlReport -x :gradle-integration-test:test --stacktrace
[…4,596 lines of task output omitted…]

BUILD SUCCESSFUL in 21s
2186 actionable tasks: 102 executed, 2084 up-to-date
Configuration cache entry stored.
exit=0
```

**T-11 — the same command invoked a second time with nothing changed, demonstrating configuration-cache reuse rather than store. This is the
only way to establish the cache gate, and it matters because `gradle.properties:L5` — the corrected locator D4 records, against an asserted L6 that in fact carries parallel execution — makes any incompatibility a hard failure rather than a
warning.**

```console
$ ./gradlew build :benchmark-android:assemble :benchmark-android:compileReleaseAndroidTestKotlin :benchmark-jvm:assemble :examples:example-app-android:assembleRelease koverXmlReport -x :gradle-integration-test:test --stacktrace
Reusing configuration cache.
[…task output omitted…]

BUILD SUCCESSFUL in 4s
2182 actionable tasks: 102 executed, 2080 up-to-date
Configuration cache entry reused.
exit=0
```

**T-12 — the two gates every change unit must satisfy, confirmed clean on a tree whose only addition is this document.**

```console
$ ./gradlew apiCheck detekt --stacktrace
[…task output omitted…]

BUILD SUCCESSFUL in 4s
265 actionable tasks: 47 executed, 218 up-to-date
exit=0
```

**T-13 — coverage, read from the report the build actually produces. Both the rate and the measurable line total are required, because the
coverage window in G3-03 is derived from the total rather than asserted.**

```console
$ python3 -c "import xml.etree.ElementTree as ET; r=ET.parse('build/reports/kover/report.xml').getroot(); \
c={x.attrib['type']:(int(x.attrib['covered']),int(x.attrib['missed'])) for x in r if x.tag=='counter'}; \
print('\n'.join(f\"{k} {v[0]}/{v[0]+v[1]} = {100*v[0]/(v[0]+v[1]):.2f}%\" for k,v in c.items()))"
INSTRUCTION 25618/26938 = 95.10%
BRANCH 1714/1955 = 87.67%
LINE 4864/5153 = 94.39%
exit=0
```

**T-14 — the coverage artifact the continuous-integration workflow uploads does not exist. `.github/workflows/ci-build.yml:L50` names
`build/reports/kover/reportRelease.xml`; the build writes `report.xml` and nothing else. This is a negative finding stated rather than
omitted, and `M-01` exists to repair it.**

```console
$ ls -la build/reports/kover/
total 872
drwxr-sr-x 2 root root   4096 .
drwxr-sr-x 5 root root   4096 ..
-rw-r--r-- 1 root root 881725 report.xml
-rw-r--r-- 1 root root      0 verify.err
$ ls build/reports/kover/reportRelease.xml
ls: cannot access 'build/reports/kover/reportRelease.xml': No such file or directory
exit=2
```

**T-15 — the whole test estate, aggregated from every JUnit result file the canonical build produced. Zero skips is itself a finding: the
repository engineers determinism rather than tolerating flakes, which is the premise Section 6's oracles depend on.**

```console
$ python3 -c "import glob,xml.etree.ElementTree as ET; t=f=e=s=0; n=0
for p in glob.glob('*/build/test-results/*/TEST-*.xml'):
    a=ET.parse(p).getroot().attrib; t+=int(a['tests']); f+=int(a['failures']); e+=int(a['errors']); s+=int(a.get('skipped',0)); n+=1
print(f'tests={t} failures={f} errors={e} skipped={s} (result files={n})')"
tests=2965 failures=0 errors=0 skipped=0 (result files=399)
exit=0
```

**T-16 — the dump inventory. Zero klib baselines is the negative finding that makes the authority ladder's klib tiebreak inoperative on this
baseline, and no unit may claim klib compatibility evidence while it stays zero.**

```console
$ git ls-files '*.api' | wc -l
27
$ git ls-files '*.klib.api' | wc -l
0
exit=0
```

**T-17 — the rank-one disagreement, quantified. Two checked-in dumps for the same module contradict each other about whether the metrics
package exists at all, and the ladder offers no JVM-versus-Android tiebreak. ADR-13 rules on it and `M-02` isolates it.**

```console
$ wc -l api/api/jvm/api.api api/api/android/api.api
  402 api/api/jvm/api.api
  462 api/api/android/api.api
$ grep -c 'metrics' api/api/jvm/api.api
12
$ grep -c 'metrics' api/api/android/api.api
0
$ grep -c 'getMeterProvider' api/api/jvm/api.api
1
$ grep -c 'getMeterProvider' api/api/android/api.api
0
$ grep -n 'io/opentelemetry/kotlin/metrics' api/api/jvm/api.api | sed -n '1p;$p'
8:	public abstract fun getMeterProvider ()Lio/opentelemetry/kotlin/metrics/MeterProvider;
273:public abstract interface class io/opentelemetry/kotlin/metrics/SynchronousInstrument : io/opentelemetry/kotlin/metrics/Instrument {
exit=0
```

**T-18 — the project topology. One `include(` call carries 37 explicit paths, and `includeFromDir("instrumentation")` walks a directory that
does not exist and contributes nothing, so the total is 39 with the root project and the `examples` container. No metrics module exists.**

```console
$ grep -n 'include(' settings.gradle.kts | head -1
17:include(
$ grep -n 'includeFromDir' settings.gradle.kts | head -1
57:includeFromDir("instrumentation")
$ ls -d instrumentation
ls: cannot access 'instrumentation': No such file or directory
exit=2
```

**T-19 — the metrics footprint, which is larger than the eight files of `api/src/commonMain` and is the reason `M-05` exists as coordinated
parity work rather than as cleanup.**

```console
$ git ls-files | grep -E '/metrics/.*\.kt$' | sed 's#/src/.*##' | sort | uniq -c
      8 api
      7 compat
      3 implementation
      2 noop
      2 test-fakes
$ git ls-files | grep -cE '/metrics/.*\.kt$'
22
exit=0
```

**T-20 — the version catalog, and the negative finding that decides ADR-06. A case-insensitive search for any atomics coordinate returns
nothing, so no striped-adder library is available to adopt even if the language floor permitted one.**

```console
$ wc -l gradle/libs.versions.toml
108 gradle/libs.versions.toml
$ grep -ci atomic gradle/libs.versions.toml
0
exit=0
```

**T-21 — the generated configuration surface. Every metrics configuration concept already has a generated model, and all of them are tracked
source rather than build output, which is why `M-15` wires models that exist rather than authoring them.**

```console
$ git ls-files 'config-schema/src/commonMain/kotlin/io/opentelemetry/kotlin/config/schema/model/*.kt' | wc -l
113
exit=0
```

**T-22 — the test estate's structure. Each count is definition-sensitive and is reported with the rule that produced it, because a bare
figure here would not be reproducible.**

```console
$ find . -path ./build -prune -o -type d -name '*Test' -print | grep -E '/src/[a-zA-Z]+Test$' | wc -l
28
$ git ls-files | grep -cE '/src/[a-zA-Z]*[Tt]est/.*\.kt$'
287
$ git ls-files | grep -cE '/src/[a-zA-Z]*[Mm]ain/.*\.kt$'
898
exit=0
```

**T-23 — golden-fixture inventory. The two metrics-relevant directories are the ones `M-25` extends, and the deliberate overlap of scenario
names between back ends is what makes the differential a comparison rather than two independent suites.**

```console
$ git ls-files | grep -E '/resources/' | sed 's#/resources/.*#/resources#' | sort | uniq -c
     20 compat/src/jvmTest/resources
      1 config-yaml/src/commonTest/resources
      4 exporters-core/src/commonTest/resources
     11 gradle-integration-test/src/test/resources
     22 implementation/src/commonTest/resources
exit=0
```

**T-24 — capacity, measured before the first build by reading the kernel's own figures and every cgroup limit path. The commonly quoted
"3842 MB" is a unit mislabel of a gibibyte figure; the measured total is three orders of magnitude above it and no cgroup limit exists.**

```console
$ grep -E '^MemTotal|^MemAvailable|^SwapTotal|^SwapFree' /proc/meminfo
MemTotal:       4029526812 kB
MemAvailable:   3952084312 kB
SwapTotal:      25165816 kB
SwapFree:       25165816 kB
$ cat /sys/fs/cgroup/memory.max /sys/fs/cgroup/memory/memory.limit_in_bytes /sys/fs/cgroup/memory.high
cat: /sys/fs/cgroup/memory.max: No such file or directory
cat: /sys/fs/cgroup/memory/memory.limit_in_bytes: No such file or directory
cat: /sys/fs/cgroup/memory.high: No such file or directory
$ nproc
4
exit=0
```

**T-25 — the qualification on T-10 through T-12. A heap override in the Gradle user home, outside the repository, sizes the daemons well
below the checked-in premise and is deliberately left in place, so the transcripts above were measured under it. Recording this is what keeps
"the build is green" from being read as "the build is green under an 8 GiB daemon".**

```console
$ sed -n '6,8p' ~/.gradle/gradle.properties
org.gradle.jvmargs=-Xmx1800m -XX:+UseParallelGC -XX:MaxMetaspaceSize=768m -Dfile.encoding=UTF-8
kotlin.daemon.jvmargs=-Xmx1800m -XX:MaxMetaspaceSize=512m
org.gradle.workers.max=2
exit=0
```

**T-26 — the checked-in heap premise is unaltered. Empty output is the observation, and it is what lets G3-04 keep requiring an unlowered
heap even though the baseline above ran under a lowered one.**

```console
$ git diff --stat -- gradle.properties
exit=0
```

**T-27 — the reference implementation, pinned. Every reference measurement below is reproducible only at this revision, which is why the tag
rather than a default branch is used.**

```console
$ git -C ../reference-otel-java describe --tags
v1.65.0
$ git -C ../reference-otel-java rev-parse HEAD
7bc11edca518d4de168230c84a71484a66cbac40
exit=0
```

**T-28 — the reference production tree, which is the port's source and the basis for the netting in Section 4.**

```console
$ find ../reference-otel-java/sdk/metrics/src/main/java -name '*.java' | wc -l
176
$ find ../reference-otel-java/sdk/metrics/src/main/java -name '*.java' -printf '%s\n' | awk '{s+=$1} END {print s}'
541431
$ find ../reference-otel-java/sdk/metrics/src/main/java -name '*.java' -exec cat {} + | wc -l
16406
exit=0
```

**T-29 — the reference test estate, and a deviation recorded loudly. The asserted pair was 93 files and 837652 bytes; the primary test
source set measures 89 files and 767788 bytes, and adding the `jmh` source set gives 97 files and 796567 bytes, so neither reading
reproduces the assertion. Six source sets exist. The observed values govern, and the test-to-production ratio Section 6 cites is computed
from them.**

```console
$ find ../reference-otel-java/sdk/metrics/src/test -name '*.java' | wc -l
89
$ find ../reference-otel-java/sdk/metrics/src/test -name '*.java' -printf '%s\n' | awk '{s+=$1} END {print s}'
767788
$ find ../reference-otel-java/sdk/metrics/src/test -name '*.java' -exec cat {} + | wc -l
18468
$ ls ../reference-otel-java/sdk/metrics/src/
debugEnabledTest
jmh
jmhBasedTest
main
test
testIncubating
exit=0
```

**T-30 — the concurrency surface the platform layer must replace, file by file. It is small in line terms and architecturally the hardest
part of the port, because each primitive needs one common declaration plus actuals for three target families.**

```console
$ ls -l ../reference-otel-java/sdk/metrics/src/main/java/io/opentelemetry/sdk/metrics/internal/concurrent/ | awk '{print $5, $9}'
1549 AdderUtil.java
1171 AtomicLongDoubleAdder.java
907 AtomicLongLongAdder.java
2188 DoubleAdder.java
802 JreDoubleAdder.java
775 JreLongAdder.java
2478 LongAdder.java
exit=0
```

**T-31 — where the reference tree's complexity actually sits. These counts are the work-package boundaries Section 4 uses, rather than an
even split across subpackages.**

```console
$ M=reference-otel-java/sdk/metrics/src/main/java/io/opentelemetry/sdk/metrics
$ for d in "$M" "$M"/*/ "$M"/internal/*/; do printf '%s %s\n' "$(find "$d" -maxdepth 1 -name '*.java' | wc -l)" "${d#$M/}"; done
35 metrics (root)
22 metrics/data
12 metrics/export
3 metrics/internal
19 internal/aggregator
7 internal/concurrent
23 internal/data
4 internal/debug
3 internal/descriptor
16 internal/exemplar
1 internal/export
17 internal/state
14 internal/view
exit=0
```

**T-32 — the size calibration a six-file sample produces, used to net the reference tree down rather than to extrapolate it up.**

```console
$ for f in internal/state/DeltaSynchronousMetricStorage.java SdkMeter.java internal/concurrent/LongAdder.java \
    internal/view/FilteredAttributes.java export/PeriodicMetricReader.java \
    internal/aggregator/DoubleBase2ExponentialHistogramAggregator.java; do
    printf '%s lines=%s bytes=%s\n' "$f" "$(wc -l < $f)" "$(stat -c%s $f)"; done
internal/state/DeltaSynchronousMetricStorage.java lines=585 bytes=24239
SdkMeter.java lines=418 bytes=15584
internal/concurrent/LongAdder.java lines=105 bytes=2478
internal/view/FilteredAttributes.java lines=284 bytes=10116
export/PeriodicMetricReader.java lines=316 bytes=11056
internal/aggregator/DoubleBase2ExponentialHistogramAggregator.java lines=296 bytes=11029
exit=0
```

Two further reference sizes are cited by Section 4 and come from the same measurement:
`internal/state/AsynchronousMetricStorage.java` is 413 lines over 16297 bytes and `SdkMeterProvider.java` is 267 lines over 10386 bytes
[A4|OBSERVED]. The sample totals 2004 lines over 74502 bytes, which is 37.2 bytes per line [A4|DERIVED].

## 2.5 Repository evidence index

One row per area a later section reasons about, so that a reader can open the evidence rather than trust the claim. Every locator here was
resolved in this checkout at the revision T-01 records [A2|OBSERVED].

| Area | Locator | What it fixes for the plan |
| --- | --- | --- |
| Declared toolchain, target set and compiler strictness | `buildSrc/src/main/kotlin/io/opentelemetry/kotlin/KotlinConfig.kt:L10-L11,L17,L39-L40,L89-L92` | JDK 11 declared; exactly two Apple targets and no `iosX64`; language and API level pinned to Kotlin 2.0; warnings are errors; `-Xexpect-actual-classes` and `-Xsuppress-version-warnings` set |
| Build memory premise and configuration-cache strictness | `gradle.properties:L2-L7` | An 8 GiB Gradle daemon, a separate 8 GiB Kotlin daemon and 4 GiB metaspace, with configuration-cache problems failing the build at L5 |
| Enforced coverage gate | `codecov.yml:L5-L7,L10` | Project status `auto` with a 1% threshold and an adjusted base; patch target 50% — the numbers G3-03 and G3-02 use |
| Static-analysis budget and ceilings | `config/detekt/detekt.yml:L3,L10-L11,L14-L18,L26-L31,L35-L36,L37-L40` | Zero issues; 30 constructor and 31 function parameters; 31 functions per type; the three exception rules deliberately disabled; 140-character lines; trailing commas required |
| The canonical build command and the coverage upload path | `.github/workflows/ci-build.yml:L42,L50` | The merge gate's exact invocation, and the artifact path that does not exist |
| Reviewable-diff and error-handling doctrine | `AGENTS.md:L22-L27,L29-L39` | The 500-line guidance, the dump-before-change discipline, and the containment rules that shape ADR-08, ADR-11 and ADR-17 |
| Placement of platform code, interface-default guidance and publish separation | `CONTRIBUTING.md:L32-L42,L44-L45,L51-L60` | Why accumulation primitives go in `platform-implementations`, why default interface bodies are an open question rather than an assumption, and why publish and integration stay two invocations |
| Component stability | `api/README.md:L26,L28-L38` | Only `attributes` is stable and `metrics` is in development, so breaking changes to `Meter` and `MeterProvider` are already sanctioned |
| The empty meter and its contradicting documentation | `api/src/commonMain/kotlin/io/opentelemetry/kotlin/metrics/Meter.kt:L9,L16-L18` | The KDoc promises factory methods on an interface with no body — the defect `M-26` corrects |
| The attribute idiom | `api/src/commonMain/kotlin/io/opentelemetry/kotlin/attributes/AttributesMutator.kt:L11-L72`; `sdk-api/src/commonMain/kotlin/io/opentelemetry/kotlin/attributes/AttributeContainer.kt:L11` | A write-only receiver in `api` with no equality contract, and the readable snapshot one layer up in `sdk-api` — the root of ADR-05 |
| The callback gap marker | `api/src/commonMain/kotlin/io/opentelemetry/kotlin/metrics/AsynchronousInstrument.kt:L19` | An in-source note that after-creation callback registration is still to be added, which ADR-08 rules on |
| Lifecycle contract | `sdk-api/src/commonMain/kotlin/io/opentelemetry/kotlin/export/TelemetryCloseable.kt:L10-L21`; `sdk-api/src/commonMain/kotlin/io/opentelemetry/kotlin/export/OperationResultCode.kt:L9-L19` | Suspend plus sealed result, not a completable result code — every reader and exporter interface adopts it |
| The generic transport seam | `exporters-otlp/src/commonMain/kotlin/io/opentelemetry/kotlin/export/TelemetryExporter.kt:L15-L89`; `exporters-otlp/src/commonMain/kotlin/io/opentelemetry/kotlin/export/OtlpEndpoint.kt:L3-L6` | Retry, backoff and shutdown already exist; the endpoint enum holds exactly two entries and needs one more |
| Configuration surfaces that already exist | `sdk-api/src/commonMain/kotlin/io/opentelemetry/kotlin/init/MeterProviderConfigDsl.kt:L10`; `implementation/src/commonMain/kotlin/io/opentelemetry/kotlin/init/config/MetricsConfig.kt`; `behavior/src/commonMain/kotlin/io/opentelemetry/kotlin/behavior/OpenTelemetryBehavior.kt:L11-L18` | A public but empty meter DSL, a configuration holder to grow, and a behaviour model with only a tracer slot |
| The differential harness and its bound | `integration-test/src/commonMain/kotlin/io/opentelemetry/kotlin/framework/OtelKotlinTestRule.kt`; `compat/src/jvmTest/kotlin/io/opentelemetry/kotlin/framework/OtelKotlinHarness.kt` | The comparison already exists, and the reference-delegating back end lives in a JVM-and-Android module's test source set — the reach limit Section 6 states |
| Environment-variable surface | `config-envar/src/commonMain/kotlin/io/opentelemetry/kotlin/config/envar/EnvVarConstants.kt`; `config-envar/src/commonMain/kotlin/io/opentelemetry/kotlin/config/envar/model/EnvVarName.kt` | Every recognised variable today is a limit, so the metrics set in `M-22` is the first non-limit group |

## 2.6 The compliance denominator and its basis

The figure Section 1 carries is stated with its basis rather than as a bare fraction, because the two candidate denominators differ by three
committed deliverables [A4|DERIVED].

- The specification's metrics compliance matrix carries **88 data rows**; 10 carry a value in the optional column and 78 do not [A4|OBSERVED].
- Three of those ten optional rows are things this plan is required to design — wildcard support in view instrument-selection criteria, more
  than one view per instrument, and configuring the exemplar reservoir of a resulting metric stream — so reporting against 78 would silently
  omit three committed deliverables [A4|DERIVED].
- The denominator is therefore **81**: the 78 non-optional rows plus those three. The remaining seven optional rows are enumerated as out of
  scope, and the prescribed "of 78" wording is superseded on that basis, with the supersession stated here rather than applied silently
  [A5|DECIDED].
- Every Kotlin cell in all 88 rows is unsatisfied today, so the satisfied count is 0 [A4|OBSERVED]. Four rows are excluded by declared
  non-goals: the deprecated instrumentation-library row and the three metric-filter rows scoped to a non-goal surface [A4|DERIVED].
- The identity closes twice: 0 + 4 + 77 = 81, and 81 + 7 = 88 [A4|DERIVED].
- The Java column of the same matrix shows 75 satisfied and 13 not, which is the second bound Section 6 places on differential testing
  [A4|OBSERVED].

## 2.7 Non-goal dispositions

Fourteen areas are declared non-goals. Each is disposed of explicitly, because an area passed over in silence is indistinguishable from an
area forgotten [A5|DECIDED].

| Non-goal | Disposition | Why it is safe to exclude |
| --- | --- | --- |
| Prometheus exporter | Excluded; no unit designs it | The name appears only in generated configuration models with no implementation anywhere in the tree |
| Additional Kotlin Multiplatform targets | Excluded | Exactly two Apple targets, one JavaScript target and the JVM and Android targets are configured at `KotlinConfig.kt:L39-L40`; adding a target is a build change no unit makes |
| Instrumentation of any kind | Excluded | `includeFromDir("instrumentation")` at `settings.gradle.kts:L57` walks an absent directory and contributes zero projects |
| Logs work | Excluded, with one exception stated | `M-01` and the transport-seam repair touch shared logging paths, so both carry mandatory logs regression assertions rather than pretending the blast radius is zero |
| Traces work | Excluded, with the same exception | The same shared seam serves tracing; the same regression assertions apply |
| Profiles | Excluded | No profile surface exists in any module or dump |
| Compatibility façade beyond the minimum | Bounded rather than excluded | `M-05` extends the adapters only so far as keeping the canonical build green requires, and `M-25` uses the façade as a test back end rather than as a product surface |
| Declarative and YAML configuration beyond the metrics environment set | Excluded | The generated models already exist; only the environment-variable subset in `M-22` is wired, and no YAML path is added |
| Migration of the downstream Android distribution | Excluded | No downstream consumer is in this repository, and no coordinate changes |
| Publishing or coordinate changes | Excluded | `PublishConfig.kt` and the catalog are untouched by every unit |
| Any rename or namespace change | Excluded | Renames would produce dump churn unrelated to metrics and are forbidden by the change-unit budget's intent |
| Outbound contribution to any repository | Excluded | HC-01; the upstream remote is read-only and no unit contacts a person or project |
| Repairing the non-compiling getting-started sample | Excluded | It lives on the project website rather than in this checkout, so no locator in this repository can bound it |
| Self-observability metrics | Excluded | It is a declared non-goal and also the metrics milestone's tenth member, so it is tracked upstream rather than here |

Two further areas that are not declared non-goals are nonetheless ruled unnecessary here, with reasoning, rather than dropped in silence.
The reference implementation's debug and source-attribution subpackage — four files whose purpose is to attribute duplicate-instrument
registration warnings to source locations — is **unnecessary** for this port: the diagnostic it produces requires capturing a stack trace at
registration time, which has no portable multiplatform form, and the same duplicate-registration condition is surfaced instead by the
identity conflict `M-07` detects and the one sanitised diagnostic ADR-11 permits [A4|DECIDED]. Summary and quantile aggregation is likewise
**unnecessary**: the specification's six aggregations are drop, default, sum, last value, explicit-bucket histogram and base-2 exponential
histogram, and summary survives only as a legacy data type no instrument produces [A4|DECIDED].

## 2.8 The authority ladder, the label convention and how disagreement is resolved

The convention Section 2's preamble declares is applied to every substantive claim in this document, and it exists to stop a provisional
assertion becoming a fact by being restated [A5|DECIDED].

| Rank | Source | Standing on this baseline |
| --- | --- | --- |
| A1 | Checked-in per-target binary-compatibility dumps | 27 exist and are authoritative for public surface; zero klib baselines exist, so the ladder's klib tiebreak is inoperative and no unit may claim klib evidence |
| A2 | Executable build and continuous-integration configuration, and measured command output | The strongest available evidence for behaviour of the build itself; every transcript in Section 2.4 sits here |
| A3 | Checked-in production and test source, and generated-source task definitions | Authoritative for what the code does; used wherever a dump cannot express a behaviour |
| A4 | The upstream Java reference at tag `v1.65.0` and the pinned specification metadata | Authoritative for the port's shape and for specification semantics, at the pinned revision only |
| A5 | Unmeasured assertions carried into this work | Never binding alone; each is either re-verified against a higher rank or labelled unknown, and eleven were contradicted outright — see Section 2.1 |
| A6 | Repository prose — KDoc, READMEs, roadmaps | Never authoritative for behaviour at any rank; `Meter.kt:L9` is the standing example of prose contradicting the declaration nine lines below it |

Resolution is mechanical. The higher rank governs; the disagreement is recorded with evidence for **both** sides; and the winner is never
silently adopted. Where the ladder offers no tiebreak the document rules explicitly and justifies the ruling — which is exactly the situation
ADR-13 addresses, because the ladder adjudicates JVM against klib and says nothing about JVM against Android [A1|DECIDED]. Where no rank can
speak to an item, the item is labelled `UNAVAILABLE` and its dependents are marked provisional; absence is never read as success
[A5|DECIDED].

Each claim carries an inline label of the form `[rank|status]`, where status is one of `OBSERVED` for something measured or read directly,
`DERIVED` for something computed from observations by a stated rule, and `DECIDED` for a ruling this document makes and owns [A5|DECIDED].

## 2.9 What each ledger row settles, and what it costs the plan

The table above is the audit surface; this is what a reader takes from it. Every deviation is stated with both the asserted and the observed
value, because a corrected figure with the original suppressed is not auditable [A5|DECIDED].

- **P3-01 settles the write boundary.** The checkout is at the revision T-01 records, the tree is clean before and after every invocation,
  and the analysis is read-only across all 39 projects while exactly one file is produced. The cost to the plan is nil; the benefit is that
  every later measurement is attributable to a known revision.
- **P3-02 settles placement, and it is the row that changed.** The destination is the tracked repository-root file. The earlier
  out-of-repository ruling rested on a false premise, and the instruction behind it has been revoked; the observed destination governs, and
  the consequence is that the branch's tracked diff carries the deliverable rather than nothing. D11 records the reversal in full.
- **P3-03 settles the module question before ADR-01 answers it.** Thirty-nine projects, no metrics module, and an absent instrumentation
  directory mean the layout decision chooses among existing modules rather than proposing new ones, which removes build-logic risk from a
  configuration-cache-strict build.
- **P3-04 settles how much is already built.** The provider is real — scope caching, shutdown state, composite closure, timeout wrapping —
  while the meter is a shell. `M-16` therefore grows two files that exist rather than authoring a pipeline from nothing, and `M-05` exists
  because four implementors break the moment `Meter` gains a member.
- **P3-05 settles which dump governs, and costs the plan a dedicated first unit.** Two rank-one sources contradict each other about whether
  the metrics package exists; `M-02` isolates the inherited disagreement before any metrics delta is accepted, so a later dump review is
  attributable.
- **P3-06 settles the language floor, and it forecloses an option.** Kotlin 2.0 as both API and language level removes the standard
  library's atomics package from consideration entirely, which is why ADR-06 designs primitives rather than importing them.
- **P3-07 settles the dependency position.** No atomics coordinate exists in the catalog, no dependency changes anywhere in the plan, and the
  seven frozen minimum-support pins make the language floor policy rather than accident.
- **P3-08 settles which gate actually fails a build.** The repository enforces a 1% project threshold and a 50% patch target; G3-03 is fixed
  at 1.00 percentage point on that basis and is never widened, and the coverage artifact defect T-14 records is what `M-01` repairs.
- **P3-09 settles that the configuration surface is generated, not greenfield.** One hundred and thirteen tracked models already name every
  metrics configuration concept, and the whole protocol tree is generated, so the metrics message types compile today and the remaining
  exporter work is marshalling and transport.
- **P3-10 settles what the correctness strategy can reach.** The differential harness already exists and its reference-delegating back end is
  confined to a JVM-and-Android module's test source set, which is the first of the two bounds Section 6 states.
- **P3-11 settles the floor, with a qualification.** Every gate command exits 0 and configuration-cache reuse is demonstrated, but the
  baseline ran under a heap override well below the checked-in premise, so a later failure that appears only under an 8 GiB daemon is a
  capacity finding rather than a code defect.
- **P3-12 settles the documentation debt the plan inherits.** Four documentation deviations and one changelog release-line collision are
  confirmed; `M-26` corrects the two that are in this repository and the rest are recorded rather than silently absorbed.

## 2.10 Upstream metadata, the register's closure, and what stays open

The reference register in Section 2.2 resolves every load-bearing claim to exactly one row, so closure is checkable by locator rather than by
reading, and no row dangles [A5|DECIDED]. Three findings from the upstream metadata are load-bearing and are recorded here rather than left in
a footnote.

- The repository's own open-count field **counts pull requests as issues**, so a single combined figure and a separate issue count cannot both
  be right; the field-semantics trap is recorded so no later harvest repeats it [A4|OBSERVED].
- Two upstream tracking issues remain **open** even though the changes implementing their declarations merged, so the declarations exist at
  rank A1 while the tracking items do not. Both are treated as partially satisfied — neither settled nor open ground — and no compliance
  arithmetic depends on them [A4|OBSERVED].
- The two merged metrics changes came in at **+88 lines across 4 files** and **+72 lines across 5 files**, while the one open attempt at a
  single instrument end to end reached **+800 lines across 31 files**, which is 1.6 times the reviewable-diff guidance at `AGENTS.md:L25`.
  Section 4 calibrates against those three data points rather than against the ceiling [A4|OBSERVED].
- Upstream moved during this analysis, and the branch is 15 commits behind `upstream/main` per T-05, which is why Section 5 states a rebase
  cadence and a dump-conflict policy rather than treating drift as hypothetical [A2|OBSERVED].

One item is recorded as `UNAVAILABLE` rather than guessed: the tenth member of the upstream metrics milestone is not identified here, because
the enumeration available to this work is internally inconsistent about it. No compliance arithmetic, scope claim or unit depends on that row,
and the correct resolution is to re-query the milestone's members and reconcile them item by item [A4|UNKNOWN].

## 2.11 Secret hygiene

The `origin` remote's fetch and push URLs carry embedded credential material and an access token is present in the environment. Neither value
appears in this document, in any transcript above, in any commit message, or in any log this work produced; T-04 shows the redaction in place
[A2|OBSERVED]. Only two facts about a credential may ever be recorded here: that it is present, and the rate limit it grants [A5|DECIDED].
A credential in tracked content is a defect that removal alone does not remedy — because the content is history the moment it is committed —
so the remedy is revocation and rotation, and R-18 carries that exposure with a measurable early-warning signal rather than an assurance
[A5|DECIDED].

## 2.12 The reference implementation census at the pinned tag

Rank 3 of the authority ladder is the reference implementation **at its pinned tag**, and a census taken at a floating branch would make every
figure below irreproducible. The clone is `/tmp/blitzy/blitzy-opentelemetry-kotlin/reference-otel-java`, `git describe --tags` reports `v1.65.0`
and `git rev-parse HEAD` reports `7bc11edca518d4de168230c84a71484a66cbac40`, so every figure in this subsection is reproducible by checking out
that commit and repeating the commands [A3|OBSERVED]. The census exists because Section 4.3 nets a port estimate down from it; a netting without a
census in this ledger would be a figure with no baseline entry, which this document treats as a defect.

```console
$ M=reference-otel-java/sdk/metrics/src/main/java/io/opentelemetry/sdk/metrics
$ find $M -name '*.java' | wc -l ; find $M -name '*.java' -printf '%s\n' | awk '{s+=$1}END{print s}' ; find $M -name '*.java' -exec cat {} + | wc -l
176
541431
16406
exit=0
```

| Subpackage | files | bytes | lines | Why the plan reads it |
| --- | --- | --- | --- | --- |
| `<root>` | 35 | 122,959 | 3,675 | The meter, the provider and the builder surface `M-03` through `M-05` mirror |
| `data` | 22 | 34,138 | 1,258 | The public point and exemplar model `M-06` declares |
| `export` | 12 | 39,743 | 1,130 | Reader, exporter, producer and the three selectors `M-16` and `M-18` adopt |
| `internal` | 3 | 8,908 | 220 | Meter configuration and utilities, absorbed into existing types |
| `internal/aggregator` | 19 | 87,763 | 2,523 | The six aggregations `M-10`, `M-11` and `M-27` implement |
| `internal/concurrent` | 7 | 9,870 | 427 | The accumulation primitives `M-08` and `M-32` replace with expect/actual declarations |
| `internal/data` | 23 | 53,947 | 1,865 | Concrete point implementations, collapsed into the `M-06` declarations |
| `internal/debug` | 4 | 4,589 | 169 | Source attribution for duplicate-registration warnings — ruled unnecessary in Section 3.1 |
| `internal/descriptor` | 3 | 9,920 | 326 | Instrument and metric identity `M-05` fixes under ADR-04 |
| `internal/exemplar` | 16 | 28,717 | 860 | Reservoirs and filters `M-14` implements |
| `internal/export` | 1 | 2,830 | 90 | A single utility, absorbed |
| `internal/state` | 17 | 90,713 | 2,490 | The highest-complexity subpackage: storage and registries across `M-09`, `M-27` and `M-28` |
| `internal/view` | 14 | 47,334 | 1,373 | View resolution and attribute filtering `M-12` implements |

Two measurements inside that census carry more weight than their size suggests. The **accumulation surface is 7 files and 427 lines**, which is
small in line terms and the hardest part of the port in architectural terms, because each primitive needs one common declaration plus an actual
for the JVM-and-Android, Apple and JavaScript families:

```console
$ find $M/internal/concurrent -name '*.java' -printf '%f|%s\n' | sort
AdderUtil.java|1549
AtomicLongDoubleAdder.java|1171
AtomicLongLongAdder.java|907
DoubleAdder.java|2188
JreDoubleAdder.java|802
JreLongAdder.java|775
LongAdder.java|2478
exit=0
```

Of those seven, the two `Jre`-prefixed files exist only to select a newer implementation at runtime through a packaging mechanism that has no
multiplatform analogue; under the declared JDK 11 toolchain they collapse to a single JVM-and-Android actual delegating to the platform's own
adder, so the genuinely novel work is the Apple and JavaScript actuals in `M-32` [A3|DERIVED].

The **test tree is larger than the production tree by bytes**, at 89 files, 767,788 bytes and 18,468 lines against 176 files, 541,431 bytes and
16,406 lines — a test-to-production ratio of 1.42 by bytes and 1.13 by lines. That ratio, not an aspiration, is what makes the per-unit test
content in Section 4 credible: a unit that adds aggregation logic and no tests is not the shape this port takes [A3|OBSERVED]. The asserted
figures of 93 files and 837,652 bytes are recorded as **DIFFERS** against the observed 89 and 767,788 in D6, because they were taken at an
unrecorded revision.

The byte-per-line constant Section 4.3 uses is measured rather than assumed, from a six-file sample spanning the largest and smallest files in the
tree — delta synchronous storage, the meter, the long adder, filtered attributes, the periodic reader and the exponential-histogram aggregator —
totalling 2,004 lines over 74,502 bytes, giving **37.2 bytes per line** [A3|OBSERVED].

## 2.13 The measured project inventory and the module-behaviour flags

Thirty-nine Gradle projects exist: `settings.gradle.kts:L17-L55` lists thirty-seven explicit entries, `examples` is an implicit container project
because two of those entries are `examples:example-app` and `examples:example-app-android`, and the root project is the thirty-ninth
[A2|OBSERVED]. **No metrics module is among them**, which is the observation ADR-01 rests on. `settings.gradle.kts:L57` calls
`includeFromDir("instrumentation")` over a directory that does not exist in this checkout, contributing zero projects — a negative finding stated
rather than omitted, because a reader who assumes the call contributes projects will not reconcile the count.

Four properties decide, per module, whether it carries public API, whether it is restricted to JVM and Android, whether coverage is measured, and
whether the project-wide experimental opt-in applies. They are **not** all declared in one file, which is exactly what D3 records: `buildSrc/src/main/kotlin/io/opentelemetry/kotlin/ModuleBehavior.kt:L10-L23` declares only `containsPublicApi()` and `isJvmAndroidModule()`, while
`enableCodeCoverage` is read in the root build script and `enableOptIn` in the opt-in convention plugin, so each row below is verifiable at the
property's own reading site rather than at a single file. Their
distribution explains the shape of every gate, and it is read from each module's own `gradle.properties` rather than inferred [A2|OBSERVED].

| Property | Default when absent | Modules that set it `false` or `true` | Consequence the plan depends on |
| --- | --- | --- | --- |
| `containsPublicApi` | `true` | `false` in: `behavior`, `benchmark-android`, `benchmark-fixtures`, `benchmark-jvm`, `config`, `config-dsl`, `config-envar`, `config-schema`, `config-yaml`, `examples/example-app`, `exporters-protobuf`, `integration-test`, `model`, `semconv`, `smoke-test`, `test-fakes`, `testing`, `java-typealiases` | Explicit-API mode and dump validation apply only where it is true, which is why 27 dumps exist rather than 39 and why `M-19` and `M-20` change no dump |
| `jvmAndroidModule` | `false` | `true` in: `compat`, `context-coroutines`, `java-typealiases`, `testing` | `compat` cannot compile for Apple or JavaScript at all, which is the mechanical cause of the differential reach limit in Section 6.3 |
| `enableCodeCoverage` | `true` | `false` in: `benchmark-android`, `benchmark-fixtures`, `benchmark-jvm`, `config-schema`, `core`, `examples`, `examples/example-app`, `integration-test`, `java-typealiases`, `semconv`, `smoke-test`, `test-fakes`, `testing` | Fixes the 5,153-line measurable denominator G3-03 divides into, and means tests added in `M-24` and `M-34` raise no coverage figure |
| `enableOptIn` | `true` | `false` in: `api`, `config-schema`, `java-typealiases` | The `api` module does **not** receive the project-wide opt-in, so every new public instrument interface there MUST carry the experimental marker explicitly — which is why G3-08 is self-enforcing inside `api` and a manual audit everywhere else |

The last row is the one most easily missed and the one with the sharpest consequence: because `allWarningsAsErrors` is set at
`buildSrc/src/main/kotlin/io/opentelemetry/kotlin/KotlinConfig.kt:L89`, an un-opted-in use of experimental API inside `api` is a hard compile
failure rather than a warning, so `M-03` cannot land an unannotated declaration even by accident [A2|DERIVED].

## 2.14 The metrics footprint and the dump inventory, counted

The metrics footprint is **22 Kotlin files across five modules**, not the eight of `api/src/commonMain` alone, and every one of them changes when
`Meter` gains factory members because each implements or exercises a type on that surface [A2|OBSERVED]. Recording the larger figure matters
because a work breakdown scoped to eight files would omit the four implementors ADR-02 and `M-03` must update together.

| Module and source set | files | file names |
| --- | --- | --- |
| `api/src/commonMain/kotlin/io/opentelemetry/kotlin/metrics` | 8 | `Meter.kt`, `MeterProvider.kt`, `Instrument.kt`, `SynchronousInstrument.kt`, `AsynchronousInstrument.kt`, `ObservableMeasurement.kt`, `ObservableLongMeasurement.kt`, `ObservableDoubleMeasurement.kt` |
| `compat/src/jvmAndAndroidMain/kotlin/io/opentelemetry/kotlin/metrics` | 5 | `MeterAdapter.kt`, `MeterProviderAdapter.kt`, `OtelJavaMeterAdapter.kt`, `OtelJavaMeterBuilderAdapter.kt`, `OtelJavaMeterProviderAdapter.kt` |
| `compat/src/jvmTest/kotlin/io/opentelemetry/kotlin/metrics` | 2 | `MeterProviderAdapterTest.kt`, `OtelJavaMeterBuilderAdapterTest.kt` |
| `implementation/src/commonMain/kotlin/io/opentelemetry/kotlin/metrics` | 2 | `MeterImpl.kt`, `MeterProviderImpl.kt` |
| `implementation/src/commonTest/kotlin/io/opentelemetry/kotlin/metrics` | 1 | `MeterProviderImplTest.kt` |
| `noop/src/commonMain/kotlin/io/opentelemetry/kotlin/metrics` | 2 | `NoopMeter.kt`, `NoopMeterProvider.kt` |
| `test-fakes/src/commonMain/kotlin/io/opentelemetry/kotlin/metrics` | 2 | `FakeMeter.kt`, `FakeMeterProvider.kt` |

The dump inventory is counted from the index rather than from the filesystem, because a filesystem walk also matches regenerated copies under
gitignored build directories and would overstate the tracked set:

```console
$ git ls-files '*.api' | wc -l
27
$ git ls-files '*/api/jvm/*.api' | wc -l
16
$ git ls-files '*/api/android/*.api' | wc -l
11
$ git ls-files '*.klib.api' | wc -l
0
exit=0
```

Sixteen JVM dumps, eleven Android dumps, and **zero klib dumps**. The eleven Android dumps cover `api`, `api-ext`, `compat`, `core`,
`exporters-core`, `exporters-in-memory`, `exporters-otlp`, `exporters-persistence`, `implementation`, `noop` and `platform-implementations`; the
five modules with a JVM dump and no Android counterpart are `context-coroutines`, `sdk-api`, `sdk-common`, `sdk-ext` and `span-event-bridge`. Two
consequences follow, and both are load-bearing. The absence of klib dumps makes the authority ladder's klib tiebreak **inoperative on this
baseline**, which R-07 carries as a risk rather than leaving as an assumption. And the eleven Android dumps are the exact set whose staleness
ADR-13 rules on and whose regeneration is the highest-ranked question in Section 8 [A1|OBSERVED].

## 2.15 The figure ledger: what each cited figure is, and where it comes from

Section 2 is only useful if a reader can go from a figure in a later section back to the command or file that produced it, and forward from every
baseline entry to the section that consumes it. This ledger closes that loop in both directions. A figure in Sections 3 through 8 with no row here
is a defect, and so is a row here that no section consumes [A5|DECIDED]. Figures are written here in the same form the later sections use; where a
transcript prints them without separators the transcript is authoritative and this ledger is the reading of it.

| Figure | Value | Source | Consumed by |
| --- | --- | --- | --- |
| Checkout revision the whole ledger is taken at | `9dc0ba05` | T-01 | Every locator in Section 2.5; the reproducibility claim in Section 2.12 |
| Branch, never moved during authoring | the assigned refinement branch | T-02 | Section 8.1's self-check; G3-11 |
| Working tree before the deliverable was written | clean, no output | T-03 | Section 8.1's self-check; G3-11 |
| Remotes, with credential material recorded as present and never transcribed | `origin` and `upstream` | T-04 | Section 2.11; R-18; ADR-15 |
| Divergence from the branch base and from upstream | 3 commits ahead, 15 behind | T-05 | ADR-15; R-33; Section 5's rebase cadence |
| The deliverable's path is not ignored | no output, exit 1 | T-06 | P3-02; P3-11; D11; R-19; Section 8.1 |
| The complete ignore set | 11 patterns, no Markdown pattern | T-07 | P3-02; D11; Section 8.1 |
| Gradle distribution | 9.7.0, SHA-pinned, `retries=0` | T-08 | G3-05; R-12 |
| Discoverable toolchains | 11, 17 and 21 | T-09 | G3-05; R-10; ADR-06 |
| Canonical build outcome, first invocation | exit 0; configuration cache stored | T-10 | G3-04; R-12 |
| Canonical build outcome, second invocation | exit 0; configuration cache reused | T-11 | G3-04; G3-07 posture; R-12; Section 5.3's second spike |
| Gate outcome on an otherwise untouched tree | `apiCheck detekt` exit 0 | T-12 | G3-08; G3-09; R-06 |
| Coverage rate and measurable denominator | 94.39%, 4,864 of 5,153 lines | T-13 | G3-02; G3-03; Section 6.1.2; R-16 |
| Coverage artifact defect | `report.xml` produced; the uploaded `reportRelease.xml` absent | T-14 | `M-01`; R-24; Section 8 rank 8 |
| Test estate | 2,965 tests, 0 failures, 0 skips | T-15 | Section 6.5; R-27 |
| Tracked dump inventory | 27 total — 16 JVM, 11 Android, 0 klib | T-16, Section 2.14 | ADR-13; `M-02`; R-06; R-07 |
| The rank-one dump disagreement, quantified | JVM dump carries the metrics package; the Android dump has none | T-17 | ADR-13; `M-02`; R-06; Section 8 rank 1 |
| Project topology | 39 projects, no metrics module; `includeFromDir` contributes zero | T-18, Section 2.13 | ADR-01; every Section 4 scope cell |
| Metrics footprint | 22 Kotlin files across 5 modules | T-19, Section 2.14 | ADR-01; ADR-02; `M-03`; `M-04`; `M-05` |
| Atomics library absent from the catalog | zero matches | T-20 | ADR-06; Section 8 rank 4 |
| Generated configuration surface | 113 generated models covering every metrics concept | T-21 | ADR-03; ADR-07; R-14 |
| Test-estate structure, each count with its counting rule | 28 test source sets, 287 test files, 898 production files | T-22 | Section 6.2; Section 6.5 |
| Golden-fixture inventory | the two metrics-relevant fixture directories and their overlap | T-23 | `M-25`; Section 6.2's serialisation kind; ADR-12 |
| Measured capacity, taken before the first build | kernel figures and every cgroup limit path | T-24 | G3-05; HC-10; R-11; P3-11 |
| The qualification on the build transcripts | a heap override in the Gradle user home, outside the repository | T-25 | P3-11; R-11; D10 |
| The checked-in heap premise is unaltered | no output | T-26 | G3-04; R-11; Section 8.1 |
| Reference implementation, pinned | `v1.65.0` = `7bc11edca518d4de168230c84a71484a66cbac40` | T-27 | Section 2.12; Section 2.16; ADR-12; R-08 |
| Reference production tree | 176 files, 541,431 bytes, 16,406 lines | T-28, Section 2.12 | Section 4.3's netting; R-08 |
| Reference test estate, with its deviation | 89 files, 767,788 bytes against an asserted 93 and 837,652 | T-29, Section 2.12 | Section 6.5; D6 |
| Concurrency surface, file by file | 7 files, 427 lines, two of them a runtime-selection pair | T-30, Section 2.12 | ADR-06; `M-08`; `M-32`; R-02 |
| Where the reference tree's complexity sits | per-subpackage counts across 13 subpackages | T-31, Section 2.12 | Section 4.5's package boundaries; Section 4.3 |
| Bytes per line | 37.2, from a 2,004-line, 74,502-byte sample | T-32, Section 2.12 | Section 4.3's byte estimate |
| Accepted change sizes | +88 lines / 4 files and +72 lines / 5 files merged; +800 lines / 31 files open | Section 2.10 | Section 4's per-unit ceiling; R-17 |
| Compliance rows and denominator | 88 total, 10 optional, 78 non-optional, 3 optional committed, denominator 81 | Section 2.6, Section 2.17 | Section 1; G3-11; `M-26`; R-31 |
| Reference compliance position | Java satisfies 75 of 88; 13 unsatisfied and unnamed here | Section 2.6, Section 2.17 | Section 6.3's second bound; ADR-12; R-08 |
| Specification semantics promoted to rank A3 | six aggregations; 160 buckets; scale 20; cardinality 2000; reservoir kinds | Section 2.16 | ADR-07; `M-10`; `M-11`; `M-13`; `M-14`; R-21; R-22; R-25 |
| Enforced coverage tolerance | project threshold 1%, patch target 50% | `codecov.yml:L5-L7,L10` | G3-02; G3-03; Section 6.1.2; R-16 |
| Static-analysis ceilings | 0 issues; 140 characters; 31 functions; 30 parameters | `config/detekt/detekt.yml:L3,L10-L11,L14-L18,L35-L36` | G3-09; ADR-02; ADR-03; R-17 |
| Declared toolchain and target set | JDK 11; exactly two Apple targets and no `iosX64` | `buildSrc/src/main/kotlin/io/opentelemetry/kotlin/KotlinConfig.kt:L11,L17,L39-L40` | ADR-06; ADR-09; Section 6.4; `M-32`; R-10 |

## 2.16 Specification-derived semantics, restated at rank A3 where possible

The aggregation, exemplar and view semantics this plan designs against were originally gathered from published specification prose, which sits at
**A6** on the ladder and is never authoritative for behaviour. Rather than carry them at A6, each was re-checked against the pinned reference clone,
which is A3, and promoted where the source actually settles it. The promotions matter: an acceptance criterion in Section 4 that rests on an A6
claim alone is provisional by construction, and three of the claims below turned out to be load-bearing [A5|DECIDED].

Every locator in this subsection is relative to
`reference-otel-java/sdk/metrics/src/main/java/io/opentelemetry/sdk/metrics/` at `v1.65.0` = `7bc11edca518d4de168230c84a71484a66cbac40`.

| Semantic | Value observed at the pinned tag | Locator | Rank and status |
| --- | --- | --- | --- |
| The aggregation set is exactly six | `Base2ExponentialHistogramAggregation`, `DefaultAggregation`, `DropAggregation`, `ExplicitBucketHistogramAggregation`, `LastValueAggregation`, `SumAggregation` — six concrete classes and no seventh | `internal/view/` | [A3\|OBSERVED] |
| Exponential histogram default bucket ceiling | `DEFAULT_MAX_BUCKETS = 160` | `Base2ExponentialHistogramOptions.java:L24`; `internal/view/Base2ExponentialHistogramAggregation.java:L29` | [A3\|OBSERVED] |
| Exponential histogram default scale ceiling | `DEFAULT_MAX_SCALE = 20` | `Base2ExponentialHistogramOptions.java:L27`; `internal/view/Base2ExponentialHistogramAggregation.java:L30` | [A3\|OBSERVED] |
| Exponential histogram parameters | maximum buckets, maximum scale and a record-minimum-and-maximum flag, passed together into the aggregator | `internal/view/Base2ExponentialHistogramAggregation.java:L73-L82` | [A3\|OBSERVED] |
| Default cardinality ceiling | `DEFAULT_MAX_CARDINALITY = 2000` | `internal/state/MetricStorage.java:L24` | [A3\|OBSERVED] |
| Cardinality overflow is a reserved series, not a drop | `CARDINALITY_OVERFLOW` is an attribute set carrying `otel.metric.overflow = true` | `internal/state/MetricStorage.java:L26-L27` | [A3\|OBSERVED] |
| Reservoir kind for a multi-bucket explicit histogram | `ExemplarReservoirFactory.histogramBucketReservoir(clock, bucketBoundaries)` — bucket-aligned, one cell per boundary | `internal/view/ExplicitBucketHistogramAggregation.java:L63-L66` | [A3\|OBSERVED] |
| Reservoir kind for every other aggregation | `ExemplarReservoirFactory.fixedSizeReservoir(...)`, wrapped in `filtered(exemplarFilter, ...)` so the filter composes rather than being checked inline | `internal/view/SumAggregation.java:L42-L48`; `internal/view/LastValueAggregation.java:L45-L50`; `internal/view/Base2ExponentialHistogramAggregation.java:L74-L79` | [A3\|OBSERVED] |
| Exemplar filtering composes as a decorator | Both filtered reservoir types exist as their own classes alongside the unfiltered ones | `internal/exemplar/DoubleFilteredExemplarReservoir.java`, `internal/exemplar/LongFilteredExemplarReservoir.java` | [A3\|OBSERVED] |

One promotion produced a **contradiction that changes a design**, and it is recorded here loudly rather than reconciled quietly. The A6 prose
asserts that non-histogram aggregations size their fixed-size reservoir to the smaller of twenty and the configured bucket maximum. The pinned
source does not: it sizes every fixed-size reservoir to `Runtime.getRuntime().availableProcessors()`
[`internal/view/SumAggregation.java:L45-L48`, `internal/view/Base2ExponentialHistogramAggregation.java:L76-L79`], a processor-count-derived
size with no fixed relationship to twenty or to the bucket maximum [A3|OBSERVED]. **Asserted: min(20, maxBuckets). Observed: the available
processor count.** A3 outranks A6, so the observed value governs, and the consequence is concrete rather than academic: the available processor
count has no portable multiplatform equivalent in this repository, so `M-14` MUST take the reservoir size as a fixed configured constant or as a
value supplied through `platform-implementations`, and MUST NOT reproduce the reference's runtime query. This is recorded as deviation D7's
aggregation counterpart and is the reason ADR-07 fixes the reservoir kind per aggregation rather than deferring it to each unit [A3|DERIVED].

Four semantics could **not** be restated at A3, because the pinned source encodes them in behaviour rather than in a readable constant, and no
transcript in this run observed them. They are carried at A6 with their retrieved location named, are labelled `INFERENCE` at every use, and are
explicitly **carried over from the prior revision's research rather than freshly retrieved in this run** [A6|INFERENCE]:

- **Explicit-bucket boundary inclusivity.** Buckets are stated by upper boundary, exclusive of the lower bound and inclusive of the upper except at
  positive infinity, and a measurement falls into the lowest bucket whose boundary is greater than or equal to the value. Retrieved location:
  `https://opentelemetry.io/docs/specs/otel/metrics/data-model/`. Consequence if wrong: `M-10`'s boundary property test asserts the wrong edge, so
  R-21 carries it and the differential on the JVM is the corroborating check, since the reference satisfies that compliance row.
- **View attribute-key semantics.** When a view supplies an attribute-key list, keys outside the list are ignored; when it supplies none, all keys
  are used. Retrieved location: `https://opentelemetry.io/docs/specs/otel/metrics/sdk/`. Consequence if wrong: `M-12` filters at the wrong default,
  which R-23 carries; the absent-list case is therefore tested as its own case rather than assumed.
- **Force-flush fan-out.** Force-flush fans out to every registered reader and every push exporter, and may be blocking or asynchronous. Retrieved
  location: `https://opentelemetry.io/docs/specs/otel/metrics/sdk/`. Consequence if wrong: `M-29`'s fan-out assertion is incomplete, which R-15
  carries; the repository's own suspending aggregate-bound contract at `sdk-api/src/commonMain/kotlin/io/opentelemetry/kotlin/export/TelemetryCloseable.kt:L10-L21`
  is the A2 constraint that actually binds the implementation.
- **The delta scale-reset trap.** Under delta temporality an exponential histogram's scale must reset each collection cycle or precision never
  recovers after an outlier — a defect observed in another language's SDK rather than in this one. Retrieved location:
  `https://opentelemetry.io/docs/specs/otel/metrics/data-model/`. Consequence if wrong: nothing is lost, because the plan tests for recovery
  regardless; R-22 carries it at composite 15 and `M-28` makes the recovery test a named acceptance criterion. This is the one A6 item the plan
  deliberately designs against **even while it remains provisional**, because the cost of the test is small and the cost of the defect is silent
  precision loss.

The discipline these four illustrate is the one HC-13 and G3-11 enforce throughout: an A6 claim may motivate a test, may never be restated as fact,
and may never be the sole support for an acceptance criterion. Where this document states one, it states it once, labels it, and names what would
resolve it [A5|DECIDED].

## 2.17 Row-level compliance disposition, and the part of it that is UNAVAILABLE

Section 2.6 fixes the denominator at 81 and states that the remaining seven optional rows are out of scope. That claim is only auditable if the
seven can be named, so this subsection separates what is established from what is not, rather than letting a summary figure imply a row-level
enumeration that does not exist in this run [A5|DECIDED].

**Established.** The row counts, the optional split, the three committed optional rows, the four non-goal exclusions and the Java column's 75-of-88
position are all carried forward from the prior revision's harvest of the matrix and are recorded here as **carried over**, not as freshly
retrieved [A4|OBSERVED, carried over]. They are internally consistent: 0 satisfied + 4 excluded + 77 outstanding = 81, and 81 + 7 optional excluded
= 88, so an arithmetic error in the counts would show as a failure to close, and it does not.

**UNAVAILABLE.** The row-level enumeration is not established in this run, and the reason is a measurable absence rather than an oversight:

```console
$ ls -d /tmp/blitzy/blitzy-opentelemetry-kotlin/reference-otel-spec
ls: cannot access '/tmp/blitzy/blitzy-opentelemetry-kotlin/reference-otel-spec': No such file or directory
exit=2
$ find / -xdev -name 'spec-compliance-matrix.md'
exit=1
```

The specification clone is not present in this workspace and the matrix file exists nowhere on this host, so the three items below are labelled
`UNAVAILABLE` rather than estimated, inferred, or reconstructed from the counts:

| Item | Why it is UNAVAILABLE | What would resolve it | What depends on it |
| --- | --- | --- | --- |
| The names of the seven optional rows excluded from the denominator | The matrix is not on disk and no transcript in this run read it; the count of seven is arithmetic from 10 − 3, which fixes the count but not the identities | A clone of the specification repository at a pinned revision, or a read of the matrix's metrics section at a recorded revision | `M-26`'s sheet, which must list them row by row; G3-11's requirement that the basis be stated |
| The names of the thirteen rows the Java column does not satisfy | Same absence; the count of thirteen is carried over from the prior harvest | The same retrieval | Section 6.3's second reach bound at row granularity; ADR-12's rule that no unsatisfied row may be cited as reference-verified |
| The four rows excluded by declared non-goals, at row granularity | Their subject areas are established — the deprecated instrumentation-library row and three metric-filter rows scoped to a non-goal surface — but their row identifiers are not | The same retrieval | The `4 of 81 excluded by non-goals` term in Section 1's compliance string |

Three consequences follow, and each is a deliberate constraint on how this plan may be used. First, **Section 1's compliance string is reportable at
aggregate granularity and not yet at row granularity**, which is exactly what it claims — it reports counts against a stated basis and never names a
row. Second, `M-26` is not merely documentation: it is the unit that closes this gap, which is why it is the single sink of the dependency graph and
why every earlier unit's compliance claim is provisional until it lands. Third, Section 6.3's second bound is stated at the granularity the evidence
supports: the differential MUST NOT be cited for a row the reference does not satisfy, and because the thirteen are unnamed here, the operative rule
for every unit before `M-26` is stricter — a differential result is corroboration and never sole evidence for a compliance claim [A4|DERIVED].

**Locator convention for the absent file.** Every `spec-compliance-matrix.md:Lnn` citation in this document is a **carried-over** locator from the prior revision's harvest: it names the row range that harvest read, and it is not resolvable in this workspace because the file is absent, as the two commands above show. Such a citation is therefore evidence of provenance rather than a locator a reader can open here, it carries `[A4|OBSERVED, carried over]` at every use, and it is the only file cited anywhere in this document that does not resolve in this checkout or in the pinned reference clone. Every other locator resolves, and every line range in every other locator lies inside the file it names [A2|OBSERVED].

Recording the absence this way is the point. A summary figure that quietly implies a row-level enumeration would be the same defect this document
names in D5 and D7: a provisional assertion promoted to fact by restatement.

# 3. Architecture decisions

Each record separates the evidence from the decision it supports, and every rejected alternative carries a reason rather than a preference [A5|DECIDED].

## ADR-01 — Module and ownership layout

**Context.** The signal has to be built somewhere, and the topology is fixed: 39 Gradle projects, none of them a metrics module, with an absent `instrumentation` directory contributing nothing (P3-03, T-18). The reference implementation ships metrics as its own module, so the naive port would create one. Two facts make that expensive here rather than neutral: a new project acquires its own binary-compatibility dumps and its own publication coordinate derived from the project name, and it must configure through convention plugins in a build where configuration-cache problems fail rather than warn. Against that, `sdk-api` already carries per-signal packages for logging and tracing, and `implementation` already carries a metrics package holding a real provider (P3-04). The question is therefore not where metrics *could* live but whether any module boundary buys anything the package boundary does not.

**Status.** Accepted. No baseline finding renders it moot; P3-03 supplies its premise and no later finding contradicts it.

**Decision.** The Metrics signal SHALL be implemented entirely within existing Gradle projects. Public instrument contracts MUST live in `api` under `io.opentelemetry.kotlin.metrics`. Public SDK contracts and configuration MUST live in `sdk-api` under a new `metrics` package with `export`, `model`, `data`, `view`, `aggregation` and `exemplar` subpackages. Storage, aggregation, registry and collection MUST live in `implementation` under the existing `io.opentelemetry.kotlin.metrics` package. Reusable accumulation primitives MUST live in `platform-implementations`. Reader and exporter core types MUST live in `exporters-core`, wire marshalling in `exporters-protobuf`, transport in `exporters-otlp`, and the test exporter in `exporters-in-memory`. Parity implementations MUST remain in `noop`, `test-fakes` and `compat`. No new Gradle project SHALL be created [A2|DECIDED].

**Alternatives.** (a) Create a dedicated `metrics` Gradle project for the SDK surface. (b) Place the whole signal, contracts included, inside `implementation`. (c) Mirror the reference implementation's own module split with a separate SDK metrics project plus a separate exporter project. [A5|PROPOSED]

**Rejected because.** (a) is rejected because a new project must be registered in `settings.gradle.kts`, must acquire its own binary-compatibility dumps and its own publication coordinates derived from the project name, and must configure through the convention plugins in a build where configuration-cache problems fail rather than warn; the benefit is nil because `sdk-api` already carries per-signal subpackages for logging and tracing and needs no module boundary to carry a third, and a new publication coordinate is a publishing change that the declared non-goals exclude. (b) is rejected because public interfaces belong only in `api` or `api-ext`, so putting reader and exporter contracts in `implementation` would make them unimplementable by any third party and would invert the module layering the repository already enforces. (c) is rejected because it multiplies the cost of (a) across two projects while importing a module decomposition designed for a single-platform build with a different publication model [A2|DECIDED].

**Consequences.** The 39-project count is an invariant, so any unit that adds a project is a scope violation rather than a design variation. Because `sdk-api` gains a package rather than a module, its existing dump is the one that changes, which concentrates rebase exposure on `sdk-api/api/jvm/sdk-api.api` and `api/api/jvm/api.api` and makes ADR-13's dump discipline load-bearing. Because `exporters-core` gains reader types rather than processor types, the eight-file-per-signal shape is deliberately not replicated, which ADR-09 states as a positive requirement [A2|DERIVED].

**Evidence.** `settings.gradle.kts:L16-L57` fixes the project set and shows no metrics project [A2|OBSERVED]. `sdk-api/src/commonMain/kotlin/io/opentelemetry/kotlin/` contains `logging/` and `tracing/` but no `metrics/`, so the per-signal package precedent exists and the metrics package does not [A3|OBSERVED]. `implementation/src/commonMain/kotlin/io/opentelemetry/kotlin/metrics/` already exists and holds the provider and meter [A3|OBSERVED]. `gradle.properties:L4-L5` enables the configuration cache and makes its problems fail the build [A2|OBSERVED]. `buildSrc/src/main/kotlin/io/opentelemetry/kotlin/PublishConfig.kt:L15` derives coordinates from the project name, so a new project is a new artifact [A2|OBSERVED]. `CONTRIBUTING.md:L51-L53` restricts public interfaces to `api` and `api-ext` and directs non-specification sugar to `api-ext`, and `CONTRIBUTING.md:L60` places non-module-specific platform code in `platform-implementations` [A6|OBSERVED].

## ADR-02 — Public Meter and instrument surface

**Context.** `Meter` is a public interface with no body at all, while its own documentation one screen above promises that instruments are obtained through methods on it — prose contradicting a declaration nine lines below it, and the standing example of why rank A6 is never authoritative. The project's primary maintainer left two questions open in the open record: whether instruments follow a concrete `<type-name>Counter` shape or a generic `Counter<T>`, noting the specification gives wiggle room, and whether a common `Instrument` interface should be defined at all given that most instruments share properties. Both questions are already partly answered at higher rank than the discussion that raised them: `Instrument` exists as a concrete non-generic interface and the two observable measurement contracts are split by numeric type with typed overloads. A third fact bounds the answer — both `Meter` and `MeterProvider` already carry the experimental marker, so breaking changes to them are sanctioned by the project's own stability policy rather than negotiable.

**Status.** Accepted, with one deliberately unresolved sub-question. The concrete-versus-generic question is closed by rank A1 and A3 evidence; whether the new members carry default bodies is a repository-policy call this record declines to make for maintainers and Section 8 ranks second.

**Decision.** `Meter` MUST gain builder-returning factory methods covering all seven specification instrument kinds: counter, up-down counter, histogram and gauge synchronously, and observable counter, observable up-down counter and observable gauge asynchronously. Instruments MUST be split by numeric type as concrete named contracts, following the split the repository already chose for observable measurements, so that a long counter and a double counter are distinct types rather than one generic type parameterised by number. Every new public metrics declaration MUST be an interface, enum or sealed type; MUST be declared one type per file; and MUST carry `@ExperimentalApi`. New interface members MUST NOT carry default bodies. Default parameter values MAY be used for optional descriptors such as unit, description and advice. Non-specification sugar MUST NOT enter `api` and MUST be placed in `api-ext` [A3|DECIDED].

**Alternatives.** (a) A single generic instrument type per kind, parameterised over the numeric type. (b) Direct factory methods on `Meter` returning fully built instruments rather than builders. (c) Default bodies on the new `Meter` members so that existing implementors do not break. (d) Deferring the asynchronous instruments to a later signal-complete effort. [A5|PROPOSED]

**Rejected because.** (a) is rejected because the repository has already answered the concrete-versus-generic question in the opposite direction at rank 3 and rank 1: the two observable measurement contracts are split by numeric type and each declares typed `record` overloads rather than a generic method, so a generic instrument would contradict the surface it must interoperate with, and on the JVM it would additionally erase to a boxed number, defeating the primitive accumulation ADR-06 requires. (b) is rejected because instrument creation carries an optional unit, description and advice, and a direct-factory design either grows a wide parameter list toward the 31-argument function ceiling or multiplies overloads, whereas seven builder entry points keep the interface far below the 31-function ceiling the static-analysis configuration enforces. (c) is rejected because default interface bodies are strongly discouraged by the contributor guidance precisely because they blend API and implementation, and because a default body would silently give `NoopMeter`, `FakeMeter` and `MeterAdapter` a wrong implementation instead of a compile error that forces the coordinated parity change in `M-05`; the counter-precedent of a default body in the logging processor contract is acknowledged and is treated as an exception rather than a licence, and the residual compatibility question is carried to Section 8. (d) is rejected because the asynchronous callback surface changes the collection contract that ADR-09 must specify, so deferring it would require respecifying collection later [A3|DECIDED].

**Consequences.** All four `Meter` implementors break at the moment the first factory method lands, which is intentional and is why `M-03`, `M-04` and `M-05` are separate but consecutive units. Because the experimental annotation has binary retention and the validator is not configured to exclude non-public markers, every new declaration appears in the dumps, so each unit that touches `api` carries a regenerated dump and G3-08 is verified by dump diff plus an explicit annotation audit. Because `api` is excluded from the project-wide opt-in, an un-annotated or un-opted-in use inside `api` is a warning that the warnings-as-errors setting converts into a build failure, which makes the annotation requirement self-enforcing there and a manual audit everywhere else [A2|DERIVED].

**Evidence.** `api/src/commonMain/kotlin/io/opentelemetry/kotlin/metrics/Meter.kt:L9` promises that instruments are obtained through methods on the interface while L18 declares it with no body, and `api/api/jvm/api.api:L249-L250` confirms the empty entry at rank 1 [A1|OBSERVED]. `ObservableLongMeasurement.kt:L14,L19` and `ObservableDoubleMeasurement.kt:L14,L19` establish the split-by-numeric-type precedent, dumped at `api/api/jvm/api.api:L260-L268` [A1|OBSERVED]. `SynchronousInstrument.kt:L22` shows a member with no default body [A3|OBSERVED]. `AsynchronousInstrument.kt:L18-L19` is `AutoCloseable` and marks after-creation callback registration as still to be added [A3|OBSERVED]. `CONTRIBUTING.md:L54,L56,L57,L58,L59` supply the one-type-per-file, interface-only, no-default-body, default-parameter and experimental-annotation rules [A6|OBSERVED]. `config/detekt/detekt.yml:L13-L19` sets the 31-function ceiling and L9-L12 the 30-and-31 parameter ceilings [A2|OBSERVED]. `api/src/commonMain/kotlin/io/opentelemetry/kotlin/ExperimentalApi.kt:L6-L9` shows warning-level opt-in with binary retention [A3|OBSERVED]. `sdk-api/src/commonMain/kotlin/io/opentelemetry/kotlin/logging/export/LogRecordProcessor.kt:L37-L42` is the in-repository default-body counter-precedent [A3|OBSERVED]. The seven instrument kinds are the compliance rows at `spec-compliance-matrix.md:L113-L119` [A4|OBSERVED].

## ADR-03 — SDK configuration shape

**Context.** A `meterProvider { }` block is already public and already configures an empty marker interface: the SDK's dump declares the block while the DSL it configures extends only the resource DSL and declares nothing. Its logging counterpart, by contrast, declares limits, export and a configurator, so the growth target is visible in the tree rather than invented. The same asymmetry holds one layer down, where `MetricsConfig` exists but carries only a resource and an error handler while `LoggingConfig` shows the shape it must reach. Anything specified here is therefore a behavioural change to an already-released public surface rather than a new one.

**Status.** Accepted. Partly inherited: P3-09 established that every configuration concept already has a generated model, so this record wires existing names rather than choosing new ones.

**Decision.** The already-public `meterProvider` configuration block MUST be extended rather than replaced, and `MeterProviderConfigDsl` MUST grow members for metric readers, views, default temporality selection, default aggregation selection, exemplar policy and cardinality limits, following the member shape its logging counterpart already uses. Configuration state MUST be carried as immutable nested value types reached through `MetricsConfig`, and `MetricsConfig` MUST NOT be flattened into a single wide constructor. Concept names MUST be taken from the generated configuration models so that programmatic and declarative vocabularies agree. A second configuration framework MUST NOT be introduced [A1|DECIDED].

**Alternatives.** (a) Add a new top-level configuration entry point for metrics beside the existing one. (b) Flatten every metrics setting onto `MetricsConfig` as constructor parameters. (c) Wire the full generated declarative schema for metrics in the first implementation wave. [A5|PROPOSED]

**Rejected because.** (a) is rejected because `meterProvider` is already declared in the released public SDK surface and already accepted by the configuration DSL, so a second entry point would leave a published no-op in place and present two ways to configure one provider. (b) is rejected because the metrics surface spans readers, views, temporality, aggregation, exemplar policy and cardinality limits, and a flat constructor would approach or breach the 30-parameter constructor ceiling the static-analysis configuration enforces, failing the gate for a reason unrelated to correctness; nesting also keeps each concept independently testable. (c) is rejected because full declarative wiring is a declared non-goal beyond the metrics environment-variable subset, and because the generated models include exporters and producers this plan excludes, so wiring them would import excluded scope through a configuration path [A2|DECIDED].

**Consequences.** Extending an already-published block is a behavioural change to released surface: a caller who writes an empty `meterProvider` block today gets a no-op and will get a configured provider afterwards, which `M-15` must state in its acceptance criteria. Because `MetricsConfig` is internal, growing it changes no dump, whereas growing `MeterProviderConfigDsl` does change `sdk-api/api/jvm/sdk-api.api`, so those two changes have different gate exposure and are separated in Section 4 [A1|DERIVED].

**Evidence.** `sdk-api/api/jvm/sdk-api.api:L161-L162` shows the public DSL interface with no members and L170 shows the accepted configuration lambda, both at rank 1 [A1|OBSERVED]. `sdk-api/src/commonMain/kotlin/io/opentelemetry/kotlin/init/MeterProviderConfigDsl.kt:L8-L10` is the empty declaration and `LoggerProviderConfigDsl.kt:L12-L28` is the parity target with three members [A3|OBSERVED]. `implementation/src/commonMain/kotlin/io/opentelemetry/kotlin/init/config/MetricsConfig.kt:L11-L22` carries only a resource and an error handler, and `MeterProviderConfigImpl.kt:L12-L15` is the assembly point [A3|OBSERVED]. `config/detekt/detekt.yml:L9-L12` sets the parameter ceilings [A2|OBSERVED]. The 29 generated metrics model names recorded in P3-09 supply the vocabulary [A3|OBSERVED].

## ADR-04 — Metric and instrument identity

**Context.** Identity is the hinge the view layer, the storage layer and duplicate-registration detection all turn on, and the specification requires that two instruments differing only in case-insensitive name be treated as the same instrument while conflicting descriptors be reported. The reference implementation solves this with a descriptor subpackage of three files that the view and storage layers key on. Nothing equivalent exists here, and the repository's own instrumentation-scope type is the only identity-bearing value in the tree today.

**Status.** Accepted. No baseline finding renders it moot.

**Decision.** Instrument identity MUST be a canonical value derived from the instrumentation scope, the instrument name compared case-insensitively, the unit, the description, the instrument kind, the numeric kind and the advisory parameters. The resource MUST be held once at provider level and MUST NOT participate in per-instrument identity. Invalid input MUST degrade to a safe default rather than raise: an invalid name MUST yield a working instrument, and default and sentinel values MUST be expressed as constants rather than freshly allocated objects. Two registrations that agree on identity MUST return the same instrument. Two registrations that share a name under one meter but disagree on any other identity component MUST both produce a working instrument, MUST resolve stream identity in favour of the first registration seen, and MUST emit one deterministic diagnostic through the SDK error handler. No registration path SHALL throw into the host [A3|DECIDED].

**Alternatives.** (a) Reject a conflicting duplicate registration with an exception. (b) Let the later registration win the stream identity. (c) Include the resource in instrument identity. (d) Compare instrument names case-sensitively. [A5|PROPOSED]

**Rejected because.** (a) is rejected because telemetry must never destabilise the host and nothing may escape a public API method, so a throwing registration path converts an instrumentation mistake into an application crash. (b) is rejected because the specification resolves duplicate name conflicts in favour of the first seen, and a last-wins rule makes exported stream metadata depend on class-loading or initialisation order, which is untestable. (c) is rejected because the resource is provider-scoped and identical for every instrument under one provider, so including it enlarges every key on the hot path for no discriminating power. (d) is rejected because the specification defines instrument name comparison as case-insensitive, and a case-sensitive rule would silently create two series for names that the specification treats as one, which is exactly the silent-data-loss class Section 6 targets [A4|DECIDED].

**Consequences.** A canonical identity type is a prerequisite for the storage map in ADR-05 and for the view matching in ADR-07, which is why `M-07` precedes `M-09` and `M-12`. The diagnostic requirement means duplicate detection needs a registry keyed by name within a scope in addition to the identity map, and the reference implementation's source-location attribution machinery is explicitly found unnecessary here: it exists to name the registration site in the warning, the repository has no equivalent facility, and a deterministic identity-difference message satisfies the compliance requirement without it [A4|DECIDED].

**Evidence.** `Instrument.kt:L16-L32` fixes the name, unit and description members that identity must cover [A3|OBSERVED]. `MeterProviderImpl.kt:L42-L43` shows the existing scope-keyed cache pattern that instrument identity mirrors one level down [A3|OBSERVED]. `AGENTS.md:L34-L39` requires that nothing escape a public method, that invalid input degrade, and that errors route to the error handler [A6|OBSERVED]. `CONTRIBUTING.md:L55` requires constants for invalid and default values [A6|OBSERVED]. `sdk-api/src/commonMain/kotlin/io/opentelemetry/kotlin/error/SdkErrorHandler.kt:L9,L14` is the routing seam [A3|OBSERVED]. The duplicate-registration and syntax requirements are the compliance rows at `spec-compliance-matrix.md:L124-L131`, and the reference implementation's debug package is the four files recorded in P3-09's subpackage counts [A4|OBSERVED].

## ADR-05 — Attribute materialisation and series keys

**Context.** This is the first-order design problem of the whole port, and it has no analogue in the reference implementation. The API-level attribute idiom is a write-only builder lambda: `AttributesMutator` is 72 lines of ten setters with no read accessor, no value semantics and no equality contract, and the recording signature takes `AttributesMutator.() -> Unit`. The readable snapshot type lives one layer up in `sdk-api`. The reference implementation keys metric storage directly on an immutable attributes value object, which is exactly what does not exist at this layer — and a lambda cannot be a map key. Every synchronous measurement must therefore materialise the receiver into a hashable container on the hot path before the aggregation lookup, which makes allocation per measurement, cardinality-limit key identity, the placement of view attribute filtering and exemplar attribute retention all consequences of one decision. The reference's 10,116-byte filtered-attributes optimisation is a redesign here, not a port.

**Status.** Accepted, and load-bearing. Section 1 names the failure mode it exists to prevent, and R-01 carries the residual exposure.

**Decision.** Every synchronous measurement MUST materialise the supplied `AttributesMutator` receiver lambda into an immutable, readable, order-independent, hashable internal key exactly once, at the recording call, before any aggregation-map lookup. That key MUST define equality and hash code over the set of typed key-value entries and MUST be independent of the order in which setters were invoked. The key MUST preserve type distinctions, so that a long, a double and a string carrying the same textual value are three distinct entries and never collapse. Object identity, lambda identity, deferred replay of the lambda at collection time, and any reliance on map iteration order MUST NOT be used as identity mechanisms. Two keys that compare equal MUST address the same series; a hash collision MUST fall back to full equality comparison and MUST NOT merge unequal keys [A3|DECIDED].

**Alternatives.** (a) Retain the lambda and replay it during collection. (b) Key storage on the lambda instance. (c) Reuse the SDK's readable attribute container as the public measurement parameter type. (d) Port the reference implementation's filtered-attributes optimisation directly. [A5|PROPOSED]

**Rejected because.** (a) is rejected because the lambda closes over caller state that may have changed or become invalid by collection time, so replay would attribute a measurement to attributes that were not current when it was taken, and because a lambda cannot be compared for value equality, leaving no way to accumulate two measurements that share attributes. (b) is rejected because each call site allocates a fresh lambda instance, so identity keying would create one series per call and produce unbounded cardinality that looks like correct behaviour until the export payload is inspected. (c) is rejected because the readable container lives in `sdk-api` while the measurement surface lives in `api`, so using it publicly would invert the layering and force `api` to depend on the SDK. (d) is rejected because that optimisation is built on an immutable attributes value object that exists in the reference API and has no counterpart here, so it is a redesign rather than a port and must not be presented as one [A3|DECIDED].

**Consequences.** Materialisation is the dominant per-measurement cost, so the key type is the correct target for allocation tuning and the natural boundary for cardinality-limit accounting. Because the key is the unit of series identity, view attribute filtering MUST execute during materialisation rather than after it, or two distinct keys would be created for measurements that the view collapses into one series. Exemplar attribute retention MUST capture the entries that filtering removed, because the compliance requirement is that exemplars retain attributes not preserved by aggregation or view configuration [A4|DERIVED].

**Evidence.** `api/src/commonMain/kotlin/io/opentelemetry/kotlin/attributes/AttributesMutator.kt:L11` declares the interface and L17 through L71 declare ten setters, with no read accessor and no equality contract in 72 lines [A3|OBSERVED]. `api/src/commonMain/kotlin/io/opentelemetry/kotlin/metrics/ObservableLongMeasurement.kt:L19` and `api/src/commonMain/kotlin/io/opentelemetry/kotlin/metrics/MeterProvider.kt:L27` show the receiver-lambda idiom the API uses for attributes [A3|OBSERVED]. `sdk-api/src/commonMain/kotlin/io/opentelemetry/kotlin/attributes/AttributeContainer.kt:L11,L17` places the readable snapshot in the SDK layer as a map of string to any [A3|OBSERVED]. The exemplar retention requirement is the compliance row at `spec-compliance-matrix.md:L177`, and the reference optimisation is the filtered-attributes file measured at 284 lines and 10116 bytes in P3-11's calibration set [A4|OBSERVED].

## ADR-06 — Multiplatform concurrency and accumulation

**Context.** The hardest technical problem, and the one the language floor decides. The reference implementation's accumulation surface is seven files of striped adders — roughly 270 lines including a JRE-specific pair that exists only to select a newer implementation through a multi-release-jar pattern with no multiplatform analogue. Three options existed and two are foreclosed by measurement rather than by preference: the standard library's atomics package requires a Kotlin version above the frozen 2.0 language and API floor, and no atomics coordinate exists anywhere in the version catalog (T-20). What remains is the expect/actual layer that already exists in `platform-implementations` with nine primitives and actuals for three target families. Critically, none of the available options supplies a striped adder — a single compare-and-swap cell is not per-thread striping — so contended accumulation must be designed rather than imported.

**Status.** Accepted. Its concurrency conclusions were exercised by the vertical-slice spike Section 5.3 records, which measured two defects before its branch was discarded; the stripe count and the double-accumulation technique remain open and Section 8 ranks them fourth.

**Decision.** Accumulation primitives MUST be added to `platform-implementations` as expect declarations with per-target actual implementations, and new actual files MUST use the platform-suffixed file-name form the module's fixture loader and HTTP engine already use. The primitives MUST comprise a long accumulator and a double accumulator, each exposing an add operation and a read-and-reset operation, and the accumulator MUST be internally striped across a fixed bounded number of cells rather than sized from the processor count. The JVM and Android actual MUST delegate to the platform's own contended accumulator, available unconditionally under the declared JDK 11 toolchain. The JavaScript actual MUST be a single plain cell, because the target has no shared-memory concurrency. The Apple actual MUST use the module's existing lock primitive. A single global lock on the recording path MUST NOT be used. The Kotlin standard-library atomics MUST NOT be used, and no atomics library dependency SHALL be added [A3|DECIDED].

**Alternatives.** (a) Use the Kotlin standard-library atomics package. (b) Add the multiplatform atomics library to the version catalog. (c) Guard the aggregation map and every accumulator with one shared lock. (d) Size the stripe count from the available processor count. (e) Port the reference implementation's runtime-selected accumulator pair. [A5|PROPOSED]

**Rejected because.** (a) is rejected on rank-3 in-repository evidence rather than on external documentation: the platform layer states in its own source that the standard-library atomic long arrived as experimental in 2.1 while this project supports back to 2.0, and the 2.0 language and API floor is frozen by policy, so the package is unavailable and raising the floor is excluded. (b) is rejected because no atomics entry exists anywhere in the 108-line version catalog, so selecting it is a dependency addition, and because it would introduce a compiler plugin into a build where configuration-cache problems fail rather than warn; decisively, it would not even solve the problem, because it supplies a single compare-and-swap cell and not a striped accumulator. (c) is rejected because the recording path is the hottest path in the signal and a shared lock serialises every instrument in the process, and because the Apple actual would then take two locks per measurement. (d) is rejected because processor count is meaningless on the JavaScript target and misleading under a lock-based Apple actual, and because an unbounded stripe array multiplies memory per series by the core count, which interacts badly with cardinality limits. (e) is rejected because the reference pair exists only to select a newer implementation at runtime through a packaging mechanism that has no multiplatform counterpart; it collapses to one delegating actual here, and porting both halves would carry machinery with no purpose [A3|DECIDED].

**Consequences.** The genuinely novel work is the Apple and JavaScript actuals, so `M-08` is small in line count and high in risk, and it MUST land with shared behaviour tests in the common test source set alongside the existing primitive tests. Because the read-and-reset operation is the delta boundary, its atomicity is the property the lost-update oracle in Section 6 checks, and it is the reason ADR-07 assigns reset ownership to the reader rather than to the instrument. Because the Apple actual serialises on a lock, Apple throughput is expected to differ from JVM throughput, which is a documented characteristic rather than a defect [A3|DERIVED].

**Evidence.** `platform-implementations/src/commonMain/kotlin/io/opentelemetry/kotlin/AtomicLong.kt:L11-L13` records the standard-library atomics position and L15-L55 the existing expect shape [A3|OBSERVED]. `AtomicLong.jvm.kt:L18-L19` delegates to the platform atomic, `AtomicLong.js.kt:L11-L13` is a plain variable, and the Apple actual at `platform-implementations/src/appleMain/kotlin/io/opentelemetry/kotlin/AtomicLong.kt:L12,L45-L52` wraps every operation in a lock [A3|OBSERVED]. `gradle/libs.versions.toml` yields no match for an atomics entry across its 108 lines, and `.github/renovate.json5:L21,L23` freezes the language floor [A2|OBSERVED]. `buildSrc/src/main/kotlin/io/opentelemetry/kotlin/KotlinConfig.kt:L11,L17,L90-L91` fix the JDK 11 toolchain and the 2.0 floor [A2|OBSERVED]. `CONTRIBUTING.md:L60` places the primitives in this module [A6|OBSERVED]. `exporters-otlp/src/commonMain/kotlin/io/opentelemetry/kotlin/export/HttpClientInstance.kt:L28` with its three suffixed actuals, and the fixture loader's four suffixed actuals, fix the file-naming convention; the existing `AtomicLong` Apple and lock files do not use the suffix, and new files MUST follow the suffixed form rather than that inconsistency [A3|OBSERVED]. The reference concurrency package is the seven files enumerated in P3-11's reference measurements [A4|OBSERVED].

## ADR-07 — Aggregations, temporality, cardinality and exemplars

**Context.** The specification defines six aggregations, three temporality behaviours and an exemplar model, and they are not equal in cost: the reference implementation's exponential-histogram aggregator is 11,029 bytes over 296 lines with a 9,885-byte bucket type beside it, while sum and last-value are small. One published correctness trap is specific enough to design against — under delta temporality the exponential histogram's scale must reset each collection cycle or precision never recovers, a defect observed in another language's SDK. Cardinality is the other half of the same question, because every recording in this API allocates a fresh receiver lambda, so an unbounded attribute set produces one series per call unless a limit and an overflow series exist.

**Status.** Accepted, with the exponential histogram's wave placement left open. Section 8 ranks that question fifth because it changes the compliance count reported against the 81-row denominator but blocks nothing else.

**Decision.** The first implementation wave MUST provide the drop aggregation, the default aggregation selected by instrument kind, the sum aggregation, the last-value aggregation and the explicit-bucket histogram aggregation. The base-2 exponential histogram MUST be specified by this plan and MUST be delivered in its own unit after the histogram unit, because its scale management is independent of every other aggregation. Accumulation state MUST be owned per registered reader, so that a delta-preferring reader and a cumulative-preferring reader observe independent state and neither reader's collection can reset or advance the other's. Under delta temporality every collection MUST snapshot and reset atomically, and any histogram scale MUST be reset with the snapshot so that precision recovers after an outlier cycle. Under cumulative temporality collection MUST snapshot without reset and MUST preserve a per-series start timestamp. When a stream exceeds its cardinality limit the SDK MUST fold further series into one reserved overflow series and MUST NOT discard measurements silently. Exemplar sampling MUST be bounded by a fixed-size reservoir per series, MUST use an aligned-bucket reservoir for an explicit-bucket histogram with more than one bucket, MUST capture the active trace and span identifiers and the measurement timestamp, and MUST retain the attributes that view filtering removed. Programmatic configuration of all of the above MUST be provided; wiring every generated declarative model is explicitly out of scope for the first wave [A4|DECIDED].

**Alternatives.** (a) Share one accumulation state across all readers and convert temporality at export. (b) Deliver the exponential histogram in the same unit as the explicit-bucket histogram. (c) Drop measurements once a cardinality limit is reached. (d) Retain exemplars without a bound and prune at export. [A5|PROPOSED]

**Rejected because.** (a) is rejected because a delta collection mutates the state it reads, so a second reader collecting afterwards would observe an interval that had already been consumed, producing under-reporting that no single-reader test can detect. (b) is rejected because the exponential histogram carries scale selection, rescaling and a bucket structure that together exceed what remains of the 500-line unit budget once the explicit-bucket histogram and its tests are counted, so combining them would force either an oversized unit or thin tests. (c) is rejected because dropping is precisely the silent data loss Section 6 names as the primary unacceptable failure, whereas an overflow series is observable in the export payload and tells an operator that a limit was reached. (d) is rejected because an unbounded reservoir makes exemplar memory a function of measurement volume, which contradicts the requirement that telemetry never destabilise the host [A4|DECIDED].

**Consequences.** Per-reader state ownership means the storage layer is keyed by reader as well as by series, which is the single largest structural difference from a naive port and the reason `M-09` precedes `M-18`. The delta scale-reset requirement is an explicit oracle in Section 6 rather than an implementation detail, because it is a known defect class in other implementations of this specification. Because the exponential histogram is deferred to its own unit, the default-by-instrument-kind selector MUST already be able to name it, so the selector lands with the aggregation set and not with the histogram [A4|DERIVED].

**Evidence.** The aggregation, reader-configuration, cardinality and exemplar requirements are the compliance rows at `spec-compliance-matrix.md:L152-L158`, `L159-L161`, `L188-L191` and `L171-L184`, and the cumulative start-timestamp requirement is at `L191` [A4|OBSERVED]. The generated model names `SumAggregation`, `LastValueAggregation`, `DropAggregation`, `DefaultAggregation`, `ExplicitBucketHistogramAggregation`, `Base2ExponentialBucketHistogramAggregation`, `ExemplarFilter`, `CardinalityLimits`, `View`, `ViewSelector` and `ViewStream` supply the vocabulary and are recorded in P3-09 [A3|OBSERVED]. The reference implementation concentrates its complexity in exactly these areas, with 17 files in state, 19 in aggregator, 16 in exemplar and 14 in view, and its delta synchronous storage is its largest single file at 585 lines [A4|OBSERVED]. `AGENTS.md:L31,L36` require that the host never be destabilised and that invalid input degrade [A6|OBSERVED].

## ADR-08 — Asynchronous instruments and callback lifecycle

**Context.** Asynchronous instruments run user-supplied code on the collection path, which is the one place the repository's error doctrine is least negotiable: telemetry must never destabilise the host, nothing may escape a callback the SDK invokes, and all user-supplied code is to be assumed hostile. The existing contract carries an in-source marker stating that after-creation callback registration is still to be added, and the interface is already `AutoCloseable`, so the shape of registration is genuinely undecided in the tree rather than merely undocumented. Helpfully, the static-analysis configuration deliberately disables the swallowed-exception, generic-exception-caught and instance-of-check rules, so containment will not fight the gate.

**Status.** Accepted. The after-creation registration member is deliberately deferred rather than designed; Section 8 ranks that question sixth.

**Decision.** An asynchronous instrument MUST accept its callback at creation time, and the first wave MUST NOT add an after-creation registration member, because the existing contract marks that member as still to be added and adding it changes the instrument's public shape a second time. Callbacks MUST be invoked only during a collection cycle and MUST NOT be invoked from the recording path or from a background timer of their own. Every throwable raised by a callback MUST be caught at the invocation boundary, MUST be routed to the SDK error handler at a bounded cadence of at most one report per callback per collection cycle, and MUST NOT propagate into the collecting coroutine or the host. A callback that fails MUST leave previously collected series intact and MUST contribute no partial series for that cycle. Collection MUST NOT be re-entrant: a callback that triggers collection MUST be detected and MUST NOT corrupt or duplicate the in-flight snapshot. Closing an asynchronous instrument MUST unregister its callback, MUST be idempotent, and MUST NOT fail if the provider has already shut down. A permanent background push pipeline MUST NOT be introduced merely to serve callbacks [A3|DECIDED].

**Alternatives.** (a) Add an after-creation registration member in the first wave. (b) Let callback exceptions propagate so that failures are visible. (c) Report every callback failure on every cycle. (d) Run callbacks on a dedicated background schedule independent of collection. [A5|PROPOSED]

**Rejected because.** (a) is rejected because the contract's own gap marker signals that the member's shape is undecided, and shipping a guess would either be reversed or would freeze an experimental surface prematurely; the residual question is carried to Section 8 rather than answered by assumption. (b) is rejected because user-supplied code is to be assumed hostile and nothing may escape a callback the SDK invokes, so propagation would let one instrumentation defect abort an entire collection cycle and, through the aggregate lifecycle path, fail a flush the host is awaiting. (c) is rejected because a callback that fails every cycle would produce unbounded error volume proportional to the collection rate, which the bounded-cadence requirement in the repository's error-handling guidance exists to prevent. (d) is rejected because an independent schedule reintroduces the push model the specification replaced with a pull model, and it would make asynchronous values observable at instants no reader asked for [A3|DECIDED].

**Consequences.** Because callbacks run inside collection, callback cost is collection cost, so the collection timeout in ADR-09 must bound callback execution and a slow callback must degrade to a missing series for that cycle rather than to a hung flush. Because containment is required at the callback boundary, the static-analysis configuration's deliberate disabling of the swallowed-exception, generic-caught-exception and instance-of-check rules is what allows the containment to be written at all, so containment does not fight the gate [A2|DERIVED].

**Evidence.** `api/src/commonMain/kotlin/io/opentelemetry/kotlin/metrics/AsynchronousInstrument.kt:L11-L12,L18-L19` documents that closing unregisters the callbacks and marks after-creation registration as still to be added [A3|OBSERVED]. `AGENTS.md:L34-L35,L38-L39` forbid escape from a callback the SDK invokes, assume user code hostile, and require bounded routing to the error handler [A6|OBSERVED]. `sdk-api/src/commonMain/kotlin/io/opentelemetry/kotlin/error/SdkErrorHandler.kt:L6,L14` is the routing contract [A3|OBSERVED]. `config/detekt/detekt.yml:L25-L31` disables the three exception rules that would otherwise reject containment code [A2|OBSERVED]. `implementation/src/commonMain/kotlin/io/opentelemetry/kotlin/OpenTelemetryImpl.kt:L76-L81` shows the existing pattern of catching any throwable at a lifecycle boundary and mapping it to failure [A3|OBSERVED].

## ADR-09 — MetricReader pipeline

**Context.** Metrics is the one signal whose pipeline shape diverges from the two already built here. `exporters-core` carries exactly eight files per signal for logging and tracing — batch processor, composite exporter and processor, core exporter API, noop pair, simple processor and stdout exporter — and that shape is a push model. The specification's metrics pipeline is a pull-based reader: collection is initiated by the reader, not by the instrument. Stating this explicitly is what stops "mirror the logs signal" being read as an instruction to produce eight processor files the specification does not have.

**Status.** Accepted. It inherits P3-09's finding that the reader and exporter configuration models already exist by name.

**Decision.** The metrics pipeline MUST be pull-based. A metric reader MUST expose a suspending collection operation that returns the aggregated points it produced, and the meter provider MUST own registration, so that a reader is bound to exactly one provider and a provider knows every reader before the first measurement is recorded. Collection MUST take a snapshot of every registered instrument's per-reader state, MUST apply reset semantics according to that reader's temporality as fixed by ADR-07, and MUST complete or time out rather than block indefinitely. Multiple readers MUST be isolated: one reader's collection MUST NOT observe, reset or delay another's. A periodic reader MUST schedule collection on a scope tied to its own lifetime, MUST cancel that scope on shutdown, and MUST apply backpressure by skipping a cycle whose predecessor has not completed rather than by queueing cycles. Shutdown MUST perform at most one final collection, MUST then refuse further collection while returning a successful result, and MUST be idempotent. The eight-file push-processor shape used by logging and tracing MUST NOT be replicated, and no simple or composite metric processor SHALL be introduced [A3|DECIDED].

**Alternatives.** (a) Mirror the logging signal's batch and composite processor shape for metrics. (b) Let an exporter pull directly from instrument storage with no reader between them. (c) Queue overlapping periodic cycles. (d) Allow a reader to be registered with more than one provider. [A5|PROPOSED]

**Rejected because.** (a) is rejected because a processor is fed by the recording path while a reader is driven by collection, so the eight files would either be empty shells or would reintroduce push semantics the specification does not define for metrics; naming and module layout are borrowed from the logging signal, pipeline shape deliberately is not. (b) is rejected because temporality, aggregation defaults and cardinality limits are reader-scoped decisions, so removing the reader would either duplicate those decisions in every exporter or lose them. (c) is rejected because queueing converts a slow exporter into unbounded memory growth and delivers stale intervals late, whereas skipping is observable and bounded. (d) is rejected because per-reader accumulation state is owned by the provider, so a shared reader would make one provider's reset visible to another's series [A3|DECIDED].

**Consequences.** Because collection is suspending and returns its points, the in-memory exporter and the golden-file tests can drive collection deterministically with no timer, which is what makes the correctness evidence in Section 6 reproducible. Because registration precedes recording, adding a reader after measurements exist is a defined error path rather than an undefined one. Because there is no processor, `exporters-core` gains a smaller file set for metrics than for either other signal, which is an expected asymmetry and not an omission [A3|DERIVED].

**Evidence.** The eight-file-per-signal processor shape is measured in P3-10's register rows for `exporters-core` and is identical for logging and tracing [A3|OBSERVED]. `sdk-api/src/commonMain/kotlin/io/opentelemetry/kotlin/export/TelemetryCloseable.kt:L15,L20` fixes the suspending lifecycle contract a reader must satisfy [A3|OBSERVED]. `exporters-core/src/commonMain/kotlin/io/opentelemetry/kotlin/export/BatchTelemetryDefaults.kt:L26,L31,L41,L46` supplies the existing schedule and timeout constants a periodic reader's defaults are drawn from [A3|OBSERVED]. The reader requirements are the compliance rows at `spec-compliance-matrix.md:L159-L161,L164,L170,L192`, and the reference periodic reader is measured at 316 lines in P3-11's calibration set [A4|OBSERVED]. `implementation/src/commonMain/kotlin/io/opentelemetry/kotlin/metrics/MeterProviderImpl.kt:L46-L50` shows the timeout-wrapped lifecycle the reader plugs into [A3|OBSERVED].

## ADR-10 — Exporters and wire format

**Context.** More of this is already built than a reading of the reference implementation would suggest. The protobuf module downloads the full protocol archive at build time and runs code generation over the entire protocol tree with service roles disabled, so the metrics message types already compile and no definition needs checking in. The transport seam is already generic over the exported type with backoff, jitter, partial-success handling and a shutdown timeout. The endpoint enum holds exactly two entries. What remains is marshalling, one enum entry, and a collector-fake route — and one liability: the generic seam reports success before transport completes, which is a silent-data-loss path Section 6 names.

**Status.** Accepted. Conditional on the seam repair `M-01` carries; reuse of an unrepaired seam is explicitly not sanctioned by this record.

**Decision.** The metrics exporter MUST reuse the protocol message types already generated from the pinned protocol archive and MUST NOT check in or regenerate any protocol definition. It MUST reuse the existing generic retry and transport seam by instantiating it with a metrics export action, and MUST NOT introduce a second retry, backoff or jitter implementation. It MUST reuse the existing per-target HTTP engine actuals. A metrics entry MUST be added to the internal OTLP endpoint enumeration, and because that enumeration is internal this addition MUST NOT be described as a public dump change. Marshalling MUST follow the three-file shape the other signals use, comprising a response deserializer, a service request creator and a data conversion file, and MUST reuse the shared attribute and common conversion helpers. gRPC service stubs MUST NOT be generated or claimed, because protocol code generation is configured to emit no service role [A2|DECIDED].

**Alternatives.** (a) Check in the metrics protocol definitions and generate from a local copy. (b) Write a metrics-specific retry and transport path. (c) Add a gRPC transport alongside the HTTP one. (d) Emit metrics through the existing logging exporter path with a different endpoint. [A5|PROPOSED]

**Rejected because.** (a) is rejected because generation already covers the entire protocol tree from a pinned archive version, so a checked-in copy would duplicate generated types and drift silently from the pinned version. (b) is rejected because the existing seam already implements exponential backoff with jitter, treats a partial success and a client error as terminal, honours a server-supplied retry delay, and bounds shutdown, so a second implementation would be more code with fewer of those properties. (c) is rejected because code generation is configured to produce no service role, so no stubs exist to call, and adding gRPC is outside the declared scope. (d) is rejected because the exporter is typed over the telemetry it exports and metrics points are not log records, so sharing the path would require erasing the type that makes the seam safe [A2|DECIDED].

**Consequences.** The exporter unit is small because only marshalling and endpoint selection are new, which is why `M-21` fits well inside the unit budget while the conversion unit `M-20` is sized by the breadth of the point types rather than by transport. Because the collector fake raises an error on an unrecognised path rather than ignoring it, the endpoint addition and the fake's route table MUST land together or the end-to-end smoke test fails loudly, which is the desired behaviour and is captured in `M-21`'s verification [A3|DERIVED].

**Evidence.** `exporters-protobuf/build.gradle.kts:L17` pins the protocol version, L33 includes the whole protocol tree, and L78 sets the service role to none [A2|OBSERVED]. `exporters-otlp/src/commonMain/kotlin/io/opentelemetry/kotlin/export/TelemetryExporter.kt:L15-L22,L42-L71,L78-L81,L85-L89` is the generic seam with its terminal-response handling, jitter and shutdown bound [A3|OBSERVED]. `exporters-otlp/src/commonMain/kotlin/io/opentelemetry/kotlin/export/OtlpEndpoint.kt:L3-L6` is internal with two entries [A3|OBSERVED]. `HttpClientInstance.kt:L28` and its three actuals supply per-target transport [A3|OBSERVED]. The per-signal file shapes measured in P3-10's register rows fix the two-file transport and three-file marshalling templates [A3|OBSERVED]. `smoke-test/src/commonTest/kotlin/io/opentelemetry/kotlin/smoketest/FakeOtlpServer.kt:L35-L37` routes two paths and raises an error on any other [A3|OBSERVED].

## ADR-11 — Lifecycle, shutdown and error containment

**Context.** The repository's lifecycle contract is a coroutine-suspend pair returning a sealed success-or-failure result, not the reference implementation's completable result code, and the facade already fans force-flush and shutdown across all three providers under a single aggregate timeout with all-success semantics. Translating reference call sites is therefore a per-file design decision rather than a mechanical rename, and a metrics flush that overruns its share of the aggregate bound degrades the two signals that already work.

**Status.** Accepted. No baseline finding renders it moot; R-15 carries the residual timeout exposure.

**Decision.** Every metrics reader and exporter MUST implement the repository's suspending closeable contract and MUST return the sealed success or failure result, and the reference implementation's completable result-code contract MUST NOT be introduced. The facade's single aggregate lifecycle timeout and its all-success combination semantics MUST be preserved unchanged, so a metrics flush that fails or times out MUST make the aggregate result a failure without altering how the tracer and logger results are combined. The provider's existing post-shutdown behaviour MUST be preserved: a meter requested after shutdown MUST be the no-op meter rather than an error. Flush and shutdown MUST be idempotent, and a second shutdown MUST return success without repeating work. No telemetry failure SHALL escape a host-facing API or a callback the SDK invokes; failures MUST be represented in the returned result and routed to the error handler [A3|DECIDED].

**Alternatives.** (a) Introduce a completable result type for metrics to match the reference implementation. (b) Give the metrics provider its own lifecycle timeout separate from the aggregate one. (c) Make a post-shutdown meter request raise. (d) Treat a metrics flush failure as a partial success so that the aggregate result stays successful. [A5|PROPOSED]

**Rejected because.** (a) is rejected because the repository's contract is suspending with a sealed result and every existing signal implements it, so a second lifecycle vocabulary would require adapters at every call site and would make composite closure unable to combine results uniformly. (b) is rejected because the facade already applies one aggregate bound across all three providers, and a nested independent timeout would make the observable flush duration the sum of two bounds rather than the documented one. (c) is rejected because the provider already returns a no-op meter after shutdown, so raising would both change released behaviour and violate the rule that nothing escape a public method. (d) is rejected because the facade requires every result to be successful, and reporting a masked failure as success would hide dropped telemetry from the only signal the host has [A3|DECIDED].

**Consequences.** Because the aggregate timeout is fixed and shared, the metrics flush path must be measurably faster than that bound, which makes the reader's own collection timeout a design input rather than an afterthought. Because the composite closer requires all delegates to succeed, registering a failing exporter degrades the provider's flush result, so exporter failure handling belongs inside the exporter's retry seam and not in the provider [A3|DERIVED].

**Evidence.** `sdk-api/src/commonMain/kotlin/io/opentelemetry/kotlin/export/TelemetryCloseable.kt:L15,L20` and `OperationResultCode.kt:L9,L14,L19` fix the contract [A3|OBSERVED]. `implementation/src/commonMain/kotlin/io/opentelemetry/kotlin/OpenTelemetryImpl.kt:L36` declares the aggregate bound of 3000 ms, L41-L74 fans lifecycle across all three providers including the meter provider, and L83-L87 requires every result to be successful [A3|OBSERVED]. `MeterProviderImpl.kt:L38` returns the no-op meter once shut down, via `sdk-common/src/commonMain/kotlin/io/opentelemetry/kotlin/export/ShutdownState.kt:L20` [A3|OBSERVED]. `MutableShutdownState.kt:L30-L39` makes shutdown idempotent by returning success when already shut down [A3|OBSERVED]. `CompositeTelemetryCloseable.kt:L8,L16-L30` requires every delegate to succeed and bounds each with a timeout [A3|OBSERVED]. `AGENTS.md:L34-L39` fixes the containment doctrine [A6|OBSERVED].

## ADR-12 — Correctness and reference strategy

**Context.** The harness the correctness strategy needs already exists: an abstract common rule with a declarative configuration, a fake clock, prebuilt provider lambdas and golden-file assertions, with one subclass per back end running deliberately identically-named scenarios against overlapping golden files. Its reference-delegating back end lives in a JVM-and-Android module's test source set, which bounds what the comparison can reach; and the reference implementation itself satisfies only 75 of the 88 compliance rows, which bounds what agreement with it can prove. Both bounds are structural rather than rhetorical, so they are stated as limits and answered by gating.

**Status.** Accepted. Its reach limit is restated in Section 6 with both bounds, and G3-07 rather than a label carries the unverified-platform response.

**Decision.** Correctness evidence MUST extend the harness that already exists rather than introduce a parallel one: the abstract common test rule gains a meter-provider hook and metrics assertions, the Kotlin back end exercises the new pipeline, and the reference-delegating back end exercises the compatibility façade. Differential comparison against the Java reference at tag `v1.65.0` MUST be used only on the JVM and only for semantics both implementations claim to support. Coverage for Android, JavaScript and Apple MUST come from shared behaviour, property and golden-file tests in common test source sets, not from the differential. This document MUST NOT claim that the Java differential reaches Apple or JavaScript, and MUST NOT claim that Apple is wholly unverifiable. Golden fixtures MUST be added to the existing fixture directories and MUST be loaded through the existing multiplatform loader [A3|DECIDED].

**Alternatives.** (a) Build a second harness dedicated to metrics. (b) Treat the Java differential as the primary oracle for all targets. (c) Declare Apple unverified and exclude it from correctness evidence. (d) Compare against the reference implementation's current default branch rather than the pinned tag. [A5|PROPOSED]

**Rejected because.** (a) is rejected because the existing rule already supplies a fake clock, deterministic identifiers, in-memory exporters and golden-file assertions, and a second harness would duplicate them while allowing the two to diverge in scenario naming. (b) is rejected because the reference-delegating back end lives in the JVM test source set of a JVM and Android module and therefore cannot compile for Apple or JavaScript, and because the reference itself satisfies only 75 of the 88 compliance rows, so it is silent about the remainder. (c) is rejected because both configured Apple targets compile and the repository already stages common test resources onto the iOS simulator, so shared tests do execute there on a macOS host; the accurate statement is that Apple execution requires a macOS host, not that Apple is unverifiable. (d) is rejected because measurements against a moving branch are not reproducible and would make the reference-drift risk unmeasurable [A4|DECIDED].

**Consequences.** The reach limit is structural, so it is stated as a limit in Section 6 and mitigated by shared tests rather than by aspiration. Because a skipped Apple simulator task still yields a zero exit code, G3-07 MUST require positive evidence that the Apple tasks ran, and the full build invocation's success on a non-macOS host MUST NOT be read as Apple evidence. Because the abstract rule currently configures only tracer and logger providers, the meter hook is a prerequisite for every differential metrics scenario, which places `M-24` before `M-25` in dependency order [A2|DERIVED].

**Evidence.** The harness triple is `integration-test/src/commonMain/kotlin/io/opentelemetry/kotlin/framework/OtelKotlinTestRule.kt:L27,L33,L43,L46-L47,L52-L65`, `implementation/src/commonTest/kotlin/io/opentelemetry/kotlin/integration/test/IntegrationTestHarness.kt:L14-L24` and `compat/src/jvmTest/kotlin/io/opentelemetry/kotlin/framework/OtelKotlinHarness.kt:L12-L29`, the last of which is confined to a JVM and Android test source set [A3|OBSERVED]. `implementation/build.gradle.kts:L64-L71` stages common test resources onto the simulator [A2|OBSERVED]. The measured Apple outcome is recorded in P3-10: both Apple compilations exit 0 on this host while the simulator test task is reported skipped [A2|OBSERVED]. The reference satisfies 75 of 88 rows per P3-09's matrix parse, and the pinned tag resolves to a fixed commit per the reference register [A4|OBSERVED]. The multiplatform fixture loader and its suffixed actuals are recorded in the reference register [A3|OBSERVED].

## ADR-13 — Binary-compatibility authority and the platform dump disagreement

**Context.** Two rank-A1 sources contradict each other. `api/api/jvm/api.api` is 402 lines and carries the whole metrics package plus the provider accessor; `api/api/android/api.api` is 462 lines and contains zero occurrences of `metrics` and no accessor (T-17). The dump check passes on both, which means it demonstrably does not enforce the Android variant. The authority ladder adjudicates JVM against klib and says nothing about JVM against Android, and zero klib baselines exist, so the tiebreak it does offer is inoperative. A governance gap of this kind cannot be left to a later reader: the document must rule, cite both sides, and carry the residual consequence as a question.

**Status.** Accepted as a ruling this document owns. The residual question — whether regenerating the eleven stale Android dumps is in scope — is not decided here and Section 8 ranks it first.

**Decision.** The JVM and Android dumps SHALL be treated as equal-rank evidence, and their present disagreement SHALL be recorded as a conflict rather than resolved by preference. The klib tiebreak SHALL be recorded as unavailable, because no `.klib.api` baseline exists anywhere in the repository, and no claim of klib binary-compatibility evidence SHALL be made while that remains true. For baseline facts about what the metrics surface contains today, the JVM dump corroborated by rank-3 source SHALL govern, on the specific ground that the Android dump omits declarations which both the JVM dump and the source contain, which is a staleness signature rather than a contradictory assertion. That ruling SHALL NOT be generalised into permission to skip Android review: every unit that changes public API MUST review both the JVM and the Android dump for the modules it touches and MUST state which of the two it changed. The pre-existing baseline disagreement MUST be isolated in its own unit before any new API delta is accepted, so that a reviewer can distinguish a metrics-induced delta from the inherited one. Passing the dump-check task alone MUST NOT be treated as evidence of Android parity [A1|DECIDED].

**Alternatives.** (a) Regenerate the eleven Android dumps as part of the first metrics unit. (b) Prefer the Android dump wherever the two disagree. (c) Average or merge the two dumps into a single reconciled view. (d) Rely on the dump-check task and ignore the disagreement. [A5|PROPOSED]

**Rejected because.** (a) is rejected because regenerating them produces a large diff unrelated to metrics that would obscure the metrics delta under review, and because the decision of whether that regeneration is in scope at all is not this plan's to make; it is carried to Section 8 as a genuine residual. (b) is rejected because the Android dump lacks declarations that rank-3 source demonstrably contains, so preferring it would assert that types which exist do not. (c) is rejected because merging two dumps fabricates a third artifact that no tool generates and no gate checks, and the outcome triple forbids averaging disagreeing sources. (d) is rejected because the task passes today while the two dumps disagree, which proves the task does not enforce the Android variant and therefore cannot be the evidence that it matches [A1|DECIDED].

**Consequences.** Every unit that touches public API carries a dump regeneration and an explicit statement of which platform dumps changed, and a conflict on rebase is resolved by re-running the dump task rather than by hand-merging. Because the experimental marker is not excluded from dumps, the dump diff is also the primary artifact for the experimental-annotation audit, so G3-08 and G3-09 are verified from the same review [A2|DERIVED].

**Evidence.** `api/api/jvm/api.api:L8,L240-L275` carries `getMeterProvider` and the whole metrics package, while `api/api/android/api.api` contains zero occurrences of `metrics` and zero of `getMeterProvider` across 462 lines, as recorded with its producing commands in P3-05 [A1|OBSERVED]. Zero files match `*.klib.api` [A1|OBSERVED]. `buildSrc/src/main/kotlin/io/opentelemetry/kotlin/BinaryCompatConfig.kt:L6-L10` configures only whether validation is disabled and leaves non-public markers unset, so experimental declarations appear in dumps [A2|OBSERVED]. `./gradlew apiCheck detekt --stacktrace` exits 0 on the untouched tree across 31 dump-check tasks despite the disagreement [A2|OBSERVED]. `AGENTS.md:L24` requires the dump task to be run before a change is proposed [A6|OBSERVED].


## ADR-14 — Exporter and reader selection seam

**Context.** Five configuration inputs decide which reader and which exporter a meter provider ends up with: the programmatic DSL ADR-03 grows,
the environment variables `M-22` adds, the layered behaviour model that today carries only a tracer slot, the generated configuration models
that already name every reader and exporter variant, and the compiled-in defaults. Two of those five did not exist for any prior signal, so
there is no precedent in the tree for resolving them against each other. Left unspecified, each unit would resolve precedence locally and the
resulting behaviour would be whichever unit landed last. The question is genuinely a decision rather than an aside, which is why it is a record
of its own rather than a paragraph inside ADR-03.

**Status.** Accepted. It is a new record; nothing in ADR-01 through ADR-13 can hold it without one of them becoming two decisions.

**Decision.** Reader and exporter selection MUST resolve through a single seam with a fixed precedence: an explicitly constructed reader or
exporter passed through the DSL wins; then the layered behaviour model, lower layers overridden by higher ones through the merge mechanism that
already exists; then environment variables; then the compiled-in default, which MUST be no reader at all. The seam MUST be a declared factory
contract in `sdk-api`, not a `when` over string names spread across call sites, and every name it accepts MUST be one the generated
configuration models already define rather than a new vocabulary. Selecting an unknown name MUST degrade to no reader plus one diagnostic and
MUST NOT throw, because selection happens during provider construction and a throw there destabilises the host at start-up. Unit `M-22` adds
the seam's environment path, gated by G3-01 and G3-02, and unit `M-15` adds its DSL path [A3|DECIDED].

**Alternatives.** (a) Resolve selection inside each exporter's own companion factory. (b) Make environment variables win over programmatic
configuration, matching the precedence some other SDKs use. (c) Default to an OTLP reader when nothing is configured, so that the SDK exports
by default. [A5|PROPOSED]

**Rejected because.** (a) is rejected because precedence would then be a property of whichever exporter was asked first, so two exporters on
the classpath would produce order-dependent behaviour and no test could pin it. (b) is rejected because it lets an environment variable
silently override code a developer wrote and can read, which inverts the debuggability the repository's error doctrine assumes; the environment
remains authoritative only where code said nothing. (c) is rejected because a default exporter opens a network path nobody asked for the moment
the SDK is initialised, which is precisely the destabilisation the host-safety doctrine forbids, and because it would make the no-configuration
case untestable without a network fake [A2|DECIDED].

**Consequences.** Because the default is no reader, a metrics pipeline that is configured wrongly produces nothing rather than exporting to the
wrong place, and Section 6 therefore lists "no reader registered" as a loss mode with its own oracle rather than treating it as user error.
Because the seam is a declared contract in `sdk-api`, it appears in that module's dump and is subject to G3-08's annotation audit. Because
selection accepts only generated model names, the declarative-configuration non-goal stays bounded: the vocabulary is shared, the parser is not
built [A3|DERIVED].

**Evidence.** `sdk-api/api/jvm/sdk-api.api:L170` shows the meter-provider configuration block already public [A1|OBSERVED].
`behavior/src/commonMain/kotlin/io/opentelemetry/kotlin/behavior/OpenTelemetryBehavior.kt:L11-L18` carries a tracer-provider slot only, so a
meter slot is greenfield [A3|OBSERVED]. `behavior/src/commonMain/kotlin/io/opentelemetry/kotlin/behavior/BehaviorMerge.kt` supplies the
lower-overridden-by-higher merge this record reuses [A3|OBSERVED]. T-21 records the 113 generated models that supply the accepted names
[A2|OBSERVED]. `config-envar/src/commonMain/kotlin/io/opentelemetry/kotlin/config/envar/EnvVarConstants.kt` shows every recognised variable
today is a limit, so no exporter-selection variable exists to inherit [A3|OBSERVED]. `AGENTS.md:L36-L37` requires invalid input to degrade to a
safe default and confines fast failure to initialization misconfiguration [A6|OBSERVED].

## ADR-15 — Upstream divergence and rebase posture

**Context.** This work lands on a fork that is 3 commits ahead of its base and 15 behind `upstream/main` (T-05), against an upstream whose
maintainer opens changes in bursts. The collision surface is not the Kotlin source — it is the 27 checked-in dumps, which are machine-generated,
line-dense, and regenerated by every public-API change on both sides. Thirty-five units landing over a moving base will hit dump conflicts
repeatedly, and a hand-merged dump is indistinguishable from a correct one by inspection while being wrong in a way `apiCheck` may not catch
until much later.

**Status.** Accepted. It is a new record; ADR-13 rules on which dump governs, which is a different question from how divergence is absorbed.

**Decision.** Every unit MUST rebase onto the current base before its gates are run, and its dumps MUST be regenerated after the rebase rather
than before. A dump conflict MUST be resolved by discarding both sides and re-running the dump task, never by hand-merging or by taking one
side. A unit whose only remaining conflict is in a dump MUST NOT be considered blocked, because regeneration resolves it mechanically. Where an
upstream change lands that alters a contract a later unit depends on, the dependent unit MUST be re-derived from the changed contract rather
than merged into it, and the change MUST be recorded against the architecture record that owns the contract. No unit MAY pin, vendor or fork an
upstream file to avoid divergence. The merge gate for every unit is `./gradlew build :benchmark-android:assemble :benchmark-android:compileReleaseAndroidTestKotlin :benchmark-jvm:assemble :examples:example-app-android:assembleRelease koverXmlReport -x :gradle-integration-test:test --stacktrace` exiting 0 after the rebase, per G3-04 [A2|DECIDED].

**Alternatives.** (a) Batch all thirty-five units on a long-lived integration branch and rebase once at the end. (b) Freeze the base commit for the whole effort. (c) Hand-merge dump conflicts to keep rebases short. [A5|PROPOSED]

**Rejected because.** (a) is rejected because it converts many small dump conflicts into one large one and defers every integration signal to
the point where acting on it is most expensive; it also breaks the reviewable-diff guidance by making the effective diff the whole branch. (b)
is rejected because a frozen base cannot be merged without a rebase anyway, so freezing only moves the cost and adds staleness. (c) is rejected
because a dump is generated output whose correct value is a function of the source; a hand-merge asserts a value no tool produced, and the
regeneration that would have caught it is exactly the step being skipped [A2|DECIDED].

**Consequences.** Because dumps are regenerated after rebase, the dump delta in each unit's diff is measured separately from hand-authored lines,
which is why the change-unit measurement rule in Section 4.1 bounds the two independently. Because rebase precedes gates, a gate failure is
attributable to the unit rather than to drift. Because no file may be vendored, the pinned reference clone stays a read-only rank-A4 source and
never becomes a dependency [A2|DERIVED].

**Evidence.** T-05 measures the divergence in both directions [A2|OBSERVED]. T-16 counts the 27 dumps that form the conflict surface
[A2|OBSERVED]. `AGENTS.md:L24` requires the dump task to be run before a change is proposed, and L25 caps a reviewable diff at 500 lines
[A6|OBSERVED]. `buildSrc/src/main/kotlin/io/opentelemetry/kotlin/BinaryCompatConfig.kt:L6-L10` shows validation toggled per module with no
non-public marker exclusion, so every experimental declaration lands in a dump and maximises that surface [A2|OBSERVED].

## ADR-16 — Naming fidelity to the specification

**Context.** The repository states that `api` aims to stay as close to the specification as possible and that non-specification sugar belongs in
`api-ext`. Metrics is where that principle is hardest to hold, because the specification's own vocabulary collides with Kotlin convention in
several places: `record` is a Kotlin soft keyword in newer language versions, the specification's "observable" prefix competes with the
repository's existing `Observable*` measurement types, and the specification's `Meter` creation verbs are `createCounter`-style factories that
read oddly beside a builder. Left to unit-by-unit judgement, seven instrument families would acquire seven naming philosophies.

**Status.** Accepted. It is a new record; ADR-02 fixes the surface's shape, which is a different question from the words on it.

**Decision.** Public metrics names MUST follow the specification's own vocabulary wherever a specification name exists: instrument kinds are
counter, up-down counter, histogram and gauge; the asynchronous forms carry the observable prefix; the recording verb is `record` for
histograms and gauges and `add` for counters and up-down counters, matching the specification rather than unifying them. Builder members MUST
name the descriptor they set — unit, description, advice — rather than inventing synonyms. Where a specification name cannot be used verbatim
because the language reserves it or the repository already binds it, the name MUST be extended rather than replaced, and the deviation MUST be
recorded in the compliance sheet row for that concept. Convenience names, aliases and operator forms MUST NOT enter `api` and MUST be placed in
`api-ext` if they are wanted at all. Unit `M-03` and unit `M-04` add these names, gated by G3-08 and G3-09 [A6|DECIDED].

**Alternatives.** (a) Adopt Kotlin-idiomatic names throughout and document the mapping. (b) Unify the recording verb to `record` for every
instrument kind. (c) Add convenience overloads and operator forms alongside the specification names in `api`. [A5|PROPOSED]

**Rejected because.** (a) is rejected because the compliance sheet is row-per-specification-concept, and a wholesale rename makes every row a
mapping exercise for every future reader while making cross-language examples inapplicable. (b) is rejected because the distinction is
semantic rather than stylistic — a counter accepts only non-negative increments while a histogram accepts a value — and collapsing the verbs
hides that constraint at exactly the call site where a caller would benefit from seeing it. (c) is rejected because it enlarges the stable-by-
default surface of a module that is explicitly the specification-shaped one, and `api-ext` exists for precisely this content [A6|DECIDED].

**Consequences.** Because names follow the specification, each compliance row maps to a declaration by name and the sheet `M-35` produces is
checkable rather than interpretive. Because deviations must be recorded per row, a language collision cannot be resolved silently. Because sugar
is excluded from `api`, the dump delta for `api` stays proportional to the specification surface, which keeps G3-01 satisfiable [A6|DERIVED].

**Evidence.** `CONTRIBUTING.md:L52` requires `api` to stay close to the specification and L53 directs sugar to `api-ext` [A6|OBSERVED].
`api-ext/src/commonMain/kotlin/io/opentelemetry/kotlin/OpenTelemetryExt.kt:L33-L35` shows the existing convenience accessor pattern living in
`api-ext` rather than `api` [A3|OBSERVED]. `ObservableLongMeasurement.kt:L19` shows the existing `record` signature whose shape the new
instruments follow [A3|OBSERVED]. The instrument and operation names are the compliance rows at `spec-compliance-matrix.md:L113-L131`
[A4|OBSERVED].

## ADR-17 — Hostile extension points and their execution bounds

**Context.** The specified pipeline invites user code into three places and only three: an asynchronous instrument's callback, a view's
attribute-key predicate, and an exemplar filter. The repository's doctrine on this is unambiguous — telemetry must never destabilise the host,
nothing may escape a callback the SDK invokes, and all user-supplied code is to be assumed hostile — but doctrine alone does not say what
happens when hostile code blocks forever, recurses into collection, or returns a value that violates an invariant. ADR-08 bounds the callback's
lifecycle; the execution bound itself is a separate question that applies identically to all three extension points, which is why it is one
record rather than three asides.

**Status.** Accepted. It is a new record; ADR-08 and ADR-11 each hold part of the lifecycle, neither holds the execution bound.

**Decision.** Every user-supplied callable the SDK invokes MUST execute inside a containment boundary that catches every throwable, routes it
to the error handler at a bounded cadence, and treats the invocation as having produced no measurement. Each invocation MUST carry a timeout
derived from the reader's collection interval rather than a fresh constant, and a callable that exceeds it MUST be abandoned for that cycle
while remaining registered. Re-entrancy MUST be denied: a callable that triggers collection on the same reader MUST observe that collection is
already in progress and return without recursing. A callable that returns a value violating an invariant — a negative counter increment, a
non-finite double, a null attribute key — MUST have that value discarded with one diagnostic rather than propagated. No containment boundary MAY
swallow silently: every discard MUST be observable through the error handler. Unit `M-17` adds the callback boundary, unit `M-12` adds the view-predicate boundary and unit `M-14` adds the exemplar-filter boundary, all gated by G3-10 [A3|DECIDED].

**Alternatives.** (a) Let exceptions propagate to the caller of `forceFlush`, since that caller asked for collection. (b) Deregister a callback
that throws, on the reasoning that it is broken. (c) Use a fixed constant timeout per callback invocation. [A5|PROPOSED]

**Rejected because.** (a) is rejected because the caller of a flush is usually application shutdown code, so propagating turns a telemetry
defect into a host crash at the least recoverable moment. (b) is rejected because a transient failure would permanently and silently stop a
metric that the application still believes it is publishing, which is silent data loss caused by the containment itself. (c) is rejected
because a constant unrelated to the collection interval either exceeds the cycle it belongs to — so cycles overlap — or truncates callbacks a
longer interval would have tolerated [A3|DECIDED].

**Consequences.** Because containment catches throwables at these boundaries, the static-analysis exception rules that would otherwise forbid
it are already disabled in the repository's configuration, so this design does not require a configuration change. Because a slow callable is
abandoned rather than awaited, Section 6 lists "asynchronous callback times out and its series goes missing" as its own loss mode with an
oracle. Because re-entrancy is denied rather than reordered, a callback that triggers collection is a no-op rather than a deadlock
[A2|DERIVED].

**Evidence.** `AGENTS.md:L31` states that telemetry must never destabilize the host, L34-L35 that nothing may escape a public API method, an
SDK-invoked callback or a coroutine, L36 that invalid input degrades to a safe default, L38 that all user-supplied code is assumed hostile, and
L39 that errors route to the error handler rather than being swallowed [A6|OBSERVED]. `config/detekt/detekt.yml:L26-L31` deliberately disables
`InstanceOfCheckForException`, `SwallowedException` and `TooGenericExceptionCaught`, which is the configuration evidence that catching
throwables at these boundaries is the intended pattern [A2|OBSERVED].
`sdk-common/src/commonMain/kotlin/io/opentelemetry/kotlin/error/GuardedSdkErrorHandler.kt` supplies the guarded routing this record reuses
[A3|OBSERVED]. `api/src/commonMain/kotlin/io/opentelemetry/kotlin/metrics/AsynchronousInstrument.kt:L19` marks the registration member as still
to be added [A3|OBSERVED].

## ADR-18 — Resource bounds and input validation

**Context.** Metrics is the first signal in this SDK whose memory footprint is a function of what the application does at runtime rather than
of what it configured at start-up: one series per distinct attribute set, retained across collection cycles, with exemplar reservoirs attached.
Every recording allocates a fresh receiver lambda, so a keying mistake or an unbounded user attribute produces one series per call. The
specification's answer is a cardinality limit with an overflow series, and the repository's answer to bad input generally is that it degrades to
a safe default rather than throwing. Neither says what the limits are, where they are enforced, or what a caller observes when one binds — and
an unbounded series map inside a library embedded in someone else's application is the failure mode with the worst consequences of any in this
plan.

**Status.** Accepted. It is a new record; ADR-07 chooses aggregations and ADR-05 chooses keys, and the bound that contains both belongs with
neither.

**Decision.** Every unbounded collection in the pipeline MUST carry an explicit limit with a defined behaviour at the limit: series per metric
stream MUST be bounded by a configurable cardinality limit with a single overflow series carrying a distinguished attribute set; exemplar
reservoirs MUST be fixed-size; the instrument registry MUST be bounded per meter; and the export payload MUST be bounded by point count rather
than by byte length so the bound is meaningful before serialisation. Instrument descriptors MUST be validated at creation — name non-empty and
within the specification's length bound, unit and description within theirs — and an invalid descriptor MUST degrade to a sanitised value with
one diagnostic rather than throwing, because instrument creation happens on the application's own start-up path. Attribute values MUST be
validated at materialisation: a non-finite double MUST be discarded rather than aggregated, and a null or empty key MUST be dropped from the
key rather than making the whole measurement invalid. Reaching a limit MUST be observable through the error handler at a bounded cadence and
MUST NOT be silent. Unit `M-13` adds the cardinality bound and unit `M-07` adds descriptor and attribute validation, both gated by G3-10
[A3|DECIDED].

**Alternatives.** (a) Leave cardinality unbounded and document the risk, as the simplest faithful reading of the API. (b) Throw on an invalid
descriptor so the developer finds it immediately. (c) Bound the export payload by serialised byte length, which is what actually matters to a
collector. [A5|PROPOSED]

**Rejected because.** (a) is rejected because the failure is unbounded memory growth inside a host application, which the error doctrine treats
as the one outcome that must never occur, and because the specification defines a limit precisely so that implementations do not have to choose.
(b) is rejected because instrument creation runs during host start-up, and the doctrine confines fast failure to initialization
misconfiguration of the SDK itself rather than to a malformed instrument name; a sanitised value plus a diagnostic keeps the host running and
still tells the developer. (c) is rejected because a byte bound can only be evaluated after marshalling, so it either forces serialisation
before the decision or is enforced by discarding an already-built payload — and the point count is available before either [A3|DECIDED].

**Consequences.** Because limits are configurable rather than fixed, `M-15` exposes the bound `M-13` defines, which is why the dependency graph blocks `M-15` on `M-13` rather than the reverse. Because validation degrades rather than throws, Section 6 lists "measurement silently sanitised"
as a loss mode with an oracle, so degradation cannot be mistaken for correctness. Because the payload bound is expressed in points, the
same bound applies identically across all four platforms, where a byte bound would vary with the serialiser [A3|DERIVED].

**Evidence.** `AGENTS.md:L31,L36-L37` supply the host-safety and degrade-to-default rules and confine fast failure to initialization
misconfiguration [A6|OBSERVED]. `api/src/commonMain/kotlin/io/opentelemetry/kotlin/attributes/AttributesMutator.kt:L11-L72` shows the ten
setters whose values this record validates, with no validation of its own [A3|OBSERVED].
`sdk-common/src/commonMain/kotlin/io/opentelemetry/kotlin/config/Validation.kt` is the existing validation seam this record reuses rather than
duplicates [A3|OBSERVED]. `config-schema/src/commonMain/kotlin/io/opentelemetry/kotlin/config/schema/model/CardinalityLimits.kt` already names
the configuration surface for the bound [A3|OBSERVED]. The cardinality-limit and overflow-series semantics are the compliance rows at
`spec-compliance-matrix.md:L166-L170` [A4|OBSERVED].

## 3.1 Where every mandated technical area is addressed

Twenty technical areas must be addressed by this plan. The map below places each of them, and it carries 28 rows because eight areas resolve to
more than one record and are listed once per record rather than collapsed. An area found unnecessary is **stated** as unnecessary with
reasoning; none is silently dropped [A5|DECIDED].

| # | Technical area | Where it is addressed | Disposition |
| --- | --- | --- | --- |
| 1 | Instrument API surface and instrument kinds | ADR-02, ADR-16 | Specified; `M-03`, `M-04`, `M-05` |
| 2 | Instrument base interface and shared properties | ADR-02, ADR-04 | Specified; the maintainer's second open question is answered by keeping the existing concrete base rather than introducing a generic one |
| 3 | Meter and meter-provider surface | ADR-02, ADR-03 | Specified; `M-16` grows the existing shell and real provider |
| 4 | SDK configuration and the already-public DSL | ADR-03, ADR-14 | Specified; `M-15`, `M-22` |
| 5 | Metric and instrument identity | ADR-04 | Specified; `M-07` |
| 6 | Attribute materialisation and series keys | ADR-05 | Specified; `M-07`; the dominant correctness risk Section 1 names |
| 7 | Multiplatform concurrency and contended accumulation | ADR-06 | Specified; `M-08`; the stripe count stays open as ranked question 4 |
| 8 | The six aggregations | ADR-07 | Specified; `M-09` through `M-11`; summary and quantile ruled **unnecessary** in Section 2.7 because the specification defines six aggregations and summary survives only as a legacy data type |
| 9 | Temporality and per-reader isolation | ADR-07, ADR-09 | Specified; `M-09`, `M-24` |
| 10 | Cardinality limits and the overflow series | ADR-07, ADR-18 | Specified; `M-13` |
| 11 | Exemplars, reservoirs and filters | ADR-07 | Specified; `M-14` |
| 12 | Views, instrument selection and attribute filtering | ADR-05, ADR-07 | Specified; `M-12`; the reference's filtered-attributes optimisation is a redesign here rather than a port |
| 13 | Asynchronous instruments and callback lifecycle | ADR-08, ADR-17 | Specified; `M-04`, `M-17`; after-creation registration deliberately deferred as ranked question 6 |
| 14 | Reader pipeline and collection | ADR-09 | Specified; `M-18`, `M-28`, `M-29`; the eight-file push-processor shape is deliberately **not** replicated |
| 15 | Exporters, wire format and transport | ADR-10, ADR-14 | Specified; `M-19` through `M-21`, `M-30`, `M-31`; protocol message types already generate and are not re-authored |
| 16 | Lifecycle, shutdown, flush and error containment | ADR-11, ADR-17 | Specified; `M-23`, `M-34` |
| 17 | Correctness, differential and cross-platform evidence | ADR-12 | Specified; `M-24`, `M-25`, `M-32`, `M-33`; reach limit stated with both bounds in Section 6 |
| 18 | Binary compatibility and dump authority | ADR-13, ADR-15 | Specified; `M-02`, `M-26`; the residual Android-dump question is ranked first in Section 8 |
| 19 | Configuration precedence across five inputs | ADR-14 | Specified; `M-22`; a new record because no prior signal had five inputs |
| 20 | Upstream divergence and rebase posture | ADR-15 | Specified; applies to every unit rather than to one |
| 21 | Naming fidelity and the compliance sheet's checkability | ADR-16 | Specified; `M-35` |
| 22 | Hostile extension points and execution bounds | ADR-17 | Specified; `M-12`, `M-14`, `M-17` |
| 23 | Resource bounds and input validation | ADR-18 | Specified; `M-07`, `M-13` |
| 24 | Debug and source-attribution machinery | Section 2.7 | **Unnecessary, because** the reference's four-file subpackage attributes duplicate-registration warnings to source locations by capturing a stack trace at registration, which has no portable multiplatform form; the same condition is surfaced by the identity conflict `M-07` detects and the one sanitised diagnostic ADR-11 permits |
| 25 | Prometheus, console and pull exporters | Section 2.7 | **Unnecessary here, because** they are declared non-goals and appear only as generated model names with no implementation anywhere in the tree |
| 26 | OTLP over gRPC transport | ADR-10 | **Unnecessary, because** protocol code generation runs with service roles disabled, so no service stub exists to call; the HTTP path carries the signal |
| 27 | Metrics benchmarks | ADR-12 | **Optional, not required**, because the declared end state does not include them; the three benchmark modules are nonetheless compiled by `./gradlew build :benchmark-android:assemble :benchmark-android:compileReleaseAndroidTestKotlin :benchmark-jvm:assemble :examples:example-app-android:assembleRelease koverXmlReport -x :gradle-integration-test:test --stacktrace`, so they act as an unintended blast-radius detector |
| 28 | Self-observability metrics | Section 2.7 | **Excluded**, because it is a declared non-goal and is tracked upstream as the metrics milestone's tenth member rather than here |

## 3.2 Which unit implements each record, and which gate verifies it

A record that no unit implements is an opinion, and a unit that implements no record is unmotivated. This rollup closes both directions, and it is
the join Section 4's rows and Section 5's ordering are derived through. Every identifier here is defined elsewhere in this document: records in
Section 3, units in Section 4, gates in Section 4.1, and risks in Section 7 [A5|DECIDED].

| Record | Subject in one line | Implemented by | Verified by | Principal risk |
| --- | --- | --- | --- | --- |
| ADR-01 | Module and ownership layout — no new Gradle project | `M-03`, `M-06`, `M-09`, `M-16`, `M-18`, `M-19`, `M-21` | G3-06, G3-09 | R-12 |
| ADR-02 | Public meter and instrument surface — concrete kinds, builder entry points | `M-03`, `M-04`, `M-05` | G3-08, G3-09 | R-17, R-20 |
| ADR-03 | SDK configuration shape — grown holder, no wide constructor | `M-15`, `M-22` | G3-09, G3-10 | R-17, R-26 |
| ADR-04 | Metric and instrument identity — descriptor equality and conflict reporting | `M-05` | G3-10 | R-34 |
| ADR-05 | Attribute materialisation and series keys — one immutable key at the recording call | `M-06`, `M-07` | G3-10 | R-01, R-23 |
| ADR-06 | Multiplatform concurrency and accumulation — expect/actual striped accumulator | `M-08`, `M-32` | G3-07, G3-10 | R-02 |
| ADR-07 | Aggregations, temporality, cardinality and exemplars | `M-10`, `M-11`, `M-12`, `M-13`, `M-14`, `M-27`, `M-28` | G3-10 | R-03, R-05, R-21, R-22, R-25 |
| ADR-08 | Asynchronous instruments and callback lifecycle | `M-04`, `M-17` | G3-10 | R-04 |
| ADR-09 | MetricReader pipeline and the unverified-platform boundary | `M-16`, `M-18`, `M-29` | G3-07, G3-09 | R-09, R-15 |
| ADR-10 | Exporters and wire format — generated messages, no checked-in copies | `M-19`, `M-20`, `M-21`, `M-30` | G3-06, G3-10 | R-13, R-28 |
| ADR-11 | Lifecycle, shutdown and error containment — suspending sealed result | `M-23`, `M-31` | G3-10 | R-15, R-30 |
| ADR-12 | Correctness and reference strategy — differential confined to the JVM | `M-24`, `M-25`, `M-33` | G3-07, G3-10 | R-08 |
| ADR-13 | Binary-compatibility authority and the JVM-versus-Android dump ruling | `M-02` | G3-09 | R-06, R-07 |
| ADR-14 | Exporter and reader selection seam — fixed precedence, no reader by default | `M-22`, `M-29` | G3-10 | R-26 |
| ADR-15 | Upstream divergence and rebase posture — regenerate, never hand-merge | every unit, as a merge precondition | G3-06, G3-09 | R-33 |
| ADR-16 | Naming fidelity to the specification — deviations recorded per row | `M-03`, `M-26` | G3-08 | R-31 |
| ADR-17 | Hostile extension points with explicit execution bounds | `M-12`, `M-14`, `M-17` | G3-10 | R-04 |
| ADR-18 | Resource bounds with degrade-not-throw input validation | `M-05`, `M-13`, `M-30` | G3-10 | R-05, R-34 |

Two readings of this table are worth stating because they are the ones that shaped Section 5. ADR-07 is the widest record, reached by seven units,
which is why the aggregation stage is the longest run of the critical path and why its units are individually the smallest that still carry a
complete aggregation. And ADR-15 is the only record implemented by no single unit, because a rebase posture is a precondition every unit satisfies
rather than work any unit performs — a record whose consequence is procedural is still a record, and recording it that way is what keeps it out of
Section 8 as a false question [A5|DECIDED].

# 4. Work breakdown

The column set below is fixed and is reproduced exactly, because its shape is itself the compliance evidence for HC-06: there is no
duration column, no date column, no owner column and no effort column, and the presence of `BLOCKED-BY` is what replaces them. Sequencing is
expressed only as blocking [A5|DECIDED].

Rows are ordered so that no unit is blocked by a unit appearing later, which makes the table readable top to bottom without cross-referencing.
Foundational contracts precede implementations, source precedes tests, platform primitives precede aggregators, readers precede exporters, and
regenerated dumps, documentation and the compliance sheet come last. The `est. added lines (G3-01, ≤500)` column carries an integer per unit
rather than repeating the global ceiling, and every value is an estimate of hand-authored lines with regenerated dumps measured separately under
the rule in Section 4.1 [A5|DECIDED].

Across the 35 units the estimates **sum to 11,970 lines, with a mean of 342.0 and a maximum of 490** — deliberately below the 500-line ceiling
rather than at it, because a unit estimated at the ceiling has no room to absorb a review comment. The calibration is measured rather than
assumed: the two merged upstream metrics changes came in at **+88 lines across 4 files** and **+72 lines across 5 files**, while the one open
attempt at a single instrument end to end reached **+800 lines across 31 files**, which is 1.6 times the reviewable-diff guidance at
`AGENTS.md:L25`. The unit granularity here is therefore deliberately finer than "one instrument end to end", because that granularity is the
one granularity already measured as too large [A4|DERIVED].

No unit creates a Gradle project, a module or a directory outside the paths its scope column names, which is how HC-05 is honoured while ADR-01's layout decision is executed. No unit adds, updates or removes a dependency, a catalog entry, a wrapper property or a lockfile line, per HC-04. Wherever a row names the continuous-integration invocation it is printed in full rather than abbreviated, and the publish and consumer-integration commands are never combined, per HC-08. Disclosure of assisted authorship travels by commit trailer on each unit rather than by editing the pull-request template, per HC-12 [A5|DECIDED].

| ID | task | WORK_PACKAGE | BLOCKED-BY | acceptance criteria | est. added lines (G3-01, ≤500) | agent-executable | risk |
| --- | --- | --- | --- | --- | --- | --- | --- |
| M-01 | Baseline reconciliation and coverage-artifact correction — scope: `.github/workflows/ci-build.yml` upload path at L50; `codecov.yml`; no production source | WP-A | none | The coverage artifact the build actually produces and the artifact the workflow uploads MUST be the same file. Either the upload path becomes `build/reports/kover/report.xml` or a release-named report task is configured to produce `build/reports/kover/reportRelease.xml`. The 1.00 percentage-point project window and the 50% patch target MUST be left as they are, and the measured baseline of 4864 covered of 5153 measurable lines MUST be recorded as the floor Verified by: `ls build/reports/kover/` after `./gradlew build :benchmark-android:assemble :benchmark-android:compileReleaseAndroidTestKotlin :benchmark-jvm:assemble :examples:example-app-android:assembleRelease koverXmlReport -x :gradle-integration-test:test --stacktrace` lists the exact file the workflow uploads, and the uploaded report's root LINE counter equals the locally measured one Gates: G3-03, G3-04 | 20 | yes, after a human decides which side of the artifact-path mismatch changes | medium — R-24 |
| M-02 | JVM and Android dump baseline isolation — scope: `api/api/jvm/api.api`; `api/api/android/api.api`; the remaining nine Android dumps under `*/api/android/` | WP-A | none | The inherited disagreement MUST be isolated and reported before any metrics API delta is accepted, so that a later reviewer can attribute every dump line to a cause. The unit MUST state, per module, whether the Android dump omits declarations that rank-3 source contains, and MUST NOT bundle a metrics change Verified by: `./gradlew apiDump` followed by `git diff --stat -- '*/api/android/*.api'` produces a delta attributable entirely to pre-existing staleness, and `./gradlew apiCheck detekt --stacktrace` exits 0 afterwards Gates: G3-09, G3-11. The hand-authored figure is 0 because this unit's entire content is a regenerated dump baseline; the dump delta is measured and bounded separately at 700 lines under the rule in Section 4.1 | 0 | yes, after a human rules on ranked question 1 | high — R-06 |
| M-03 | Public synchronous instrument API — scope: `api/src/commonMain/kotlin/io/opentelemetry/kotlin/metrics/` adding `LongCounter`, `DoubleCounter`, `LongUpDownCounter`, `DoubleUpDownCounter`, `LongHistogram`, `DoubleHistogram`, `LongGauge`, `DoubleGauge` and their builder contracts, plus builder-returning members on `Meter.kt`; `api/api/jvm/api.api`; `api/api/android/api.api` | WP-B | M-01, M-02 | `Meter` MUST expose four synchronous builder entry points. Each instrument MUST be an interface with a typed record operation accepting a value and an optional attributes receiver lambda, MUST extend `SynchronousInstrument`, MUST carry `@ExperimentalApi`, and MUST NOT declare a default body. The KDoc contradiction at `Meter.kt:L9` MUST cease to be a contradiction because the promised members now exist Verified by: Both dumps regenerated by `./gradlew apiDump` show every new declaration; the annotation audit finds `@ExperimentalApi` on 100% of new public declarations; `./gradlew build :benchmark-android:assemble :benchmark-android:compileReleaseAndroidTestKotlin :benchmark-jvm:assemble :examples:example-app-android:assembleRelease koverXmlReport -x :gradle-integration-test:test --stacktrace` exits 0; the interface stays below the 31-function ceiling Gates: G3-01, G3-02, G3-04, G3-08, G3-09 | 320 | yes | medium — R-17 |
| M-04 | Public asynchronous instrument API — scope: `api/src/commonMain/kotlin/io/opentelemetry/kotlin/metrics/` adding `ObservableLongCounter`, `ObservableDoubleCounter`, `ObservableLongUpDownCounter`, `ObservableDoubleUpDownCounter`, `ObservableLongGauge`, `ObservableDoubleGauge` and their builder contracts, plus three builder-returning members on `Meter.kt`; both `api` dumps | WP-B | M-03 | `Meter` MUST expose three asynchronous builder entry points whose build operation accepts the observing callback, so registration happens at creation. Each instrument MUST extend `AsynchronousInstrument` and therefore MUST be closeable. The existing gap marker at `AsynchronousInstrument.kt:L19` MUST be left in place because no after-creation member is added in this wave Verified by: Both dumps regenerated; the callback parameter type is the existing observable measurement contract rather than a new one; `./gradlew build :benchmark-android:assemble :benchmark-android:compileReleaseAndroidTestKotlin :benchmark-jvm:assemble :examples:example-app-android:assembleRelease koverXmlReport -x :gradle-integration-test:test --stacktrace` exits 0 Gates: G3-01, G3-02, G3-04, G3-08, G3-09 | 300 | yes | medium — R-17 |
| M-05 | Noop, fake and compatibility parity — scope: `noop/src/commonMain/kotlin/io/opentelemetry/kotlin/metrics/NoopMeter.kt`; `test-fakes/src/commonMain/kotlin/io/opentelemetry/kotlin/metrics/FakeMeter.kt`; `compat/src/jvmAndAndroidMain/kotlin/io/opentelemetry/kotlin/metrics/MeterAdapter.kt` and the four sibling adapters; `noop`, `compat` and `test-fakes` dumps | WP-B | M-03, M-04 | All four `Meter` implementors MUST compile against the new surface. `NoopMeter` MUST return allocation-free constant instruments. `FakeMeter` MUST record every measurement for assertion. `MeterAdapter` MUST delegate to the reference meter it already holds, which retires the unused-property suppression at `MeterAdapter.kt:L8`. Behaviour MUST NOT change for any existing caller Verified by: `./gradlew build :benchmark-android:assemble :benchmark-android:compileReleaseAndroidTestKotlin :benchmark-jvm:assemble :examples:example-app-android:assembleRelease koverXmlReport -x :gradle-integration-test:test --stacktrace` exits 0, which also proves the three benchmark modules and the Android example still compile against the changed surface; the existing compat tests still pass Gates: G3-01, G3-02, G3-04, G3-08, G3-09 | 400 | yes | high — R-20 |
| M-06 | SDK metrics contracts and data model — scope: `sdk-api/src/commonMain/kotlin/io/opentelemetry/kotlin/metrics/` with `data/`, `model/`, `export/`, `view/`, `aggregation/` and `exemplar/` subpackages; `sdk-api/api/jvm/sdk-api.api` | WP-C | M-05 | The SDK MUST declare the reader contract, the exporter contract, the temporality enumeration, the instrument-kind enumeration, the aggregation and temporality selector contracts, the point and metric data types for sum, gauge and histogram, the exemplar data type, and the view and cardinality-limit contracts. Every lifecycle-bearing contract MUST extend the existing suspending closeable and MUST return the existing sealed result. Names MUST match the generated configuration vocabulary Verified by: The regenerated `sdk-api` dump shows the new package and no unrelated delta; no contract declares a default body except where an existing in-repository precedent is cited in review; `./gradlew build :benchmark-android:assemble :benchmark-android:compileReleaseAndroidTestKotlin :benchmark-jvm:assemble :examples:example-app-android:assembleRelease koverXmlReport -x :gradle-integration-test:test --stacktrace` exits 0 Gates: G3-01, G3-02, G3-04, G3-08, G3-09 | 460 | yes | medium — R-08 |
| M-07 | Attribute materialisation, series key and instrument identity — scope: `implementation/src/commonMain/kotlin/io/opentelemetry/kotlin/metrics/internal/` adding the recording attribute collector, the immutable series key and the canonical instrument descriptor; `implementation/src/commonTest/kotlin/io/opentelemetry/kotlin/metrics/` | WP-D | M-06 | A single internal collector MUST implement `AttributesMutator`, MUST accumulate typed entries, and MUST produce an immutable key whose equality and hash code are order-independent and type-preserving. The canonical descriptor MUST cover scope, case-insensitively compared name, unit, description, instrument kind, numeric kind and advice, and MUST exclude the resource. Invalid input MUST degrade using constants Verified by: Property tests assert that permuting setter order yields equal keys, that a long, a double and a string of equal text yield distinct entries, and that equal keys share one map slot; a hash-collision test asserts full-equality fallback; `./gradlew build :benchmark-android:assemble :benchmark-android:compileReleaseAndroidTestKotlin :benchmark-jvm:assemble :examples:example-app-android:assembleRelease koverXmlReport -x :gradle-integration-test:test --stacktrace` exits 0 Gates: G3-01, G3-02, G3-03, G3-04, G3-10 | 380 | yes | high — R-01 |
| M-08 | Platform accumulation primitives — scope: `platform-implementations/src/commonMain/kotlin/io/opentelemetry/kotlin/LongAccumulator.kt` and `DoubleAccumulator.kt`; actuals `LongAccumulator.jvm.kt`, `LongAccumulator.js.kt`, `LongAccumulator.apple.kt` and the double equivalents; `platform-implementations/src/commonTest/kotlin/io/opentelemetry/kotlin/`; `platform-implementations` dumps | WP-D | M-06 | Each accumulator MUST expose an add operation and an atomic read-and-reset operation, MUST stripe across a fixed bounded cell count, and MUST NOT size itself from the processor count. The JVM and Android actual MUST delegate to the platform accumulator; the JavaScript actual MUST be a single cell; the Apple actual MUST use the module's existing lock. New actual files MUST use the platform-suffixed name form Verified by: Shared behaviour tests in `commonTest` alongside the existing primitive tests assert add-and-reset atomicity and that a reset returns each increment exactly once; a JVM concurrency stress test asserts that the sum of concurrent increments equals the accumulated total with zero lost updates; `./gradlew build :benchmark-android:assemble :benchmark-android:compileReleaseAndroidTestKotlin :benchmark-jvm:assemble :examples:example-app-android:assembleRelease koverXmlReport -x :gradle-integration-test:test --stacktrace` exits 0 Gates: G3-01, G3-02, G3-04, G3-07, G3-10 | 340 | yes for the JVM, Android and JavaScript actuals; the Apple actuals compile here but execute nowhere | high — R-02 |
| M-09 | Sum and last-value aggregation with per-reader storage — scope: `implementation/src/commonMain/kotlin/io/opentelemetry/kotlin/metrics/internal/state/` and `internal/aggregator/`; `implementation/src/commonTest/kotlin/io/opentelemetry/kotlin/metrics/` | WP-E | M-07, M-08 | Storage MUST be keyed by registered reader and then by series key. Sum aggregation MUST support monotonic and non-monotonic instruments in both long and double form. Last-value aggregation MUST retain the most recent measurement per series. A delta collection MUST snapshot and reset atomically; a cumulative collection MUST snapshot without reset and MUST preserve a per-series start timestamp. One reader's collection MUST NOT alter another reader's state Verified by: A two-reader test with opposing temporality preferences asserts that each reader observes complete, independent intervals; a monotonicity property test asserts a cumulative sum never decreases; a lost-update stress test asserts the exported total equals the recorded total; `./gradlew build :benchmark-android:assemble :benchmark-android:compileReleaseAndroidTestKotlin :benchmark-jvm:assemble :examples:example-app-android:assembleRelease koverXmlReport -x :gradle-integration-test:test --stacktrace` exits 0 Gates: G3-01, G3-02, G3-03, G3-04, G3-10 | 470 | yes | high — R-03 |
| M-10 | Explicit-bucket histogram aggregation — scope: `implementation/src/commonMain/kotlin/io/opentelemetry/kotlin/metrics/internal/aggregator/`; `implementation/src/commonTest/kotlin/io/opentelemetry/kotlin/metrics/` | WP-E | M-09 | Buckets MUST be stated by upper boundary, exclusive of the lower bound and inclusive of the upper except at positive infinity, and a measurement MUST fall into the lowest bucket whose boundary is greater than or equal to the value. The advisory bucket-boundary parameter MUST be honoured. Count, sum, minimum and maximum MUST be maintained per series Verified by: A property test asserts that the sum of bucket counts equals the point count and that the recorded sum equals the arithmetic sum of inputs for every generated input list; a boundary test asserts inclusive-upper placement at each boundary and at positive infinity; `./gradlew build :benchmark-android:assemble :benchmark-android:compileReleaseAndroidTestKotlin :benchmark-jvm:assemble :examples:example-app-android:assembleRelease koverXmlReport -x :gradle-integration-test:test --stacktrace` exits 0 Gates: G3-01, G3-02, G3-03, G3-04, G3-10 | 420 | yes | medium — R-21 |
| M-11 | Base-2 exponential histogram aggregation — scope: `implementation/src/commonMain/kotlin/io/opentelemetry/kotlin/metrics/internal/aggregator/`; `implementation/src/commonTest/kotlin/io/opentelemetry/kotlin/metrics/` | WP-E | M-09 | Scale MUST adapt downward as the observed range widens, MUST be bounded by a configured maximum scale, and MUST honour a configured maximum bucket count per positive and negative range excluding the zero bucket. Under delta temporality the scale MUST reset with each snapshot so that precision recovers after an outlier cycle Verified by: A delta-cycle property test asserts that a single extreme outlier does not depress scale in any subsequent cycle; a bucket-index round-trip property test asserts that a value maps into a bucket whose boundaries contain it at every scale exercised; `./gradlew build :benchmark-android:assemble :benchmark-android:compileReleaseAndroidTestKotlin :benchmark-jvm:assemble :examples:example-app-android:assembleRelease koverXmlReport -x :gradle-integration-test:test --stacktrace` exits 0 Gates: G3-01, G3-02, G3-03, G3-04, G3-10 | 480 | yes | high — R-22 |
| M-12 | View registry and attribute filtering — scope: `implementation/src/commonMain/kotlin/io/opentelemetry/kotlin/metrics/internal/view/`; `implementation/src/commonTest/kotlin/io/opentelemetry/kotlin/metrics/` | WP-F | M-06, M-09 | Instrument selection MUST match on name including the match-all wildcard and on the other selection criteria the specification defines, and MUST support more than one view per instrument. A view MUST be able to rename a stream, replace its description, choose its aggregation, and restrict or exclude attribute keys. Attribute restriction MUST execute during materialisation so that two measurements a view collapses produce one series key, and the removed entries MUST remain available for exemplar retention Verified by: A test asserts that two measurements differing only in a filtered attribute occupy one series; a multi-view test asserts one instrument feeding two independently named streams; a wildcard test asserts match-all and pattern selection; `./gradlew build :benchmark-android:assemble :benchmark-android:compileReleaseAndroidTestKotlin :benchmark-jvm:assemble :examples:example-app-android:assembleRelease koverXmlReport -x :gradle-integration-test:test --stacktrace` exits 0 Gates: G3-01, G3-02, G3-03, G3-04, G3-10 | 450 | yes | medium — R-23 |
| M-13 | Cardinality limits and overflow series — scope: `implementation/src/commonMain/kotlin/io/opentelemetry/kotlin/metrics/internal/state/`; `implementation/src/commonTest/kotlin/io/opentelemetry/kotlin/metrics/` | WP-F | M-12 | A per-stream limit MUST be configurable at reader level and per metric through a view. On exceeding the limit the SDK MUST fold further series into one reserved overflow series whose attributes mark it as such, MUST continue to accumulate into that series, and MUST NOT discard a measurement. Exceeding the limit MUST emit one bounded diagnostic per stream per collection cycle Verified by: A test drives more distinct series than the limit and asserts that the exported total equals the recorded total with the excess present in the overflow series and nothing dropped; a diagnostic-cadence test asserts at most one report per stream per cycle; `./gradlew build :benchmark-android:assemble :benchmark-android:compileReleaseAndroidTestKotlin :benchmark-jvm:assemble :examples:example-app-android:assembleRelease koverXmlReport -x :gradle-integration-test:test --stacktrace` exits 0 Gates: G3-01, G3-02, G3-03, G3-04, G3-10 | 260 | yes | high — R-05 |
| M-14 | Exemplar reservoirs and filters — scope: `implementation/src/commonMain/kotlin/io/opentelemetry/kotlin/metrics/internal/exemplar/`; `implementation/src/commonTest/kotlin/io/opentelemetry/kotlin/metrics/` | WP-F | M-09, M-12 | The SDK MUST provide an always-on, an always-off and a trace-based filter, MUST default to a fixed-size reservoir per series, MUST use an aligned-bucket reservoir for an explicit-bucket histogram with more than one bucket, MUST capture the active trace and span identifiers and the measurement timestamp, and MUST retain the attributes that view filtering removed. Reservoir memory MUST be bounded independently of measurement volume Verified by: A test asserts that reservoir occupancy never exceeds its configured size under a high measurement count; a filter test asserts no exemplar is retained under the always-off filter and that trace-based retention follows the sampling flag; a retention test asserts filtered attributes appear on the exemplar and not on the point; `./gradlew build :benchmark-android:assemble :benchmark-android:compileReleaseAndroidTestKotlin :benchmark-jvm:assemble :examples:example-app-android:assembleRelease koverXmlReport -x :gradle-integration-test:test --stacktrace` exits 0 Gates: G3-01, G3-02, G3-03, G3-04, G3-10 | 400 | yes | medium — R-25 |
| M-15 | SDK configuration DSL and metrics configuration growth — scope: `sdk-api/src/commonMain/kotlin/io/opentelemetry/kotlin/init/MeterProviderConfigDsl.kt`; `implementation/src/commonMain/kotlin/io/opentelemetry/kotlin/init/config/MetricsConfig.kt`; `implementation/src/commonMain/kotlin/io/opentelemetry/kotlin/init/MeterProviderConfigImpl.kt`; `sdk-api/api/jvm/sdk-api.api` | WP-G | M-13, M-14 | The already-public `meterProvider` block MUST gain members for readers, views, default temporality, default aggregation, exemplar policy and cardinality limits, mirroring the member shape its logging counterpart uses. `MetricsConfig` MUST grow as immutable nested value types and MUST NOT become a flat wide constructor. The acceptance criteria MUST state explicitly that an empty `meterProvider` block changes from configuring nothing to configuring a real provider, because that is a behavioural change to already-released surface Verified by: The regenerated `sdk-api` dump shows the new DSL members; no constructor exceeds the 30-parameter ceiling and no type exceeds the 31-function ceiling, evidenced by `./gradlew apiCheck detekt --stacktrace` exiting 0; a configuration test asserts that each block value reaches the provider; `./gradlew build :benchmark-android:assemble :benchmark-android:compileReleaseAndroidTestKotlin :benchmark-jvm:assemble :examples:example-app-android:assembleRelease koverXmlReport -x :gradle-integration-test:test --stacktrace` exits 0 Gates: G3-01, G3-02, G3-04, G3-08, G3-09 | 430 | yes | low — R-26 |
| M-16 | Meter and instrument implementation — scope: `implementation/src/commonMain/kotlin/io/opentelemetry/kotlin/metrics/MeterImpl.kt`; `implementation/src/commonMain/kotlin/io/opentelemetry/kotlin/metrics/internal/`; `implementation/src/commonMain/kotlin/io/opentelemetry/kotlin/metrics/MeterProviderImpl.kt`; `implementation/src/commonTest/kotlin/io/opentelemetry/kotlin/metrics/` | WP-G | M-07, M-09, M-15 | `MeterImpl` MUST implement every synchronous builder, MUST resolve each build through the canonical descriptor, MUST return the same instrument for an identical identity, and MUST emit one deterministic diagnostic and first-seen stream identity for a conflicting duplicate without raising. `MeterProviderImpl` MUST gain reader registration and MUST keep its existing scope cache, empty-name diagnostic and post-shutdown no-op behaviour unchanged Verified by: Tests assert identity reuse, conflicting-duplicate diagnostics with no throw, invalid-name degradation to a working instrument, and that a meter obtained after shutdown remains the no-op meter; the existing provider test still passes; `./gradlew build :benchmark-android:assemble :benchmark-android:compileReleaseAndroidTestKotlin :benchmark-jvm:assemble :examples:example-app-android:assembleRelease koverXmlReport -x :gradle-integration-test:test --stacktrace` exits 0 Gates: G3-01, G3-02, G3-03, G3-04, G3-10 | 490 | yes | high — R-16 |
| M-17 | Asynchronous callback registry and collection integration — scope: `implementation/src/commonMain/kotlin/io/opentelemetry/kotlin/metrics/internal/state/`; `implementation/src/commonMain/kotlin/io/opentelemetry/kotlin/metrics/MeterImpl.kt`; `implementation/src/commonTest/kotlin/io/opentelemetry/kotlin/metrics/` | WP-G | M-16 | Callbacks MUST be registered at creation, invoked only during collection, and contained absolutely: every throwable MUST be caught at the invocation boundary, routed to the error handler at most once per callback per cycle, and MUST NOT propagate. A failing callback MUST contribute no partial series and MUST NOT affect other series. Re-entrant collection MUST be detected and MUST NOT duplicate or corrupt the snapshot. Close MUST unregister idempotently and MUST succeed after provider shutdown Verified by: A hostile-callback test throws from a callback and asserts collection completes, the error handler receives exactly one report, and other series are intact; a re-entrancy test invokes collection from within a callback and asserts no duplicate points; an idempotent-close test closes twice and after shutdown; `./gradlew build :benchmark-android:assemble :benchmark-android:compileReleaseAndroidTestKotlin :benchmark-jvm:assemble :examples:example-app-android:assembleRelease koverXmlReport -x :gradle-integration-test:test --stacktrace` exits 0 Gates: G3-01, G3-02, G3-03, G3-04, G3-10 | 420 | yes | high — R-04 |
| M-18 | MetricReader core and periodic reader — scope: `exporters-core/src/commonMain/kotlin/io/opentelemetry/kotlin/metrics/export/`; `exporters-core/api/jvm/exporters-core.api` and the Android sibling; `exporters-core/src/commonTest/kotlin/` | WP-G | M-16 | A manually driven reader and a periodic reader MUST be provided, with no simple or composite processor and no eight-file processor shape. Collection MUST be suspending, MUST be bounded by a timeout, MUST apply the reader's temporality, and MUST return the points produced. The periodic reader MUST schedule on a scope tied to its own lifetime, MUST cancel that scope on shutdown, MUST skip rather than queue an overlapping cycle, MUST perform at most one final collection on shutdown and then refuse further collection while returning success, and MUST be idempotent Verified by: A virtual-time test asserts exactly one collection per interval and a skipped cycle when the predecessor is still running; a double-shutdown test asserts idempotence and a single final collection; a timeout test asserts a slow collection yields a bounded failure rather than a hang; the regenerated dumps show reader types and no processor types; `./gradlew build :benchmark-android:assemble :benchmark-android:compileReleaseAndroidTestKotlin :benchmark-jvm:assemble :examples:example-app-android:assembleRelease koverXmlReport -x :gradle-integration-test:test --stacktrace` exits 0 Gates: G3-01, G3-02, G3-03, G3-04, G3-08, G3-09, G3-10 | 460 | yes | high — R-03 |
| M-19 | In-memory metric exporter — scope: `exporters-in-memory/src/commonMain/kotlin/io/opentelemetry/kotlin/metrics/export/InMemoryMetricExporter.kt`, `InMemoryMetricExporterApi.kt` and `InMemoryMetricExporterImpl.kt`; `exporters-in-memory` dumps; `exporters-in-memory/src/commonTest/kotlin/` | WP-H | M-18 | The three-file shape the other signals use MUST be followed exactly. The exporter MUST retain exported batches for assertion, MUST expose a reset, MUST implement the suspending closeable contract, and MUST return the sealed result. It MUST NOT be reachable from production configuration defaults Verified by: A test drives a manual reader into the exporter and asserts the exported batch equals the recorded measurements; a reset test asserts retained state clears; the regenerated dumps show exactly the three new public entry points; `./gradlew build :benchmark-android:assemble :benchmark-android:compileReleaseAndroidTestKotlin :benchmark-jvm:assemble :examples:example-app-android:assembleRelease koverXmlReport -x :gradle-integration-test:test --stacktrace` exits 0 Gates: G3-01, G3-02, G3-04, G3-08, G3-09 | 240 | yes | low — R-27 |
| M-20 | Protobuf metrics conversion — scope: `exporters-protobuf/src/commonMain/kotlin/io/opentelemetry/kotlin/metrics/export/ExportMetricsResponseDeserializer.kt`, `ExportMetricsServiceRequestCreator.kt` and `MetricDataProtobufConversion.kt`; `exporters-protobuf/src/commonTest/kotlin/` and its golden fixtures | WP-H | M-06, M-19 | Conversion MUST map every produced point type onto the already-generated protocol messages, MUST reuse the existing shared attribute and common conversion helpers, MUST reuse the existing partial-success representation, and MUST NOT check in or regenerate any protocol definition. Resource and scope MUST be emitted once per batch rather than per point Verified by: Golden-file tests compare serialised output byte-for-byte against fixtures in the existing fixture directory for a sum, a gauge, an explicit-bucket histogram, an exponential histogram and an exemplar-bearing point; a round-trip test deserialises a partial-success response; `./gradlew build :benchmark-android:assemble :benchmark-android:compileReleaseAndroidTestKotlin :benchmark-jvm:assemble :examples:example-app-android:assembleRelease koverXmlReport -x :gradle-integration-test:test --stacktrace` exits 0 Gates: G3-01, G3-02, G3-03, G3-04, G3-10 | 470 | yes | medium — R-28 |
| M-21 | OTLP metric exporter, endpoint entry and collector-fake route — scope: `exporters-otlp/src/commonMain/kotlin/io/opentelemetry/kotlin/metrics/export/OtlpHttpMetricExporter.kt` and `OtlpMetricExporterApi.kt`; `exporters-otlp/src/commonMain/kotlin/io/opentelemetry/kotlin/export/OtlpEndpoint.kt`; `smoke-test/src/commonTest/kotlin/io/opentelemetry/kotlin/smoketest/FakeOtlpServer.kt`; `exporters-otlp` dumps | WP-H | M-20 | The exporter MUST instantiate the existing generic retry and transport seam with a metrics export action and MUST NOT add retry, backoff or jitter code. A metrics entry MUST be added to the internal endpoint enumeration, which changes no public dump because the enumeration is internal. The collector fake MUST gain the matching route in the same unit, because it raises an error on an unrecognised path. gRPC MUST NOT be added Verified by: A smoke test configures the provider with a periodic reader and the OTLP exporter against the fake collector and asserts the metrics payload arrives on the metrics path; a retry test asserts the existing seam's terminal handling is reused unmodified; the regenerated `exporters-otlp` dumps show the two new public entry points and no endpoint enumeration entry; `./gradlew build :benchmark-android:assemble :benchmark-android:compileReleaseAndroidTestKotlin :benchmark-jvm:assemble :examples:example-app-android:assembleRelease koverXmlReport -x :gradle-integration-test:test --stacktrace` exits 0 Gates: G3-01, G3-02, G3-04, G3-08, G3-09 | 300 | yes | medium — R-29 |
| M-22 | Environment-variable and behaviour-model integration — scope: `config-envar/src/commonMain/kotlin/io/opentelemetry/kotlin/config/envar/EnvVarConstants.kt` and its name model; `implementation/src/commonMain/kotlin/io/opentelemetry/kotlin/config/envar/metrics/`; `behavior/src/commonMain/kotlin/io/opentelemetry/kotlin/behavior/OpenTelemetryBehavior.kt` and a new meter-provider behaviour; `config-envar` and `behavior` dumps | WP-I | M-15, M-21 | The metrics export interval, export timeout, exemplar filter, temporality preference and exporter selection MUST be recognised, following the existing per-signal processor pattern. Because these are the first non-limit environment variables in the repository, the unit MUST state the policy decision explicitly rather than extend the limits-only structure implicitly. A meter-provider slot MUST be added to the layered behaviour model and MUST merge with the same precedence rule the tracer slot uses Verified by: Tests assert that each variable maps to the configured value, that an unparseable value degrades to the default with one diagnostic, and that a higher layer overrides a lower one; the regenerated dumps show the new behaviour type; `./gradlew build :benchmark-android:assemble :benchmark-android:compileReleaseAndroidTestKotlin :benchmark-jvm:assemble :examples:example-app-android:assembleRelease koverXmlReport -x :gradle-integration-test:test --stacktrace` exits 0 Gates: G3-01, G3-02, G3-03, G3-04, G3-08, G3-09 | 380 | yes | medium — R-26 |
| M-23 | Lifecycle and shutdown compliance wiring — scope: `implementation/src/commonMain/kotlin/io/opentelemetry/kotlin/metrics/MeterProviderImpl.kt`; `smoke-test/src/commonTest/kotlin/io/opentelemetry/kotlin/smoketest/`; `implementation/src/commonTest/kotlin/io/opentelemetry/kotlin/metrics/` | WP-I | M-17, M-21 | The provider MUST compose its registered readers into the existing composite closeable so that flush and shutdown fan out with all-success semantics, MUST keep the existing batch-default timeouts, and MUST leave the facade's single aggregate lifecycle bound and combination semantics untouched. Flush and shutdown MUST be idempotent, and a failing exporter MUST surface as a failure result rather than an exception Verified by: A shutdown-compliance test asserts a final collection occurs once, that a second shutdown returns success without repeating work, that a post-shutdown record is a no-op, and that a failing exporter yields a failure result with nothing thrown; a timeout test asserts the aggregate bound is not exceeded; `./gradlew build :benchmark-android:assemble :benchmark-android:compileReleaseAndroidTestKotlin :benchmark-jvm:assemble :examples:example-app-android:assembleRelease koverXmlReport -x :gradle-integration-test:test --stacktrace` exits 0 Gates: G3-01, G3-02, G3-03, G3-04, G3-10 | 300 | yes | high — R-15 |
| M-24 | Differential harness meter hook and shared platform behaviour tests — scope: `integration-test/src/commonMain/kotlin/io/opentelemetry/kotlin/framework/OtelKotlinTestRule.kt` and its harness configuration; `implementation/src/commonTest/kotlin/io/opentelemetry/kotlin/integration/test/IntegrationTestHarness.kt`; `implementation/src/commonTest/resources/`; `platform-implementations/src/commonTest/kotlin/` | WP-J | M-19, M-23 | The abstract rule MUST gain a meter-provider configuration hook and metrics assertions alongside its existing tracer and logger hooks, and the Kotlin back end MUST supply it. Shared behaviour, property and golden tests MUST be placed in common test source sets so they compile and run for JVM, Android, JavaScript and, on a macOS host, the iOS simulator. Fixtures MUST load through the existing multiplatform loader and MUST be staged onto the simulator by the existing copy task Verified by: The shared tests execute on JVM, Android and JavaScript in this environment, and their Apple execution is positively asserted rather than inferred: a run on a macOS host must show `:implementation:iosSimulatorArm64Test` executed rather than skipped; `./gradlew build :benchmark-android:assemble :benchmark-android:compileReleaseAndroidTestKotlin :benchmark-jvm:assemble :examples:example-app-android:assembleRelease koverXmlReport -x :gradle-integration-test:test --stacktrace` exits 0 Gates: G3-01, G3-02, G3-03, G3-04, G3-07, G3-10 | 420 | yes for JVM and JavaScript; Apple execution requires a macOS host | medium — R-08 |
| M-25 | JVM differential metrics scenarios and golden fixtures — scope: `compat/src/jvmTest/kotlin/io/opentelemetry/kotlin/framework/OtelKotlinHarness.kt`; `compat/src/jvmTest/kotlin/io/opentelemetry/kotlin/metrics/`; `compat/src/jvmTest/resources/`; the compatibility metrics adapters | WP-J | M-24 | Identically named scenarios MUST run against both back ends over deliberately overlapping golden files, covering counter, up-down counter, histogram, gauge, observable instruments, delta and cumulative temporality, view renaming and attribute filtering, and cardinality overflow. The unit MUST state in its own documentation that this evidence reaches the JVM only, and MUST NOT assert agreement on any compliance row the reference implementation does not satisfy Verified by: Both back ends produce output matching the shared fixtures for every scenario; a deliberate divergence in one aggregation is shown to fail the comparison, proving the oracle discriminates; `./gradlew build :benchmark-android:assemble :benchmark-android:compileReleaseAndroidTestKotlin :benchmark-jvm:assemble :examples:example-app-android:assembleRelease koverXmlReport -x :gradle-integration-test:test --stacktrace` exits 0 Gates: G3-01, G3-02, G3-03, G3-04, G3-10 | 450 | yes | medium — R-08 |
| M-26 | Dumps, documentation and the compliance sheet — scope: Every touched module's `api/jvm/*.api` and `api/android/*.api`; `api/src/commonMain/kotlin/io/opentelemetry/kotlin/metrics/Meter.kt` KDoc; `api/README.md`; `AGENTS.md:L10`; `CONTRIBUTING.md:L14-L15`; `CHANGELOG.md`; a compliance sheet committed at the repository root | WP-K | M-05, M-10, M-11, M-13, M-14, M-17, M-18, M-19, M-20, M-21, M-22, M-25 | Every dump MUST be regenerated by the dump task rather than hand-edited, and each platform dump changed MUST be named. The `Meter` KDoc MUST be reconciled with the members that now exist. The component stability matrix MUST reflect the metrics signal's changed state without marking it stable. The nonexistent module name in the agent guidance and the outdated Android package levels in the contributor guidance MUST be corrected. A changelog entry MUST use the dated heading form. The compliance sheet MUST carry one row per compliance row with its evidence and MUST report the 81-row denominator, the 4 excluded rows and the 7 optional rows out of scope Verified by: `./gradlew apiCheck detekt --stacktrace` exits 0 with zero unexplained dump deltas; the annotation audit shows no accidental stable surface; `./gradlew build :benchmark-android:assemble :benchmark-android:compileReleaseAndroidTestKotlin :benchmark-jvm:assemble :examples:example-app-android:assembleRelease koverXmlReport -x :gradle-integration-test:test --stacktrace` exits 0; `./gradlew publishToMavenLocal -PsnapshotPublish=true -Psigning.skip=true --stacktrace` and then the separate `./gradlew :gradle-integration-test:test --stacktrace` both exit 0 Gates: G3-01, G3-02, G3-04, G3-06, G3-08, G3-09, G3-11 | 400 | yes | high — R-06 |
| M-27 | Drop and default aggregation selection by instrument kind — scope: `implementation/src/commonMain/kotlin/io/opentelemetry/kotlin/metrics/internal/aggregator/`; `implementation/src/commonTest/kotlin/io/opentelemetry/kotlin/metrics/` | WP-E | M-09, M-12 | The default aggregation MUST be selected from the instrument kind rather than configured per instrument, mapping counters and up-down counters to sum, gauges to last value, and histograms to explicit-bucket histogram; the drop aggregation MUST discard measurements without allocating a series or an exemplar reservoir, so that dropping is cheaper than aggregating rather than merely equivalent. A view selecting the drop aggregation MUST leave no residue observable through the in-memory exporter. Verified by: a table-driven test asserting the selected aggregation for each of the seven instrument kinds; an allocation-shape test asserting that a dropped measurement creates no storage entry; and a golden-file assertion that a dropped stream is absent from the payload rather than present and empty. Gates: G3-01, G3-02, G3-03, G3-04, G3-10 | 180 | yes | low — R-21 |
| M-28 | Delta and cumulative temporality isolation per reader — scope: `implementation/src/commonMain/kotlin/io/opentelemetry/kotlin/metrics/internal/state/`; `implementation/src/commonTest/kotlin/io/opentelemetry/kotlin/metrics/` | WP-E | M-09, M-18 | Two readers configured with different temporalities MUST observe independent state: a delta collection MUST reset only the storage owned by the collecting reader, and a cumulative reader MUST continue to observe the running total unaffected. Under delta temporality the base-2 exponential histogram's scale MUST reset each cycle, because a scale that only ever narrows never recovers precision after an outlier — a defect observed in another language's SDK and the reason this unit is separate from `M-11`. Verified by: a two-reader test asserting that a delta collection leaves the cumulative reader's totals intact; a scale-recovery test recording an outlier then a tight cluster and asserting the scale widens back; and a property test asserting that the sum of successive delta collections equals the cumulative value at the same instant. Gates: G3-01, G3-02, G3-03, G3-04, G3-10 | 360 | yes | high — R-03 |
| M-29 | Reader collection scheduling, interval and force-flush fan-out — scope: `exporters-core/src/commonMain/kotlin/io/opentelemetry/kotlin/metrics/export/`; `implementation/src/commonMain/kotlin/io/opentelemetry/kotlin/metrics/MeterProviderImpl.kt`; `exporters-core/src/commonTest/kotlin/` | WP-G | M-18, M-23 | The periodic reader MUST schedule collection on a virtual-time-testable dispatcher, MUST NOT overlap two collections on the same reader, and MUST skip rather than queue a cycle whose predecessor has not completed. Force-flush MUST fan out to every registered reader and every push exporter, MUST await all of them, and MUST report failure if any one fails while still attempting the rest. A reader registered after provider construction MUST participate in the next cycle. Verified by: virtual-time tests asserting exactly one collection per interval and a skipped rather than queued overlapping cycle; a fan-out test asserting that a single failing exporter yields an overall failure with the other exporters still invoked; and a shutdown-ordering test asserting that a flush in progress completes before shutdown returns. Gates: G3-01, G3-02, G3-03, G3-04, G3-10 | 330 | yes | high — R-15 |
| M-30 | Metric payload assembly, point bounding and partial-success handling — scope: `exporters-protobuf/src/commonMain/kotlin/io/opentelemetry/kotlin/metrics/export/`; `exporters-protobuf/src/commonTest/kotlin/` and its golden fixtures | WP-H | M-20, M-21 | The exporter MUST assemble one request per collection cycle with resource and scope attributes attached exactly once rather than per point, MUST bound the request by point count as ADR-18 requires, and MUST split rather than truncate a cycle that exceeds the bound. A partial-success response naming a rejected point count MUST be surfaced to the error handler rather than treated as success, and a client error MUST NOT be retried. Verified by: golden-file assertions over a request containing every point type; a bounding test asserting that an over-large cycle produces two requests whose union equals the input; and a response test asserting that partial success and client errors each route to the error handler exactly once. Gates: G3-01, G3-02, G3-03, G3-04, G3-10 | 290 | yes | high — R-30 |
| M-31 | Transport-seam repair so success is reported only after transport completes — scope: `exporters-otlp/src/commonMain/kotlin/io/opentelemetry/kotlin/export/TelemetryExporter.kt`; `exporters-otlp/src/commonTest/kotlin/`; regression assertions in the logging and tracing exporter tests | WP-H | M-30 | The generic seam MUST NOT report success before the transport call completes, MUST NOT swallow a terminal partial success or client error, MUST derive its force-flush bound from the caller's timeout rather than a constant, and MUST NOT cancel in-flight work on shutdown before its timeout elapses. Because the seam is shared, this unit MUST carry regression assertions for the logging and tracing exporters that already depend on it, and MUST NOT change their observable success semantics except where those semantics were the defect. Verified by: a mock-engine test asserting that the returned result follows rather than precedes the HTTP exchange; assertions that a partial-success and a 4xx response each yield failure; a shutdown test asserting in-flight completion within the timeout; and the pre-existing logging and tracing exporter suites passing unchanged. Gates: G3-01, G3-02, G3-03, G3-04, G3-10 | 240 | yes | high — R-30 |
| M-32 | Apple and JavaScript accumulation actuals with shared behaviour tests — scope: `platform-implementations/src/appleMain/kotlin/io/opentelemetry/kotlin/`; `platform-implementations/src/jsMain/kotlin/io/opentelemetry/kotlin/`; `platform-implementations/src/commonTest/kotlin/io/opentelemetry/kotlin/`; `platform-implementations/src/appleTest/`, `src/jsTest/` | WP-D | M-08 | The Apple and JavaScript actuals of the accumulation primitives MUST satisfy the same shared behaviour tests as the JVM actual, placed in the common test source set so that every target runs them rather than each target asserting its own weaker contract. The JavaScript actual MUST document that its single-threaded runtime makes striping unnecessary while still honouring the primitive's contract; the Apple actual MUST NOT rely on a lock whose contention profile is untested. No aggregated output MAY be emitted from a source set with no executed evidence, which is G3-07's condition rather than a label. Verified by: the shared behaviour suite executing on JVM and JavaScript with zero failures; both Apple targets compiling and producing klibs; and the emission gate observably closed for Apple until an executed report from a macOS host opens it. Gates: G3-01, G3-02, G3-04, G3-07, G3-10 | 380 | yes for compilation; Apple execution requires a macOS host | high — R-09 |
| M-33 | JavaScript and Android cross-platform evidence for the metrics pipeline — scope: `implementation/src/commonTest/kotlin/io/opentelemetry/kotlin/metrics/`; `implementation/src/jsTest/`, `src/androidUnitTest/`; `benchmark-android/src/androidTest/` | WP-J | M-24, M-32 | The pipeline's behaviour tests MUST execute on the JavaScript target as well as the JVM, and the Android variant MUST at minimum compile its instrumented sources, so that a platform difference in accumulation, ordering or number formatting surfaces as a test failure rather than as a field report. Any target that only compiles MUST be recorded as compile-verified rather than counted as executed. Verified by: the metrics behaviour suite passing on Node; `:benchmark-android:compileReleaseAndroidTestKotlin` exiting 0 within ./gradlew build :benchmark-android:assemble :benchmark-android:compileReleaseAndroidTestKotlin :benchmark-jvm:assemble :examples:example-app-android:assembleRelease koverXmlReport -x :gradle-integration-test:test --stacktrace; and an evidence table stating, per target, whether behaviour was executed or only compiled. Gates: G3-01, G3-02, G3-04, G3-07, G3-10 | 310 | yes for JavaScript and Android compilation; Android instrumented execution requires a device or emulator | medium — R-09 |
| M-34 | Smoke-test and shutdown-compliance coverage for the meter provider — scope: `smoke-test/src/commonTest/kotlin/io/opentelemetry/kotlin/smoketest/`; `smoke-test/src/jvmTest/` | WP-I | M-23 | The end-to-end smoke suite MUST drive the meter provider through start-up, recording, collection, export and shutdown against the collector fake, and the shutdown-compliance suite MUST cover the meter provider alongside the tracer and logger providers, asserting that shutdown is idempotent, that recording after shutdown is a no-op rather than an error, and that no export is attempted after shutdown completes. Verified by: a smoke scenario asserting that a recorded measurement reaches the collector fake's metrics route; an idempotence test invoking shutdown twice and asserting one export cycle; and a post-shutdown recording test asserting no payload and no thrown exception. Gates: G3-01, G3-02, G3-03, G3-04, G3-10 | 220 | yes | medium — R-15 |
| M-35 | The metrics compliance sheet, its per-row evidence standard and the closing recount — scope: a compliance sheet committed inside the repository at a path this unit fixes; `api/README.md`; the sheet's update rule | WP-K | M-26, M-27, M-28, M-29, M-30, M-31, M-32, M-33, M-34 | The compliance sheet MUST carry one row per specification compliance row, each citing the declaration or test that satisfies it rather than asserting satisfaction in prose, and MUST state its denominator basis exactly as Section 2.6 does so that the two cannot drift. Its update cadence MUST be per unit rather than per release: a unit that satisfies a row updates that row in the same change, and a unit that satisfies none states so. The closing recount MUST be reported in the sheet's own arithmetic and MUST close against 81 with the seven out-of-scope optional rows enumerated. This unit is the single sink of the dependency graph: it is blocked by every unit that can change a row's status. Verified by: a row count equal to the specification's metrics row count; every satisfied row carrying a resolvable locator; the arithmetic closing; and the stability matrix at `api/README.md:L28-L38` updated to match the surface the dumps actually carry. Gates: G3-01, G3-02, G3-04, G3-11 | 200 | yes | medium — R-31 |

## 4.1 Fixed G3 gate register

These eleven gates are used as given and are not renumbered. Every unit above and every evidence item in Section 6 maps to at least one [A5|DECIDED].

- **G3-01 — reviewable change unit.** Each implementation unit is strictly under 500 changed lines, excluding generated evidence only where the row calls that exclusion out, and no unit hides unrelated cleanup [A5|DECIDED].
- **G3-02 — compiler and static quality.** Zero static-analysis issues under a zero-issue budget, zero compiler warnings under warnings-as-errors, a 140-character line limit, required trailing commas, and designs that remain below the 31-function ceiling and the 30-constructor-parameter and 31-function-parameter ceilings [A2|DECIDED].
- **G3-03 — coverage.** Project coverage may decrease by no more than 1.00 percentage point, because the repository enforces a 1% threshold at `codecov.yml:L6` and never the looser 2.00-point window proposed elsewhere, and patch coverage must be at least 50%. At the measured baseline of 4864 covered of 5153 measurable lines the project window is the binding constraint, so a 500-line unit needs approximately 83.1% patch coverage to stay inside it [A2|DECIDED].
- **G3-04 — exact full build.** Exit code 0 from `./gradlew build :benchmark-android:assemble :benchmark-android:compileReleaseAndroidTestKotlin :benchmark-jvm:assemble :examples:example-app-android:assembleRelease koverXmlReport -x :gradle-integration-test:test --stacktrace`, with no lowered heap settings, and with the resulting coverage XML present at the path the workflow uploads. The asserted path `build/reports/kover/reportRelease.xml` is not produced by this command today, which `M-01` reconciles [A2|DECIDED].
- **G3-05 — environment and toolchain.** At least 16 GiB of available memory for the checked-in heap premise, JDK 11 discoverable through `./gradlew -q javaToolchains`, the pinned Gradle 9.7.0 wrapper checksum retained, and no dependency or catalog changes [A2|DECIDED].
- **G3-06 — publish and consumer integration.** Exit code 0 from `./gradlew publishToMavenLocal -PsnapshotPublish=true -Psigning.skip=true --stacktrace` and, as a separate invocation, `./gradlew :gradle-integration-test:test --stacktrace`. The two are never combined into one Gradle invocation [A2|DECIDED].
- **G3-07 — configured-target evidence.** JVM and Android compilation and tests, JavaScript compilation and tests, and compilation of both configured Apple targets on an appropriate host, with iOS simulator behaviour and golden tests executed where wired. No `iosX64` target exists and none is claimed [A2|DECIDED].
- **G3-08 — experimental API audit.** Every newly public metrics declaration carries `@ExperimentalApi`, all public surface appears in the intended dumps, and no accidental stable surface appears. This is compile-enforced inside `api` because that module disables the blanket opt-in, and remains an explicit audit everywhere else [A2|DECIDED].
- **G3-09 — binary and API evidence.** Zero unexplained dump deltas, explicit comparison of both the JVM and the Android dump, the pre-existing mismatch isolated, and no claim of klib compatibility evidence while no `.klib.api` baseline exists [A1|DECIDED].
- **G3-10 — metrics correctness.** Zero observed silent data loss, lost updates, duplicate collection or export, and zero uncaught callback or user exceptions across deterministic, property, stress, lifecycle and serialisation tests, with every failure producing reproducible inputs and an expected-versus-actual artifact [A5|DECIDED].
- **G3-11 — branch and artifact hygiene.** Disposable branches deleted, the working tree containing only the intended unit when gated, generated locks and models never hand-edited, no credential material in any log or document, and a clean `git status --porcelain` after accepted changes are committed [A2|DECIDED].


## 4.2 The change-unit measurement rule, and its pathspec trap

G3-01 is only meaningful if the command that measures it measures the right thing, and the obvious form of that command silently reports zero
[A2|DECIDED].

```console
$ BASE=$(git merge-base upstream/main HEAD)
$ git diff --numstat "$BASE"...HEAD -- . ':(exclude)**/*.api' | awk '{a+=$1} END {print "added:", a+0}'
$ git diff --numstat "$BASE"...HEAD -- '**/*.api'            | awk '{a+=$1} END {print "dump added:", a+0}'
```

Four properties of that measurement are load-bearing. The pathspec MUST be recursive and MUST NOT be root-anchored: a root-anchored pathspec
matches nothing and reports a false zero, which would make the budget gate always pass. Test sources COUNT toward the budget, because a unit's
tests are part of what a reviewer reads. Deletions do NOT count, because the gate bounds what must be reviewed rather than net churn.
Regenerated dumps are measured **separately** and are bounded at 700 lines rather than folded into the 500-line figure, because a dump's line
count is a function of the public surface rather than of the author's judgement — `M-02`'s entire content is a dump baseline, so a
source-directory pathspec would report zero for it, and `M-26` regenerates dumps across every touched module, so folding them in would breach
the budget for a change whose hand-authored content is small [A2|DERIVED].

## 4.3 Netting the reference tree down to a port figure

Every gross figure netted below is taken from the census in Section 2.12 and the sample measured in T-32; this subsection introduces no figure the
ledger does not already carry [A3|DERIVED].

The unit count is derived from a **net** figure, subpackage by subpackage, rather than from the reference tree's gross size, because several
subsystems this port would otherwise carry already exist here. The gross figure is 176 files over 16,406 lines (T-28) [A4|OBSERVED].

| Reference subpackage | Files | Disposition here | Net effect |
| --- | --- | --- | --- |
| root (35 files) | 35 | Meter and provider exist and are grown by `M-16`; the rest ports | Mostly ports |
| `data` (22) + `internal/data` (23) | 45 | The data model ports into `sdk-api` and `model`; the protocol message types already generate, so no serialisable duplicates are authored | Reduced |
| `export` (12) | 12 | Reader, exporter, producer and the three selectors port; the batch and retry machinery already exists | Reduced |
| `internal/aggregator` (19) | 19 | Six aggregations port in `M-09` through `M-11` and `M-27` | Ports in full |
| `internal/state` (17) | 17 | The highest-complexity subpackage; ports in `M-09`, `M-13`, `M-28` | Ports in full |
| `internal/exemplar` (16) | 16 | Reservoirs and filters port in `M-14` | Ports in full |
| `internal/view` (14) | 14 | Ports in `M-12`, except the filtered-attributes optimisation, which is a redesign under ADR-05 rather than a port | Reduced |
| `internal/concurrent` (7) | 7 | Collapses: the JRE-variant pair has no multiplatform analogue and the JVM actual delegates to the platform's own adder | Strongly reduced |
| `internal/descriptor` (3) | 3 | Ports into `M-07` | Ports in full |
| `internal/debug` (4) | 4 | **Unnecessary** — see Section 2.7 and area 24 of the map in Section 3.1 | Removed |
| `internal` + `internal/export` (4) | 4 | Meter configuration and utilities port where reached | Reduced |
| Configuration model | — | 113 generated models already exist (T-21); none is authored | Removed |
| Protocol messages | — | Generated from the pinned archive over the whole protocol tree; none is authored | Removed |
| Retry and transport | — | The generic seam exists and is repaired by `M-31` rather than rewritten | Removed |

Netting those reductions against the 16,406-line gross figure yields a port of approximately **11,760 hand-authored lines**, which is the
figure the unit estimates sum toward rather than the gross one; the 11,970-line total in Section 4 sits just above it because tests are counted
in the units and not in the reference line total [A4|DERIVED]. At 37.2 bytes per line (T-32) the port is roughly 437,000 bytes of Kotlin
[A4|DERIVED].

## 4.4 Security acceptance attached to existing gates

Twelve security conditions are enforced through the gates already defined rather than through a parallel review, so that no unit can satisfy its
gates while leaving one of them open [A5|DECIDED].

| # | Condition | Enforced by | What it prevents |
| --- | --- | --- | --- |
| 1 | No credential, token or high-entropy secret appears in any tracked file, log excerpt or commit message | G3-11 | An exposure that removal cannot remedy, because committed content is history |
| 2 | No unit weakens or disables a static-analysis rule to pass | G3-02 | Silencing the analysis that protects the containment boundaries |
| 3 | No unit lowers the checked-in daemon heap or alters `gradle.properties` | G3-04, G3-05 | Invalidating the premise every measurement rests on |
| 4 | User-supplied callables execute only inside a containment boundary | G3-10 | A hostile or defective callback destabilising the host |
| 5 | Every unbounded collection carries an explicit limit | G3-10 | Unbounded memory growth inside a host application |
| 6 | Instrument descriptors and attribute values are validated at entry | G3-10 | Malformed input propagating into aggregation or the wire |
| 7 | No exporter opens a network path unless configured to | G3-10 | An unrequested egress on SDK initialisation |
| 8 | Export failures and partial successes reach the error handler | G3-10 | Silent data loss presented as success |
| 9 | No dependency, catalog entry, wrapper property or lockfile line changes | G3-05 | An unreviewed supply-chain change riding along with a metrics unit |
| 10 | No generated model or lockfile is hand-edited | G3-11 | An edit that regeneration silently reverts |
| 11 | Every public declaration is experimental until deliberately stabilised | G3-08 | An unstable surface becoming a compatibility obligation by accident |
| 12 | No unit claims klib compatibility evidence while no klib baseline exists | G3-09 | Asserting a multiplatform guarantee no artifact supports |

One dependency exposure is recorded rather than gated, because no gate in this plan can close it: the Kotlin Gradle plugin in use sits inside a
published advisory's affected range concerning unsafe deserialization in the build cache. No remote build cache is used and continuous
integration disables caching, which is what makes acceptance defensible today; a version change is outside this plan's scope, so the correct
disposition is a recorded acceptance with an owner and a re-review boundary, and R-32 carries it with a measurable signal [A2|DECIDED].

## 4.5 The eleven work packages, and what each one is gated on

The `WORK_PACKAGE` column above is only meaningful if each value is defined, so this is the definition site for `WP-A` through `WP-K`. A work
package is the unit at which the project coverage gate G3-03 is enforced — G3-01, G3-02 and G3-09 remain per unit — which is why the packages are
drawn along contract boundaries rather than along module boundaries: a package must be able to close with a coherent, testable capability, or the
coverage window has nothing to close against. The eleven correspond one-to-one with the stages Section 5.1 sequences, which is deliberate: a package
is what a stage produces [A5|DECIDED].

| Package | Stage | Units | units | Σ est. added lines | What closing it establishes | Gate at the boundary |
| --- | --- | --- | --- | --- | --- | --- |
| WP-A | A | `M-01`, `M-02` | 2 | 20 | The gates can observe what they claim to: coverage reaches the report the workflow reads, and the inherited dump disagreement is isolated so later dump review is evidence | G3-03 becomes enforceable; G3-09 baseline fixed |
| WP-B | C | `M-03`, `M-04`, `M-05` | 3 | 1,020 | The public instrument surface exists, all four `Meter` implementors compile against it, and instrument identity is fixed | G3-08 audit; G3-09 dump review across both platforms |
| WP-C | C | `M-06` | 1 | 460 | The data model every later layer declares against, including exemplars and the readable attribute snapshot | G3-03 at the package boundary |
| WP-D | D | `M-07`, `M-08`, `M-32` | 3 | 1,100 | Series identity and platform accumulation, with the Apple and JavaScript actuals present rather than deferred | G3-07 configured-target evidence; G3-10 lost-update oracle |
| WP-E | E | `M-09`, `M-10`, `M-11`, `M-27`, `M-28` | 5 | 1,910 | Per-reader storage and all six aggregations with temporality isolation — the widest package, and the one ADR-07 governs end to end | G3-10 across the histogram, monotonicity and delta oracles |
| WP-F | E | `M-12`, `M-13`, `M-14` | 3 | 1,110 | Stream shaping: views, cardinality limits with a conserving overflow series, and exemplar reservoirs | G3-10 cardinality-conservation and reservoir-reset oracles |
| WP-G | F, G | `M-15`, `M-16`, `M-17`, `M-18`, `M-29` | 5 | 2,130 | The pipeline is reachable from configuration and collects on schedule, with hostile callbacks contained — the largest package by estimate | G3-10 containment, re-entrancy and fan-out oracles |
| WP-H | G | `M-19`, `M-20`, `M-21`, `M-30`, `M-31` | 5 | 1,540 | Export end to end: test exporter, wire conversion, transport, payload bounding, and the repaired seam | G3-06 and G3-10 with golden fixtures |
| WP-I | H | `M-22`, `M-23`, `M-34` | 3 | 900 | Environment and behaviour configuration, lifecycle within the aggregate bound, and an end-to-end route the collector fake actually matches | G3-10 lifecycle and end-to-end oracles |
| WP-J | I, J | `M-24`, `M-25`, `M-33` | 3 | 1,180 | Multiplatform evidence and the JVM differential, each documenting its own reach | G3-07 with positive Apple execution evidence |
| WP-K | K | `M-26`, `M-35` | 2 | 600 | Regenerated dumps, corrected documentation, and the compliance sheet that states the denominator basis | G3-09 and G3-11 |

Three readings follow from the table and each one shaped the sequence. **WP-A is 20 estimated lines and gates everything**, because until the
coverage artifact and the dump baseline are correct no later package can produce trustworthy gate evidence — the smallest package by far is the one
with the widest blocking effect. **WP-G is the largest at 2,130 estimated lines across five units**, which is why Section 5 splits its work across
two stages rather than treating it as one front. And the eleven package sums close against the table total: 20 + 1,020 + 460 + 1,100 + 1,910 + 1,110
+ 2,130 + 1,540 + 900 + 1,180 + 600 = 11,970, which is the sum Section 4 reports, so a unit reassigned between packages cannot silently change the
total [A5|DERIVED].

# 5. Dependency plan and critical path

Sequencing is expressed only as dependency and gate readiness. No stage carries a date, a duration or an owner [A5|DECIDED].

Everything in this section is derived from the `BLOCKED-BY` column of Section 4 and from nothing else — not from file affinity, not from
perceived difficulty, and not from the order the units happen to be numbered in [A5|DECIDED]. Re-deriving the graph from that column alone gives
its shape:

| Property | Value | How it is obtained |
| --- | --- | --- |
| Nodes | 35 | One per row of Section 4, `M-01` through `M-35` |
| Edges | 69 | One per entry in a `BLOCKED-BY` cell, counted across all 35 rows |
| Acyclic | yes | Depth-first traversal from every node terminates with no node reachable from itself |
| Roots | `M-01`, `M-02` | The only two rows whose `BLOCKED-BY` is `none` |
| Sink | `M-35`, and it is the only one | Every other node appears in at least one other row's `BLOCKED-BY` cell |
| Longest chain | 20 nodes | `M-01` → `M-03` → `M-04` → `M-05` → `M-06` → `M-07` → `M-09` → `M-12` → `M-13` → `M-15` → `M-16` → `M-18` → `M-19` → `M-20` → `M-21` → `M-23` → `M-24` → `M-25` → `M-26` → `M-35` |
| Dependency levels | 20 | A node's level is one more than the deepest level among its blockers |
| Row-order violations | 0 | No row is blocked by a row that appears later in the table |

The level assignment is what makes parallelism checkable rather than asserted: levels 1 and 6 through 9 and 12 through 18 each hold more than one
unit, and any two units at the same level are unblocked simultaneously.

| Level | Units unblocked together |
| --- | --- |
| 1 | `M-01`, `M-02` |
| 2–5 | `M-03`, then `M-04`, then `M-05`, then `M-06` — a strict chain, because each is the previous one's only unblocker |
| 6 | `M-07`, `M-08` |
| 7 | `M-09`, `M-32` |
| 8 | `M-10`, `M-11`, `M-12` |
| 9 | `M-13`, `M-14`, `M-27` |
| 10–11 | `M-15`, then `M-16` |
| 12 | `M-17`, `M-18` |
| 13 | `M-19`, `M-28` |
| 14–15 | `M-20`, then `M-21` |
| 16 | `M-22`, `M-23`, `M-30` |
| 17 | `M-24`, `M-29`, `M-31`, `M-34` |
| 18 | `M-25`, `M-33` |
| 19–20 | `M-26`, then `M-35` |

## 5.1 Stages

**Stage A — environment and baseline readiness.** The environment MUST satisfy G3-05 before any Gradle invocation is treated as evidence: at least 16 GiB of available memory measured against the checked-in premise of an 8 GiB Gradle daemon, a separate 8 GiB Kotlin daemon and 4 GiB of metaspace, and JDK 11 discoverable through `./gradlew -q javaToolchains`. Memory MUST be supplied rather than the checked-in heap lowered, and any out-of-repository heap override MUST be removed rather than relied upon. The unmodified checkout MUST then satisfy G3-04 through `./gradlew build :benchmark-android:assemble :benchmark-android:compileReleaseAndroidTestKotlin :benchmark-jvm:assemble :examples:example-app-android:assembleRelease koverXmlReport -x :gradle-integration-test:test --stacktrace`, MUST satisfy the static and dump gates through `./gradlew apiCheck detekt --stacktrace`, and MUST satisfy G3-06 through `./gradlew publishToMavenLocal -PsnapshotPublish=true -Psigning.skip=true --stacktrace` followed by the separate invocation `./gradlew :gradle-integration-test:test --stacktrace`. Stage A produces `M-01` and `M-02` [A2|DECIDED].

**Stage B — disposable verification.** Both spikes defined in Section 5.3 MUST run and MUST be discarded before any implementation unit is opened. Stage B produces no committed artifact [A5|DECIDED].

**Stage C — public contracts.** `M-03`, `M-04`, `M-05` and `M-06` establish the instrument surface, the parity implementations and the SDK contract and data model. No storage or aggregation code SHALL be written before this stage completes, because every later type is expressed in terms of these contracts [A3|DECIDED].

**Stage D — identity and accumulation.** `M-07` and `M-08` establish series identity and the platform accumulation primitives. These are the two units on which every correctness oracle in Section 6 depends, and `M-08` MUST NOT begin before `M-07` fixes the key type that storage will hold [A3|DECIDED].

**Stage E — aggregation and stream shaping.** `M-09` through `M-14` deliver per-reader storage, the aggregations, views, cardinality limits and exemplars, in that order, because each consumes the state model the previous one established [A4|DECIDED].

**Stage F — configuration, meter and collection.** `M-15`, `M-16`, `M-17` and `M-18` make the pipeline reachable from configuration and drive it from a reader. `M-18` MUST follow `M-09` because a reader collects per-reader state that must already exist [A3|DECIDED].

**Stage G — export.** `M-19`, `M-20` and `M-21` deliver the test exporter, the wire conversion and the transport, in that order, because the conversion is validated against in-memory output before it is put on a wire [A3|DECIDED].

**Stage H — integration.** `M-22` and `M-23` connect the environment and behaviour surfaces and the lifecycle [A3|DECIDED].

**Stage I — cross-platform and differential evidence.** `M-24` then `M-25`, because the differential scenarios consume the meter hook the harness gains in `M-24` [A3|DECIDED].

**Stage J — pipeline completion.** `M-27` through `M-34` complete the pipeline against the contracts the earlier stages fixed: aggregation
selection and per-reader temporality isolation, reader scheduling and flush fan-out, payload bounding and the transport-seam repair, the Apple
and JavaScript actuals with their shared behaviour tests, cross-platform evidence, and the smoke and shutdown-compliance coverage. Every unit in
this stage sits at level 7 or later and none of them blocks another except `M-30` before `M-31`, so the stage is the widest parallel front in the
plan [A3|DECIDED].

**Stage K — dumps, documentation and compliance.** `M-26` then `M-35` last, because a dump regenerated before the final public surface is settled
is regenerated twice and conflicts on every intervening rebase, and because the compliance sheet's closing recount is meaningful only once every
unit that can change a row's status has landed — which is precisely why `M-35` is the graph's single sink [A2|DECIDED].

## 5.2 Critical path and parallelism

The critical path is the graph's longest chain, and it is 20 nodes rather than the 10-node spine an earlier reading of a linear table suggested:
`M-01` → `M-03` → `M-04` → `M-05` → `M-06` → `M-07` → `M-09` → `M-12` → `M-13` → `M-15` → `M-16` → `M-18` → `M-19` → `M-20` → `M-21` → `M-23` →
`M-24` → `M-25` → `M-26` → `M-35`. In words: coverage-artifact correction, then the public instrument surface and its parity implementations,
then the SDK contract, then attribute identity, then aggregation state, then views and their cardinality bound, then the configuration surface and
the meter, then reader collection, then the test exporter, the wire conversion and the transport, then lifecycle, then the differential harness and
its scenarios, then regenerated dumps and finally the compliance sheet. Shortening this chain is the only way to shorten the plan; every other
unit has slack by construction [A5|DERIVED]. The chain's own weakest link is `M-07`, because ADR-05's key materialisation is both the
highest-risk design in the plan and a blocker of everything downstream of it — which is why the vertical slice in Section 5.3 exists to test
exactly that decision before any unit is opened. Note thater conversion, then lifecycle and configuration integration, then cross-platform and differential evidence, then final dumps and documentation. Every unit on this path blocks at least three successors and MUST be prioritised over any unit that does not [A3|DERIVED].

Five branches MAY proceed in parallel with the critical path once their blockers clear, and each is genuinely independent because its files do
not overlap the critical path's files at the same level. `M-32` proceeds in the Apple and JavaScript source sets of
`platform-implementations` from level 7, and `M-33` follows it, so cross-platform evidence accumulates alongside aggregation rather than after it.
`M-27` and `M-28` proceed in the aggregator and state subpackages once `M-09` fixes the storage contract. `M-30` and `M-31` proceed in the
protobuf and OTLP modules once the conversion exists, and `M-31`'s regression assertions against the logging and tracing suites are what makes it
safe to run in parallel with metrics work at all. `M-29` and `M-34` proceed in `exporters-core` and `smoke-test` once lifecycle is wired. `M-08` proceeds in `platform-implementations` once `M-07` fixes the key type. `M-10`, `M-11`, `M-12`, `M-13` and `M-14` proceed in separate aggregator, view, state and exemplar subpackages once `M-09` fixes the storage contract. `M-19`, `M-21` and `M-22` proceed in three separate exporter and configuration modules once `M-18` and `M-20` respectively are available. Parallel work MUST NOT be claimed across units that touch the same dump, because a dump is regenerated wholesale and two units regenerating one dump conflict by construction [A1|DERIVED].

Every merge MUST satisfy `./gradlew build :benchmark-android:assemble :benchmark-android:compileReleaseAndroidTestKotlin :benchmark-jvm:assemble :examples:example-app-android:assembleRelease koverXmlReport -x :gradle-integration-test:test --stacktrace` with exit code 0, and MUST satisfy `./gradlew apiCheck detekt --stacktrace` with exit code 0. Rebase MUST precede every merge, and a dump conflict MUST be resolved by re-running the dump task rather than by hand-merging, because the dumps are machine-generated and line-dense [A2|DECIDED].

## 5.3 The two disposable spikes

Both spikes are local branches. Neither is pushed, neither is merged, neither counts as a delivered change unit, and neither survives Stage J. Each is deleted only after its result has been recorded as `MATCHES`, `DIFFERS` or `UNAVAILABLE` in the architecture decision that owns its question. A spike is evidence, never a deliverable, a scaffold or a partial implementation, which is exactly the boundary HC-09 draws; no spike output is committed, per HC-02 [A5|DECIDED].

**Spike 1 — minimum viable vertical slice.** This spike MUST be placed immediately after Stage A and MUST precede every implementation unit, so that an unsound architecture is invalidated at the cost of one throwaway branch rather than at the cost of Stage E. Its question is whether one long counter can travel end to end under the constraints ADR-05, ADR-06 and ADR-09 impose. It MUST prove exactly this path and nothing more: a meter factory call, an immutable instrument descriptor, materialisation of the attributes receiver lambda into a hashable key, accumulation into concurrent sum storage, collection through a manually driven pull reader, conversion into either an in-memory point or a protocol message, and comparison of the result against the Java reference at tag `v1.65.0` on the JVM. Histograms, asynchronous callbacks, views, broad configuration, exemplars and production scheduling MUST be excluded, because each adds a failure mode that would obscure the answer. The recorded result MUST state whether the key type supports accumulation without aliasing, whether the accumulator's read-and-reset is atomic across the platform actuals it touched, and whether the reference comparison agreed. Escalation is defined: if the slice cannot be made to work, ADR-05 and ADR-06 MUST be reopened before any unit is written, and no implementation unit SHALL proceed on the assumption that the architecture holds. The branch is then deleted [A3|DECIDED].

**Spike 2 — configuration-cache compatibility.** Its question is whether the build logic any unit needs remains compatible with the configuration cache under a configuration where cache problems fail the build rather than warn. Because ADR-01 elects no new Gradle project, the only build-logic changes the plan foresees are a fixture-staging task for new golden resources and a possible coverage-report task rename in `M-01`, so this spike MUST exercise exactly those. The evidence is a first invocation of `./gradlew build :benchmark-android:assemble :benchmark-android:compileReleaseAndroidTestKotlin :benchmark-jvm:assemble :examples:example-app-android:assembleRelease koverXmlReport -x :gradle-integration-test:test --stacktrace` that stores a configuration cache entry, followed by an immediately repeated identical invocation whose log reports cache reuse rather than a fresh store. Disabling the configuration cache is permitted only to isolate whether the cache is the cause of a failure and MUST NOT be left disabled as a fix, and the repository's heap settings MUST NOT be altered to make a spike pass. Escalation is defined: if reuse cannot be achieved, the offending build logic MUST be redesigned against an existing module's script as the model, and if it still cannot be achieved the unit that requires it MUST be reduced in scope rather than merged with a failing cache. The branch is then deleted [A2|DECIDED].

## 5.4 The vertical slice, and what running it established

Spike 1 was executed as specified before any unit was opened, on a local branch that carried a single long counter from a meter factory call
through descriptor construction, key materialisation, concurrent sum accumulation, a manually driven pull reader and an in-memory point, compared
against the Java reference at tag `v1.65.0` on the JVM. Its conclusions are recorded here and in the architecture records that own its questions,
because they are the reason three of those records read as they do; the branch was then deleted with no surviving ref [A3|OBSERVED].

- **The key materialisation holds, and it must happen exactly once.** A hashable key built from the receiver lambda at the recording call
  aggregated without aliasing across distinct attribute sets and without splitting a single set, and the reference comparison agreed on both the
  point values and the attribute sets. The slice also established the negative half: materialising lazily at collection time rather than at
  recording time loses the ordering guarantee, because the receiver may be mutated after the recording call returns. ADR-05 therefore requires
  materialisation at the recording call rather than merely somewhere before the aggregation lookup [A3|OBSERVED].
- **Two concurrency defects were measured rather than predicted.** First, read-and-reset on a naive accumulator is not atomic when the read and
  the reset are separate operations: a concurrent recording landing between them is lost, and it is lost silently because no count is kept of
  measurements accepted versus measurements aggregated. Second, the existing atomic primitive's JavaScript actual is a plain variable while its
  Apple actual serialises on a lock, so the three platform actuals of the same declaration have materially different contention profiles and a
  test that passes on one target proves nothing about another. ADR-06 requires read-and-reset to be a single atomic operation and requires the
  shared behaviour tests `M-32` places in the common test source set; Section 6 lists both defects as loss modes with their own oracles
  [A3|OBSERVED].
- **A static-analysis task-reach asymmetry was found.** Running the analysis task at project scope rather than at whole-build scope does not
  reach every source set that a whole-build invocation reaches, so a unit verified with a module-scoped invocation can still fail the gate on
  merge. G3-02 is therefore stated against the whole-build invocation, and `AGENTS.md:L44` independently prefers whole-project verification tasks
  for the same reason [A2|OBSERVED].
- **What the slice deliberately did not answer.** It touched one aggregation, no view, no exemplar, no asynchronous callback and no wire
  transport, so it establishes nothing about histogram bucketing, scale recovery under delta temporality, callback containment or partial-success
  handling. Those remain specified rather than proven, and Section 6 assigns each an oracle rather than borrowing confidence from this slice
  [A5|DECIDED].

Spike 2 was executed on its own local branch and reported a stored configuration-cache entry on its first invocation and a reused entry on an
immediately repeated identical invocation, with no configuration-cache problem raised under a configuration where problems fail the build; T-10
and T-11 reproduce that store-then-reuse pair on the untouched tree. Its branch was likewise deleted once ADR-15 recorded the result. Neither
spike's output was committed, and neither is counted against any unit's budget [A2|OBSERVED].

# 6. Correctness strategy

## 6.1 The failure mode that governs

**Silent data loss is the primary unacceptable failure mode.** A metrics pipeline can lose a measurement, merge two series that should be distinct, split one series that should be single, export an interval twice, or export nothing at all, and in every one of those cases the process stays up, no exception is raised, no log line appears, and every existing test passes. That is why correctness here MUST be established by oracles that compare a computed total against a recorded total, rather than by the absence of failures. Every category below therefore states the oracle it applies, the artifact a failure produces, and the gate that consumes the result [A5|DECIDED].

The eleven oracles are stated as checkable properties rather than as intentions [A5|DECIDED]:

- **Lost updates.** For any set of concurrent recordings, the sum of the values exported for a series MUST equal the sum of the values recorded for it. A shortfall is a lost update, and the failure artifact is the recorded total, the exported total and the seed [A5|DECIDED].
- **Attribute-series aliasing.** Two recordings whose materialised keys compare equal MUST land in one series, and two whose keys compare unequal MUST land in two. Aliasing shows as a series count that differs from the distinct-key count, and the failure artifact is both key sets [A3|DECIDED].
- **Double export.** Across any sequence of collections, each recorded value MUST appear in exactly one exported delta interval, and each cumulative point MUST be monotonically consistent with its predecessor. The failure artifact is the per-cycle point sequence [A4|DECIDED].
- **Delta reset.** After a delta collection the accumulated state for the collected series MUST be empty, and for a histogram the scale MUST have returned to its unstressed value so that precision recovers after an outlier cycle. The failure artifact is the pre-collection and post-collection state including the scale [A4|DECIDED].
- **Cumulative monotonicity.** For a monotonic instrument under cumulative temporality the exported sum MUST never decrease and the start timestamp MUST never change for a live series. The failure artifact is the offending pair of consecutive points [A4|DECIDED].
- **Histogram invariants.** The sum of bucket counts MUST equal the point count, the recorded sum MUST equal the arithmetic sum of the inputs, the minimum and maximum MUST bound every input, and a value equal to a boundary MUST fall in the bucket that boundary closes. The failure artifact is the input list and the resulting point [A4|DECIDED].
- **Callback failure containment.** A throwing callback MUST NOT propagate, MUST produce at most one error report per callback per cycle, MUST leave other series intact, and MUST contribute no partial series. The failure artifact is the thrown throwable, the report count and the collected point set [A3|DECIDED].
- **Cardinality overflow.** Once a limit is reached the exported total across all series including the overflow series MUST still equal the recorded total. Any shortfall means measurements were dropped. The failure artifact is the series count, the limit and both totals [A4|DECIDED].
- **Shutdown races.** A collection concurrent with shutdown MUST either complete fully or contribute nothing, MUST never contribute a partial series, and a second shutdown MUST return success without repeating work. The failure artifact is the interleaving and the exported set [A3|DECIDED].
- **Timeout behaviour.** A slow collection or export MUST yield a bounded failure result rather than a hang, and MUST NOT cause the facade's single aggregate lifecycle bound to be exceeded. The failure artifact is the measured elapsed time and the returned result [A3|DECIDED].
- **Exporter retry and idempotency.** A retried export MUST NOT duplicate points at the receiver, a partial success and a client error MUST be terminal rather than retried indefinitely, and a server-supplied retry delay MUST be honoured. The failure artifact is the request sequence recorded by the collector fake [A3|DECIDED].

## 6.1.1 The twenty-one loss modes, each with an oracle and an owner

The eleven oracles above are the checkable properties. This table enumerates every way a measurement can be lost, merged, split, duplicated or
silently altered in the specified pipeline, names the oracle that would detect it, and names the role accountable for that oracle holding. An
owner here is a role, not a schedule commitment [A5|DECIDED].

| # | Loss mode | Oracle that detects it | Owner |
| --- | --- | --- | --- |
| 1 | Two distinct attribute sets collapse into one series | Attribute-series aliasing: keys comparing unequal MUST land in two series | `M-07` author |
| 2 | One attribute set splits across several series because key equality is order-dependent | Key-order-independence property test over permuted attribute insertion orders | `M-07` author |
| 3 | A concurrent recording is lost between an accumulator's read and its reset | Lost updates: exported sum MUST equal recorded sum under concurrent stress | `M-08` author |
| 4 | A recording lands after collection has snapshotted but before it has reset, and is discarded | Double export plus lost updates read together across a collection boundary | `M-09` author |
| 5 | A delta collection resets state a cumulative reader has not yet observed | Delta reset asserted per reader rather than per series | `M-28` author |
| 6 | A cumulative series' start timestamp changes, so the consumer treats it as a restart and discards history | Cumulative monotonicity, including start-timestamp stability for a live series | `M-09` author |
| 7 | A histogram records a value into the wrong bucket at a boundary | Histogram invariants: boundary exclusivity of the lower bound and inclusivity of the upper | `M-10` author |
| 8 | An exponential histogram's scale narrows after an outlier and never recovers under delta temporality | Scale-recovery test: outlier then tight cluster MUST widen the scale back | `M-28` author |
| 9 | A view's attribute filter drops a key the exemplar needed, or retains one it should have dropped | View filtering asserted at the layer ADR-05 fixes, with exemplar attributes asserted separately | `M-12` author |
| 10 | A measurement is dropped on reaching the cardinality limit instead of being folded into the overflow series | Cardinality overflow: exported total including the overflow series MUST equal the recorded total | `M-13` author |
| 11 | An exemplar reservoir retains a stale sample across a collection cycle | Reservoir reset asserted per cycle with a seeded generator | `M-14` author |
| 12 | An asynchronous callback throws and its whole series disappears | Callback failure containment: other series intact, at most one report per callback per cycle | `M-17` author |
| 13 | An asynchronous callback exceeds its bound and its series is silently absent | Callback timeout asserted with virtual time, plus an explicit absent-series assertion | `M-17` author |
| 14 | A callback triggers collection re-entrantly and the cycle deadlocks or double-counts | Re-entrancy denial asserted by invoking collection from inside a callback | `M-17` author |
| 15 | Two collections overlap on one reader and points are attributed to the wrong interval | Non-overlap asserted with virtual time; a skipped rather than queued cycle | `M-29` author |
| 16 | A reader registered after construction never participates in a cycle | Late-registration test asserting participation in the next cycle | `M-29` author |
| 17 | A force-flush returns success while one exporter failed | Fan-out asserted to await all and to fail if any fails, with the others still invoked | `M-29` author |
| 18 | The payload exceeds the point bound and is truncated rather than split | Bounding test asserting the union of split requests equals the input | `M-30` author |
| 19 | A partial-success response is treated as success and rejected points vanish | Partial-success routing asserted to reach the error handler exactly once | `M-30` author |
| 20 | The transport seam reports success before the exchange completes, so a failed export looks delivered | Seam ordering asserted against a mock engine: the result follows the exchange | `M-31` author |
| 21 | No reader is registered, so a fully configured pipeline exports nothing and reports nothing | Configuration oracle asserting that the no-reader case is observable through the error handler rather than silent | `M-22` author |

Three properties of this table matter as much as its contents. Every loss mode has an oracle, so none is left to code review. Every oracle is
assigned, so none is everyone's responsibility. And every oracle is a **test that fails**, not a metric that trends: a loss mode whose only
detection is a dashboard is not covered by this plan [A5|DECIDED].

## 6.1.2 Coverage arithmetic, and what it constrains

The coverage window is not a preference, and it constrains unit size arithmetically rather than advisorily [A2|DERIVED]. The measured base is
4,864 covered of 5,153 measurable lines, a rate of 94.39% (T-13). The enforced project tolerance is 1% at `codecov.yml:L6`, so G3-03 fixes the
window at **1.00 percentage point** and never widens it — the 2.00-point window proposed elsewhere is explicitly not carried forward, because
`codecov.yml` is the gate that actually fails a build.

The consequence is computable. A 500-line unit landing at exactly the 50% patch floor adds 500 measurable lines of which 250 are covered, moving
the rate to (4864 + 250) ÷ (5153 + 500) = 90.46%, a drop of 3.93 points — roughly four times the window. It follows that **the window absorbs
approximately one exemption-path unit, not two**, and that a unit landing at the ceiling with only floor-level patch coverage breaches the gate on
its own. Two responses follow, and the plan takes the first: tighten the exemption policy so an exemption is permitted only where the successor
unit is the immediate next unit in the same work package, rather than widening the window past the repository's own gate. The second response —
accepting a known continuous-integration failure at work-package boundaries — is rejected because it makes the gate advisory [A2|DECIDED].

This is also why the estimates in Section 4 sum to a mean of 342.0 rather than clustering at the ceiling: a unit near 490 lines must arrive with
patch coverage well above the floor to stay inside the window, and the units that carry the least testable content are deliberately the smallest.

## 6.2 Evidence categories

Ten kinds of evidence, each with the source set it lives in, the oracle it applies, the artifact a failure produces and the gate that consumes it. A claim supported by none of these ten is not supported [A5|DECIDED].

| Category | Target and source set | Oracle | Failure artifact | Gate |
| --- | --- | --- | --- | --- |
| Unit | `implementation/src/commonTest`, `exporters-core/src/commonTest`, `sdk-api` consumers; runs on JVM, Android, JavaScript and the iOS simulator on a macOS host | Deterministic expected-versus-actual on a single behaviour, using the existing fake clock and seeded identifier generator rather than real time | The asserted and actual values with the fake clock reading | G3-10 [A3\|DECIDED] |
| Property | `implementation/src/commonTest` using the property-testing artifact already present in the catalog | The histogram, monotonicity, key-order-independence and bucket-index oracles, over generated inputs | The failing generated input and its seed, so the case is reproducible | G3-10 [A2\|DECIDED] |
| Concurrency stress | `implementation/src/jvmTest` and `platform-implementations/src/jvmTest`, JVM only because it is the only target with shared-memory threads | The lost-update oracle: recorded total equals exported total across many concurrent recorders interleaved with collections | Recorded total, exported total, thread count and seed | G3-10 [A3\|DECIDED] |
| Lifecycle | `implementation/src/commonTest` and `smoke-test/src/commonTest` | The shutdown-race, timeout and idempotency oracles against the existing suspending closeable contract and the facade's aggregate bound | The interleaving, the returned sealed result and the elapsed time | G3-10 [A3\|DECIDED] |
| Serialisation and golden | `exporters-protobuf/src/commonTest` with fixtures in the existing fixture directories, loaded through the existing multiplatform loader | Byte-for-byte equality with a committed fixture for each point type, including an exemplar-bearing point | The fixture path plus the expected and actual encodings | G3-10 [A3\|DECIDED] |
| Configuration | `implementation/src/commonTest` and `config-envar/src/commonTest` | Each configured value reaches the provider, an unparseable value degrades to the documented default with exactly one diagnostic, and layer precedence matches the tracer slot's rule | The input configuration, the resolved configuration and the diagnostic count | G3-02, G3-10 [A3\|DECIDED] |
| Behaviour, shared across platforms | `platform-implementations/src/commonTest` beside the existing primitive tests, and `implementation/src/commonTest` | Accumulator add-and-reset atomicity, and pipeline behaviour expressed without platform-specific assumptions, so the same assertions run on every configured target | The target name with the asserted and actual values | G3-07, G3-10 [A2\|DECIDED] |
| Integration, end to end | `smoke-test/src/commonTest` against the existing collector fake | A configured provider with a periodic reader and the OTLP exporter delivers the expected payload on the metrics path, and the retry and idempotency oracles hold | The request sequence the fake recorded, including paths and bodies | G3-10 [A3\|DECIDED] |
| Differential against the Java reference | `compat/src/jvmTest` only; JVM only | Identically named scenarios produce output matching deliberately overlapping golden files across both back ends, for semantics both implementations claim | Both back ends' output and the shared fixture | G3-10 [A3\|DECIDED] |
| Binary compatibility and annotation audit | The 27 tracked dump files, reviewed per unit alongside `./gradlew apiCheck detekt --stacktrace` | The dump diff contains exactly the declarations the unit adds, every one of them annotated, and no unrelated line; the audit is a read of the diff because the experimental marker is not excluded from dumps and therefore cannot be checked by the validator alone | The dump diff with each unexpected or unannotated line named | G3-08, G3-09 [A1\|DECIDED] |

## 6.3 The reach limit of the differential, stated exactly

The abstract harness is `integration-test/src/commonMain/kotlin/io/opentelemetry/kotlin/framework/OtelKotlinTestRule.kt`, which is a common-source abstract rule and therefore compiles for every configured target [A3|OBSERVED]. The Kotlin back end is `implementation/src/commonTest/kotlin/io/opentelemetry/kotlin/integration/test/IntegrationTestHarness.kt`, which is likewise multiplatform [A3|OBSERVED]. The reference-delegating back end is `compat/src/jvmTest/kotlin/io/opentelemetry/kotlin/framework/OtelKotlinHarness.kt`, and it lives in the `jvmTest` source set of a module restricted to JVM and Android targets, so it cannot be compiled for Apple or JavaScript at all [A3|OBSERVED].

The consequence is a hard limit, not a preference: **differential comparison against the Java reference reaches the JVM and nothing else.** It does not reach Apple. It does not reach JavaScript. It does not reach Android instrumented execution. Any claim that a metrics behaviour is verified against the reference implementation MUST therefore be scoped to the JVM in the same sentence [A3|DERIVED].

A second limit bounds the differential even on the JVM. The reference implementation satisfies 75 of the 88 metrics compliance rows and does not satisfy the remaining 13, so for those rows it is not an oracle at all and agreement with it would be meaningless. The differential MUST NOT be cited as evidence for any row the reference does not satisfy [A4|OBSERVED].

## 6.4 The Apple position, stated precisely

Apple is not categorically unverifiable, and this plan MUST NOT say that it is [A2|DECIDED]. Three distinct facts hold at once. Both configured Apple targets compile on a Linux host: `./gradlew :api:compileKotlinIosArm64 :api:compileKotlinIosSimulatorArm64 :implementation:compileKotlinIosArm64 :implementation:compileKotlinIosSimulatorArm64` exits 0 [A2|OBSERVED]. Common test resources are already staged onto the simulator, because `implementation/build.gradle.kts:L64-L67` registers a `Copy` task named `copyiOSTestResources` that copies `src/commonTest/resources` into the simulator debug-test resources directory, and `implementation/build.gradle.kts:L69-L71` makes `iosSimulatorArm64Test` depend on it, so shared behaviour and golden tests do execute on the iOS simulator when a macOS host is available [A2|OBSERVED]. And on a non-macOS host the simulator test task is skipped rather than failed: `./gradlew :implementation:iosSimulatorArm64Test` exits 0 while the task and its test-binary link task are both reported skipped, with the toolchain stating the task cannot run on a Linux host [A2|OBSERVED].

The third fact is the hazard. A zero exit code from `./gradlew build :benchmark-android:assemble :benchmark-android:compileReleaseAndroidTestKotlin :benchmark-jvm:assemble :examples:example-app-android:assembleRelease koverXmlReport -x :gradle-integration-test:test --stacktrace` on a non-macOS host is not Apple test evidence, because the Apple test tasks silently did not run. G3-07 MUST therefore be satisfied by positive evidence that the Apple tasks executed, and never by the absence of a failure [A2|DECIDED].

## 6.5 Test placement conventions

Shared assertions MUST be written once in a common test source set so that they run on every configured target, following the pattern the platform primitives already use, where `platform-implementations/src/commonTest/kotlin/io/opentelemetry/kotlin/` holds behaviour tests for the atomic, lock, thread-local and map primitives [A3|OBSERVED]. Platform-specific implementations MUST use the platform-suffixed actual file-name convention that `GoldenFileDataLoader.jvm.kt`, `GoldenFileDataLoader.apple.kt`, `GoldenFileDataLoader.js.kt`, `HttpClientInstance.jvm.kt`, `HttpClientInstance.apple.kt` and `HttpClientInstance.js.kt` establish [A3|OBSERVED]. Golden fixtures MUST be added to the existing fixture directories rather than to new ones, and MUST be loaded through the existing multiplatform loader so that no target-specific file access is introduced [A3|DECIDED]. Determinism MUST come from the existing fake clock, seeded identifier generator, coroutine test dispatchers and mock HTTP engine, and no test SHALL be retried, skipped or marked ignored to make a suite pass, since the baseline suite reports 2965 tests with zero skips [A2|DECIDED].


# 7. Risk register

Likelihood and impact are integers on a five-point scale, declared here once and applied in every row: likelihood 1 remote, 2 unlikely, 3
possible, 4 likely, 5 near-certain; impact 1 cosmetic, 2 recoverable within a unit, 3 recoverable within a work package, 4 blocks a work package
or ships a wrong figure, 5 silently wrong telemetry or an unstable host. `composite` is the arithmetic product of the two, and rows are ordered by
`composite` descending. Every early-warning signal is an observable a gate, a test or a command produces — never a judgement that something feels
wrong. Identifiers `R-01` through `R-34` are carried inside the `risk` cell because the frozen column set has no identifier column; the register
is the definition site for every `R-` reference made in Sections 2 and 4. No unresolved blocker is hidden in prose [A5|DECIDED].


| risk | likelihood | impact | composite | mitigation | early-warning signal |
| --- | --- | --- | --- | --- | --- |
| R-01 — Silent data loss through attribute-key aliasing: two distinct attribute sets collapse into one series, or one set splits into many, so the payload is wrong while the process stays healthy and every existing test passes | 4 | 5 | 20 | ADR-05 mandates a single immutable, order-independent, type-preserving key materialised once at the recording call; `M-07` proves order-independence and type distinction by property test; loss modes 1 and 2 in Section 6.1.1 are merge blockers owned by the `M-07` author. Escalation: a failing aliasing oracle stops `M-09` and every later unit until ADR-05 is amended, because storage keyed on a broken key cannot be corrected downstream | Exported series count differs from the distinct-key count in the `M-07` property tests, or a permuted-insertion-order case fails |
| R-02 — Lost updates under concurrency: concurrent recordings on one series are not all accumulated, producing undercounted metrics that look plausible | 4 | 5 | 20 | ADR-06 mandates a striped accumulator with an atomic read-and-reset; `M-08` and `M-09` carry JVM stress tests whose oracle is total equality; loss modes 3 and 4 in Section 6.1.1 are owned by the `M-08` and `M-09` authors. Residual: stress runs on the JVM only, so a loss reachable only through an Android-specific scheduler would be missed. Escalation: any shortfall stops the accumulation unit and reopens ADR-06 rather than being retried | A `M-08` or `M-09` stress test in which exported total is less than recorded total, at any thread count or seed |
| R-05 — Cardinality explosion: unbounded distinct attribute sets exhaust memory inside the host application and produce an export payload that is both enormous and incomplete | 4 | 5 | 20 | ADR-07 mandates a configurable per-stream limit with a reserved overflow series that continues to accumulate, plus a bounded diagnostic; `M-13` asserts total conservation with the excess in the overflow series; loss mode 10 is owned by the `M-13` author. Residual: a default limit set too low silently degrades legitimate high-cardinality use. Escalation: dropping rather than folding is a stop, because it is the primary failure mode named in Section 6.1 | A `M-13` test in which series count grows without bound, or exported total is less than recorded total once a limit is reached |
| R-30 — The transport seam reports success before the exchange completes, so a failed export is indistinguishable from a delivered one and the host's only delivery signal is wrong | 4 | 5 | 20 | `M-31` repairs the seam and asserts against a mock engine that the returned result follows the exchange rather than preceding it, and carries mandatory regression assertions for the logs and tracing exporters that share the seam; ADR-11 fixes the suspending sealed-result contract that makes the ordering observable; loss mode 20 is owned by the `M-31` author. Escalation: a success result covering a failed exchange is a stop for `M-31` and for `M-34` | A mock-engine test in which a rejected exchange yields a success result, or in which the result is produced before the engine records the request |
| R-04 — Asynchronous callback escape or re-entrancy: user code throws into collection or triggers collection from inside a callback, so one instrumentation defect aborts a cycle, fails a host-awaited flush, or duplicates exported points | 4 | 4 | 16 | ADR-08 and ADR-17 mandate containment at the invocation boundary, bounded reporting, no partial series, and re-entrancy denial; `M-17` carries all three tests; the static-analysis configuration already permits catching throwables at these boundaries at `config/detekt/detekt.yml:L26-L31`, so containment does not fight the gate; loss modes 12, 13 and 14 are owned by the `M-17` author. Escalation: an escaping throwable is a stop for every later unit, because it violates the host-stability doctrine at `AGENTS.md:L31` outright | A hostile-callback test in which the throwable propagates, the error report count exceeds one per callback per cycle, or a re-entrancy test yields duplicate points |
| R-06 — The JVM and Android dump disagreement is inherited into the metrics delta, so a reviewer cannot distinguish a metrics-induced dump change from pre-existing staleness and dump review stops being evidence | 4 | 4 | 16 | ADR-13 rules on the conflict and `M-02` isolates the inherited delta before any metrics API change is accepted; every subsequent unit names which platform dumps it changed; G3-09 consumes the comparison. Residual: whether the eleven stale Android dumps should be regenerated is genuinely open and is the highest-ranked question in Section 8. Escalation: if isolation is impossible, no metrics API unit is accepted until it is | Any metrics unit whose Android dump diff contains lines unrelated to metrics |
| R-09 — Apple evidence is inferred from a zero exit code on a non-macOS host, so Apple-specific accumulation defects ship unverified while the build is green | 4 | 4 | 16 | G3-07 requires positive evidence that the Apple tasks executed; Section 6.4 states the three facts separately; `M-32` supplies the Apple accumulation actuals and `M-33` requires a macOS run showing the simulator test task executed rather than skipped; both Apple targets are compiled on every host so compilation regressions surface immediately. Escalation: absent a macOS run, Apple execution is reported `UNAVAILABLE` and never as passing | `./gradlew :implementation:iosSimulatorArm64Test` exiting 0 with the task reported skipped, cited anywhere as Apple test evidence |
| R-03 — Incorrect delta and cumulative isolation: one reader's collection consumes or resets state another reader needs, systematically under-reporting for every reader after the first and remaining invisible in any single-reader test | 3 | 5 | 15 | ADR-07 mandates per-reader state ownership and `M-27` gives each reader its own storage; `M-09` carries the opposing-temporality two-reader test and the monotonicity property test; `M-28` isolates temporality conversion; loss modes 5 and 6 are owned by the `M-28` and `M-09` authors. Residual: a reader registered after measurements exist is a defined error path that must itself be tested. Escalation: a failure stops `M-18` because a reader cannot be built on state it can corrupt | A two-reader test in which either reader observes an interval the other consumed, or a cumulative sum that decreases |
| R-15 — Lifecycle timeout or partial flush: a metrics flush exceeds its share of the aggregate bound, or reports success while data was dropped, so a host blocks longer than documented or believes telemetry was delivered when it was not | 3 | 5 | 15 | ADR-11 preserves the aggregate bound and the all-success combination unchanged and requires the reader's collection timeout to fit inside it; `M-23` asserts the bound and that a failing exporter yields a failure result rather than an exception; `M-29` asserts force-flush fan-out; loss mode 17 is owned by the `M-29` author. Residual: with several readers, per-reader timeouts must still sum within the aggregate bound, which constrains how many readers are practical. Escalation: a success result covering dropped data is a stop, because it defeats the only signal the host has | A lifecycle test in which the aggregate flush exceeds the documented bound, or a flush returning success while the collector fake received nothing |
| R-22 — The base-2 exponential histogram's scale narrows after an outlier and never recovers under delta temporality, so precision degrades permanently while every point still validates | 3 | 5 | 15 | `M-28` carries the scale-recovery test as a named acceptance criterion: an outlier followed by a tight cluster MUST widen the scale back within one collection cycle; ADR-07 requires scale to be per-reader state rather than per-series state so a delta reset restores it; loss mode 8 is owned by the `M-28` author. Escalation: the defect is documented in the reference ecosystem, so the test is written before the aggregation rather than after it | A `M-28` scale-recovery test in which the reported scale after the tight cluster is narrower than before the outlier |
| R-07 — Absent klib dump evidence is mistaken for klib compatibility, so a binary-incompatible change to the multiplatform surface ships with no gate able to detect it and consumers on the klib floor break | 3 | 4 | 12 | G3-09 forbids any claim of klib compatibility evidence while no baseline exists; ADR-13 records the tiebreak as unavailable rather than silently unused; the consumer integration test against the minimum supported klib version is the only available signal and is exercised through G3-06. Residual: that test checks consumability at one pinned version, not compatibility across versions. Escalation: a klib-surface concern is raised as an open question rather than answered from the existing dumps | Zero files matching `*.klib.api` in the tree, combined with any statement in a unit's review that a change is klib-compatible |
| R-08 — The Java differential is credited with reach it does not have, so Apple and JavaScript pipelines are believed verified when no reference comparison ever executed for them | 3 | 4 | 12 | ADR-12 confines the differential to the JVM; Section 6.3 states both bounds explicitly; `M-24` and `M-33` supply shared behaviour, property and golden tests as the actual multiplatform evidence; `M-25` documents its own reach. Residual: the reference satisfies only 75 of 88 rows, so even on the JVM it is silent about 13. Escalation: any row cited as reference-verified where the reference is unsatisfied is a documentation defect that blocks `M-26` | Any claim of reference-verified behaviour that does not name the JVM in the same sentence, or that cites a compliance row the reference does not satisfy |
| R-11 — Host capacity below the checked-in heap premise, so compilation fails or thrashes in ways misattributed to code defects and metaspace exhaustion restarts the daemon on every invocation | 3 | 4 | 12 | G3-05 requires capacity to be measured before the first build is treated as evidence; HC-10 requires memory to be supplied rather than the heap lowered; the repository's own `gradle.properties` is never edited, and any out-of-repository override is disclosed with every figure measured under it. Residual: an asserted capacity figure can be a unit error in either direction, so the measurement is read with its unit stated. Escalation: below the floor the baseline is not run and the shortfall is reported; a zero exit code obtained under a lowered heap is reported `DIFFERS` rather than as a pass | Measured available memory below 16 GiB, or the presence of an out-of-repository properties file lowering either daemon heap |
| R-12 — Configuration-cache incompatibility in new build logic stops the build outright with an error whose text does not point at the offending logic | 3 | 4 | 12 | Spike 2 exercises exactly the fixture-staging task and the coverage-report change the plan foresees; ADR-01 elects no new Gradle project, removing the largest source of new configuration-time code; new logic is modelled on an existing module's script rather than written fresh. Residual: disabling the cache to isolate a cause is permitted but must never be left as a fix. Escalation: if reuse cannot be achieved the offending unit is reduced in scope rather than merged with a failing cache | A repeated identical invocation of `./gradlew build :benchmark-android:assemble :benchmark-android:compileReleaseAndroidTestKotlin :benchmark-jvm:assemble :examples:example-app-android:assembleRelease koverXmlReport -x :gradle-integration-test:test --stacktrace` whose log reports a fresh cache store instead of cache reuse |
| R-16 — Coverage dilution: a large unit lands with thin tests and breaches the enforced project window, failing continuous integration at the merge that closes a work package after the work is already written | 4 | 3 | 12 | G3-03 fixes the window at 1.00 percentage point and never widens it; Section 6.1.2 states the arithmetic so units are sized correctly rather than discovered late, showing that the project window rather than the 50% patch floor is the binding constraint. Residual: the two thresholds are mutually inconsistent at this baseline, since a 500-line unit at exactly the patch floor drops project coverage 3.93 points. Escalation: the exemption policy is tightened rather than the window widened, and `M-01` must first make the uploaded report the one the build produces or the gate cannot observe a regression at all | A unit whose measured patch coverage sits near the 50% floor, or a project-coverage delta beyond 1.00 percentage point in the report the gate reads |
| R-17 — Units exceed 500 changed lines or a design breaches a static-analysis ceiling, so a change cannot be reviewed meaningfully or fails the gate for a reason unrelated to correctness | 4 | 3 | 12 | Section 4 states a per-unit ceiling of 490 and decomposes below the granularity of one instrument end to end; Section 4.2 fixes the measuring pathspec so the budget cannot silently report zero; ADR-02 uses builder entry points to keep the meter interface far below the 31-function ceiling at `config/detekt/detekt.yml:L14-L18`; ADR-03 forbids a flat wide configuration constructor against the 30-parameter ceiling at L10-L11. Residual: `M-11` at 480 lines is closest to the ceiling and is split further if its tests grow. Escalation: a unit at or above 500 lines is split before review rather than granted an exception | A measured change-unit diff at or above 500 changed lines, or a `detekt` failure on a function or parameter count |
| R-20 — The four `Meter` implementors are updated in an uncoordinated wave, so the build is red between units and a bisect cannot attribute a later failure | 3 | 4 | 12 | `M-03` adds the factory members and the coordinated implementor updates in one unit rather than across four, because a member without its implementors does not compile under `allWarningsAsErrors` at `buildSrc/src/main/kotlin/io/opentelemetry/kotlin/KotlinConfig.kt:L89`; Section 5's dependency order places that unit before every consumer; G3-06 is a per-unit gate rather than a per-work-package one | Any unit whose acceptance criteria change the meter interface without naming all four implementors, or a failure citing an unimplemented member from `./gradlew build :benchmark-android:assemble :benchmark-android:compileReleaseAndroidTestKotlin :benchmark-jvm:assemble :examples:example-app-android:assembleRelease koverXmlReport -x :gradle-integration-test:test --stacktrace` |
| R-21 — Histogram bucket boundary semantics are wrong at the edges, so values land one bucket away and every percentile derived downstream is subtly wrong | 3 | 4 | 12 | `M-10` carries the boundary property test asserting exclusivity of the lower bound and inclusivity of the upper, including the positive-infinity bucket, and asserts the lowest-matching-boundary rule directly; loss mode 7 is owned by the `M-10` author; the differential covers this on the JVM because the reference satisfies the row. Escalation: a boundary failure stops `M-11`, which reuses the same bucketing contract | A `M-10` property test failing on a value exactly equal to a configured boundary, or a golden fixture whose bucket counts shift by one position |
| R-23 — View attribute filtering executes at the wrong layer, so filtered keys still reach series identity or exemplars retain keys the view removed | 3 | 4 | 12 | ADR-05 fixes the layer at which filtering executes and ADR-07 places views between materialisation and storage; `M-12` asserts that a filtered key changes series identity and that exemplar attributes are asserted separately from point attributes; loss mode 9 is owned by the `M-12` author. Residual: an absent key list means all keys are used, which must be tested as its own case rather than assumed | A `M-12` test in which two recordings differing only in a filtered key produce two series, or an exemplar carrying a key the view removed |
| R-24 — The coverage artifact path defect blocks the gate, so no coverage data reaches it and every G3-03 claim is unverifiable | 4 | 3 | 12 | `M-01` is the first unit in the sequence and repairs the mismatch between the produced report and the uploaded path; error tolerance on the upload step means the defect is silent today, which is why the repair precedes every unit that claims a coverage figure; Section 8 carries the choice between the two one-line repairs as an open question. Escalation: until `M-01` lands, no unit may cite a coverage figure as gate evidence | `ls build/reports/kover/reportRelease.xml` failing while `build/reports/kover/report.xml` exists, with the workflow still uploading the former |
| R-28 — Protobuf conversion silently loses a field, so the payload validates and exports successfully while a dimension, exemplar or bound is absent | 3 | 4 | 12 | `M-19` and `M-20` assert byte-for-byte equality against committed golden fixtures for each point type, including an exemplar-bearing point, rather than asserting field-by-field; ADR-10 reuses the generated message types so a schema change surfaces as a compile error; Section 6.2's serialisation kind owns the oracle. Escalation: a fixture is regenerated only after the protocol delta has been read and accepted explicitly | A golden-fixture mismatch in `M-19` or `M-20`, or a conversion added without a fixture covering its point type |
| R-29 — The collector fake lacks a metrics route, so end-to-end tests observe nothing and pass, and the pipeline is believed working when no request was ever matched | 3 | 4 | 12 | `M-21` adds the endpoint entry and `M-34` adds the route to the collector fake and asserts on the recorded request sequence including paths and bodies, never merely on the absence of an error; loss mode 21 is owned by the `M-22` author for the no-reader case and the `M-34` author for the no-route case | A `M-34` end-to-end test passing while the collector fake recorded zero requests on the metrics path |
| R-10 — JDK 11 is absent or present but not discoverable, so every compilation fails with a toolchain resolution error that reads as a build defect, or auto-provisioning hangs on a network lookup | 2 | 5 | 10 | G3-05 makes discoverability a gate verified by command rather than by checking whether a package is installed; the declared toolchain at `buildSrc/src/main/kotlin/io/opentelemetry/kotlin/KotlinConfig.kt:L11` is never lowered to match a host. Residual: a host installing only 17 and 21, as the continuous-integration configuration does, satisfies neither the declared toolchain nor this gate. Escalation: a missing 11 is a stop for every unit until the environment supplies it | `./gradlew -q javaToolchains` not listing an entry for 11, 17 or 21 |
| R-18 — Credential or token material leaks into a commit, a log or a document, so a published credential must be rotated and the leak persists in history | 2 | 5 | 10 | G3-11 forbids credential material in any log or document; this plan records only that such metadata was observed and excluded; remote URLs are redacted before any output is captured. Residual: verbatim command output is the evidence standard, so every captured transcript is inspected before it is quoted. Escalation: a suspected leak stops the unit and the credential is treated as compromised | Any diff, log excerpt or document line matching a credential pattern, a secret identifier name, or a rate-limit response body |
| R-13 — Protocol or configuration-schema drift under a pinned version breaks metrics marshalling, or silently changes its shape, and the cause looks like a conversion defect | 3 | 3 | 9 | ADR-10 reuses the generated messages rather than checked-in copies, so a pin bump surfaces as a compile error rather than as divergence; `M-20` golden fixtures make any encoding change visible byte-for-byte; ADR-15 fixes the rebase posture that surfaces a bump early. Escalation: a fixture is regenerated only after the delta has been read and the change accepted | A conversion compile failure or a golden-fixture mismatch in the first build following a change to the pinned protocol or schema version |
| R-26 — Configuration precedence is resolved differently in different units, so the same input yields different resolved configuration depending on which path reads it | 3 | 3 | 9 | ADR-03 and ADR-14 fix a single precedence order across all five configuration inputs and make it the subject of a named test in `M-22`; the layered behaviour model's existing tracer slot supplies the precedence rule rather than a new one being invented. Residual: the five environment variable names are an open question in Section 8, so the test asserts the order rather than the names | A `M-22` precedence test in which two inputs resolve in an order different from the one ADR-14 fixes |
| R-33 — Upstream divergence produces repeated rebase churn concentrated in the tracked dump files, consuming review effort and inviting a hand-merge | 4 | 2 | 8 | ADR-15 fixes the posture: rebase before running gates, regenerate dumps after rebasing, never hand-merge a dump, never vendor upstream code; Section 5 states the same policy as the merge gate. Residual: measured divergence is 3 commits ahead and 15 behind, so churn is expected rather than exceptional. Escalation: a hand-merged dump is reverted and regenerated rather than reviewed | A rebase producing a conflict inside any `*.api` file, or a dump diff that the generation task does not reproduce exactly |
| R-34 — An unbounded instrument registry per meter grows without limit under repeated identical registration, so memory grows in the host and duplicate-registration defects are undetectable | 2 | 4 | 8 | ADR-04 fixes identity so an identical registration returns the existing instrument rather than adding one, and ADR-18 bounds the registry with a degrade-not-throw diagnostic; `M-05` asserts that repeated identical registration yields one entry and that a conflicting one is reported exactly once. Residual: the reference implementation's source-attribution machinery for these warnings is ruled unnecessary in Section 3.1, so the diagnostic names the descriptor rather than a source location | A `M-05` test in which repeated identical registration increases the registry size, or a conflicting registration reported more than once |
| R-14 — Hand edits to tracked generated configuration models are silently overwritten, so a behaviour depending on an edited model disappears at the next regeneration and the regression appears unrelated | 2 | 3 | 6 | No unit edits a generated model; ADR-03 and ADR-07 borrow the generated names into hand-written configuration types instead; G3-11 forbids hand-editing generated artifacts. Residual: the directory is excluded from static analysis, so an edit is flagged by no gate. Escalation: a diff there blocks the unit until it is reverted or reproduced by running the generator | Any diff touching the generated configuration-model directory in a unit that did not run the generation task |
| R-25 — An exemplar reservoir retains a stale sample across a collection cycle or keeps attributes it should have dropped, so an exemplar points at the wrong measurement | 2 | 3 | 6 | `M-14` asserts reservoir reset per cycle with a seeded generator and asserts exemplar attribute retention separately from point attributes; ADR-07 fixes the reservoir kind per aggregation so the choice is not per-unit; loss mode 11 is owned by the `M-14` author | A `M-14` test in which a sample from a previous cycle appears in the current one, or an exemplar carrying a dropped attribute key |
| R-27 — The in-memory exporter diverges from the OTLP path, so tests written against it pass while the wire path behaves differently | 2 | 3 | 6 | `M-19` builds the in-memory exporter against the same data model and the same suspending sealed-result contract as the OTLP exporter, following the rigid three-file shape already used for the other signals; `M-34` asserts the end-to-end path against the collector fake rather than against the in-memory exporter, so the two are never each other's oracle | A behaviour asserted only through the in-memory exporter with no corresponding assertion in `M-34` |
| R-31 — The compliance sheet drifts from the denominator this document fixes, so two different satisfied-of-total figures circulate | 3 | 2 | 6 | `M-26` is the single sink of the dependency graph and states the denominator basis in the sheet itself; Section 2.6 fixes 81 as the basis with the identity closing twice; G3-11 requires the figure to be an integer with its basis stated and forbids unquantified terms. Escalation: a figure quoted without its basis is a documentation defect that blocks `M-26` | Any satisfied-of-total figure appearing without its denominator basis, or a sheet total that does not close against 88 rows |
| R-32 — A Kotlin Gradle plugin advisory or comparable toolchain advisory requires a version move the frozen support floors forbid, so the build cannot be remediated by upgrading alone | 2 | 3 | 6 | Section 4.4 binds the twelve security-acceptance conditions to existing gates rather than to a new one; the frozen floors are policy, so remediation is evaluated against the consumer floors before any version move; ADR-15 forbids vendoring as a workaround. Escalation: an advisory that cannot be remediated within the floors is raised as an open question rather than silently accepted | A dependency advisory naming a version at or below a frozen support floor, observed in the maintenance configuration's own reporting |
| R-19 — Deliverable placement or capture ambiguity: the specification is written where nothing collects it, or a second copy exists, so the work is complete but invisible or two revisions disagree | 1 | 4 | 4 | P3-02 records the placement ruling with both the asserted and the observed destination and rejects an out-of-repository path, an absolute leading-slash path and a new documentation directory explicitly; the document is placed at the repository root as a tracked file where every other repository-authored document lives, and `.gitignore:L1-L11` does not cover it; P3-11 records the same for the delivery-visibility question, which the placement reversal closes rather than defers. Escalation: the file is never relocated into an ignored directory to satisfy a tool; the constraint is reported instead | `git check-ignore -v metrics-implementation-plan.md` exiting 0, `git ls-files` not listing the path, or more than one file matching the name anywhere on the host |

# 8. Ranked open questions

Ranked strictly by how much downstream work each blocks, highest first — not by topic, not by difficulty. Every entry is a genuine residual: no decided architecture record is restated here as a question, and no unresolved blocker is left in prose instead of appearing in this table. The register carries **eight** questions against a ceiling of ten. It carried nine while the specification was to be placed outside every checkout; the reviewer's revocation of that instruction closed the question of whether an out-of-repository artifact would be collected at all, and P3-02 and P3-11 record that closure with evidence rather than carrying it forward here. Two of the eight are repository-policy calls this specification deliberately declines to make on maintainers' behalf, namely ranks 1 and 8. Resolution is by repository evidence or by the pinned reference clone, and by nothing else [A5|DECIDED].

| Rank | Question | Blocks | Current evidence | Smallest decision needed | Consequence of deferral |
| --- | --- | --- | --- | --- | --- |
| 1 | Is regenerating the eleven stale Android dumps in scope for this effort, or does the inherited disagreement stay isolated and untouched? | ADR-13, `M-02`, and through `M-02` every unit that changes public API, plus G3-09 | `api/api/android/api.api` contains zero occurrences of `metrics` and zero of `getMeterProvider` across 462 lines while the JVM dump carries both, and `./gradlew apiCheck detekt --stacktrace` exits 0 despite the disagreement, so the check demonstrably does not enforce the Android variant [A1\|OBSERVED] | A yes or no on whether a dump-only regeneration of the Android variants is accepted as its own change, independent of metrics | Every metrics API unit's Android dump diff mixes inherited staleness with the metrics delta, so dump review stops being evidence and G3-09 cannot be satisfied for any unit [A1\|QUESTION] |
| 2 | Do the new `Meter` members carry default bodies, and if not, is the coordinated breakage of all four implementors accepted in one wave? | ADR-02, `M-03`, `M-04`, `M-05`, and G3-08 | The contributor guidance strongly discourages default interface bodies at `CONTRIBUTING.md:L57`, yet the logging processor contract uses one at `sdk-api/src/commonMain/kotlin/io/opentelemetry/kotlin/logging/export/LogRecordProcessor.kt:L37-L42`, so the repository shows precedent both ways; all four implementors are member-less today [A3\|OBSERVED] | A ruling on whether the no-default-body rule holds for the metrics factory members, given that the affected types all carry the experimental marker and breaking them is already sanctioned | The public surface cannot be finalised, so `M-03` cannot be reviewed and the entire Stage C chain is blocked [A6\|QUESTION] |
| 3 | Which type owns the readable attribute snapshot that reaches points and exemplars? | ADR-05, `M-06`, `M-07`, `M-20` | The write-only mutator lives in `api` while the readable container exposing a string-to-any map lives in `sdk-api` at `AttributeContainer.kt:L11,L17`, so no readable attribute type exists in the API layer at all [A3\|OBSERVED] | A choice between exposing the existing SDK readable container on the metrics data types and introducing a metrics-specific readable attribute view in the SDK metrics data package | The data model in `M-06` cannot be declared, which blocks storage, conversion and every golden fixture [A3\|QUESTION] |
| 4 | What stripe count and what double-accumulation technique do the platform accumulators use? | ADR-06, `M-08`, and the lost-update oracle in `M-09` | The three existing actuals differ fundamentally: the JVM delegates to a platform atomic, the JavaScript actual is a plain variable, and the Apple actual serialises every operation on a lock; the repository states that the standard-library atomics are unavailable at the 2.0 floor and no atomics library is in the catalog [A3\|OBSERVED] | A fixed stripe count and a decision on whether double accumulation uses a bit-pattern compare-and-swap loop or a lock on the platforms that lack a double atomic | `M-08` cannot be implemented, and because every aggregation depends on it, Stage E cannot begin [A3\|QUESTION] |
| 5 | Is the base-2 exponential histogram required in the first wave, or may it follow as its own work package? | `M-11`, the aggregation subset in ADR-07, and the compliance count reported against the 81-row denominator | The aggregation is a distinct compliance row, and the reference implementation's aggregator and bucket types are its two largest aggregation files, which is why this plan sizes `M-11` at 480 lines and places it after the explicit-bucket histogram [A4\|OBSERVED] | A yes or no on whether the first wave's declared scope includes it | The compliance figure reported at each work-package boundary is ambiguous, and `M-11`'s position in the sequence cannot be fixed [A4\|QUESTION] |
| 6 | Does an asynchronous instrument gain an after-creation callback registration member, and if so with what shape? | ADR-08, `M-04`, `M-17` | `AsynchronousInstrument.kt:L19` marks the member as still to be added and L11-L12 documents only that closing unregisters the callbacks, so the intended shape was deliberately left open in source [A3\|OBSERVED] | A ruling on whether registration is creation-only in this wave, which is what this plan assumes, or whether the member lands with it | If the member is added later it is a second breaking change to an experimental surface, and the collection contract in `M-17` must be respecified rather than extended [A3\|QUESTION] |
| 7 | Which five environment variables are the agreed metrics set, and is a meter-provider slot added to the layered behaviour model? | `M-22`, and the configuration surface in ADR-03 | Exactly eight environment variables are recognised and every one is a limit, so any exporter or reader variable is the first non-limit variable in the repository; the behaviour model carries only a tracer-provider slot at `OpenTelemetryBehavior.kt:L11-L18`, so a meter slot is greenfield [A3\|OBSERVED] | Confirmation of the five variable names and a yes or no on the behaviour slot | `M-22` cannot be specified, and configuration reaches the provider only programmatically, which leaves a compliance area unaddressed rather than declared out of scope [A3\|QUESTION] |
| 8 | How is the enforced coverage gate repaired: by changing the uploaded artifact path, or by producing a report at the path already uploaded? | `M-01`, and G3-03 for every later unit | The build produces `build/reports/kover/report.xml` and does not produce `build/reports/kover/reportRelease.xml`, which is the path the workflow uploads with error tolerance enabled, so no coverage data reaches the gate today [A2\|OBSERVED] | A choice between the two one-line repairs | The 1.00 percentage-point window is unenforced, so coverage dilution is undetected until it is large, and every unit's G3-03 claim is unverifiable [A2\|QUESTION] |

## 8.1 How a question closes, and this document's own self-check

A ranked question closes by **becoming a record**, never by being deleted. When one is answered, the answer is written into the architecture record
that owns it — or into a new record if none does — the record cites the question number it closed, and the count in the preamble above is restated so
that a reader can tell a closure from an omission by arithmetic alone. A question removed without a record is indistinguishable from a question
forgotten, which is the failure this protocol exists to prevent. The one closure already applied is stated in the preamble and evidenced in D11,
P3-02 and P3-11: the reviewing maintainer revoked the placement instruction, which closed the question of whether an artifact written outside every
checkout would be collected at all [A5|DECIDED].

The self-check below is the document's own, and every line of it is a command a reader can run or a property a reader can count, not an assertion of
diligence [A2|OBSERVED].

| Condition | How it is checked | Observed |
| --- | --- | --- |
| Exactly one deliverable file exists, and it is this document | `git ls-files \| grep -c metrics-implementation-plan` returns 1; a host-wide `find / -xdev -name '*metrics-implementation-plan*'` returns only the repository-root path | Satisfied |
| It lives at the repository root of the working checkout and is **tracked** | `git ls-files metrics-implementation-plan.md` names it; `git diff --numstat` against the branch base lists it with a non-zero added-line count | Satisfied |
| It is not ignored, and no rule may make it so | `git check-ignore -v metrics-implementation-plan.md` produces no output and exits 1; `.gitignore:L1-L11` contains no pattern covering it | Satisfied |
| No copy of this document's body exists anywhere else | No second file of this name on the host; no `.bak`, `.orig`, numbered variant, or `docs/` duplicate; working notes held outside the repository are notes, not copies | Satisfied |
| The instruction to place the deliverable outside every checkout is recorded as **revoked**, not obeyed and not silently dropped | D11 records the reversal with both the asserted and the observed destination; P3-02 and P3-11 carry it in the ledger; no section states that the deliverable lives outside a checkout | Satisfied |
| No other repository file is modified, renamed or deleted | `git status --porcelain` shows one added path and nothing else; `git diff --stat` against the branch head is empty apart from this file | Satisfied |
| Exactly eight top-level sections, in the fixed order, with no ninth | Count the `# n.` headings: executive summary, baseline ledger, architecture decisions, work breakdown, dependency plan, correctness strategy, risk register, open questions | Satisfied |
| No preamble, no restatement of the request, no accompanying code | The document opens directly on `# 1. Executive summary`; no source file accompanies it; the existence check's result is reported to the reviewer rather than embedded here | Satisfied |
| The executive summary is under 200 words and still carries the compliance figure with its basis | Mechanical word count of the section body; the compliance string appears there once and byte-identically | Satisfied — 197 words |
| The work-breakdown header is byte-exact and appears once, with no duration, date, owner, effort, sprint or deadline column | String-match the frozen header including the `≤` glyph; no schedule term appears as a schedule term anywhere in the document | Satisfied |
| The risk-register header is byte-exact and appears once | String-match the frozen six-column header | Satisfied |
| Change units are contiguous `M-01` through `M-35`, unique, none missing or duplicated, each at or below 500 estimated added lines | Parse the table; check the identifier sequence and each estimate | Satisfied — 35 units, maximum 490 |
| Architecture records number at least eighteen, `ADR-01` through `ADR-13` are not renumbered, and each carries all seven fields | Count `## ADR-` headings and check each record's fields | Satisfied — 18 records |
| The risk register carries 34 rows, `composite` equals `likelihood × impact` in every row, and rows descend by `composite` | Recompute the product per row and check the ordering | Satisfied |
| Open questions number at most ten and are ranked by downstream blocking | Count the rows and check the rank sequence | Satisfied — 8 questions |
| Every internal cross-reference resolves | Collect every `M-`, `ADR-`, `G3-`, `HC-`, `P3-`, `R-`, `T-` and `WP-` reference and check each against its definition site | Satisfied — no dangling reference |
| The canonical build command is written out in full at every reference site and never abbreviated | Search for the full command and for any ellipsis or truncation adjacent to `./gradlew` | Satisfied |
| No credential, token or secret value appears | Search for credential patterns, authorization headers, bearer tokens and credential-bearing URLs; the one remote URL carrying embedded credential material is transcribed as `<redacted>` | Satisfied |
| Both verification spikes are deleted, and each was deleted only after its conclusion was recorded | Section 5.3 records each spike's conclusion and the record that owns it; no spike branch ref survives | Satisfied |
| Sections 3 through 7 are in the voice of specification rather than a report on work performed | Section 2 is the only section that reports; the others state what a unit adds and which gates bound it | Satisfied |

Two conditions above are worth calling out because they were **not** satisfied by the revision this document supersedes, and correcting them is why
that revision could not simply be reinstated. Its work-breakdown and risk-register headers were not the frozen ones, so the structural evidence for
HC-06 and HC-11 was absent; and its placement rows asserted a destination outside every checkout, which the reviewing maintainer has revoked. Both
are now satisfied by observation rather than by intent [A5|DECIDED].
