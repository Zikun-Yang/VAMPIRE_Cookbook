---
title: logo
parent: Usage
layout: default
nav_order: 13
---

# **logo**
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## **Introduction**

The `logo` command in VAMPIRE is designed to analyze and visualize the motif variation. It can plot sequence LOGO in three types -- `count`, `probability` and `information` based on `motif` file or `annotation` file.

---

## **Input and Output**

### **Input**

| Input         | Format              | Description                                              | Default |
|:------------- |:------------------- |:---------------------------------------------------------|:--------|
| Annotation    | TSV                 | Detailed annotation results from anno command            | None    |
| Motif         | TSV                 | Statistics of detected motifs from anno command          | None    |


### **Output**

| Output             | Format     | Description                                    | Default |
|:------------------ |:---------- |:---------------------------------------------- |:--------|
| Base Counts        | TSV        | Counts of A/T/C/G/- for each loci              | None    |
| Base Counts        | PDF/PNG    | Seq logo in count                              | None    |
| Base Probability   | PDF/PNG    | Seq logo in probability                        | None    |
| Base Information   | PDF/PNG    | Seq logo in information                        | None    |


---

## **Usage Examples**

```bash
vampire logo -t motif -f pdf annotation_prefix output_prefix
```

---

## **Parameters**

