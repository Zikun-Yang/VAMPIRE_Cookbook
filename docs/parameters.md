---
title: Parameters
layout: default
nav_order: 4
---

# Parameters
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

I/O Options:
  input                 FASTA file you want to annotate
  output                output prefix

## General parameters

| parameters        | description          | default |
|:-------------|:------------------|:------|
| -c, --config CONFIG   | config file in JSON, CONFLICT with --AUTO | None |
| -t, --thread THREAD   | number of threads | 1 |
| --AUTO                | automatically estimate parameters | False |
| --debug               | output running time of each module and alignments | False |
| -w, --window-length | parallel window size | 5000 |
| -o, --overlap-length | windows overlap size | 1000 |
| -r, --resource | memory limit (GB) | 50 |

## Decomposition parameters

| parameters        | description          | default |
|:-------------|:------------------|:------|
| -k, --ksize      | k-mer size for building De Bruijn graph | 5 |
| -m, --motif      | reference motif set path | None |
| -n, --motifnum | maximum number of motifs | 30 |
| --abud-threshold | min threshold campared with top edge weight | 0.01 |
| --abud-min | min edge weight in De Bruijn graph | 3 |
| --plot | paint De bruijn graph for each windows | False |
| --no-denovo | not de novo find motifs, use reference motifs to annotate | False |

## Annotation parameters

| parameters        | description          | default |
|:-------------|:------------------|:------|
| --force | add reference motifs into annotation | False |
| --annotation-dist-ratio | max distance to map | 0.4 |
| --finding-dist-ratio | max distance to query in reference motif set | 0.4 |
| --match-score | score per matched base | 1 |
| --lendif-penalty | hope that mapped sequence is as long as the motif | 0.01 |
| --gap-penalty | penalty per skipped base | 1 |
| --distance_penalty | penalty per distance | 1.5 |
| --perfect-bonus | bonus per matched base if there is no difference in alignment | 0.5 |

## Output parameters

| parameters        | description          | default |
|:-------------|:------------------|:------|
| --quiet | don't output thread completion informing | False |
| --file | minimum output score | 5 |
| -s, --score | minimum output score | 5 |

