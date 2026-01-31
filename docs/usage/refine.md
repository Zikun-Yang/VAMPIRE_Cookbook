---
title: refine
parent: Usage
layout: default
nav_order: 6
---

# **refine**
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## **Introduction**

The `refine` command performs manual curation of VAMPIRE annotations by merging, replacing, or deleting motifs based on user-defined actions. This is essential for:

- **Correcting annotation errors**: Fix motifs that were incorrectly assigned
- **Merging similar motifs**: Combine variants that should be treated as one
- **Removing artifacts**: Delete spurious motifs from noise or assembly errors
- **Standardizing labels**: Ensure consistent motif naming across samples

### Supported Operations

Three refinement operations are supported:

- **MERGE**: Combine multiple motifs into a single alternative
- **REPLACE**: Substitute one motif with another
- **DELETE**: Remove motifs and reassign affected regions

---

## **Input and Output**

### **Input**

| Input         | Format | Description                                              | Default |
|:------------- |:------ |:---------------------------------------------------------|:--------|
| Annotation    | TSV    | Annotation file from `anno` command                   | None    |
| Action File   | TSV    | User-defined refinement actions (source → operation)      | None    |
| Motif         | TSV    | Motif statistics file from `anno` command          | None    |
| Settings      | JSON   | Parameter settings from `anno` command                | None    |

### **Output**

| Output         | Format | Description                                | Default |
|:-------------- |:------ |:-------------------------------------------|:--------|
| Annotation    | TSV    | Refined annotation file                  | None    |
| Concise       | TSV    | Collapsed summary of refined annotations  | None    |
| Motif         | TSV    | Updated motif statistics file              | None    |
| Dist          | TSV    | Updated distance matrix                    | None    |

**Output files** (with default naming `{prefix}.revised`):
- `{prefix}.revised.annotation.tsv` - Refined per-instance annotations
- `{prefix}.revised.concise.tsv` - Collapsed blocks after refinement
- `{prefix}.revised.motif.tsv` - Updated motif counts and labels
- `{prefix}.revised.dist.tsv` - Filtered distance matrix

---

## **Usage Examples**

### **1. Merge similar motifs**

Merge motifs 0 and 1 into the best alternative based on edit distance.
```bash
# Action file content:
# source    action
# 0,1       MERGE

vampire refine annotation_prefix merge_actions.tsv -o refined_anno
```

### **2. Replace motif with another**

Replace motif 2 with motif 5 throughout the annotation.
```bash
# Action file content:
# source          action
# 2       REPLACE

vampire refine annotation_prefix replace_actions.tsv -o refined_anno
```

### **3. Delete motif**

Remove motif 3 and reassign affected regions to best alternatives.
```bash
# Action file content:
# source    action
# 3          DELETE

vampire refine annotation_prefix delete_actions.tsv -o refined_anno
```

### **4. Multiple actions**

Perform multiple refinements in one run.
```bash
# Action file content:
# source    action
# 0,1       MERGE
# 3          DELETE
# 4       REPLACE

vampire refine annotation_prefix multi_actions.tsv -o refined_anno
```

### **5. Strand-specific refinement**

Operate only on specific strand orientations.
```bash
# Action file content:
# source    action
# 0+         MERGE
# 1-         DELETE

vampire refine annotation_prefix strand_actions.tsv -o refined_anno
```

---

## **Parameters**

### **Required Parameters**

| Parameter | Type   | Description                                     | Default |
|:-----------|:-------|:------------------------------------------------|:--------|
| `prefix`   | string | Input prefix from `anno` command         | Required |
| `action`   | string | Path to action TSV file                   | Required |

### **Optional Parameters**

| Parameter      | Type    | Description                                         | Default |
|:--------------|:--------|:----------------------------------------------------|:--------|
| `-o, --out`   | string | Output prefix (defaults to `{prefix}.revised`) | `{prefix}.revised` |
| `-t, --thread` | integer | Number of threads for parallel processing              | 8       |

---

## **Action File Format**

### **Structure**

The action file is a TSV file with two columns:
1. **source**: Motif ID(s) to modify
2. **action**: Operation type (MERGE, REPLACE, DELETE)

### **Format Specifications**

**Source column syntax:**
- Single motif: `0`, `0+`, or `0-`
- Multiple motifs: `0,1,2` (comma-separated, no direction)
- With direction: `0+,1-` (comma-separated with direction)

**Direction codes:**
- `+` or blank: Forward strand only
- `-`: Reverse complement only
- No suffix: Both strands

**Action column:**
- `MERGE`: Combine motifs into best alternative
- `REPLACE`: Substitute one motif with another
- `DELETE`: Remove motifs entirely

### **Examples**

**MERGE syntax:**
```
# Merge motifs 0 and 1 (both strands)
0,1    MERGE

# Merge only forward strand of motif 0
0+    MERGE

# Merge multiple motifs
0,1,2,3    MERGE
```

**REPLACE syntax:**
```
# Replace motif 2 with motif 5 (both strands)
2    REPLACE

# Replace specific strand
0-    REPLACE

# Replace motif specified in source column
0,1    REPLACE
```

**DELETE syntax:**
```
# Delete motif 3 (both strands)
3    DELETE

# Delete only reverse complement of motif 4
4-    DELETE

# Delete multiple motifs
5,6,7    DELETE
```

### **Complete Action File Example**

```
source	action
0,1	MERGE
3	DELETE
0	REPLACE
4+	MERGE
5-	DELETE
```

---

## **Operation Details**

### **MERGE**

**Purpose**: Combine multiple motifs into a single representative.

**Algorithm**:
1. Identifies all instances of source motifs
2. For each alternative motif, computes total edit distance to all instances
3. Selects alternative with minimum total distance
4. Reassigns instances if distance < threshold

**Use cases**:
- Merging highly similar motif variants
- Correcting over-fragmentation
- Combining motifs with low copy number

**Example workflow**:
```bash
# After evaluation, find motifs 0 and 1 are very similar
vampire evaluate anno eval

# Check eval_raw_merged.pdf - motifs 0 and 1 cluster together

# Merge them
echo -e "0,1\tMERGE" > merge_action.tsv
vampire refine anno merge_action.tsv -o anno_merged
```

### **DELETE**

**Purpose**: Remove motifs from annotation entirely.

**Algorithm**:
1. Identifies all instances of target motifs
2. Finds best alternative motif (not in delete list) for each instance
3. Reassigns instances if distance < threshold
4. Removes deleted motifs from all output files

**Use cases**:
- Removing spurious motifs from noise
- Eliminating artifacts from assembly errors
- Deleting low-quality motif assignments

**Example workflow**:
```bash
# Check motif counts
cat anno.motif.tsv

# Motif 7 has only 2 occurrences - likely noise
echo -e "7\tDELETE" > delete_action.tsv
vampire refine anno delete_action.tsv -o anno_clean
```

### **REPLACE**

**Purpose**: Substitute one motif with another throughout annotation.

**Algorithm**:
1. Identifies all instances of old motif
2. For each instance, checks if new motif is a good match
3. Reassigns instances if distance < threshold
4. Removes instances that don't match well

**Use cases**:
- Correcting misassigned motif IDs
- Standardizing motif names across samples
- Applying corrections from manual inspection

**Example workflow**:
```bash
# Manual inspection shows motif 2 should be motif 5
echo -e "2\tREPLACE" > replace_action.tsv
vampire refine anno replace_action.tsv -o anno_corrected
```

---

## **Validation and Quality Control**

### **Before Refinement**

1. **Review motif statistics**:
```bash
# Check copy numbers and labels
cat anno.motif.tsv | column -t
```

2. **Examine evaluation results**:
```bash
# Look for outliers or unexpected clusters
open anno_eval_raw_merged.pdf
```

3. **Inspect specific motifs**:
```bash
# View instances of motif 0
grep -P '\t0\t' anno.annotation.tsv
```

### **After Refinement**

1. **Verify changes**:
```bash
# Compare motif counts before/after
wc -l anno.motif.tsv anno_revised.motif.tsv
```

2. **Re-evaluate**:
```bash
# Check if refinement improved quality
vampire evaluate anno_revised anno_revised_eval
```

3. **Cross-check with sequence**:
```bash
# Verify assignments make biological sense
grep 'contig1' anno_revised.annotation.tsv
```

---

## **Best Practices**

### **Iterative Refinement**

For complex datasets, refine in multiple rounds:

```bash
# Round 1: Remove obvious artifacts
vampire refine anno round1_delete.tsv -o anno_r1

# Round 2: Merge similar variants
vampire evaluate anno_r1 anno_r1_eval
vampire refine anno_r1 round2_merge.tsv -o anno_r2

# Round 3: Final corrections
vampire evaluate anno_r2 anno_r2_eval
vampire refine anno_r2 round3_final.tsv -o anno_final
```

### **Population Consistency**

When working with multiple samples:

```bash
# Step 1: Refine sample 1
vampire refine sample1 refine1.tsv -o sample1_refined

# Step 2: Check if same motifs need refinement in sample 2
# Compare sample1_refined.motif.tsv and sample2.motif.tsv

# Step 3: Apply consistent actions
vampire refine sample2 refine2.tsv -o sample2_refined
vampire refine sample3 refine3.tsv -o sample3_refined
```

### **Threshold Management**

The distance threshold is automatically set from annotation settings (`annotation_dist_ratio`). To adjust:

1. **Modify original annotation** parameters
2. **Re-run anno** with new threshold
3. **Refine** with new annotation

Or manually edit `annotation_dist_ratio` in `.setting.json` before refining.

---

## **Common Issues**

### **No changes after refine**

**Cause**: Distance threshold too high or too low.

**Solution**: Check settings and adjust `annotation_dist_ratio` in original annotation.

```bash
# Check current threshold
cat anno.setting.json | grep annotation_dist_ratio

# Re-anno with adjusted threshold if needed
vampire anno --annotation-dist-ratio 0.6 data.fa anno
```

### **Too many deletions**

**Cause**: Aggressive delete actions without good alternatives.

**Solution**: Delete motifs gradually, re-evaluating after each step.

### **Merge results in wrong motif**

**Cause**: Source motifs too diverse.

**Solution**: Merge more similar motifs first, or use REPLACE for specific instances.

### **Strand confusion**

**Cause**: Motif on wrong strand due to action specification.

**Solution**: Specify direction explicitly (+ or -) in source column.

```bash
# Check strand distribution
cut -f5 anno.annotation.tsv | sort | uniq -c

# Use strand-specific actions
echo -e "0+\tMERGE" > action.tsv
```

---

## **See Also**

- [Parameters](../parameters.html) - Full parameter reference
- [anno](anno.html) - Generate annotations to refine
- [evaluate](evaluate.html) - Assess refinement quality
