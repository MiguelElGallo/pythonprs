# What the checks found in iParq

Applying pythonprs to an existing project can reveal failures in code whose tests already pass. These examples come from iParq, a command-line tool for inspecting Parquet files, when it adopted the checks on 27 September 2026. Its baseline had 126 passing tests alongside complexity and documentation failures. [Recorded baseline](https://github.com/MiguelElGallo/iparq/blob/51dc01d357f923f32a8fd674504906e86a253afe/reports/pythonprs-baseline-2026-09-27.md).

The examples show one McCabe finding, the highest cognitive complexity finding, and a short-docstring finding. Each includes the original diagnostic and the completed repair. The linked revisions preserve those results; they are historical examples. Code blocks are excerpts, and complexity scores describe the **whole function**, including code omitted here.

## McCabe complexity: too many decision paths

McCabe complexity counts decision points plus one. Ruff reports `C901` when a function exceeds this project's maximum of **10**. [Ruff C901](https://docs.astral.sh/ruff/rules/complex-structure/).

In iParq, `inspect` expanded file patterns, removed duplicate filenames, displayed headings, handled each file's errors, and assembled JSON output. Its McCabe score was **14**:

```text
src/iparq/source.py:749:5: C901 `inspect` is too complex (14 > 10)
```

The repair moved pattern expansion, headings, and JSON emission into named helpers. For example, the JSON portion of `inspect` originally contained these decisions:

```python
if format == OutputFormat.JSON and json_results:
    payload: object = json_results[0] if len(unique_files) == 1 else json_results
    print(json.dumps(payload, indent=2))
elif format == OutputFormat.JSON and len(unique_files) > 1:
    print("[]")
```

After the repair, that portion delegates output to a helper:

```python
if format == OutputFormat.JSON:
    _emit_json_results(json_results, len(unique_files))
```

The helper retains the choice between a single JSON object, an array for multiple input files, and an empty array when multiple inputs yield no successful results. Together with the other extractions, this brought the whole function within both limits:

| Measure for `inspect` | Before | After | Allowed maximum |
| --- | ---: | ---: | ---: |
| McCabe complexity | 14 — FAIL | 8 — PASS | 10 |
| Cognitive complexity | 28 — FAIL | 13 — PASS | 15 |

These are two measurements of the same function. A helper's own complexity is checked too. The smaller caller score does not, by itself, establish that the repair preserves behavior.

Source: [`inspect` before](https://github.com/MiguelElGallo/iparq/blob/51dc01d357f923f32a8fd674504906e86a253afe/src/iparq/source.py#L749-L837), [`inspect` and its helpers after](https://github.com/MiguelElGallo/iparq/blob/7fbda13d2cb35cecd06a235683e764adcfa1336f/src/iparq/source.py#L840-L934).

## Cognitive complexity: decisions inside decisions

Cognitive complexity gives extra weight to nesting: a reader must remember the surrounding conditions while following an inner decision. complexipy checks this score against a maximum of **15**. [How complexipy measures complexity](https://complexipy.com/#what-is-cognitive-complexity).

`print_min_max_statistics` had the highest cognitive score in the baseline, **49**. The complexipy output included:

```text
src/iparq/source.py print_min_max_statistics 49
```

It looped over groups of rows and each group's columns, then searched for the matching column in iParq's metadata model. Inside those two outer loops, it copied statistics with further conditions:

```python
for col in column_info.columns:
    if col.row_group == i and col.column_index == j:
        if column_chunk.is_stats_set:
            stats = column_chunk.statistics
            col.has_min_max = stats.has_min_max
            col.null_count = (
                stats.null_count if stats.has_null_count else None
            )
            col.distinct_count = (
                stats.distinct_count if stats.has_distinct_count else None
            )
            col.statistics_num_values = stats.num_values
```

The repair separates finding a matching column from updating its statistics. Inside the same two outer loops, the caller now reads:

```python
column = next(
    (
        col
        for col in column_info.columns
        if col.row_group == i and col.column_index == j
    ),
    None,
)
if column is not None:
    _update_column_statistics(column, column_chunk)
```

The update helper returns early when statistics are absent:

```python
def _update_column_statistics(
    column: ColumnInfo, column_chunk: pq.ColumnChunkMetaData
) -> None:
    """Update one column using only statistics declared by its chunk."""
    if not column_chunk.is_stats_set:
        column.has_min_max = False
        return
    _apply_column_statistics(column, column_chunk.statistics)
```

The whole function's cognitive score fell from **49 (FAIL)** to **11 (PASS)**. Both new statistics helpers also pass the limit. The repair retains the first matching model column and distinguishes missing statistics from valid zero values. Those details matter when reviewing a refactor, even when its score improves.

Source: [statistics update before](https://github.com/MiguelElGallo/iparq/blob/51dc01d357f923f32a8fd674504906e86a253afe/src/iparq/source.py#L381-L418), [statistics update and helpers after](https://github.com/MiguelElGallo/iparq/blob/7fbda13d2cb35cecd06a235683e764adcfa1336f/src/iparq/source.py#L408-L456).

## Docstrings: enough characters, too few words

`output_json` already had a docstring and an `Args` section:

```python
"""
Outputs the parquet information in JSON format.

Args:
    meta_model: The Parquet metadata model
    column_info: The column information model
    compression_codecs: Set of compression codecs used
"""
```

Its first paragraph contained **7 words and 41 non-whitespace characters**. It met the character minimum of 40, but missed the word minimum of 8:

```text
./src/iparq/source.py:659: output_json: DOC002 summary has 7 words/41 characters; requires >= 8 words and >= 40 non-whitespace characters
```

Interrogate treats this function as documented. The project's AST checker reads the source without running it and still fails this function because **both summary minimums must pass**. The `Args` section follows a blank line, so it contributes nothing to the summary counts.

The repaired first paragraph names the selected content and says it goes to standard output (`stdout`):

```python
"""Write the selected Parquet metadata and column details to stdout as JSON."""
```

That summary has **12 words and 62 non-whitespace characters**, so it passes both minimums. The original parameter descriptions remain below it, and the repair also documents the `metadata_only` option. The added detail explains the function's behavior as well as meeting the count.

Source: [`output_json` before](https://github.com/MiguelElGallo/iparq/blob/51dc01d357f923f32a8fd674504906e86a253afe/src/iparq/source.py#L659-L677), [`output_json` after](https://github.com/MiguelElGallo/iparq/blob/7fbda13d2cb35cecd06a235683e764adcfa1336f/src/iparq/source.py#L724-L742).

## What these examples show

A passing test suite and a failed quality check can describe the same project. Tests exercise behavior; these checks also inspect structure and documentation. iParq's [repair report](https://github.com/MiguelElGallo/iparq/blob/7fbda13d2cb35cecd06a235683e764adcfa1336f/reports/pythonprs-fixes-2026-09-27.md) records unchanged check limits, added regression tests, and comparisons of CLI output and exit codes alongside the improved scores.

For your own findings, use [Fix a failed check](../how-to/fix-failed-checks.md). The [check reference](../reference/checks.md) gives the exact requirements, and [Docstrings and types](docstrings-and-types.md) explains the separate documentation checks.
