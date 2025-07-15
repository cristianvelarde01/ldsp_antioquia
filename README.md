# Nanopore-based *Mycobacterium tuberculosis* Variant Calling and Resistance Analysis Pipeline

This repository contains a Bash pipeline designed for processing **Oxford Nanopore** sequencing data of *Mycobacterium tuberculosis* (MTB). The workflow automates basecalling, demultiplexing, quality control, alignment, variant calling, phasing, and functional annotation—focusing on drug resistance mutations and lineage classification.

---

## Overview

This pipeline is optimized for:
- Clinical and epidemiological surveillance of *M. tuberculosis*
- Identification of **SNPs linked to antibiotic resistance**
- Annotation of **WHO canonical mutations** and **lineage-specific variants**
- **Consensus genome reconstruction** from Nanopore reads

---

## Key Features

- **Basecalling & demultiplexing** with `guppy_basecaller`
- **Read filtering** with `artic guppyplex`
- **Quality reports** using `pycoQC` and `nanoq`
- **Alignment** with `minimap2` and `samtools`
- **Variant calling & phasing** with `bcftools` and `whatshap`
- **Codon-level annotation** using `vcf-annotator` and GenBank
- **Resistance and lineage annotation** via `bcftools annotate`
- **Consensus genome creation** with `bcftools consensus`

---

## Pipeline Structure

### 1. **Environment Initialization**
- Activates relevant Conda environments:
  - `artic-ncov2019`, `pycoQC`, `py38`, `NEXTFLOW`, `PYTHON`
- Prepares directories and configuration files

### 2. **Basecalling and Demultiplexing**
- Converts `.fast5` signals to `.fastq` using Guppy (`hac` model)
- Assigns reads to barcodes using `EXP-NBD196`

### 3. **Read Filtering**
- Uses `artic guppyplex` to select reads within desired length (50–6000 bp)

### 4. **Run-Level QC Report**
- Generates HTML reports with `pycoQC` from the sequencing summary

### 5. **Per-Sample Loop (`sample1` to `sampleN`)**
Each sample undergoes the following steps:

#### a. **Read Alignment**
- Aligns reads to the reference genome (`NC_000962.3`) using `minimap2`
- Sorts and indexes BAM files with `samtools`

#### b. **Quality Statistics**
- Produces read statistics using `nanoq`

#### c. **Variant Calling**
- Calls variants with `bcftools mpileup`
- Adds allele-specific annotations
- Removes duplicates and normalizes variants

#### d. **Variant Filtering**
- Filters for SNPs with `PASS` status and minimum depth/quality
- Adds read group headers to BAM files

#### e. **Phasing**
- Runs `whatshap` to phase variants using BAM + VCF
- Falls back to a template VCF if no reads are detected

#### f. **Codon Annotation**
- Uses Dockerized `vcf-annotator` and GenBank to annotate codon changes

#### g. **Post-processing**
- Adds missing INFO fields using `awk`
- Processes phased VCF with `process_whatshap.py`

#### h. **Resistance and Lineage Annotation**
- Annotates variants using `bcftools annotate` against:
  - Drug resistance database
  - WHO canonical SNPs
  - Lineage-defining variants

#### i. **Filtering & Merging Annotations**
- Filters annotated VCFs to retain only `WHO_CANONICAL` mutations
- Concatenates filtered variants across databases

#### j. **Consensus Genome Generation**
- Calls variants again to generate `.fasta` consensus sequence
- Uses `seqtk` to convert FASTQ to FASTA

---

## Output Per Sample

| Output File | Description |
|-------------|-------------|
| `sample.bam`, `sample.sorted.rg.bam` | Aligned reads |
| `sample.mpileup.annotated.processed.PASS.vcf` | Filtered SNPs |
| `sample.phased.codon.fixed.vcf` | Codon-annotated, phased variants |
| `sample.phased.processed.sorted.annotated.{ANTIBIOTICS,WHO,LINEAGES}.vcf` | Functional annotations |
| `sample.final.Concatenado.vcf` | Merged filtered variants |
| `sample_consensus.fasta` | Consensus genome |

---

## Dependencies

This pipeline relies on:
- **Guppy** (ONT basecaller)
- **ARTIC tools**
- **Minimap2**
- **Samtools**
- **Bcftools**
- **Whatshap**
- **vcf-annotator** (Docker)
- **Nanoq**
- **PycoQC**
- Custom Python scripts: `process_mpileup.py`, `process_whatshap.py`

Ensure that the required Conda environments are set up as described in the `environment.yml` or your custom Conda configuration.

---

## Notes

- This pipeline is tailored for *Mycobacterium tuberculosis* and may require adaptation for other organisms or barcoding kits.
- Make sure `.fast5` files are accessible and your GPU setup is compatible with `guppy_basecaller` (e.g., CUDA support).
- Annotation databases used (resistance, WHO, lineage) must be updated periodically from trusted repositories.

---

## License

MIT License

---

## 👥 Authors

Developed by the [Laboratorio Departamental de Salud publica de Antioquia].  
For questions or issues, please open a GitHub issue or contact [idabely.betancur@antioquia.gov.co].

---

## Citation

If you use this pipeline, please cite: `Analytical Comparison of Illumina and Oxford Nanopore Sequencing for Drug Resistance Detection in Mycobacterium tuberculosis: A Cross-Sectional Study from Antioquia, Colombia`

