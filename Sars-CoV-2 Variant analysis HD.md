# SARS-CoV-2 Omicron Variant Analysis

## BIOLSC903 – Bioinformatics Analysis on the Command Line

### Student:
Hugo Deane

### Project Aim

The aim of this project was to reconstruct and analyse a SARS-CoV-2 Omicron genome from raw paired-end sequencing reads using a command-line bioinformatics workflow. The analysis involved quality control, read alignment, variant calling, variant filtering, and biological interpretation of high-confidence genomic mutations relative to the Wuhan-Hu-1 reference genome (NC_045512.2).

---

# Project Directory Setup

A structured project directory was created to organise sequencing data, quality control reports, alignment files, variant files, and final outputs.

```bash
mkdir -p sars_cov2_project/{raw_data,qc,reference,aligned,variants,results}
```

The project directory structure was organised as follows:

BIOLSC903_SARS_CoV2_Project/
├── raw_data/
├── qc/
├── reference/
├── aligned/
├── variants/
└── results/

---

# Software Installation

Bioinformatics tools were installed on macOS using Homebrew package manager.

```bash
brew install fastqc bwa samtools bcftools sra-tools
```

The following software tools were used:

- FastQC
- BWA
- SAMtools
- BCFtools
- SRA Toolkit

Software versions were confirmed using:

```bash
fastqc --version
samtools --version
bcftools --version
```

---

# Data Acquisition

## Sequencing Reads

Paired-end SARS-CoV-2 sequencing reads were downloaded from the NCBI Sequence Read Archive (SRA).

Dataset accession:
- SRR17054502

The dataset corresponds to SARS-CoV-2 Omicron sequencing data from South Africa.

Reads were downloaded using the SRA Toolkit:

```bash
prefetch SRR17054502
fasterq-dump SRR17054502
```

Generated paired-read FASTQ files:

- SRR17054502_1.fastq
- SRR17054502_2.fastq

The FASTQ files were moved into the `raw_data` directory:

```bash
mv SRR17054502_1.fastq SRR17054502_2.fastq raw_data/
```

---

# Quality Control Analysis

FastQC was used to assess sequencing quality prior to downstream analysis.

Quality control analysis was performed using:

```bash
fastqc raw_data/SRR17054502_1.fastq raw_data/SRR17054502_2.fastq -o qc/
```

The `-o` parameter specified the output directory for FastQC reports.

Generated FASTQC reports:
- SRR17054502_1_fastqc.html
- SRR17054502_2_fastqc.html

---


## Quality Control Results

FastQC analysis demonstrated generally high sequencing quality across both paired-end read files.

### Read 1
- Per-base sequence quality passed
- High overall Phred quality scores (>Q30)
- Minimal adapter contamination detected
- Elevated sequence duplication levels observed

### Read 2
- Per-base sequence quality showed reduced quality towards the 3' end of reads
- Quality degradation at terminal bases is commonly observed in Illumina paired-end sequencing datasets
- Adapter content remained minimal
- Sequence duplication levels were elevated

High duplication levels were considered expected due to the deep coverage and small genome size characteristic of viral sequencing workflows.

As adapter contamination was minimal and overall sequencing quality remained acceptable, read trimming was not performed prior to alignment.

---

# Reference Genome Preparation

The Wuhan-Hu-1 SARS-CoV-2 reference genome (NC_045512.2) was downloaded from NCBI in FASTA format and saved as:

```text
reference/reference.fasta
```

Prior to sequence alignment, the reference genome was indexed using BWA. Reference indexing creates the necessary data structures required for efficient read mapping.

```bash
bwa index reference/reference.fasta
```

BWA indexing generated multiple reference index files including:

- reference.fasta.amb
- reference.fasta.ann
- reference.fasta.bwt
- reference.fasta.pac
- reference.fasta.sa

These files are required for downstream alignment using the BWA-MEM algorithm.

Reference indexing completed successfully with no errors.

---

# Read Alignment

Following quality control and reference genome preparation, paired-end SARS-CoV-2 sequencing reads were aligned to the Wuhan-Hu-1 reference genome using the BWA-MEM algorithm.

BWA-MEM is a widely used alignment algorithm optimised for aligning short Illumina sequencing reads to a reference genome. The algorithm identifies the most likely genomic location for each sequencing read based on sequence similarity.

Read alignment was performed using the following command:

```bash
bwa mem reference/reference.fasta \
raw_data/SRR17054502_1.fastq \
raw_data/SRR17054502_2.fastq > aligned/aligned.sam
```

Command components:

- `bwa mem` invoked the BWA-MEM alignment algorithm
- `reference/reference.fasta` specified the indexed SARS-CoV-2 reference genome
- `SRR17054502_1.fastq` represented forward paired-end reads
- `SRR17054502_2.fastq` represented reverse paired-end reads
- `>` redirected alignment output to a SAM file

The alignment process generated a SAM (Sequence Alignment Map) file containing mapped sequencing reads relative to the SARS-CoV-2 reference genome.

The resulting alignment file was saved as:

```text
aligned/aligned.sam
```

The SAM file contains information including:
- read identifiers
- genomic alignment positions
- mapping quality scores
- alignment flags
- CIGAR strings describing alignment structure

This SAM file is the primary alignment output for downstream BAM conversion, sorting, indexing, and variant calling analysis.

---

# SAM to BAM Conversion and BAM Processing

The initial alignment output was generated in SAM (Sequence Alignment Map) format. SAM files are human-readable text files but are inefficient for large-scale downstream analysis due to their large file size.

To improve storage efficiency and compatibility with downstream tools, the SAM file was converted into BAM (Binary Alignment Map) format using SAMtools.

SAM to BAM conversion was performed using:

```bash
samtools view -Sb aligned/aligned.sam > aligned/aligned.bam
```

Command components:

- `samtools view` was used to manipulate alignment files
- `-S` specified SAM input format
- `-b` specified BAM output format
- `>` redirected output into a BAM file

The resulting BAM file was saved as:

```text
aligned/aligned.bam
```

---

# BAM Sorting

The BAM alignment file was sorted by genomic coordinate using SAMtools.

Sorting is required for efficient downstream processing, including BAM indexing and variant calling.

Sorting was performed using:

```bash
samtools sort aligned/aligned.bam -o aligned/aligned.sorted.bam
```

Command components:

- `samtools sort` sorted alignment records by genomic position
- `-o` specified the output filename

The sorted BAM file was saved as:

```text
aligned/aligned.sorted.bam
```

---

# BAM Indexing

The sorted BAM file was indexed using SAMtools.

Indexing creates a `.bai` index file that enables rapid random access to genomic regions within the BAM file during downstream analysis.

BAM indexing was performed using:

```bash
samtools index aligned/aligned.sorted.bam
```

This generated the BAM index file:

```text
aligned.sorted.bam.bai
```

The sorted and indexed BAM file was subsequently used for alignment quality assessment and variant calling.

---

# Alignment Quality Assessment

Alignment quality statistics were assessed using SAMtools `flagstat`.

The following command was used:

```bash
samtools flagstat aligned/aligned.sorted.bam
```

This analysis provided summary statistics describing read mapping performance and paired-end alignment quality.

Key alignment statistics included:

- Total reads: 227,200
- Mapped reads: 130,192 (57.30%)
- Properly paired reads: 128,658 (56.88%)
- Singleton reads: 476 (0.21%)

The majority of mapped reads were successfully aligned as properly paired sequencing fragments, indicating acceptable overall alignment quality.

The observed mapping rate (~57%) was considered sufficient for downstream variant analysis. Reduced mapping percentages can occur due to low-quality reads, non-target sequences, primer artefacts, or sequence divergence relative to the reference genome.

Very low singleton read counts indicated strong concordance between paired-end sequencing reads during alignment.

---

# Coverage Depth Analysis

Sequencing coverage depth across the SARS-CoV-2 genome was assessed using SAMtools `depth`.

Coverage depth analysis was performed using:

```bash
samtools depth aligned/aligned.sorted.bam > aligned/depth.txt
```

This command generated a text file containing per-base sequencing depth across the reference genome.

The output file was saved as:

```text
aligned/depth.txt
```

The first few lines of the coverage depth file were inspected using:

```bash
head aligned/depth.txt
```

Example output:

```text
NC_045512.2    28    12
NC_045512.2    29    12
NC_045512.2    30    12
NC_045512.2    31    14
NC_045512.2    32    83
NC_045512.2    33    95
NC_045512.2    34    142
NC_045512.2    35    226
NC_045512.2    36    238
NC_045512.2    37    239
```

Output columns represent:

- Column 1: Reference genome identifier
- Column 2: Genomic position
- Column 3: Sequencing depth (number of aligned reads covering the position)

Coverage depth increased substantially across the early genomic positions, indicating strong sequencing support across the SARS-CoV-2 genome.

Coverage depth metrics were subsequently used during variant filtering to retain only high-confidence variants supported by sufficient read depth.

---

# Variant Calling

Genomic variants relative to the Wuhan-Hu-1 SARS-CoV-2 reference genome were identified using BCFtools.

Variant calling was performed using the sorted and indexed BAM alignment file generated during previous alignment processing steps.

The following command was used:

```bash
bcftools mpileup -f reference/reference.fasta aligned/aligned.sorted.bam | \
bcftools call -mv -Ov -o variants/variants.vcf
```

Command components:

- `bcftools mpileup` generated genotype likelihood information from aligned sequencing reads
- `-f reference/reference.fasta` specified the SARS-CoV-2 reference genome
- `aligned.sorted.bam` provided the sorted alignment file for analysis
- `bcftools call` identified genomic variants relative to the reference genome
- `-m` enabled multiallelic variant calling
- `-v` output variant sites only
- `-O v` specified VCF text output format
- `-o variants/variants.vcf` specified the output VCF filename

The variant calling workflow generated a Variant Call Format (VCF) file containing candidate genomic mutations identified within the SARS-CoV-2 sample.

The output VCF file was saved as:

```text
variants/variants.vcf
```

The VCF file contains detailed variant information including:
- genomic position
- reference allele
- alternate allele
- quality scores
- read depth metrics
- genotype information

The generated VCF file was inspected using:

```bash
head variants/variants.vcf
```

Representative variants identified included:
- 14408 C>T (ORF1ab)
- 23063 A>T (Spike N501Y)
- 21764 deletion (Spike Δ69-70)

Inspection confirmed successful generation of variant call records and VCF metadata headers required for downstream filtering and interpretation.

---

---

# Variant Filtering

Raw variant calls generated during the initial variant calling stage included both high-confidence and lower-confidence candidate mutations.

To retain only reliable genomic variants suitable for downstream biological interpretation, variants were filtered according to the thresholds specified in the given project brief.

Filtering criteria applied:
- Variant quality score (QUAL) ≥ 50
- Read depth (DP) > 50

Variant filtering was performed using the following command:

```bash
bcftools filter -i 'QUAL>=50 && DP>50' \
variants/variants.vcf > variants/filtered.vcf
```

Command components used:

- `bcftools filter` applied filtering criteria to variant records
- `-i` specified inclusion conditions
- `QUAL>=50` retained variants with high-confidence quality scores
- `DP>50` retained variants supported by sufficient sequencing depth
- `variants/filtered.vcf` specified the output filtered VCF file

The filtering process removed lower-confidence variants and retained mutations supported by strong sequencing evidence.

The filtered high-confidence variants were saved as:

```text
variants/filtered.vcf
```

The total number of retained high-confidence variants was determined using:

```bash
grep -v "^#" variants/filtered.vcf | wc -l
```

This command excluded VCF metadata header lines beginning with `#` and counted only genomic variant records.

Following filtering, a total of:

```text
38 high-confidence variants
```

were identified within the SARS-CoV-2 sample.

The number of retained variants was broadly consistent with the expected number of mutations anticipated for the SARS-CoV-2 Omicron lineage following quality and depth filtering.

These filtered variants were then used for downstream annotation, interpretation, and creation of the final attached variant summary table on excel.


# Conclusion

A complete SARS-CoV-2 bioinformatics workflow was successfully performed using command-line tools including FastQC, BWA, SAMtools and BCFtools.

Sequencing reads from SRR17054502 were quality assessed, aligned to the Wuhan-Hu-1 reference genome, and analysed for genomic variation.

Following filtering (QUAL ≥50 and DP >50), 38 high-confidence variants were identified.

Several mutations characteristic of the Omicron lineage were detected, including Spike mutations A67V, Δ69-70, ins214EPE, Q498R and N501Y.

Most variants exhibited high allele frequencies, indicating strong support from the sequencing data.

The resulting variant summary table provides biological interpretation of these mutations and demonstrates how genomic surveillance workflows can be used to identify and characterise emerging viral variants.
