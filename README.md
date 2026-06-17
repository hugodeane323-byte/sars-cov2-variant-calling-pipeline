# sars-cov2-variant-calling-pipeline
End-to-end SARS-CoV-2 variant calling pipeline using FastQC, BWA, SAMtools and BCFtools

## Overview

This project demonstrates an end-to-end next-generation sequencing (NGS) workflow for identifying genomic variants in SARS-CoV-2 sequencing data.

## Dataset

- Sample: SRR17054502
- Reference Genome: NC_045512.2 (Wuhan-Hu-1)

## Workflow

1. Quality Control (FastQC)
2. Reference Genome Indexing (BWA)
3. Read Alignment (BWA MEM)
4. BAM Processing (SAMtools)
5. Variant Calling (BCFtools)
6. Variant Interpretation

## Tools

- FastQC
- BWA
- SAMtools
- BCFtools

## Results

- 38 variants identified
- Coverage depth calculated
- Variant annotation performed

## Skills Demonstrated

- Linux command line
- NGS data analysis
- Sequence alignment
- Variant calling
- VCF interpretation
