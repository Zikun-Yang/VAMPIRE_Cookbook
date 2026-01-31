---
title: evaluate
parent: Usage
layout: default
nav_order: 4
---

# **evaluate**
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## **Introduction**

The `evaluate` command assesses the quality of VAMPIRE annotations by computing edit distance matrices between actual observed motifs and reference motifs. It generates clustered heatmaps to visualize motif relationships and annotation accuracy.

### Key Features

- **Multi-metric evaluation**: Raw edit distances and normalized comparisons
- **Strand-aware**: Handles merged and separate strand analysis
- **Visual output**: Clustered heatmaps with hierarchical clustering
- **Configurable thresholds**: Adjustable outlier detection and color scales

### Evaluation Modes

**Distance modes:**
- **raw**: Direct edit distances between motifs
- **normalized**: Absolute difference between distances to reference (corrects for reference motif quality)

**Strand modes:**
- **merged**: Treats forward and reverse complement motifs as the same
- **separate**: Keeps forward and reverse complement motifs distinct

---

## **Input and Output**

### **Input**

| Input         | Format | Description                                   | Default |
|:------------- |:------ |:----------------------------------------------|:--------|
| Annotation    | TSV    | Annotation file from `anno` command         | None    |
| Motif         | TSV    | Motif statistics file from `anno` command | None    |

### **Output**

| Output              | Format | Description                                | Default |
|:--------------------|:------ |:-------------------------------------------|:--------|
| Distance Matrix      | TSV    | Edit distance matrix between motifs            | None    |
| Clustered Heatmap    | PDF    | Hierarchically clustered heatmap visualization | None    |

**Four output combinations** (for each mode/strand combination):
- `{output}_raw_merged.pdf` - Raw distances, merged strands
- `{output}_raw_separate.pdf` - Raw distances, separate strands
- `{output}_normalized_merged.pdf` - Normalized distances, merged strands
- `{output}_normalized_separate.pdf` - Normalized distances, separate strands

---

## **Usage Examples**

### **1. Basic evaluation**
Generate all four evaluation plots with default settings.
```bash
vampire evaluate annotation_prefix eval_output
```

### **2. Adjusted thread count**
Use more threads for faster processing on large datasets.
```bash
vampire evaluate -t 16 annotation_prefix eval_output
```

### **3. Custom outlier threshold**
Adjust the percentile for heatmap color scaling.
```bash
vampire evaluate -p 90 annotation_prefix eval_output
```

### **4. Show distance values**
Display edit distance values on heatmap cells.
```bash
vampire evaluate -s annotation_prefix eval_output
```

---

## **Parameters**

### **Required Parameters**

| Parameter | Type   | Description                                   | Default |
|:-----------|:-------|:----------------------------------------------|:--------|
| `prefix`   | string | Input prefix from `anno` command                 | Required |
| `output`   | string | Output prefix for evaluation results and figures  | Required |

### **Optional Parameters**

| Parameter              | Type    | Description                                                                              | Default |
|:----------------------|:--------|:-----------------------------------------------------------------------------------------|:--------|
| `-t, --thread`       | integer | Number of threads for parallel distance calculation                                         | 6       |
| `-p, --percentage`   | integer | Threshold percentile for identifying abnormal values in heatmap (0-100)                        | 75      |
| `-s, --show-distance` | boolean | Display actual edit distance values on heatmap cells                                      | False   |

### **Parameter Details**

**Thread count**: Higher values speed up processing but increase memory usage. Typical range: 4-16.

**Percentage threshold**: Controls heatmap color scaling. Values at or above this percentile saturate the red color. Lower values increase contrast for small distances.

---

## **Understanding the Output**

### **Distance Matrix File**

Tab-separated file with motif IDs as rows and columns:
```
ref_motif	motif0+	motif1-	motif2+	motif3-
motif0+	0.00	2.50	1.00	3.50
motif1-	2.50	0.00	1.50	2.00
...
```

- `motifX+`: Forward strand motif with ID X
- `motifX-`: Reverse complement of motif with ID X

### **Heatmap Visualization**

**Interpretation:**

- **Red cells**: Large edit distances (poor matches)
- **White cells**: Small edit distances (good matches)
- **Diagonal**: Self-comparisons (always 0)
- **Clustering**: Similar motifs group together

**Raw vs Normalized:**

- **Raw**: Direct edit distances. Useful for assessing overall motif diversity.
- **Normalized**: Differences in distance from reference. Useful for assessing annotation consistency - if a motif is far from its reference, it may be misannotated.

**Merged vs Separate:**

- **Merged**: Combines forward and reverse complement. Reduces complexity when strand direction is not biologically meaningful.
- **Separate**: Keeps strands distinct. Important when strand-specific patterns matter.

### **Cluster Dendrogram**

The hierarchical clustering groups similar motifs. This helps:

- Identify motif families
- Detect potential annotation errors (outliers)
- Understand motif variation patterns

---

## **Interpreting Results**

### **Good Quality Indicators**

- **Diagonal dominance**: Cells near the diagonal are light (low distances)
- **Clear clusters**: Motifs form cohesive groups
- **Low variance**: Within-cluster distances are small
- **Reference alignment**: Each motif is close to its labeled reference

### **Potential Issues**

**Scattered heatmap** (no clear structure):
- Annotation may have errors
- k-mer size may be inappropriate
- Consensus motifs may be too divergent

**Outlier motifs** (far from all clusters):
- May be annotation errors
- Could represent rare variants
- Check manually in original sequence

**High intra-cluster variance**:
- High motif variation
- May need more lenient parameters
- Consider splitting into sub-groups

---

## **Workflow Integration**

### **Quality Control Pipeline**

```bash
# Step 1: Annotate
vampire anno -t 8 data.fa anno

# Step 2: Evaluate
vampire evaluate anno eval

# Step 3: Inspect heatmaps
# Open eval_*_*.pdf files

# Step 4: If issues found, refine
vampire refine anno refine_actions.tsv -o anno_refined

# Step 5: Re-evaluate
vampire evaluate anno_refined eval_refined

# Step 6: Compare results
# Compare eval and eval_refined heatmaps
```

### **Parameter Optimization**

```bash
# Test different k-mer sizes
for k in 7 9 11 13; do
  vampire anno -k $k data.fa anno_k$k
  vampire evaluate anno_k$k eval_k$k
  # Inspect eval_k$k_raw_merged.pdf
done
```

---

## **Common Issues**

### **Long computation time**

**Cause**: Large number of motifs with all-vs-all comparisons.

**Solutions:**
1. Increase `--thread` count
2. Reduce motif number with `--motifnum` in anno
3. Evaluate subsets of data first

### **Heatmap all red**

**Cause**: Distances exceed color scale threshold.

**Solutions:**
1. Decrease `--percentage` threshold (e.g., 50)
2. Check if annotation has errors (try de novo mode)
3. Verify motif database is appropriate

### **Empty heatmap**

**Cause**: Input files missing or malformed.

**Solutions:**
1. Verify input prefix: `ls anno.annotation.tsv anno.motif.tsv`
2. Check annotation file format
3. Ensure motifs file is not empty

---

## **See Also**

- [Parameters](../parameters.html) - Full parameter reference
- [anno](anno.html) - Generate annotations to evaluate
- [refine](refine.html) - Fix issues identified by evaluate
