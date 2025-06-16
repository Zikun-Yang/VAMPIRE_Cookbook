---
title: anno
parent: Usage
layout: default
nav_order: 3
---

# **anno**

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}s

---

## **Introduction**

The `anno` command in VAMPIRE is used to annotate tandem repeat loci with motif information and variation patterns. This is the core function for analyzing how motifs vary across different loci in an assembly.

It takes a FASTA file of tandem repeat loci as input and produces an annotated TSV table along with optional motif logo plots.

---

## **Usage Examples**

### **1. *de novo* annotate**
```bash

```

### **2. ** 
```bash

```

### **3. decompose using a consensus motif**
```bash

```


## **Input and Output**

### **Input:**

| Input      | Format | Description                                                    | Default |
|:---------- |:------ |:---------------------------------------------------------------|:--------|
| Sequence   | FASTA  | The sequence to be annotated                                   | None    |
| Motif DB   | FASTA  | Motif database used to refine and align motifs during analysis | base    |


### **Output**

| Output     | Format | Description                               | Default |
|:---------- |:------ |:------------------------------------------|:--------|
| Config     | JSON   | Parameter set used for the annotation job | None    |
| Annotation | TSV    | Detailed annotation results               | None    |
| Concise    | TSV    | Summary of annotation results             | None    |
| Motif      | TSV    | Statistics of detected motifs             | None    |
| Distance   | TSV    | Edit distances between detected motifs    | None    |

## **Parameters**




