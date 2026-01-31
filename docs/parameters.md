---
title: Parameters
layout: default
nav_order: 30
---

# **Parameters**
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## **anno**

The `anno` command is the core annotation engine for VAMPIRE. It performs de novo tandem repeat motif discovery and annotation using De Bruijn graph construction and dynamic programming for optimal path selection.

### I/O Options

| Parameter        | Type   | Description                                    | Default |
|:----------------|:-------|:-----------------------------------------------|:--------|
| `input`         | string | FASTA file containing TR sequences to annotate   | Required |
| `prefix`        | string | Output prefix for all generated files             | Required |

### General Options

| Parameter           | Type    | Description                                                                      | Default |
|:-------------------|:--------|:---------------------------------------------------------------------------------|:--------|
| `-t, --thread`     | integer | Number of parallel threads for processing                                          | 1       |
| `--AUTO`            | boolean | Automatically estimate optimal parameters (ksize) from input data                      | False   |
| `--debug`           | boolean | Output running time of each module and alignments for performance profiling              | False   |
| `--window-length`   | integer | Parallel window size (bp) for processing large sequences                             | 5000    |
| `--overlap-length`   | integer | Overlap size (bp) between adjacent windows to prevent boundary effects                | 1000    |
| `-r, --resource`    | integer | Memory limit (GB) for processing                                                | 50      |

### Decomposition Options

| Parameter              | Type    | Description                                                                                     | Default |
|:----------------------|:--------|:------------------------------------------------------------------------------------------------|:--------|
| `-k, --ksize`        | integer | k-mer size for building De Bruijn graph (typically 3-5x expected motif length)              | 9       |
| `-m, --motif`        | string  | Path to reference motif database (FASTA) or 'base' for built-in database                | base    |
| `-n, --motifnum`     | integer | Maximum number of motifs to identify per sequence                                         | 30      |
| `--abud-threshold`    | float   | Minimum threshold compared with top edge weight in De Bruijn graph (relative)            | 0.01    |
| `--abud-min`         | integer | Minimum edge weight in De Bruijn graph (absolute count)                              | 3       |
| `--plot`             | boolean | Paint De Bruijn graph for each window (debugging output)                            | False   |
| `--no-denovo`        | boolean | Skip de novo motif discovery; use only reference motifs for annotation                 | False   |

{: .note }
> **Note on ksize selection**: The k-mer size should be approximately 3-5 times the expected motif length. Smaller k values capture short motifs but increase computational cost. Larger values may miss shorter motifs.

### Annotation Options

| Parameter                  | Type    | Description                                                                                      | Default |
|:--------------------------|:--------|:----------------------------------------------------------------------------------------------------|:--------|
| `-f, --force`            | boolean | Add reference motifs into annotation even if not detected de novo                              | False   |
| `--annotation-dist-ratio`  | float   | Maximum distance ratio for mapping actual sequences to reference motifs (distance ≤ ratio × motif length) | 0.4     |
| `--finding-dist-ratio`     | float   | Maximum distance ratio for querying reference motif set (distance ≤ ratio × motif length)        | 0.2     |
| `--match-score`           | float   | Score added per matched base in dynamic programming                                        | 1       |
| `--lendif-penalty`       | float   | Penalty for length difference between actual and reference motifs (encourages proper length)    | 0.01    |
| `--gap-penalty`          | float   | Penalty per skipped base (gap) in alignment                                             | 1       |
| `--distance-penalty`     | float   | Penalty per edit distance from reference motif                                             | 1.5     |
| `--perfect-bonus`        | float   | Bonus added for perfect matches (encourages exact motif identification)                     | 0.5     |

{: .note }
> **Scoring formula**: `Score = match_score × length - distance_penalty × edit_distance - lendif_penalty × |length_diff| - gap_penalty × gaps + perfect_bonus (if edit_distance = 0)`

### Output Options

| Parameter          | Type    | Description                                            | Default |
|:------------------|:--------|:-------------------------------------------------------|:--------|
| `--quiet`         | boolean | Suppress thread completion messages                      | False   |
| `-s, --score`    | float   | Minimum output score threshold for filtering annotations | 5       |

---

## **generator**

The `generator` command creates simulated tandem repeat sequences with user-defined motifs and mutation rates for testing and benchmarking.

### Required Parameters

| Parameter             | Type   | Description                                              | Default |
|:---------------------|:-------|:---------------------------------------------------------|:--------|
| `-m, --motifs`      | string | One or more motif sequences (space-separated)                  | Required |
| `-p, --prefix`       | string | Output prefix for generated files                             | Required |

### Optional Parameters

| Parameter                | Type    | Description                                   | Default |
|:------------------------|:--------|:----------------------------------------------|:--------|
| `-l, --length`         | integer | Length of simulated tandem repeat sequence (bp) | 1000    |
| `-r, --mutation-rate`   | float   | Mutation rate (0-1) for SNVs and indels        | 0       |
| `-s, --seed`           | integer | Random seed for reproducibility                  | 42      |

---

## **mkref**

The `mkref` command extracts motif sequences from annotation results to create a reference motif database for downstream use.

### Parameters

| Parameter   | Type   | Description                                          | Default |
|:-----------|:-------|:-----------------------------------------------------|:--------|
| `prefix`   | string | Input prefix from `anno` command (reads `.motif.tsv`) | Required |
| `output`   | string | Output FASTA file path for reference motif database     | Required |

---

## **evaluate**

The `evaluate` command assesses annotation quality by computing edit distance matrices between actual and reference motifs, generating clustered heatmaps.

### Required Parameters

| Parameter | Type   | Description                                   | Default |
|:-----------|:-------|:----------------------------------------------|:--------|
| `prefix`   | string | Input prefix from `anno` command                 | Required |
| `output`   | string | Output prefix for evaluation results and figures  | Required |

### Optional Parameters

| Parameter              | Type    | Description                                                                              | Default |
|:----------------------|:--------|:-----------------------------------------------------------------------------------------|:--------|
| `-t, --thread`       | integer | Number of threads for parallel distance calculation                                         | 6       |
| `-p, --percentage`   | integer | Threshold percentile for identifying abnormal values in heatmap (0-100)                        | 75      |
| `-s, --show-distance` | boolean | Display actual edit distance values on heatmap cells                                      | False   |

{: .note }
> **Output files**: Four heatmaps are generated combining modes (`raw`, `normalized`) with strand options (`merged`, `separate`), saved as PDF files.

---

## **refine**

The `refine` command performs manual curation of annotations by merging, replacing, or deleting motifs based on user-defined actions.

### Required Parameters

| Parameter | Type   | Description                                     | Default |
|:-----------|:-------|:------------------------------------------------|:--------|
| `prefix`   | string | Input prefix from `anno` command                  | Required |
| `action`   | string | Path to action TSV file (format: `source<TAB>action`) | Required |

### Action File Format

The action file defines refinement operations in TSV format:
- **source**: Motif ID(s) to modify (supports direction: `0+`, `0-`, or `0` for both)
- **action**: One of `MERGE`, `REPLACE`, or `DELETE`

**Examples:**
```
0,1    MERGE    # Merge motifs 0 and 1
0       DELETE    # Delete motif 0
2       REPLACE   # Replace motif 2 (see REPLACE action format)
```

For `REPLACE` action, use: `old_id<TAB>new_id` in the source column.

### Optional Parameters

| Parameter      | Type    | Description                                         | Default |
|:--------------|:--------|:----------------------------------------------------|:--------|
| `-o, --out`   | string | Output prefix (defaults to `{prefix}.revised`) | `{prefix}.revised` |
| `-t, --thread` | integer | Number of threads for parallel processing              | 8       |

{: .note }
> **Action types:**
> - **MERGE**: Combines multiple motifs into the best alternative based on edit distance
> - **REPLACE**: Replaces one motif with another while preserving sequence context
> - **DELETE**: Removes motifs and reassigns affected regions to best alternatives

---

## **logo**

The `logo` command generates sequence logos to visualize motif variation patterns across different metrics.

### Required Parameters

| Parameter | Type   | Description                                               | Default |
|:-----------|:-------|:----------------------------------------------------------|:--------|
| `prefix`   | string | Input prefix (reads `.motif.tsv` or `.annotation.tsv`)  | Required |
| `output`   | string | Output filename prefix (without extension)               | Required |

### Optional Parameters

| Parameter            | Type    | Description                                                          | Default |
|:--------------------|:--------|:---------------------------------------------------------------------|:--------|
| `-t, --type`        | string  | Input type: `motif` (from `.motif.tsv`) or `annotation` (from `.annotation.tsv`) | motif    |
| `-f, --format`      | string  | Output image format: `pdf` or `png`                                  | pdf      |

{: .note }
> **Output files**: Three logo types are generated for each input:
> - `{output}_count.{format}`: Base counts per position
> - `{output}_prob.{format}`: Probability distribution
> - `{output}_info.{format}`: Information content (bits)

---

## **identity**

The `identity` command calculates pairwise identity matrices between annotated TR sequences using alignment-based methods.

### Required Parameters

| Parameter | Type   | Description                                   | Default |
|:-----------|:-------|:----------------------------------------------|:--------|
| `prefix`   | string | Input prefix from `anno` command                 | Required |
| `output`   | string | Output prefix for identity matrix (BED format)    | Required |

### Optional Parameters

| Parameter               | Type    | Description                                                                             | Default |
|:-----------------------|:--------|:----------------------------------------------------------------------------------------|:--------|
| `-w, --window-size`   | integer | Window size in units of motifs for sequence comparison                                      | 100     |
| `-t, --thread`        | integer | Number of threads for parallel processing                                                   | 30      |
| `--mode`               | string  | Processing mode: `raw` (default) or `invert` (reverse complement minus strand motifs)            | raw     |
| `--max-indel`         | integer | Maximum indel length to include in identity calculation (set to 0 to ignore indels)     | 0       |
| `--min-indel`         | integer | Minimum indel length to include in identity calculation                                    | 0       |

{: .note }
> **Indel handling**: By default, indels are excluded from identity calculations. Set `--max-indel` and `--min-indel` to include indels within a specific length range.

{: .note }
> **Visualization**: After generating the identity matrix, use the provided scripts to create heatmaps:
> ```bash
> python scripts/get_visualization_data.py --prefix [annotation_prefix] --repeat [repeatmasker_annotation] --output [output_prefix]
> Rscript scripts/SG_aln_plot.R -t 30 -b [identity_bed_file] -a [visualization_data] -p [figure_output_prefix]
> ```

---

## **I/O Options (Common)**

These options apply across multiple commands:

| Parameter | Type   | Description                                      |
|:-----------|:-------|:-------------------------------------------------|
| `input`    | string | FASTA file you want to annotate (anno command)     |
| `output`   | string | Output prefix or file path                         |

## **General parameters (Common)**

| Parameter        | Type    | Description                     | Default |
|:----------------|:----------|:--------------------------------|:--------|
| `-t, --thread`  | integer   | Number of threads               | 1       |
| `--debug`       | boolean   | Output running time information  | False   |
| `-w, --window-length` | integer | Parallel window size (bp)   | 5000    |
| `-o, --overlap-length` | integer | Windows overlap size (bp) | 1000    |
| `-r, --resource` | integer  | Memory limit (GB)             | 50      |

## **Decomposition parameters (anno)**

| Parameter        | Type    | Description                                                  | Default |
|:----------------|:----------|:-------------------------------------------------------------------|:--------|
| `-k, --ksize`   | integer   | k-mer size for building De Bruijn graph                            | 9       |
| `-m, --motif`   | string    | Reference motif set path                                           | None    |
| `-n, --motifnum` | integer   | Maximum number of motifs                                           | 30      |
| `--abud-threshold` | float   | Minimum threshold compared with top edge weight                    | 0.01    |
| `--abud-min`    | integer   | Minimum edge weight in De Bruijn graph                             | 3       |
| `--plot`        | boolean   | Paint De Bruijn graph for each window                              | False   |
| `--no-denovo`   | boolean   | Not de novo find motifs, use reference motifs to annotate             | False   |

## **Annotation parameters (anno)**

| Parameter                 | Type    | Description                                | Default |
|:-------------------------|:----------|:---------------------------------------------------|:--------|
| `--force`               | boolean   | Add reference motifs into annotation        | False   |
| `--annotation-dist-ratio` | float | Max distance to map = ratio × motif length | 0.4     |
| `--finding-dist-ratio`     | float | Max distance to query in reference motif set = ratio × motif length | 0.2     |
| `--match-score`          | float    | Score per matched base                       | 1       |
| `--lendif-penalty`      | float    | Penalty for length difference                | 0.01    |
| `--gap-penalty`         | float    | Penalty per skipped base                     | 1       |
| `--distance-penalty`    | float    | Penalty per distance                         | 1.5     |
| `--perfect-bonus`       | float    | Bonus per matched base if there is no difference in alignment | 0.5     |

## **Output parameters (anno)**

| Parameter        | Type    | Description                    | Default |
|:----------------|:----------|:---------------------------------------|:--------|
| `--quiet`       | boolean   | Don't output thread completion informing | False   |
| `--file`        | integer   | Minimum output score            | 5       |
| `-s, --score`  | float     | Minimum output score            | 5       |
