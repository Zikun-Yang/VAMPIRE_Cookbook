---
title: identity
parent: Usage
layout: default
nav_order: 9
---

# **identity**
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## **Introduction**

The `identity` command calculates pairwise identity matrices between annotated tandem repeat (TR) sequences using alignment-based methods. This is useful for:

- **Population structure**: Identify sequence similarity groups
- **Phylogenetic analysis**: Understand TR evolution
- **Quality control**: Detect outliers and potential errors
- **Variant detection**: Find divergent sequences

### Key Features

- **Window-based comparison**: Compares sequences in user-defined windows
- **Indel handling**: Optional inclusion of insertion/deletion events
- **Strand modes**: Raw or invert (reverse complement) modes
- **BED output**: Standard format for downstream tools
- **Alignment-based**: Precise identity calculation using CIGAR strings

---

## **Input and Output**

### **Input**

| Input         | Format | Description                                     | Default |
|:------------- |:------ |:------------------------------------------------|:--------|
| Motif         | TSV    | Motif statistics file from `anno` command | None    |
| Dist          | TSV    | Distance matrix file from `anno` command   | None    |
| Concise       | TSV    | Summary annotation from `anno` command  | None    |
| Annotation    | TSV    | Detailed annotation from `anno` command  | None    |

### **Output**

| Output        | Format | Description                                | Default |
|:-------------|:------ |:-------------------------------------------|:--------|
| Identity Matrix | BED    | Pairwise identity scores between sequences | None    |

**Output file format** (BED6+1):
```
#query_name	query_start	query_end	reference_name	reference_start	reference_end	perID_by_events
contig1	10000	12000	contig1	10000	12000	98.5
contig1	10000	12000	contig2	15000	17000	45.2
contig2	15000	17000	contig2	15000	17000	100.0
...
```

Columns:
- `query_name`: Query sequence identifier
- `query_start`: Query window start coordinate
- `query_end`: Query window end coordinate
- `reference_name`: Reference sequence identifier
- `reference_start`: Reference window start coordinate
- `reference_end`: Reference window end coordinate
- `perID_by_events`: Percentage identity (0-100)

---

## **Usage Examples**

### **1. Basic identity calculation**

Calculate identity with default 100-motif windows.
```bash
vampire identity annotation_prefix identity_output
```

### **2. Adjust window size**

Use smaller windows for higher resolution.
```bash
vampire identity -w 30 annotation_prefix identity_output
```

### **3. Include indels in calculation**

Account for insertions and deletions within specific size range.
```bash
vampire identity --min-indel 5 --max-indel 50 annotation_prefix identity_output
```

### **4. Reverse complement mode**

Invert minus strand motifs before comparison.
```bash
vampire identity --mode invert annotation_prefix identity_output
```

### **5. Parallel processing**

Use more threads for faster computation on large datasets.
```bash
vampire identity -t 50 annotation_prefix identity_output
```

---

## **Parameters**

### **Required Parameters**

| Parameter | Type   | Description                                   | Default |
|:-----------|:-------|:----------------------------------------------|:--------|
| `prefix`   | string | Input prefix from `anno` command                 | Required |
| `output`   | string | Output prefix for identity matrix (BED format)    | Required |

### **Optional Parameters**

| Parameter               | Type    | Description                                                                             | Default |
|:-----------------------|:--------|:----------------------------------------------------------------------------------------|:--------|
| `-w, --window-size`   | integer | Window size in units of motifs for sequence comparison                                      | 100     |
| `-t, --thread`        | integer | Number of threads for parallel processing                                                   | 30      |
| `--mode`               | string  | Processing mode: `raw` (default) or `invert` (reverse complement minus strand motifs)            | raw     |
| `--max-indel`         | integer | Maximum indel length to include in identity calculation (set to 0 to ignore indels)     | 0       |
| `--min-indel`         | integer | Minimum indel length to include in identity calculation                                    | 0       |

### **Parameter Details**

**Window size (`-w`)**:
- Controls resolution of identity matrix
- Smaller values: More detailed comparisons, more computationally intensive
- Larger values: Coarser comparisons, faster processing
- Typical range: 30-200 motifs

**Thread count (`-t`)**:
- Higher values speed up processing but increase memory
- Typical range: 10-100 depending on dataset size

**Mode (`--mode`)**:
- `raw`: Use motifs as-is (default)
- `invert`: Reverse complement minus strand motifs before comparison

**Indel handling**:
- Default (both 0): Indels are excluded from identity calculation
- Set both values: Include indels within specified length range
- Example: `--min-indel 5 --max-indel 50` includes 5-50 bp indels

---

## **Algorithm Overview**

### **1. Sequence Processing**

1. **Reads annotation files**: motif statistics, distances, concise, and detailed annotations
2. **Partitions sequences**: Divides each sequence into windows of specified size
3. **Applies mode**: Inverts minus strand motifs if `--mode invert`

### **2. Window Construction**

For each window, VAMPIRE:

1. **Collects motifs**: All motif instances within the window
2. **Extends boundaries**: Uses adjacent motifs for precise alignment
3. **Constructs sequence**: Concatenates actual motif sequences

### **3. Alignment and Identity Calculation**

For each window pair, VAMPIRE:

1. **Performs alignment**: Uses edlib with semi-global alignment (HW mode)
2. **Processes CIGAR string**:
   - Counts matched bases (`=`)
   - Counts mismatched bases (`X`)
   - Counts indels within range (`I`, `D`)
   - Ignores small indels (outside range)
3. **Calculates identity**:
```
identity = (matched_bases / (matched + mismatched + indels_in_range)) × 100
```

### **4. Extension for Precision**

To improve alignment accuracy, VAMPIRE:

1. **Extends query window**: Includes half of adjacent motif on each side
2. **Extends reference window**: Includes half of adjacent motif on each side
3. **Aligns extended sequences**: Better anchor points for alignment
4. **Calculates identity**: Uses extended sequences but reports on original window

### **5. Output Generation**

For each window pair, VAMPIRE outputs:
- Query coordinates (name, start, end)
- Reference coordinates (name, start, end)
- Percentage identity (0-100)

---

## **Identity Calculation Details**

### **CIGAR String Processing**

VAMPIRE uses CIGAR strings from annotations to compute identity:

**CIGAR symbols:**
- `=`: Match (identical bases)
- `X`: Mismatch (substitution)
- `I`: Insertion (gap in reference)
- `D`: Deletion (gap in query)
- `N`: Masked region (typically excluded)

**Example CIGAR**: `10=2X5=3I8=2D10=`

Breakdown:
- 10 matched bases
- 2 substitutions
- 5 more matches
- 3 bp insertion
- 8 more matches
- 2 bp deletion
- 10 more matches

**Identity calculation** (without indels):
```
matched = 10 + 5 + 8 + 10 = 33
mismatched = 2
indels = 0 (if outside range)
identity = (33 / (33 + 2 + 0)) × 100 = 94.3%
```

**Identity calculation** (with indels in range):
```
matched = 33
mismatched = 2
indels = 3 + 2 = 5 (if 3 and 2 are in range)
identity = (33 / (33 + 2 + 5)) × 100 = 82.4%
```

### **Window Overlap**

Windows are **non-overlapping** partitions:
- Sequence of 1000 motifs with window size 100 → 10 windows
- Window 1: motifs 1-100
- Window 2: motifs 101-200
- Etc.

Each window is compared to every other window (including itself).

---

## **Output Interpretation**

### **BED File Format**

The output is in BED format with additional columns:

```
# Columns:                     Description:
# query_name                 Query sequence identifier
# query_start                Start position in query sequence
# query_end                  End position in query sequence
# reference_name             Reference sequence identifier
# reference_start            Start position in reference sequence
# reference_end              End position in reference sequence
# perID_by_events           Percent identity (0-100)
```

**Note**: The `#` header line is included for documentation.

### **Identity Values**

- **100%**: Identical sequences (same window compared to itself)
- **High (90-99%)**: Very similar sequences
- **Medium (70-90%)**: Moderately similar, some variation
- **Low (50-70%)**: Distinct sequences
- **Very low (<50%)**: Highly divergent sequences

### **Using the Output**

**In UCSC Genome Browser:**
```bash
# Convert to BED6 format for loading
# Remove header line
tail -n +2 identity_output.bed > identity_output_nohdr.bed

# Load in genome browser
```

**In R (visualization):**
```R
library(GenomicRanges)
bed <- read.table("identity_output.bed", header=TRUE, sep="\t")
gr <- GRanges(seqnames=bed$query_name,
              ranges=IRanges(start=bed$query_start,
                            end=bed$query_end),
              identity=bed$perID_by_events)
# Plot heatmap, etc.
```

**In Python (analysis):**
```python
import pandas as pd
import seaborn as sns

df = pd.read_csv("identity_output.bed", sep="\t", comment="#")

# Create pivot table for heatmap
pivot = df.pivot(index="query_name",
               columns="reference_name",
               values="perID_by_events")

# Plot
sns.clustermap(pivot)
```

---

## **Visualization Scripts**

VAMPIRE provides additional scripts for advanced visualization:

### **Generate Visualization Data**

```bash
python scripts/get_visualization_data.py \
  --prefix [annotation_prefix] \
  --repeat [repeatmasker_annotation] \
  --output [output_prefix]
```

**Purpose**: Prepares data for plotting with RepeatMasker annotation and strand information.

**Inputs**:
- `--prefix`: Annotation prefix from `anno`
- `--repeat`: RepeatMasker GFF or BED annotation file
- `--output`: Prefix for visualization data files

### **Plot Alignment Heatmap**

```bash
Rscript scripts/SG_aln_plot.R \
  -t [threshold] \
  -b [identity_bed_file] \
  -a [visualization_data] \
  -p [figure_output_prefix]
```

**Parameters:**
- `-t`: Identity threshold for coloring (default: 30)
- `-b`: Identity matrix BED file from `vampire identity`
- `-a`: Visualization data from `get_visualization_data.py`
- `-p`: Output prefix for figures

**Output**: High-quality PDF heatmap with:
- Sequence names and coordinates
- RepeatMasker annotation overlay
- Strand information
- Identity-based color gradient

---

## **Use Cases**

### **Population Structure Analysis**

Identify similarity groups across multiple sequences:

```bash
# Calculate identity matrix
vampire identity -w 50 annotation_prefix pop_structure

# Import into R for clustering
Rscript cluster_analysis.R pop_structure.bed

# Visualize population structure
# (Expect distinct groups for related sequences)
```

### **Quality Control**

Detect outlier sequences that may have annotation errors:

```bash
# Calculate identity
vampire identity annotation_prefix qc_check

# In Python:
import pandas as pd
df = pd.read_csv("qc_check.bed", sep="\t", comment="#")

# Check self-identity (should be 100%)
self_id = df[df['query_name'] == df['reference_name']]['perID_by_events']
if not all(self_id == 100):
    print("WARNING: Self-identity not 100%!")

# Find outliers (low average identity)
avg_id = df.groupby('query_name')['perID_by_events'].mean()
outliers = avg_id[avg_id < 80].index.tolist()
print(f"Potential outliers: {outliers}")
```

### **Phylogenetic Analysis**

Use identity matrix for phylogenetic tree construction:

```bash
# Calculate identity
vampire identity annotation_prefix phylo_data

# Convert to distance matrix
python bed_to_distance_matrix.py phylo_data.bed phylo_dist.tsv

# Build phylogenetic tree
iqtree -s phylo_dist.tsv -m GTR+G -bb 1000

# Visualize tree
figtree phylo_dist.treefile
```

### **Comparative Genomics**

Compare TR regions between species or strains:

```bash
# Species 1
vampire identity species1_anno species1_identity

# Species 2
vampire identity species2_anno species2_identity

# Cross-species comparison
# (Manually merge annotations or use custom script)

# Compare identity distributions
python compare_identities.py species1_identity.bed species2_identity.bed
```

### **Variant Detection**

Find divergent windows within a sequence:

```bash
# Calculate with small windows
vampire identity -w 30 annotation_prefix variant_detection

# Find low-identity windows
python detect_variants.py variant_detection.bed --threshold 50
```

---

## **Best Practices**

### **Window Size Selection**

Choose window size based on analysis goals:

```bash
# High resolution (detecting local variation)
vampire identity -w 30 annotation_prefix high_res

# Medium resolution (general patterns)
vampire identity -w 100 annotation_prefix medium_res

# Low resolution (global patterns)
vampire identity -w 200 annotation_prefix low_res
```

**Guidelines:**
- **30-50**: Fine-scale variation, copy number changes
- **100-150**: Regional variation, motif structure
- **200+**: Global patterns, sequence relationships

### **Indel Parameter Tuning**

Adjust based on biological relevance:

```bash
# No indels (default)
vampire identity annotation_prefix no_indels

# Small indels only (local events)
vampire identity --min-indel 3 --max-indel 20 annotation_prefix small_indels

# Large indels only (structural variants)
vampire identity --min-indel 50 --max-indel 500 annotation_prefix large_indels

# All indels
vampire identity --min-indel 1 --max-indel 1000 annotation_prefix all_indels
```

### **Memory Management**

For large datasets:

```bash
# Reduce thread count if memory issues
vampire identity -t 10 annotation_prefix output

# Process subsets first
vampire identity annotation_prefix output_subset1
# Then merge or analyze separately
```

---

## **Common Issues**

### **Long computation time**

**Cause**: Large number of window pairs (all-vs-all comparison).

**Solutions**:
1. Increase window size to reduce number of windows
2. Use `--thread` to parallelize
3. Analyze subsets of data first

### **Memory errors**

**Cause**: Storing many alignment results simultaneously.

**Solutions**:
1. Reduce thread count
2. Process smaller datasets
3. Increase system memory or use a machine with more RAM

### **All 100% identity**

**Cause**: Windows are identical (expected for same sequence comparison).

**This is normal behavior** - windows compared to themselves should be 100% identity.

### **Unexpected low identity**

**Cause**: May indicate:
- Annotation errors
- Sequence contamination
- Wrong window size

**Solutions**:
1. Check annotation quality with `vampire evaluate`
2. Verify input sequences are correct
3. Adjust window size

### **Empty output**

**Cause**: Input files missing or no windows generated.

**Solution**: Verify input files exist and have valid data:
```bash
# Check required files
ls -lh anno.motif.tsv anno.dist.tsv anno.concise.tsv anno.annotation.tsv

# Verify motif count
wc -l anno.motif.tsv
```

---

## **Integration with Other Tools**

### **UCSC Genome Browser**

```bash
# Format for UCSC (requires BED6)
awk 'NR>1' identity_output.bed > ucsc.bed

# Load in browser or use track hubs
```

### **IGV (Integrative Genomics Viewer)**

```bash
# Standard BED format loads directly
igv identity_output.bed
```

### **BEDTools**

```bash
# Intersect with other annotations
bedtools intersect -a identity_output.bed -b other_features.bed

# Find overlapping low-identity windows
awk '$7 < 50' identity_output.bed | bedtools intersect -a - -b genes.bed
```

---

## **See Also**

- [Parameters](../parameters.html) - Full parameter reference
- [anno](anno.html) - Generate annotations for identity calculation
- [evaluate](evaluate.html) - Assess motif quality before identity analysis
