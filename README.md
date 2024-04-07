# Metagenomics Data Analysis Pipeline

This pipeline covers the key steps in analyzing metagenomics data, from data download and preprocessing to the creation of metagenome-assembled genomes and their annotation. The extra steps provide opportunities for further exploration and analysis of the data.

## 1. Download public data
- SRA run selector
- Download from terminal

## 2. Data preprocessing
- Quality control (QC)
- Trimmomatic
- QC
- Taxonomic profiling (Kraken2 and Krona)

## 3. Assembly of the data into contigs
- metaSPAdes

## 4. Create metagenome-assembled genomes (MAGs)
- MetaBAT2
- CheckM (for completeness estimation)

## 5. Annotation
- Prokka or eggNOG-mapper

## Extra
- Comparative metagenomics
- Machine learning in metagenomics


# Metagenomics Practical Course

This practical course will guide you through the essential steps of metagenomics data analysis, from data download to downstream analysis.

## 1. Download public data

### 1.1 SRA run selector
The Sequence Read Archive (SRA) run selector is a web-based tool that allows you to search and download metagenomics datasets. You can access it at the following link:
[SRA Run Selector](https://www.ncbi.nlm.nih.gov/Traces/study/) and for this course you can download the data with the following accession number PRJNA448333 from the paper https://microbiomejournal.biomedcentral.com/articles/10.1186/s40168-019-0618-5#Sec2

Use the search functionality to find a dataset of interest, and take note of the accession numbers for the samples you want to download.

### 1.2 Download from the terminal
Once you have the accession numbers, you can use the SRA Toolkit to download the data directly from the command line. Install the SRA Toolkit on your system, and then use the following command to download the data:

```
fastq-dump --split-files <accession_number>
```

Replace `<accession_number>` with the actual accession number of the sample you want to download.

## 2. Data preprocessing

### 2.1 Quality control (QC)
Perform quality control on the downloaded data using tools like FastQC or MultiQC. These tools will provide you with various metrics, such as read quality scores, GC content, and sequence duplication levels, which will help you identify any potential issues with the data.

```
mkdir fastqc_out

# Example command to run FastQC
for each in *.fastq.gz
do 
fastqc ${each} -o fastqc
```

### 2.2 Trimmomatic
After the initial QC, use Trimmomatic to trim adapter sequences and low-quality reads from the data.

```
# Example command to run Trimmomatic
trimmomatic PE -threads 4 -phred33 input_forward.fastq.gz input_reverse.fastq.gz \
                output_forward_paired.fastq.gz output_forward_unpaired.fastq.gz \
                output_reverse_paired.fastq.gz output_reverse_unpaired.fastq.gz \
                ILLUMINACLIP:TruSeq3-PE.fa:2:30:10 LEADING:3 TRAILING:3 SLIDINGWINDOW:4:15 MINLEN:36
```

## 3. Assembly of the data into contigs
Use the metaSPAdes tool to assemble the preprocessed reads into contigs.

```
# Example command to run metaSPAdes
metaspades.py -1 input_forward_paired.fastq.gz -2 input_reverse_paired.fastq.gz -o metaspades_output
```

## 4. Generate coverage statistics
Prior to running MetaBAT, we need to generate coverage statistics by mapping reads to the contigs. To do this, we can use bwa (http://bio-bwa.sourceforge.net/) and then the samtools software (http://www.htslib.org) to reformat the output.

```
# index the contigs file that was produced by metaSPAdes:
bwa index contigs.fasta

# map the original reads to the contigs:
bwa mem contigs.fasta ERR011322_1.fastq ERR011322_2.fastq > input.fastq.sam

# reformat the file with samtools:
samtools view -Sbu input.fastq.sam > junk
samtools sort junk input.fastq.sam
```

## 5. Create metagenome-assembled genomes (MAGs)
Use MetaBAT2 to bin the assembled contigs into putative metagenome-assembled genomes (MAGs), and then use CheckM and BUSCO or EukCC to estimate the completeness and quality of the resulting MAGs.

```
# Example command to run MetaBAT2
metabat2 -i metaspades_output/contigs.fasta -o metabat2_output/bin \
         -m 1500 --unbinned \
         --saveCls metabat2_output/bin.cls.tsv \
         --saveLog metabat2_output/metabat2.log

# Example command to run CheckM
checkm lineage_wf -t 4 -x fa metabat2_output/bin checkm_output
```

## 6. Visualising the phylogenetic tree
We will now plot and visualize the tree we have produced. A quick and user-friendly way to do this is to use the web-based **interactive Tree of Life (iTOL)**: http://itol.embl.de/index.shtml

iTOL only takes in newick formatted trees, so we need to quickly reformat the tree with **FigTree** (http://tree.bio.ed.ac.uk/software/figtree/).

Other potential software options for visualizing phylogenetic trees include:
- **Archaeopteryx**: https://sites.google.com/site/cmzmasek/home/software/archaeopteryx
- **EvolView**: https://www.evolgenius.info/evolview/
- **TreeView**: http://taxonomy.zoology.gla.ac.uk/rod/treeview.html

## 7. Annotation
Annotate the MAGs using tools like Prokka or eggNOG-mapper.

```
# Example command to run Prokka
prokka --outdir prokka_output --prefix sample_name metaspades_output/contigs.fasta
```

## Extra stuff

### Comparative metagenomics
.... its need to edited 

### Machine learning in metagenomics
For how to use Machine Learning in metagenomics read about the package **SIAMCAT** https://bioconductor.org/packages/release/bioc/html/SIAMCAT.html where you can use an input features such as: <br>
- Species abundance
- Metabolic pathway abundance
- Functional gene annotation
- Orthologous gene families
- Toxis, virulence factors

### Do you have material from other courses that I can follow?
- https://usda-ars-gbru.github.io/Microbiome-workshop/tutorials/metagenomics/
- https://ucdavis-bioinformatics-training.github.io/2021-December-Metagenomics-and-Metatranscriptomics/data_analysis/00-project_setup_mm
- https://www.ebi.ac.uk/training/online/courses/metagenomics-bioinformatics/introduction/



