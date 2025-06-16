---
title: identity
parent: Usage
layout: default
nav_order: 15
---

# **identity**
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## **Introduction**

The `identity` command in VAMPIRE is designed to calculate the self-identity of one TR sequence.

---

## **Input and Output**

### **Input**

| Input         | Format              | Description                                              | Default |
|:------------- |:------------------- |:---------------------------------------------------------|:--------|
| Annotation    | TSV                 | Detailed annotation results from anno command            | None    |
| Motif         | TSV                 | Statistics of detected motifs from anno command          | None    |

### **Output**

| Output             | Format | Description                                    | Default |
|:------------------ |:------ |:---------------------------------------------- |:--------|
| Identity Matrix    | BED    | Identity between two windows                   | None    |

---

## **Usage Examples**

### **1. plot raw identity heatmap**
```bash
vampire identity -t 30 -w 30 annotation_prefix output_prefix
```

### **2. plot inverted identity heatmap**
```bash
vampire identity -t 30 -w 30 --mode="invert" annotation_prefix output_prefix
```

### **3. plot identity heatmap including INDELs**
```bash
vampire identity -t 30 -w 30 --min-indel 1 --max-indel 10 annotation_prefix output_prefix
```

---

## **Parameters**

