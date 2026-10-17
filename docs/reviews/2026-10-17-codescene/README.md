# RepoWise: CodeScene assessment and source cross-check

Prepared 17 October 2026. This is a technical assessment for Daniel Webb, whose organisation is a CodeScene partner. It is not an independent head-to-head product trial.

**Main finding.** RepoWise has substantial technical debt concentrated in frequently changed, first-party product code. The most consequential examples sit in index updating, parsing, persistence and dead-code analysis. The evidence supports prioritising those areas. It does not establish that the whole repository is unhealthy, that every CodeScene finding is a functional bug, or that RepoWise's benchmark is invalid.

**1. Scope, provenance and method**

The local checkout and the existing CodeScene cloud analysis both identify revision **c9bbf7d4e2ed3f352bc4da0d6a5fdce561179819**. The cloud job is **7997331**, project **86250**, analysed on **17 October 2026 at 12:18:47 BST**. Its historical scope starts on 23 March 2026. The analysed remote is https://github.com/dhansak79/repowise.git.

[Verify the cloud scope and revision](https://codescene.io/projects/86250/jobs/7997331/results/scope/analysis-data/analysis-scope). The cloud rule-override view reports [no overridden Code Health rules](https://codescene.io/projects/86250/jobs/7997331/results/scope/analysis-data/overridden-code-health-rules).

The 19 reviewed source files are also byte-identical at upstream main revision adaf956dd503237b1fe4377c697b363bdb9ed995, which is the base of the accompanying PR. Repository-wide cloud metrics remain pinned to the earlier analysed revision. CodeScene project links may require access; the raw evidence is included so readers do not need access to Daniel's account.

The primary evidence comes from the connected CodeScene MCP server:
- project metrics and prioritised technical-debt hotspots;
- 19 file-level Code Health reviews;
- four file-level refactoring business cases;
- the project's configured goals and one file-specific hotspot lookup;
- Code Health and productivity explanation tools.

The source checks use Python AST parsing, direct inspection and local git history. Supplemental browser inspection verified the cloud revision, history window and rule-override state. The application test suite was not executed. No product source was changed.

The 19 files were selected because CodeScene prioritised them, they lie on adjacent critical product paths, or they provide useful healthy comparisons. This is a **risk-directed sample, not a random sample**. Its health distribution must not be extrapolated to the whole repository.

**2. What the packages directory contains**

The packages directory is RepoWise's monorepo source, not a directory of downloaded dependencies. The root [package.json](https://github.com/repowise-dev/repowise/blob/c9bbf7d4e2ed3f352bc4da0d6a5fdce561179819/package.json) declares the npm workspaces; the root [pyproject.toml](https://github.com/repowise-dev/repowise/blob/c9bbf7d4e2ed3f352bc4da0d6a5fdce561179819/pyproject.toml#L148) maps the Python source roots into the distributed repowise package. [The core package README](https://github.com/repowise-dev/repowise/blob/c9bbf7d4e2ed3f352bc4da0d6a5fdce561179819/packages/core/README.md) explicitly describes core as RepoWise's engine.

| Directory | First-party responsibility | Treatment |
|---|---|---|
| packages/core/src | Parsing, dependency graph, analysis, generation and persistence | In scope |
| packages/cli/src | RepoWise commands and local workflows | In scope |
| packages/server/src | HTTP API and MCP tools | In scope |
| packages/ui/src | Shared interactive UI | In scope |
| packages/web/src | Web application | In scope |
| packages/types/src | Shared type definitions | Hand-written definitions in scope; generated types treated separately |
| packages/api-client/src | RepoWise's own client library | In scope even though other workspace packages depend on it |
| packages/vscode/src | RepoWise's editor extension | In scope |
| packages/claude-mod/src | RepoWise's agent integration | In scope |
| node_modules, .venv, vendor/third_party paths | Installed or vendored external code | Exclude from first-party maintainability conclusions |
| tests, fixtures and generated/build output | Different ownership and maintenance characteristics | Separate from production-code conclusions |

A local scope inventory selected **3,059 tracked source files and 695,783 physical lines** under packages/*/src after recognised tests, fixtures, dependency, vendor, build and generated paths/headers were excluded. This is a directory/header heuristic, not a provenance audit of every line. These are physical source lines, not CodeScene's non-comment line metric.

Four generated paths were excluded explicitly, including the 4,470-line generated HTTP contract. The inventory and exclusions are in first-party-scope.json. None of the 19 reviewed files is one of those exclusions.

**The existing cloud average was not recomputed using this narrower inventory.** Treat the cloud KPIs as results for the configured project, not as a newly calculated production-only aggregate. The strongest conclusions below rely on named, verified first-party files, rather than repository-wide percentages.

**3. Repository-level picture**

The MCP project response reports:
- overall Code Health **8.5969**, up from **8.55** at the month reference;
- hotspot Code Health **4.7213**, up from **4.54**;
- an overall-versus-hotspot gap of about **3.88 points**.

That combination is more informative than either number alone: the code being changed most frequently is substantially less healthy than the aggregate. The month comparisons show modest improvement, not uniform deterioration.

CodeScene's prioritisation flags four recommended refactoring targets. Their computed technical-debt friction has increased against the month reference, despite the small improvement in the aggregate hotspot KPI. These metrics describe different populations and calculations, so the findings are not contradictory.

| Priority target | Cloud health | Local MCP health | Cloud revisions | Cloud LoC | Friction: month reference → current |
|---|---:|---:|---:|---:|---:|
| [CLI update](https://github.com/repowise-dev/repowise/blob/c9bbf7d4e2ed3f352bc4da0d6a5fdce561179819/packages/cli/src/repowise/cli/commands/update_cmd/command.py) | 2.23 | 2.32 | 141 | 2,030 | 0.749 → 0.792 |
| [Analysis persistence](https://github.com/repowise-dev/repowise/blob/c9bbf7d4e2ed3f352bc4da0d6a5fdce561179819/packages/core/src/repowise/core/pipeline/persist.py) | 3.27 | 3.33 | 115 | 2,091 | 0.401 → 0.561 |
| [Dead-code analyser](https://github.com/repowise-dev/repowise/blob/c9bbf7d4e2ed3f352bc4da0d6a5fdce561179819/packages/core/src/repowise/core/analysis/dead_code/analyzer.py) | 3.45 | 3.44 | 112 | 1,747 | 0.430 → 0.531 |
| [Source parser](https://github.com/repowise-dev/repowise/blob/c9bbf7d4e2ed3f352bc4da0d6a5fdce561179819/packages/core/src/repowise/core/ingestion/parser.py) | 2.56 | 2.56 | 105 | 1,802 | 0.482 → 0.566 |

The 105–141 revision counts are recorded in the analysed historical scope; they are not counts for the last month. Friction is reported here as CodeScene's raw index. It is **not a percentage of developer time or money**, and its changes must not be described as measured cost increases.

Cloud and local scores differ on the same clean revision, e.g. update 2.23 versus 2.32. The exact engine/configuration reason was not established. Keep the two columns separate; do not describe their differences as improvements or regressions over time. Business cases below use local scores.

[Open the prioritised hotspot analysis](https://codescene.io/projects/86250/jobs/7997331/results/code/technical-debt/system-map?max-code-health=10.00&min-change-freq=0&showHotspotsOnly=true&min-coverage=0.00&max-coverage=100.00#hotspots).

**4. CLI update: the clearest concentration of responsibilities**

[update_cmd/command.py](https://github.com/repowise-dev/repowise/blob/c9bbf7d4e2ed3f352bc4da0d6a5fdce561179819/packages/cli/src/repowise/cli/commands/update_cmd/command.py#L708) scores **2.32** locally and **2.23** in the cloud. CodeScene identifies a Brain Method, 26 bumpy-road regions, four levels of conditional nesting and excessive arguments.

Python's AST confirms that run_update spans **lines 708–2780: 2,073 physical lines**, with **21 parameters**. The file is 2,780 physical lines long, so this single function occupies about 75% of its physical span. Physical span includes comments and blank lines; it is not executable LoC.

The source combines workspace/single-repository dispatch, progress and console setup, linked-worktree seeding, upgrade paths, configuration and version checks, update detection, index refresh, persistence coordination, generated output and state/outcome handling. There are helpers, but much of the decision flow remains centralised.

**Why this matters to the product.** Update correctness affects whether the context given to developers and agents is current and consistent. Changes to one workflow require reviewers to reason about adjacent modes, early exits, persisted stamps and error/degraded outcomes. The engineering concern is the difficulty of making safe changes to the central update path, not an allegation that its current output is necessarily wrong.

Local git history from 17 September through the pinned 16 October HEAD contains **28 non-merge commits touching this file, 16 with a fix prefix**. Several subjects concern stale renderers, security rescanning, git history and index stamps. These are maintenance-activity indicators, not independent confirmed-defect counts.

**Practical intervention.** Separate command options and presentation from update planning and execution. Establish a typed plan for no-op, index refresh, regeneration and upgrade decisions. Extract one phase at a time behind explicit inputs, outputs and degraded-state rules. Verify each extraction against the existing observable outcomes and state stamps. Merely moving the function to another file would not resolve the concentration.

**5. Source parsing: complex logic at the start of every downstream inference**

[ingestion/parser.py](https://github.com/repowise-dev/repowise/blob/c9bbf7d4e2ed3f352bc4da0d6a5fdce561179819/packages/core/src/repowise/core/ingestion/parser.py#L1502) scores **2.56** locally. CodeScene reports 18 Complex Method findings and 14 bumpy-road function findings. Representative examples:
- _extract_references: cyclomatic complexity **23**, [line 2206](https://github.com/repowise-dev/repowise/blob/c9bbf7d4e2ed3f352bc4da0d6a5fdce561179819/packages/core/src/repowise/core/ingestion/parser.py#L2206);
- _call_receiver_from_node: complexity **23**, [line 192](https://github.com/repowise-dev/repowise/blob/c9bbf7d4e2ed3f352bc4da0d6a5fdce561179819/packages/core/src/repowise/core/ingestion/parser.py#L192);
- _extract_calls: complexity **20**, [line 2091](https://github.com/repowise-dev/repowise/blob/c9bbf7d4e2ed3f352bc4da0d6a5fdce561179819/packages/core/src/repowise/core/ingestion/parser.py#L2091);
- _refine_symbol_kind: complexity **20**, [line 769](https://github.com/repowise-dev/repowise/blob/c9bbf7d4e2ed3f352bc4da0d6a5fdce561179819/packages/core/src/repowise/core/ingestion/parser.py#L769);
- parse_file: complexity **15**, [line 1544](https://github.com/repowise-dev/repowise/blob/c9bbf7d4e2ed3f352bc4da0d6a5fdce561179819/packages/core/src/repowise/core/ingestion/parser.py#L1544).

The file spans **2,417 physical lines**. parse_file spans 149; _symbol_from_match 122; _extract_calls 114. Language-specific conventions and shared extraction logic coexist in the module.

**Why this matters to the product.** Parsed symbols, imports, references and receivers feed the graph, change-impact and dead-code layers. A parser change can affect several downstream features. That is an architectural dependency argument, not proof of a specific current parsing defect.

The source comments around _extract_references explain why references without direct calls matter: callbacks, dispatch tables and registrations need to be retained as uses. This is a genuine domain challenge. Language breadth creates necessary complexity; the refactoring opportunity is to isolate language policies from the shared extraction contract.

The recent window contains **42 non-merge touching commits, 36 fix-prefixed**. The subjects describe overload resolution, returned receiver types, constructors, attributes and registrations. This supports treating the module as an active maintenance hotspot, while also showing that the team is addressing edge cases.

**Practical intervention.** Define normalised extraction facts and dialect adapters, with shared resolution inputs. Characterise representative registration, alias, overload, namespace and callback cases before moving logic. Retain cross-language regression fixtures and shared behaviour checks. Do not simplify away language-specific semantics just to lower a metric.

**6. Persistence: synchronisation and lifecycle rules concentrated in shared writers**

[pipeline/persist.py](https://github.com/repowise-dev/repowise/blob/c9bbf7d4e2ed3f352bc4da0d6a5fdce561179819/packages/core/src/repowise/core/pipeline/persist.py#L2304) scores **3.33** locally. It spans **2,738 physical lines** and contains 58 function definitions under the AST inventory. CodeScene flags Low Cohesion, nested/bumpy logic, large methods and multiple complex methods.

Representative findings:
- persist_analysis: complexity **34**, five bumps, four-level conditional nesting;
- sweep_superseded_generated_pages: complexity **25**;
- persist_ingestion: complexity **18**;
- prune_deleted_file_rows and _prune_stale_file_rows: complexity **14** each.

persist_analysis spans 247 physical lines; prune_deleted_file_rows spans 174. The module covers multiple kinds of stored state and lifecycle cleanup. Source comments explain ordering-sensitive operations: restoring records before purging retired proposals, pruning stale rows and retiring generated pages.

**Why this matters to the product.** Index contents need to agree with the source and with each other after full and incremental runs. Errors in ordering or lifecycle handling could leave stale evidence, inconsistent views or missing results. The observations establish change risk and testing needs, not an observed current data-loss defect.

The recent window contains **42 touching commits, 19 fix-prefixed**. This is an active shared subsystem rather than a large dormant file.

**Practical intervention.** Split responsibility by persisted domain and lifecycle operation, while preserving the transaction and ordering contract in a small coordinator. Make incremental versus full replacement explicit. Verify idempotence, repository scoping, deletion and preservation of untouched rows. The existing tests already provide useful starting invariants.

**7. Dead-code analysis: a high-consequence decision deserves simpler reasoning**

[dead_code/analyzer.py](https://github.com/repowise-dev/repowise/blob/c9bbf7d4e2ed3f352bc4da0d6a5fdce561179819/packages/core/src/repowise/core/analysis/dead_code/analyzer.py#L868) scores **3.44** locally. It spans **2,410 physical lines**. CodeScene reports Low Cohesion, bumpy logic and nesting.

The [_get_unsatisfied_prop_guard method](https://github.com/repowise-dev/repowise/blob/c9bbf7d4e2ed3f352bc4da0d6a5fdce561179819/packages/core/src/repowise/core/analysis/dead_code/analyzer.py#L1823) has complexity **26** and five-level conditional nesting. It recursively follows graph predecessors, examines supplied props, caches parsed guards, tracks visited nodes and memoises results.

The [analysis pipeline](https://github.com/repowise-dev/repowise/blob/c9bbf7d4e2ed3f352bc4da0d6a5fdce561179819/packages/core/src/repowise/core/analysis/dead_code/analyzer.py#L1003) applies an ordered sequence of detection, exclusions, demotions, confidence adjustments and deletion gating. Several comments document why one step has to run before another.

**Why this matters to the product.** A false positive could encourage removal of used code; a false negative leaves dead code in place. Readability of the evidence and confidence pipeline matters for review and regression prevention.

There are also concrete safeguards: unindexed importer handling, demotions for public API/own-file use, confidence thresholds and a deletion gate. The review therefore does not support describing the analyser as indiscriminately recommending deletion.

The recent window contains **43 touching commits, 36 fix-prefixed**. Subjects address framework-loaded code, operator contracts, entry roots and alternative uses. These are evidence of active maintenance of difficult semantics; the commit labels do not establish how many user-visible incidents occurred.

**Practical intervention.** Represent guard resolution and use evidence explicitly, isolate language/framework policies, and retain tests showing that one valid use path keeps a symbol alive. Test the order of confidence and exclusion passes before extracting them into smaller stages.

**8. Adjacent risks beyond the four priority targets**

The extended review found the same pattern in other core paths:

| Area | Local score | Specific observation | Product relevance |
|---|---:|---|---|
| CLI initialisation | 2.68 | init_command spans 1,233 physical lines, 40 parameters, 18 bumps and four-level nesting | First successful index and user onboarding |
| Incremental pipeline | 3.90 | persist_incremental_index: complexity 56, 22 parameters, 481 physical lines | Keeping the persisted index aligned with changed/deleted source |
| CLI update persistence | 4.21 | _persist_full_update_async: complexity 98, 25 parameters, 705 physical lines | Coordinating full-update persistence and degraded outcomes |
| MCP context targets | 3.10 | _resolve_one_target: Brain Method, 26 bumps; AST span 1,027 physical lines | Resolving the evidence/context delivered to agents |
| MCP search | 5.22 | _structured_search: complexity 57, 10 parameters, 198 physical lines | Search modes, filtering and result selection |
| Call resolver | 4.10 | Many functions, repeated branching and complex language-specific resolution paths | Reliability of graph relationships |
| Health engine | 6.49 | _evaluate_file: complexity 21 and 17 explicit arguments excluding self | Reliability and extensibility of the health-analysis layer |
| Pipeline orchestration | 6.90 | run_pipeline: 26 parameters; AST span 816 physical lines | Coordination of ingestion, analysis and generation |
| Shared graph UI | 3.97 | GraphFlow: reported complexity 304; physical span 1,457 lines | Maintaining the interactive graph's states and interactions |

The frontend file is **packages/ui/src/graph/graph-flow.tsx**, not the much smaller web wrapper of the same name. That distinction matters: the wrapper scores 9.20. A large React component's complexity metric can include callbacks and rendering branches; 304 is not a bug count or evidence that 304 test cases are sufficient.

These files were selected for inspection, not all prioritised by the cloud hotspot algorithm. In particular, a poor local score alone does not justify placing an adjacent file ahead of a higher-activity target.

**9. Source and change-pattern cross-checks**

| File | Non-merge touching commits in recent window | Fix-prefixed touching commits |
|---|---:|---:|
| CLI update command | 28 | 16 |
| Analysis persistence | 42 | 19 |
| Dead-code analyser | 43 | 36 |
| Source parser | 42 | 36 |
| CLI initialisation | 16 | 8 |
| Incremental pipeline | 37 | 14 |
| CLI update persistence | 42 | 21 |
| MCP context targets | 24 | 12 |
| MCP search | 20 | 5 |
| Pipeline orchestration | 10 | 4 |

The explicit recent window is 17 September 2026 through the pinned HEAD, dated 16 October 2026. Counts are non-merge commits touching the exact path and do not follow renames. A fix prefix means the subject starts with fix: or fix(...). It is a noisy maintenance proxy, not a curated defect label.

Across the four cloud-prioritised files there are **143 unique touching commits**, **99 fix-prefixed**. These union counts remove duplicates between files; adding file-level counts would double-count multi-file changes.

Update-command changes co-occur with CLI update-persistence changes in **11 of its 28 recent commits (39.3%)**, with initialisation in six and with incremental.py in six. This is a direct local git calculation, not the cloud coupling algorithm. Shared commits can reflect legitimate cross-cutting work; they indicate coordination scope rather than prove architectural failure.

**10. Business context and modelled opportunity**

The business-case MCP tool proposes an **industry-average target of 5.15** for all four files. That is an intermediate target, not green/optimal health. Its returned ranges are:

| File | Current → target local health | Modelled reduction in implementation time | Modelled reduction in defects |
|---|---:|---:|---:|
| CLI update | 2.32 → 5.15 | 5.0–26.2% | 34.4–60.7% |
| Analysis persistence | 3.33 → 5.15 | 2.8–17.1% | 18.3–40.9% |
| Dead-code analyser | 3.44 → 5.15 | 2.6–16.0% | 16.7–38.5% |
| Source parser | 2.56 → 5.15 | 4.4–24.2% | 30.5–56.6% |

The MCP reports these as **90% confidence intervals**. The raw responses encode reductions as negative percentage changes. This table presents positive reduction magnitudes, sorted numerically; it does not rely on the response's optimistic/pessimistic labels, whose ordering is not intuitive for reductions.

These projections are useful for discussing where refactoring could pay off. They are not measured RepoWise savings, guaranteed causal effects, forecasts of actual incident counts or independently validated ROI estimates. The analysis has no RepoWise issue-cycle-time, engineering-hour, customer-incident or salary data. The percentages must not be added across files or applied to the whole team.

For a worked illustration only: if 100 hours of comparable future implementation work were spent in the update-command file, a 5.0–26.2% time reduction would correspond to about 5–26 hours. The 100 hours is hypothetical. No payback period can be calculated without estimating refactoring effort and observing future work.

[Code Red](https://arxiv.org/abs/2203.04374) studied 39 proprietary codebases and found associations between unhealthy code, defects and longer issue-resolution times. That provides population-level context; it does not establish that RepoWise experiences those same multipliers.

**11. Healthy areas and existing safeguards**

The assessment also found healthy, first-party code:
- health/ranking.py: **10.00**, no findings;
- health/biomarkers/registry.py: **10.00**, no findings;
- health/grading.py: **9.84**;
- persistence/models.py: **9.68**;
- the web graph wrapper: **9.20**.

The 2,930-line persistence/models.py is the largest Python file in the local package-source inventory yet scores 9.68. Its largely declarative nature differs from branch-heavy orchestration. File size is useful context, not a sufficient explanation for unhealthy code.

There are substantial visible testing and CI practices:
- Python provider/unit tests and a 3.11/3.12/3.13 matrix;
- coverage collection on the Python 3.12 unit-test leg;
- a tree-sitter minimum-version query-compilation check;
- Python lint, frontend type checks and tests, extension checks and documentation checks;
- integration tests configured for push events;
- source-level regression tests for init/update graph convergence, stale-row pruning, moved-symbol persistence and JSX guard use paths.

Examples: [CI configuration](https://github.com/repowise-dev/repowise/blob/c9bbf7d4e2ed3f352bc4da0d6a5fdce561179819/.github/workflows/ci.yml), [init/update convergence tests](https://github.com/repowise-dev/repowise/blob/c9bbf7d4e2ed3f352bc4da0d6a5fdce561179819/tests/unit/pipeline/test_init_update_graph_convergence.py), [persistence pruning tests](https://github.com/repowise-dev/repowise/blob/c9bbf7d4e2ed3f352bc4da0d6a5fdce561179819/tests/unit/persistence/test_persist_prune.py), [incremental symbol tests](https://github.com/repowise-dev/repowise/blob/c9bbf7d4e2ed3f352bc4da0d6a5fdce561179819/tests/unit/persistence/test_incremental_symbol_persist.py), [JSX guard tests](https://github.com/repowise-dev/repowise/blob/c9bbf7d4e2ed3f352bc4da0d6a5fdce561179819/tests/unit/dead_code/test_jsx_prop_guard_dead_code.py).

These files were inspected, not executed in this assessment. Their existence does not establish coverage percentages or passing status. No Code Health gate was found in the checked-in workflow text, but externally configured GitHub Apps, branch protections and private workflows were not audited.

The empty configured-goals response applies to **Daniel's analysis project**. It is not evidence that RepoWise's maintainers have no technical-debt goals or do not use their own tool.

**12. What this establishes about the README comparison**

The README prominently states a **2.3×** defect-finding comparison with CodeScene. The detailed experiment evaluates file rankings against later bug-fix-bearing files at a source-line inspection budget. A prior reproduction of the published paired rows selected 65 versus 28 of 376 labelled files, giving about 2.32×. That is a narrower operational result than demonstrating diagnosis of functional bugs.

The comparison harness uses per-file CodeScene CLI scores. It does not compare against CodeScene's complete hotspot prioritisation. The published AUC comparison is 0.731 versus 0.705 with p=0.054; the p=0.003 claim concerns effort-aware ranking/recall.

The calibration and comparison reports also need a clear account of training/evaluation separation: the main report describes using the corpus to calibrate weights, while the comparison report says neither score was tuned to these labels. Historical feature measurement and genuinely held-out prediction are separate requirements. This is a request for clarification, not a finding that the reported p-value is fabricated. The assessment did not independently rerun weight fitting or validate the full inferential procedure.

These are questions about the scope and wording of the claim. The maintainability findings above are a separate body of evidence. A team may knowingly defer debt, and a tool can find more than its authors choose to fix. Poor health in the tool's own code does not mathematically invalidate a ranking benchmark.

The technically useful challenge is: **what does RepoWise itself report for these same first-party hotspots, how does it prioritise them, and what happened when its recommendations were followed?** This assessment did not run a RepoWise self-analysis, so agreement or disagreement with its own scores remains unmeasured.

[README at the analysed commit](https://github.com/repowise-dev/repowise/blob/c9bbf7d4e2ed3f352bc4da0d6a5fdce561179819/README.md#L23). [Published comparison report](https://github.com/repowise-dev/repowise-bench/blob/0fb3c03aab35fb67dc72540fd6a70e2c2c599878/health-defect/COMPARISON_REPORT.md). [Metric implementation](https://github.com/repowise-dev/repowise-bench/blob/0fb3c03aab35fb67dc72540fd6a70e2c2c599878/health-defect/lib/stats.py). [CodeScene prioritisation](https://docs.enterprise.codescene.io/latest/guides/technical/hotspots.html).

**13. Recommended engineering sequence**

| Order | Work | Reason to prioritise it | Evidence of improvement |
|---|---|---|---|
| 1 | Characterise update outcomes and extract its decision/phase boundaries | Lowest health, highest recorded revisions among the four targets; central user workflow | Stable outcomes/stamps, smaller responsibilities, improved health |
| 2 | Simplify update and incremental persistence contracts | Several long, high-argument and high-complexity coordinators affect index consistency | Idempotence and full/incremental parity checks; unchanged-row preservation |
| 3 | Isolate parsing dialect policy from shared extraction | High maintenance activity and broad downstream influence | Cross-language extraction/resolution fixtures remain correct; complexity falls |
| 4 | Separate dead-code evidence, confidence and deletion decisions | Recommendations can lead to deletion; ordered safety rules need clarity | Used-code counterexamples remain protected; evidence stays explainable |
| 5 | Split shared graph interaction/rendering responsibilities | A single large React component accumulates many modes and callbacks | Focused interaction tests, stable controlled/uncontrolled behaviour |
| 6 | Make health-delta checking visible in the chosen development workflow | Prevent new complexity from outpacing improvements | New changes do not worsen targeted structural findings; measured trend over time |

A practical additional safeguard would be to trial **CodeScene MCP** on these paths: use code_health_review to establish a baseline, pre_commit_code_health_safeguard for uncommitted/staged changes, and analyze_change_set against the PR base. Begin by preventing new regressions and improving selected hotspots incrementally, rather than demanding a repository-wide cleanup before any feature can ship. This is a recommendation; this PR does not install CodeScene, connect an account or change CI policy.

The contributing guide already encourages RepoWise-on-RepoWise analysis and regression tests. That existing practice is acknowledged. An additional independent Code Health signal is proposed, not an assumption that the maintainers have no quality tools.

These are suggested priorities, not estimated schedules. Start with a limited extraction and the relevant regression checks rather than a multi-subsystem rewrite. Lowering a score by hiding logic in generic helpers or changing thresholds would not demonstrate improved comprehension.

To test the business case, record review time, rework, representative task completion and agent usage before and after the same kind of work. Pair structural improvements with correctness checks. Token savings alone do not show whether a change is safer or faster for the engineer.

**14. Complete local MCP sample**

The thresholds below use the MCP's stated interpretation: below 4 severe debt, 4 to below 9 problematic/below green, 9+ healthy, 10 optimal. Retain raw scores near boundaries.

| Reviewed file | Local score | Interpretation | First reported categories |
|---|---:|---|---|
| [CLI update](https://github.com/repowise-dev/repowise/blob/c9bbf7d4e2ed3f352bc4da0d6a5fdce561179819/packages/cli/src/repowise/cli/commands/update_cmd/command.py) | 2.32 | Severe debt | Lines of Code in a Single File; Brain Method; Bumpy Road Ahead |
| [Source parser](https://github.com/repowise-dev/repowise/blob/c9bbf7d4e2ed3f352bc4da0d6a5fdce561179819/packages/core/src/repowise/core/ingestion/parser.py) | 2.56 | Severe debt | Lines of Code in a Single File; Bumpy Road Ahead; Overall Code Complexity |
| [CLI initialisation](https://github.com/repowise-dev/repowise/blob/c9bbf7d4e2ed3f352bc4da0d6a5fdce561179819/packages/cli/src/repowise/cli/commands/init_cmd/command.py) | 2.68 | Severe debt | Lines of Code in a Single File; Brain Method; Bumpy Road Ahead |
| [MCP context targets](https://github.com/repowise-dev/repowise/blob/c9bbf7d4e2ed3f352bc4da0d6a5fdce561179819/packages/server/src/repowise/server/mcp_server/tool_context/targets.py) | 3.10 | Severe debt | Lines of Code in a Single File; Brain Method; Bumpy Road Ahead |
| [Analysis persistence](https://github.com/repowise-dev/repowise/blob/c9bbf7d4e2ed3f352bc4da0d6a5fdce561179819/packages/core/src/repowise/core/pipeline/persist.py) | 3.33 | Severe debt | Low Cohesion; Bumpy Road Ahead; Deep, Nested Complexity |
| [Dead-code analyser](https://github.com/repowise-dev/repowise/blob/c9bbf7d4e2ed3f352bc4da0d6a5fdce561179819/packages/core/src/repowise/core/analysis/dead_code/analyzer.py) | 3.44 | Severe debt | Low Cohesion; Bumpy Road Ahead; Deep, Nested Complexity |
| [Incremental pipeline](https://github.com/repowise-dev/repowise/blob/c9bbf7d4e2ed3f352bc4da0d6a5fdce561179819/packages/core/src/repowise/core/pipeline/incremental.py) | 3.90 | Severe debt | Low Cohesion; Bumpy Road Ahead; Overall Code Complexity |
| [Shared graph component](https://github.com/repowise-dev/repowise/blob/c9bbf7d4e2ed3f352bc4da0d6a5fdce561179819/packages/ui/src/graph/graph-flow.tsx) | 3.97 | Severe debt | Lines of Code in a Single File; Bumpy Road Ahead; Complex Method |
| [Call resolver](https://github.com/repowise-dev/repowise/blob/c9bbf7d4e2ed3f352bc4da0d6a5fdce561179819/packages/core/src/repowise/core/ingestion/call_resolver.py) | 4.10 | Below green threshold | Number of Functions in a Single Module; Bumpy Road Ahead; Overall Code Complexity |
| [CLI update persistence](https://github.com/repowise-dev/repowise/blob/c9bbf7d4e2ed3f352bc4da0d6a5fdce561179819/packages/cli/src/repowise/cli/commands/update_cmd/persistence.py) | 4.21 | Below green threshold | Low Cohesion; Bumpy Road Ahead; Overall Code Complexity |
| [MCP search](https://github.com/repowise-dev/repowise/blob/c9bbf7d4e2ed3f352bc4da0d6a5fdce561179819/packages/server/src/repowise/server/mcp_server/tool_search.py) | 5.22 | Below green threshold | Low Cohesion; Bumpy Road Ahead; Overall Code Complexity |
| [Health engine](https://github.com/repowise-dev/repowise/blob/c9bbf7d4e2ed3f352bc4da0d6a5fdce561179819/packages/core/src/repowise/core/analysis/health/engine.py) | 6.49 | Below green threshold | Lines of Code in a Single File; Bumpy Road Ahead; Overall Code Complexity |
| [Pipeline orchestration](https://github.com/repowise-dev/repowise/blob/c9bbf7d4e2ed3f352bc4da0d6a5fdce561179819/packages/core/src/repowise/core/pipeline/orchestrator.py) | 6.90 | Below green threshold | Bumpy Road Ahead; Global Conditionals; Complex Method |
| [Health scoring](https://github.com/repowise-dev/repowise/blob/c9bbf7d4e2ed3f352bc4da0d6a5fdce561179819/packages/core/src/repowise/core/analysis/health/scoring.py) | 8.95 | Below green threshold | Bumpy Road Ahead; Complex Method; Complex Conditional |
| [Web graph wrapper](https://github.com/repowise-dev/repowise/blob/c9bbf7d4e2ed3f352bc4da0d6a5fdce561179819/packages/web/src/components/graph/graph-flow.tsx) | 9.20 | Healthy / optimal | Complex Method |
| [Persistence models](https://github.com/repowise-dev/repowise/blob/c9bbf7d4e2ed3f352bc4da0d6a5fdce561179819/packages/core/src/repowise/core/persistence/models.py) | 9.68 | Healthy / optimal | Lines of Declarations in a Single File |
| [Health grading](https://github.com/repowise-dev/repowise/blob/c9bbf7d4e2ed3f352bc4da0d6a5fdce561179819/packages/core/src/repowise/core/analysis/health/grading.py) | 9.84 | Healthy / optimal | Bumpy Road Ahead |
| [Health ranking](https://github.com/repowise-dev/repowise/blob/c9bbf7d4e2ed3f352bc4da0d6a5fdce561179819/packages/core/src/repowise/core/analysis/health/ranking.py) | 10.00 | Healthy / optimal | No findings |
| [Detector registry](https://github.com/repowise-dev/repowise/blob/c9bbf7d4e2ed3f352bc4da0d6a5fdce561179819/packages/core/src/repowise/core/analysis/health/biomarkers/registry.py) | 10.00 | Healthy / optimal | No findings |

Eight of the 19 selected files score below 4. This is a sample description only, **not an estimate that 42% of RepoWise is unhealthy**.

![CodeScene scores for the 19 selected first-party files](code-health-sample.png)

**15. Measurement limits and reproducibility**

- The cloud and local code revisions match, but engine/configuration parity was not established. Cross-engine score differences are not time trends.
- Cloud LoC, CLI non-comment lines and AST physical spans are distinct measures. They are labelled separately.
- CodeScene's function spans differ from Python AST spans for some nested-function-heavy modules, notably MCP context targets and orchestration. AST spans are used for physical-length claims; the raw CLI boundaries are retained. Complexity values remain the CLI's metrics, not independently reimplemented values.
- This is maintainability and change-risk analysis, not a functional-correctness or security audit. No count of current exploitable vulnerabilities or confirmed live bugs is asserted.
- Commit prefixes are not curated defect labels. High fix activity is compatible with active hardening during feature expansion.
- A 10/10 score is not proof of correctness; a low score is not proof of incorrect behaviour.
- The narrow source-scope inventory does not retroactively change the existing cloud project metrics.
- Dependency directories, generated code and fixtures should not be attributed to the maintainers as hand-written production debt. All named reviewed files were checked against the source scope.
- No team-specific ROI, regression rate, agent failure rate, test coverage percentage or causality was measured.

The accompanying [evidence.zip](evidence.zip) contains the complete measurements and reproduction material. The selected [file-scores.csv](file-scores.csv) is also included directly for review. Its contents include:
- mcp-evidence.json: selected project data, raw MCP responses, parsed reviews and business cases;
- metadata.json: revision, cloud scope and retrieval method;
- file-scores.csv: all 19 scores;
- first-party-scope.json: source paths and exclusions;
- repowise-source-hotspot-crosschecks.json: AST spans, parameter counts and path-specific git counts;
- repowise-git-cochange-crosschecks.json: deduplicated change activity and co-change counts;
- repowise-source-inventory.json and repowise-python-functions.json: initial broad inventory and Python function spans;
- reproduce-source-checks.py: a script to regenerate source-scope, AST and git measurements from the pinned local checkout;
- code-health-sample.png / .svg: the targeted sample chart.

To refresh MCP evidence, select project 86250, list its technical-debt hotspots, review the 19 file paths in file-scores.csv, and request business cases for the four recommended files. Record the new analysis job, source revision and analysis-engine version before comparing results.

**Assessment suitable for publication:** RepoWise contains severe maintainability debt in several frequently changed, first-party components central to its functionality. The strongest evidence is the combination of low Code Health, specific structural findings and observed maintenance activity. Broader claims about its overall quality, actual financial losses or comparative product superiority require additional evidence.
