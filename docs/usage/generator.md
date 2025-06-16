---
title: generator
parent: Usage
layout: default
nav_order: 5
---

# **generator**
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## **Introduction**

The `generator` command in VAMPIRE is designed to generate simulated tandem repeat sequence with user-defined motifs, length and mutation rate. Random seed can be set to enable reproductive results.

---

## **Input and Output**

### **Input**

| Input         | Format              | Description                                              | Default |
|:------------- |:------------------- |:---------------------------------------------------------|:--------|
| Motifs        | string (parameter)  | One motif or a list of motifs to generate the sequence   | None    |
| Length        | integer (parameter) | Length of the simulated sequence                         | None    |
| Mutation Rate | float (parameter)   | Mutation rate for SNVs and INDELs                        | None    |
| Seed          | integer (parameter) | Random seed for reproducibility                          | 42      |

### **Output**

| Output            | Format | Description                                    | Default |
|:------------------|:------ |:---------------------------------------------- |:--------|
| Sequence          | FASTA  | Simulated tandem repeat sequence               | None    |
| Annotation        | TSV    | Annotation file with mutations                 | None    |
| Annotation_woMut  | TSV    | Annotation file before introducing mutations   | None    |

---

## **Usage Examples**

### **1. Generate a simple TR sequence**
```bash
vampire generator -m GGC -l 1000 -r 0.01 -p simulated_TR
```

### **2. Generate a TR sequence with multiple motifs**
```bash
vampire generator -m AATGG CCATT -l 1000 -r 0.01 -p simulated_TR
```

---

## **Parameters**

