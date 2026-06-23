# Dataflow Path Coverage Guide

Read this file when the user asks for def-use paths, variable-dependency graphs, DOT/SVG/HTML reports, test-to-path mapping, or dataflow-path coverage.

## Tool entrypoint

- PowerShell setup:

```powershell
$analyzer = "$HOME\.agents\skills\whitebox-testing\scripts\rust-dataflow-analyzer.exe"
```

- Resolve it from the skill directory, not from the current workspace.
- Version check: `& $analyzer -V`
- Help: `& $analyzer --help`

## Analyze command

```powershell
& $analyzer analyze --lang python --input D:\repos\asr_platform\app --out D:\tmp\dataflow_report
```

Prefer an isolated output directory so existing reports are not overwritten.

## Key outputs

- `index.html`: static HTML entrypoint
- `graphs/def_use_hotspots.dot`: def-use hotspot DOT
- `graphs/variable_dependencies.dot`: variable-dependency DOT
- `graphs/variable_dependencies.svg`: SVG when Graphviz `dot` is available
- `graphs/variable_dependencies.graph.json`: machine-readable variable-dependency graph
- `data/analysis-cache.json`: canonical cache for path queries and post-processing
- `data/definitions.csv`, `uses.csv`, `def_use_edges.csv`, `var_dependencies.csv`, `parse_diagnostics.csv`

## Function-level path query

```powershell
& $analyzer paths --input D:\tmp\dataflow_report\data\analysis-cache.json --function app.services.result_service::ResultService.get_task_summary --max-loop-unroll 2
```

This writes:

```text
D:\tmp\dataflow_report\data\path-query.json
```

Use qualified function names when possible. Keep `--max-loop-unroll` at `2` unless the user explicitly asks for a different unroll depth.

## Coverage standard

Use these as coverage targets instead of only line or branch counts:

1. def-use paths
2. variable-dependency paths
3. call-argument propagation edges
4. same-line def/use propagation edges

Minimum workflow:

1. Run `analyze` to generate report artifacts.
2. Read `graphs/variable_dependencies.graph.json` and `data/analysis-cache.json`.
3. List path totals, root functions, path lengths, fan-in/fan-out, and uncovered/split status.
4. Build `path_id -> tests` and `test -> path_ids` mappings.
5. Cover uncovered paths first, collapse split paths next, and document any remaining exclusions.

## Required reporting

When using dataflow-path coverage, produce at least:

- `dataflow_path_coverage.md`
- `path_mapping.csv` or `path_mapping.json`
- changed or added test-file paths
- report-directory location for HTML/DOT/SVG/JSON artifacts

`dataflow_path_coverage.md` should include:

- analyzer command(s)
- analyzed source directory
- output report directory
- covered / uncovered / split statistics
- top uncovered functions
- per-path `path_id`, root function, gap explanation, and planned or linked tests

## Constraints

- Treat static-analysis output as conservative approximation, not runtime truth.
- Call out dynamic-language risk areas: reflection, monkey patching, runtime imports, dynamic attributes.
- If SVG is unavailable, keep DOT and state that Graphviz was not available.
- Do not build a new ad-hoc def-use extractor when the bundled binary is available.
