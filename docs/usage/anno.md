---
title: anno
parent: Usage
layout: default
nav_order: 3
---

# **anno**
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## **Introduction**

The `anno` command in VAMPIRE is designed to annotate tandem repeat (TR) loci with detailed motif information and variation patterns.

By providing a FASTA file containing tandem repeat loci, `anno` generates a comprehensive set of annotated files using de novo motif discovery via De Bruijn graph construction and dynamic programming for optimal path selection through the TR sequence.

### Key Features

- **De novo motif discovery**: Automatically identifies consensus motifs from k-mer analysis
- **Reference database support**: Can use known motif databases (e.g., alpha-satellite, pCht/StSat)
- **Flexible annotation**: Supports both de novo and non-de novo modes
- **Multi-strand analysis**: Handles both forward and reverse complement motifs
- **Dynamic programming**: Optimal path selection through TR regions

---

## **Input and Output**

### **Input**

| Input      | Format | Description                                                    | Default |
|:---------- |:------ |:---------------------------------------------------------------|:--------|
| Sequence   | FASTA  | The sequence to be annotated                                   | None    |
| Motif DB   | FASTA  | Motif database used to refine and align motifs during analysis | base    |

{: .note }
> **Built-in database**: The default `base` database includes human alpha-satellite monomers and pCht/StSat motifs from:
> Altemose N, Logsdon G A, Bzikadze A V, et al. Complete genomic and epigenetic maps of human centromeres[J]. Science, 2022, 376(6588): eabl4178.

### **Output**

| Output     | Format | Description                                | Default |
|:---------- |:------ |:-------------------------------------------|:--------|
| Config     | JSON   | Parameter set used for the annotation job  | None    |
| Annotation | TSV    | Detailed annotation results                | None    |
| Concise    | TSV    | Summary of annotation results              | None    |
| Motif      | TSV    | Statistics of detected motifs              | None    |
| Distance   | TSV    | Edit distances between detected motifs     | None    |

**File descriptions:**

- `{prefix}.setting.json`: All parameters used in the annotation (for reproducibility)
- `{prefix}.annotation.tsv`: Per-instance annotation with actual sequences, CIGAR strings, and distances
- `{prefix}.concise.tsv`: Collapsed annotation grouped by motif blocks
- `{prefix}.motif.tsv`: Motif statistics (id, sequence, count, label)
- `{prefix}.dist.tsv`: Pairwise edit distances between all detected motifs

---

## **Usage Examples**

### **1. De novo annotate**
Annotate tandem repeat loci directly from a FASTA file, discovering motifs automatically.
```bash
vampire anno -t 8 tests/001-anno_STR.fa tests/001-anno_STR
```

### **2. Annotate with custom motif database**
Use a user-provided motif database for annotation (combined with de novo discovery).
```bash
vampire anno -f -m custom_motifs.fa -t 8 repeats.fasta annotation_prefix
```

### **3. Non-de novo annotation (reference only)**
Use only motifs from the database without de novo discovery.
```bash
vampire anno --no-denovo -m reference_motifs.fa repeats.fasta annotation_prefix
```

### **4. Annotate population-level sequence**
Annotating tandem repeat (TR) loci at the population level can be challenging due to highly variable copy numbers among individuals. For loci with low copy numbers, accurate annotation is more difficult. In such cases, providing a curated motif set can improve annotation quality and consistency across samples.

A three-step strategy can be applied:

**STEP-1:** Call the motif set from the population sequences
```bash
vampire anno -t 16 population_sequences.fasta population_annotation
```

**STEP-2:** Create a motif database from the annotation
```bash
vampire mkref population_annotation population_motif_set.fasta
```

**STEP-3:** Use the curated motif database to annotate the population sequences
```bash
vampire anno --no-denovo -m population_motif_set.fasta -t 16 population_sequences.fasta population_annotation.curated
```

### **5. Automatic parameter estimation**
Let VAMPIRE automatically estimate optimal k-mer size from input data.
```bash
vampire anno --AUTO repeats.fasta annotation_prefix
```

---

## **Parameters**

### **I/O Options**

| Parameter        | Type   | Description                                    | Default |
|:----------------|:-------|:-----------------------------------------------|:--------|
| `input`         | string | FASTA file containing TR sequences to annotate   | Required |
| `prefix`        | string | Output prefix for all generated files             | Required |

### **General Options**

| Parameter           | Type    | Description                                                                      | Default |
|:-------------------|:--------|:---------------------------------------------------------------------------------|:--------|
| `-t, --thread`     | integer | Number of parallel threads for processing                                          | 1       |
| `--AUTO`            | boolean | Automatically estimate optimal parameters (ksize) from input data                      | False   |
| `--debug`           | boolean | Output running time of each module and alignments for performance profiling              | False   |
| `--window-length`   | integer | Parallel window size (bp) for processing large sequences                             | 5000    |
| `--overlap-length`   | integer | Overlap size (bp) between adjacent windows to prevent boundary effects                | 1000    |
| `-r, --resource`    | integer | Memory limit (GB) for processing                                                | 50      |

### **Decomposition Options**

| Parameter              | Type    | Description                                                                                     | Default |
|:----------------------|:--------|:------------------------------------------------------------------------------------------------|:--------|
| `-k, --ksize`        | integer | k-mer size for building De Bruijn graph (typically 3-5x expected motif length)              | 9       |
| `-m, --motif`        | string  | Path to reference motif database (FASTA) or 'base' for built-in database                | base    |
| `-n, --motifnum`     | integer | Maximum number of motifs to identify per sequence                                         | 30      |
| `--abud-threshold`    | float   | Minimum threshold compared with top edge weight in De Bruijn graph (relative)            | 0.01    |
| `--abud-min`         | integer | Minimum edge weight in De Bruijn graph (absolute count)                              | 3       |
| `--plot`             | boolean | Paint De Bruijn graph for each window (debugging output)                            | False   |
| `--no-denovo`        | boolean | Skip de novo motif discovery; use only reference motifs for annotation                 | False   |

{: .note-title }
> **Note on ksize selection**
>
> The k-mer size should be approximately 3-5 times the expected motif length. Smaller k values capture short motifs but increase computational cost. Larger values may miss shorter motifs. For example:
> - Short STRs (2-6 bp): Use k=9-15
> - Variable number tandem repeats (7-20 bp): Use k=15-31
> - Satellite arrays (20+ bp): Use k=31-71

### **Annotation Options**

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

{: .note-title }
> **Scoring formula**
>
> The dynamic programming score for each annotation is calculated as:
> ```
> Score = (match_score × length)
>        - (distance_penalty × edit_distance)
>        - (lendif_penalty × |length_difference|)
>        - (gap_penalty × gaps)
>        + (perfect_bonus if edit_distance = 0)
> ```

### **Output Options**

| Parameter          | Type    | Description                                            | Default |
|:------------------|:--------|:-------------------------------------------------------|:--------|
| `--quiet`         | boolean | Suppress thread completion messages                      | False   |
| `-s, --score`    | float   | Minimum output score threshold for filtering annotations | 5       |

---

## **Algorithm Overview**

### **1. Preprocessing**
- N-character regions are identified and masked
- Sequences are split into sliding windows for parallel processing

### **2. De Bruijn Graph Construction**
- k-mers are counted within each window
- De Bruijn graph is built to identify frequent paths
- Top `--motifnum` motifs are selected based on edge weights

### **3. Motif Refinement**
- Discovered motifs are aligned to reference database
- Reverse complements are considered
- Motifs are adjusted to canonical (minimal rotation) form

### **4. Annotation**
- Each instance is annotated with the best matching motif
- Edit distance and CIGAR strings are computed
- Direction (+/-) is recorded

### **5. Dynamic Programming**
- Finds optimal path through all annotated instances
- Maximizes score while penalizing gaps and mismatches
- Produces final concise annotation

---

## **Troubleshooting**

### **Empty output files**

**Possible causes:**
- Input sequences don't contain tandem repeats
- k-mer size too large for motif length
- Thresholds too restrictive

**Solutions:**
1. Reduce `--ksize` (try 5, 7, 9 for STRs)
2. Lower `--abud-min` and `--abud-threshold`
3. Use `--AUTO` for automatic parameter estimation
4. Provide reference motifs with `-m`

### **Low annotation rate**

**Possible causes:**
- Score threshold too high
- Penalties too strict

**Solutions:**
1. Lower `--score` threshold
2. Reduce `--distance-penalty` and `--gap-penalty`
3. Increase `--match-score`

### **Memory issues**

**Possible causes:**
- Large input sequences with many windows
- High thread count

**Solutions:**
1. Reduce `--thread` count
2. Decrease `--resource` limit
3. Increase `--window-length` to reduce number of windows

---

## **See Also**

- [Parameters](../parameters.html) - Full parameter reference
- [evaluate](evaluate.html) - Evaluate annotation quality
- [refine](refine.html) - Manual refinement of annotations
