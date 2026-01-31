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

The `generator` command in VAMPIRE is designed to generate simulated tandem repeat sequences with user-defined motifs, length, and mutation rate. This is useful for:

- **Benchmarking**: Test VAMPIRE's annotation performance on known patterns
- **Parameter tuning**: Explore effects of different settings
- **Method validation**: Verify algorithms work as expected

Random seed can be set to enable reproducible results.

---

## **Input and Output**

### **Input**

| Input         | Format              | Description                                              | Default |
|:------------- |:------------------- |:---------------------------------------------------------|:--------|
| Motifs        | string (parameter)  | One motif or a list of motifs to generate the sequence   | None    |
| Length        | integer (parameter) | Length of the simulated sequence                         | 1000    |
| Mutation Rate | float (parameter)   | Mutation rate for SNVs and INDELs                        | 0       |
| Seed          | integer (parameter) | Random seed for reproducibility                          | 42      |

### **Output**

| Output            | Format | Description                                    | Default |
|:------------------|:------ |:---------------------------------------------- |:--------|
| Sequence          | FASTA  | Simulated tandem repeat sequence               | None    |
| Annotation        | TSV    | Annotation file with mutations                 | None    |
| Annotation_woMut  | TSV    | Annotation file before introducing mutations   | None    |

**File descriptions:**

- `{prefix}.fa`: Simulated TR sequences in FASTA format
- `{prefix}.anno.tsv`: Annotation results including mutations (SNVs, indels)
- `{prefix}.anno_woMut.tsv`: Annotation of perfect tandem repeats before mutations

---

## **Usage Examples**

### **1. Generate a simple TR sequence**
Generate a perfect tandem repeat with motif "GGC".
```bash
vampire generator -m GGC -l 1000 -r 0 -p simple_tr
```

### **2. Generate TR with mutations**
Introduce mutations at 1% rate.
```bash
vampire generator -m GGC -l 1000 -r 0.01 -p mutated_tr
```

### **3. Generate multi-motif TR sequence**
Create a sequence alternating between two motifs.
```bash
vampire generator -m AATGG CCATT -l 1000 -r 0 -p multi_motif_tr
```

### **4. Reproducible generation**
Set a specific random seed for reproducible results.
```bash
vampire generator -m GGC -l 1000 -r 0.01 -s 12345 -p reproducible_tr
```

---

## **Parameters**

### **Required Parameters**

| Parameter             | Type   | Description                                              | Default |
|:---------------------|:-------|:---------------------------------------------------------|:--------|
| `-m, --motifs`      | string | One or more motif sequences (space-separated)                  | Required |
| `-p, --prefix`       | string | Output prefix for generated files                             | Required |

### **Optional Parameters**

| Parameter                | Type    | Description                                   | Default |
|:------------------------|:--------|:----------------------------------------------|:--------|
| `-l, --length`         | integer | Length of simulated tandem repeat sequence (bp) | 1000    |
| `-r, --mutation-rate`   | float   | Mutation rate (0-1) for SNVs and indels        | 0       |
| `-s, --seed`           | integer | Random seed for reproducibility                  | 42      |

### **Parameter Details**

**Mutation types:**
- **SNVs**: Single nucleotide variants (substitutions)
- **Indels**: Insertions and deletions of varying lengths

**Motif patterns:**
- Single motif: Repeats continuously (e.g., `GGCGGC GGC...`)
- Multiple motifs: Alternates in specified order (e.g., `AATGG CCATT AATGG CCATT...`)

---

## **Output Format**

### **FASTA File**
Standard FASTA format with sequence name and repeat sequence.

### **Annotation File (with mutations)**
Columns include motif ID, position, actual sequence, and mutation details.

### **Annotation File (without mutations)**
Shows the perfect tandem repeat structure before mutations were introduced.

---

## **Use Cases**

### **Parameter Tuning**
Generate sequences with known structure to test different k-mer sizes:
```bash
# Test k=5
vampire generator -m GGC -l 1000 -r 0 -p test_k5
vampire anno -k 5 test_k5.fa test_k5

# Test k=9
vampire generator -m GGC -l 1000 -r 0 -p test_k9
vampire anno -k 9 test_k9.fa test_k9
```

### **Benchmarking**
Compare annotation accuracy across mutation rates:
```bash
for rate in 0 0.01 0.05 0.1; do
  vampire generator -m GGC -l 1000 -r $rate -p benchmark_$rate
  vampire anno benchmark_$rate.fa benchmark_$rate
done
```

### **Multi-motif Validation**
Test annotation with complex motif patterns:
```bash
vampire generator -m AATGG CCATT -l 2000 -r 0.01 -p complex_tr
vampire anno complex_tr.fa complex_tr
vampire evaluate complex_tr complex_tr_eval
```

---

## **See Also**

- [Parameters](../parameters.html) - Full parameter reference
- [anno](anno.html) - Annotate the generated sequences
- [evaluate](evaluate.html) - Evaluate annotation quality on simulated data
