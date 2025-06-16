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

The `anno` command in VAMPIRE is designed to annotate tandem repeat loci with detailed motif information and variation patterns.

By providing a FASTA file containing tandem repeat loci, `anno` generates a list of annotated TSV file. These files are the input for `evaluate`, `refine`, `logo`, `identity` command.

---

## **Input and Output**

### **Input**

| Input      | Format | Description                                                    | Default |
|:---------- |:------ |:---------------------------------------------------------------|:--------|
| Sequence   | FASTA  | The sequence to be annotated                                   | None    |
| Motif DB   | FASTA  | Motif database used to refine and align motifs during analysis | base    |


### **Output**

| Output     | Format | Description                                | Default |
|:---------- |:------ |:-------------------------------------------|:--------|
| Config     | JSON   | Parameter set used for the annotation job  | None    |
| Annotation | TSV    | Detailed annotation results                | None    |
| Concise    | TSV    | Summary of annotation results              | None    |
| Motif      | TSV    | Statistics of detected motifs              | None    |
| Distance   | TSV    | Edit distances between detected motifs     | None    |

---

## **Usage Examples**

### **1. *de novo* annotate**
Annotate tandem repeat loci directly from a FASTA file, discovering motifs automatically.
```bash
vampire anno [parameters] repeats.fasta annotation_prefix
```

### **2. decompose using a consensus motif or a motif set**
Decompose tandem repeats using a provided consensus motif or motif set, without de novo motif discovery.
```bash
vampire anno [parameters] --no-denovo -m consensus_motif.fasta repeats.fasta annotation_prefix
```

### **3. annotate population-level sequence**
Annotating tandem repeat (TR) loci at the population level can be challenging due to highly variable copy numbers among individuals. For loci with low copy numbers, accurate annotation is more difficult. In such cases, providing a curated motif set can improve annotation quality and consistency across samples. A three-step strategy can be applied:

**STEP-1:** Call the motif set from the population sequences
```bash
vampire anno [parameters] population_sequences.fasta population_annotation
```

**STEP-2:** Create a motif database from the annotation
```bash
vampire mkref population_annotation population_motif_set.fasta
```

**STEP-3:** Use the curated motif database to annotate the population sequences
```bash
vampire anno [parameters] --no-denovo -m population_motif_set.fasta population_sequences.fasta population_annotation.curated
```

---

## **Parameters**




