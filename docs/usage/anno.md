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
| Input        | Format        | Description                                          | Default |
|:-------------|:--------------|:-----------------------------------------------------|:--------|
| Sequence     | FASTA         | Sequence you want to annotate                        | None    |
| Motif DB     | FASTA         | Motif database used to polish (and align) the motifs | base    |


### **Output**
| Output       | Format        | Description                        | Default |
|:-------------|:--------------|:-----------------------------------|:--------|
| Config       | Json          | Parameter set used in the job      | None    |
| Annotation   | TSV           | Detailed annotation file           | None    |
| Concise      | TSV           | Concise version of annotation file | None    |
| Motif        | TSV           | Statistics of detected motifs      | None    |
| distance     | TSV           | Edit distance between motifs       | None    |

## **Parameters**




