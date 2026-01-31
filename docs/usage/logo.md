---
title: logo
parent: Usage
layout: default
nav_order: 8
---

# **logo**
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## **Introduction**

The `logo` command generates sequence logos to visualize motif variation patterns in VAMPIRE annotations. Sequence logos provide intuitive visualization of base composition and conservation across motif positions.

### Key Features

- **Multiple visualization types**: Count, probability, and information content
- **Two input modes**: Reference motifs or actual annotation instances
- **Alignment-aware**: Uses MAFFT for proper motif alignment
- **Strand handling**: Automatically handles reverse complement annotations
- **Flexible output**: PDF or PNG format options

### Logo Types Generated

Three logo types are always generated:

1. **Count logo**: Shows absolute base counts at each position
2. **Probability logo**: Shows normalized probabilities (sum = 1)
3. **Information logo**: Shows bits of information (measures conservation)

---

## **Input and Output**

### **Input**

| Input         | Format | Description                                                              | Default |
|:------------- |:------ |:-------------------------------------------------------------------------|:--------|
| Annotation    | TSV    | Annotation file from `anno` command (for `--type annotation`)        | None    |
| Motif         | TSV    | Motif statistics file from `anno` command (for `--type motif`)       | None    |

### **Output**

| Output        | Format | Description                              | Default |
|:-------------|:------ |:-----------------------------------------|:--------|
| Count Matrix  | TSV    | Base count matrix by position             | None    |
| Count Logo    | PDF/PNG | Visualization of base counts            | None    |
| Probability Logo | PDF/PNG | Visualization of probabilities      | None    |
| Information Logo | PDF/PNG | Visualization of information content  | None    |

**Output files** (with output prefix `logo_output`):
- `logo_output_count.tsv` - Count matrix used for logos
- `logo_output_count.{format}` - Count visualization
- `logo_output_prob.{format}` - Probability visualization
- `logo_output_info.{format}` - Information content visualization

---

## **Usage Examples**

### **1. Reference motif logos**

Generate logos from the reference motif statistics file.
```bash
vampire logo annotation_prefix motif_logo
```

### **2. Actual motif variation logos**

Generate logos showing actual sequence variation from annotations.
```bash
vampire logo --type annotation annotation_prefix variation_logo
```

### **3. PNG format output**

Generate logos in PNG format for web use.
```bash
vampire logo --format png annotation_prefix motif_logo_png
```

### **4. Comparative analysis**

Generate both reference and variation logos to compare.
```bash
# Reference (expected) patterns
vampire logo annotation_prefix reference_motif

# Actual (observed) variation
vampire logo --type annotation annotation_prefix actual_variation

# Compare the three logo types from both outputs
```

---

## **Parameters**

### **Required Parameters**

| Parameter | Type   | Description                                               | Default |
|:-----------|:-------|:----------------------------------------------------------|:--------|
| `prefix`   | string | Input prefix (reads `.motif.tsv` or `.annotation.tsv`)  | Required |
| `output`   | string | Output filename prefix (without extension)               | Required |

### **Optional Parameters**

| Parameter            | Type    | Description                                                          | Default |
|:--------------------|:--------|:---------------------------------------------------------------------|:--------|
| `-t, --type`        | string  | Input type: `motif` (from `.motif.tsv`) or `annotation` (from `.annotation.tsv`) | motif    |
| `-f, --format`      | string  | Output image format: `pdf` or `png`                                  | pdf      |

---

## **Input Modes**

### **Mode: motif (default)**

**Input file**: `{prefix}.motif.tsv`

**Data used**:
- Motif sequences
- Copy numbers (rep_num)

**Process**:
1. Reads motif sequences from TSV file
2. Performs multiple sequence alignment (MSA) using MAFFT
3. Calculates base counts at each aligned position
4. Weights counts by copy number

**When to use**:
- Visualizing consensus motif structures
- Comparing different motif sets
- Understanding expected variation patterns

**Advantages**:
- Reflects canonical motif representation
- Shows what motifs "should" look like
- Useful for reference database construction

### **Mode: annotation**

**Input file**: `{prefix}.annotation.tsv`

**Data used**:
- Actual motif sequences from each instance
- Strand direction (+/-)
- Position information

**Process**:
1. Reads all actual motif instances from annotations
2. Filters out N characters
3. Reverses complement for negative strand motifs
4. Performs MSA using MAFFT
5. Calculates counts weighted by occurrence
6. Handles truncated motifs at 5' and 3' boundaries

**When to use**:
- Visualizing true observed variation
- Understanding motif degradation patterns
- Detecting systematic biases or errors

**Advantages**:
- Shows actual sequence diversity
- Captures real variation patterns
- Reveals motif-specific biases

**Truncated motif handling**:
In annotation mode, motifs at TR boundaries may be incomplete. VAMPIRE:
- Identifies motifs truncated at 5' end
- Identifies motifs truncated at 3' end
- Excludes incomplete boundary regions from counting
- Only counts full motifs or internally truncated regions

---

## **Visualization Types**

### **Count Logo**

Shows **absolute** number of each base at each position.

**Interpretation**:
- Height reflects actual count of bases
- Useful for comparing motif frequencies
- Shows most common variant per position

**Best for**:
- Understanding motif abundance
- Detecting dominant variants
- Quality control (check for unexpected patterns)

### **Probability Logo**

Shows **normalized** probability of each base (sum = 1.0 per position).

**Interpretation**:
- Height reflects relative frequency
- Easy to compare positions of different conservation
- Standardized across all positions

**Best for**:
- Comparing conservation levels
- Identifying informative positions
- Publication-quality figures

### **Information Logo**

Shows **bits of information** (measures sequence conservation).

**Interpretation**:
- Height (total bits) = conservation (max 2 bits for DNA)
- Base height = contribution to information
- Tall stack = highly conserved position

**Best for**:
- Identifying functional/structural positions
- Detecting constraint levels
- Evolutionary analysis

**Information formula**:
```
Information = 2 - (entropy)
Entropy = -Σ(p_i × log2(p_i))
```
Where p_i is probability of base i at that position.

---

## **Color Scheme**

VAMPIRE uses a custom color scheme for DNA bases:

| Base | Color    | Description        |
|:-----|:--------|:------------------|
| A     | Green   | Adenine          |
| C     | Blue    | Cytosine         |
| G     | Yellow  | Guanine          |
| T     | Red     | Thymine          |
| -     | Black   | Gap/deletion      |

These colors are optimized for:
- **Accessibility**: Clear distinction between bases
- **Color blindness**: High contrast
- **Publication**: Professional appearance

---

## **Alignment Process**

### **MAFFT Integration**

VAMPIRE uses MAFFT for multiple sequence alignment:

```bash
# Internal command equivalent
mafft input_motifs.fasta > aligned_motifs.aln
```

**Alignment features**:
- **Automatic parameter selection**: MAFFT chooses optimal strategy
- **Gap handling**: Proper treatment of indels
- **Quality assessment**: Failed alignments are reported

### **Truncated Motif Handling**

In annotation mode, boundary motifs are often incomplete:

**Example**:
```
Full motif:      ATCGATCGATCG
5' truncated: ----ATCGATCGATCG
3' truncated: ATCGATCGAT----
Internal trunc: ATCG----ATCGATCG
```

**VAMPIRE's strategy**:
1. Identifies motifs starting or ending with gaps
2. Checks truncation extent (must be < 10% of length)
3. Excludes truncated boundary regions from counting
4. Only counts internally complete positions

This ensures logos reflect **true** motif variation, not boundary artifacts.

---

## **Use Cases**

### **Quality Control**

Check if motif annotations are biologically plausible:

```bash
# Generate logo from annotation
vampire logo --type annotation anno_prefix quality_check

# Inspect:
# - Are certain positions highly conserved? (expected)
# - Is there random noise? (potential error)
# - Are there unexpected patterns? (annotation issue)
```

### **Comparative Analysis**

Compare motifs across samples or conditions:

```bash
# Sample 1
vampire logo sample1_anno sample1_logo

# Sample 2
vampire logo sample2_anno sample2_logo

# Side-by-side comparison in PDF viewer:
# - Similar overall patterns? (good)
# - Position-specific differences? (interesting)
# - Divergent structures? (may need separate analysis)
```

### **Evolutionary Analysis**

Track motif evolution over time or populations:

```bash
# Ancestral motif
vampire logo ancestral_anno ancestral_logo

# Derived population
vampire logo derived_anno derived_logo

# Compare information content:
# - Which positions gained conservation? (functional constraint)
# - Which positions lost conservation? (relaxed constraint)
```

### **Strand-specific Patterns**

Examine forward and reverse complement separately:

```bash
# Check motif.tsv for strand distribution
cat anno.motif.tsv

# If motifs have direction info, filter annotation:
# Forward only
awk '$6=="+"' anno.annotation.tsv > anno_plus.tsv

# Reverse only
awk '$6=="-"' anno.annotation.tsv > anno_minus.tsv

# Generate separate logos
vampire logo anno_plus_logo  # (requires proper prefix setup)
vampire logo anno_minus_logo  # (requires proper prefix setup)
```

---

## **Best Practices**

### **File Size Considerations**

Large motif numbers can slow down MAFFT:

```bash
# Check motif count
wc -l anno.motif.tsv

# If > 500 motifs, consider:
# 1. Subset analysis
head -n 100 anno.motif.tsv > subset_motifs.tsv

# 2. Filter by copy number
awk '$3 >= 10' anno.motif.tsv > frequent_motifs.tsv

# 3. Cluster and select representatives
# (Use evaluate output to group similar motifs)
```

### **Image Format Selection**

- **PDF**: Best for publication, vector graphics, small file size
- **PNG**: Best for web, raster graphics, larger file size

Choose based on intended use:

```bash
# Publication figure
vampire logo --format pdf anno_prefix figure

# Website or presentation
vampire logo --format png anno_prefix figure
```

### **Quality Assessment**

Use logos to detect annotation errors:

**Good annotation indicators:**
- Clear conservation at core positions
- Smooth variation at flanking positions
- Biologically plausible patterns

**Potential error indicators:**
- Random noise (no conservation)
- Unexpected periodicity
- Single base dominance at all positions

---

## **Common Issues**

### **MAFFT not found**

**Error**: `RuntimeError: MAFFT failed`

**Solution**: Install MAFFT:
```bash
conda install -c bioconda mafft
# or
apt-get install mafft
```

### **Alignment fails**

**Error**: `MAFFT failed` with traceback

**Cause**: Highly divergent motifs or sequence length mismatch.

**Solutions**:
1. Check motif sequences in input file
2. Filter out extreme outliers
3. Use motif mode instead of annotation mode

### **Empty logo**

**Cause**: No motifs in annotation or all filtered out.

**Solution**: Verify input files:
```bash
# Check motif file
cat anno.motif.tsv

# Check annotation file
cat anno.annotation.tsv | head

# Ensure motifs exist
```

### **Colors not displaying**

**Cause**: PDF viewer not supporting custom colors.

**Solution**: Use vector PDF viewer or convert to PNG:
```bash
vampire logo --format png anno_prefix logo
```

---

## **Advanced Usage**

### **Custom Visualization**

Extract count matrix for custom plots:

```bash
# Generate logo (creates .tsv file)
vampire logo anno_prefix logo

# Use in R/Python for custom visualization
# R:
count_mat <- read.delim("logo_count.tsv", sep="\t")
# Custom ggplot2 code...

# Python:
import pandas as pd
import seaborn as sns
df = pd.read_csv("logo_count.tsv", sep="\t", index_col=0)
# Custom seaborn code...
```

### **Position-specific Analysis**

Identify most informative positions:

```bash
# Generate info logo
vampire logo anno_prefix info_logo

# Inspect information logo
open info_logo_info.pdf

# Note positions with:
# - High total height (conserved)
# - Uneven distribution (informative)
# - Single dominant base (highly constrained)
```

---

## **See Also**

- [Parameters](../parameters.html) - Full parameter reference
- [anno](anno.html) - Generate annotations for logo
- [evaluate](evaluate.html) - Assess motif quality before visualization
