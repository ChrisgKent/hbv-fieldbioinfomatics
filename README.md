# HBV-fieldbioinfomatics

[![DOI](https://zenodo.org/badge/DOI/10.5281/zenodo.13341402.svg)](https://doi.org/10.5281/zenodo.13341402)

This project is a custom fork of fieldbioinfomatics, to enable the analysis of circular genomes and corresponding circular primer schemes, such as HBV. 

This project `hbv-fieldbioinfomatics` has been extensively tested for; `minimap2 + medaka + hbv-600/V2.1.0L` (PrimerScheme), however, does contain some quirks due to using an older fork of `fieldbioinfomatics`. 

## Overview of changes

### Circularisation 
This mode takes the amplicon, which spans the `end -> start` of the genome, and appends the sequence to the 3' end of the reference to create `>{refID}_circular`

1. Reads are mapped to this new circular reference genome
2. Pipeline continues normally with variant calling, etc
3. `circular.py parse-vcf`: takes the pass.vcf file and maps the extended positions back into the linear reference using `modulo` to create `{}.mod.vcf`
4. `circular.py dedupe-vcf`: Step 3 enables the same vcf record to exist in `pass.vcf` and `fail.vcf`, leading to errors in consensus generation. This command maps the fail vcf back to linear and removes fail.vcf records with the same coord as in `vcf.pass.mod.vcf`

### Reference selection
HBV is a diverse virus with multiple genotypes. Therefore, mapping any sample to a single reference genome (X02763 geno A) can lead to low mapping quality scores (MAPQ) between reads and the reference. Especially if the reads are from a different genotype. This low MAPQ score leads to errors in the consensus genome, as variant callers have MAPQ thresholds, ignoring some genotype-specific indels.  

To fix this, a reference selection mode has been added. In which the reads are initially competitively against an MSA of representative genomes for each genotype (found [here](https://github.com/ChrisgKent/hep-tile-data/blob/main/data/genotype_reference_sequences.align.fasta)), with the genome with the most reads mapped acting as the reference for the analysis, with the primer-sites moved into this new reference's coordinate system.

```sh
--select-ref-file ./hep-tile-data/data/genotype_reference_sequences.align.fasta
```

## Installation

Currently, the only installation method is from the source

#### 1. Downloading the source:
```sh
git clone https://github.com/ChrisgKent/hbv-fieldbioinfomatics
cd hbv-fieldbioinfomatics
```
#### 2. Installing dependencies:
```sh
conda env create -f environment.yml
conda activate hbv-artic
```
#### 3. Installing the pipeline:
```sh
python setup.py install
```
#### 4. Test the pipeline:
```sh
artic -v
```

## Usage

`hbv-fieldbioinfomatics` expects `primer.bed` files in an older legacy format than the current schemes. Therefore, the schemes are parsed into a legacy mode using `primalbedtools`. This has been done for the `hbv/600/v2.1.0` scheme and included in the `hbv-fieldbioinfomatics` repo. 

```sh 
primalbedtools downgrade ~/primerschemes/primerschemes/hbv/600/v2.1.0/primer.bed --merge-alts > ~/hbv-fieldbioinfomatics/primerschemes/hbv-600/V2.1.0L/hbv-600.scheme.bed
```

### Example
```sh
artic minion --circular --medaka --normalise 400 --threads 8 --scheme-directory ~/hbv-fieldbioinfomatics/primerschemes --scheme-name hbv-600 --scheme-version V2.1.0L --read-file {}  --medaka-model r1041_e82_400bps_hac_v4.3.0 hbv-600/V2.1.0L output/barcode13 --select-ref-file ~/hep-tile-data/data/genotype_reference_sequences.align.fasta
```
