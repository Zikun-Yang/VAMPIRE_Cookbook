---
title: mkref
parent: Usage
layout: default
nav_order: 7
---

# **mkref**
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## **Introduction**

The `mkref` command in VAMPIRE is designed to generate motif database (FASTA format) from motif file from `anno` command.

---

## **Input and Output**

### **Input:**

| Input         | Format              | Description                                              | Default |
|:------------- |:------------------- |:---------------------------------------------------------|:--------|
| Motif         | TSV                 | Motif file from anno command                             | None    |

### **Output**

| Output            | Format | Description                                    | Default |
|:------------------|:------ |:---------------------------------------------- |:--------|
| Motif Database    | FASTA  | Motif extracted from motif file                | None    |


---

## **Usage Examples**

```bash
vampire mkref annotation_prefix motif.fasta
```

---

## **Parameters**




