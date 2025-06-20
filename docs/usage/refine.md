---
title: refine
parent: Usage
layout: default
nav_order: 11
---

# **refine**
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## **Introduction**

The `refine` command in VAMPIRE is designed to refine the annotation results based on user-defined actions.

---

## **Input and Output**

### **Input**

| Input         | Format              | Description                                              | Default |
|:------------- |:------------------- |:---------------------------------------------------------|:--------|
| Config        | JSON                | Parameter set used for the annotation job                | None    |
| Annotation    | TSV                 | Detailed annotation results from anno command            | None    |
| Motif         | TSV                 | Statistics of detected motifs from anno command          | None    |
| Action        | TSV                 | Refinement action provided by user                       | None    |

### **Output**

| Output             | Format | Description                                    | Default |
|:------------------ |:------ |:---------------------------------------------- |:--------|
| Refined Annotation | TSV    | Detailed annotation results (Refined)          | None    |
| Refined Concise    | TSV    | Summary of annotation results (Refined)        | None    |
| Refined Motif      | TSV    | Statistics of detected motifs  (Refined)       | None    |
| Refined Distance   | TSV    | Edit distances between detected motifs         | None    |


---

## **Usage Examples**

```bash
vampire refine -t 6 -o annotation_prefix.refined annotation_prefix action.tsv
```

---

## **Parameters**

