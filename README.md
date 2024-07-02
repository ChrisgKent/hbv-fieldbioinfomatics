# HBV-fieldbioinfomatics

[![DOI](https://zenodo.org/badge/790860575.svg)](https://zenodo.org/doi/10.5281/zenodo.12626841)


This project is a custom fork of fieldbioinfomatics, to enable the analysis of circular genomes and corresponding circular primer schemes, such as HBV. 

Due to time constraints this mode has only been applied/tested for one running mode;
- minimap2
- medaka
- hbv-600/V2.1.0L (PrimerScheme)


### Overview of changes

This mode takes the amplicon which spans the `end -> start` of the genome, and appends the sequence to the 3' end of the reference to create `>{refID}_circular`

1. Reads are mapped to this new circular reference genome
2. Pipeline continues normally with variant calling, etc
3. `circular.py parse-vcf`: takes the pass.vcf file and maps the extended positions back into the linear reference using `modulo` to create `{}.mod.vcf`
4. `circular.py dedupe-vcf`: Step 3 enables the same vcf record to exist in `pass.vcf` and `fail.vcf` leading to errors in consensus generation. This command maps the fail vcf back to linear and removes fail.vcf records with the same coord as in in `vcf.pass.mod.vcf`

### Installation

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



### Example
```sh
artic minion --circular --medaka --normalise 400 --threads 8 --scheme-directory ~/hbv-fieldbioinfomatics/primerschemes --read-file {}  --medaka-model r1041_e82_400bps_hac_v4.3.0 hbv-600/V2.1.0L output/barcode13
```
