---
title: evaluate
parent: Usage
layout: default
nav_order: 9
---

# **evaluate**
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## **Introduction**

The `evaluate` command in VAMPIRE is designed to evaluate the quality of annotation.

---

## **Input and Output**

### **Input**

| Input              | Format              | Description                                              | Default |
|:------------------ |:------------------- |:---------------------------------------------------------|:--------|
| Annotation         | TSV                 | Detailed annotation results from anno command            | None    |
| Motif              | TSV                 | Statistics of detected motifs from anno command          | None    |

### **Output**

| Output                               | Format | Description                                               | Default |
|:------------------------------------ |:------ |:--------------------------------------------------------- |:--------|
| Distance Matrix (raw, merged)        | PDF    | Distance matrix (raw values, merged direction)            | None    |
| Distance Matrix (raw, separate)      | PDF    | Distance matrix (raw values, separate direction)          | None    |
| Distance Matrix (normalized, merged) | PDF    | Distance matrix (normalized values, merged direction)     | None    |
| Distance Matrix (normalized, separate)| PDF   | Distance matrix (normalized values, separate direction)   | None    |
| Distance Matrix (raw, merged)        | TSV    | Distance matrix (raw values, merged direction)            | None    |
| Distance Matrix (raw, separate)      | TSV    | Distance matrix (raw values, separate direction)          | None    |
| Distance Matrix (normalized, merged) | TSV    | Distance matrix (normalized values, merged direction)     | None    |
| Distance Matrix (normalized, separate)| TSV   | Distance matrix (normalized values, separate direction)   | None    |

---

## **Usage Examples**

```bash
vampire evaluate -t 30 evaluate annotation_prefix output_prefix
```

---

## **Parameters**

