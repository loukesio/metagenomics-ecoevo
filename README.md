## Metagenomics course for the Applied Bioinformatics Masters 

### Details 
- **Location**: Online
- **Date**: April 1st 2024
- **Speakers**: Loukas Theodosiou

### Abstract
  In this metagenomics lecture, participants will be introduced to the principles and latest updates in the field of metagenomics, including amplicon sequencing, 16S sequencing, and whole-genome shotgun sequencing approaches. Participants will not only grasp the technical details for analyzing these types of data but also gain insights into the eco-evolutionary processes that these data can elucidate. As an exercise, students will break into teams at the end of the lecture to conceptualize a company they could start using metagenomics principles. During the practicum, students will become familiar with state-of-the-art tools for analyzing metagenomic data, including an introduction to machine learning techniques. Basic knowledge of R, Python, and Bash, as well as the conda package manager, is required. All students should bring their own PC with at least 5 GB of free space and be able to install conda modules.



# Metagenomics Practical Course

This practical course will guide you through the essential steps of metagenomics data analysis, from data download to downstream analysis.

## 1. Download public data

### 1.1 SRA run selector
The Sequence Read Archive (SRA) run selector is a web-based tool that allows you to search and download metagenomics datasets. You can access it at the following link:
[SRA Run Selector](https://www.ncbi.nlm.nih.gov/Traces/study/)

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
# Example command to run FastQC
fastqc *.fastq.gz
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

### 2.3 Taxonomic profiling
Perform taxonomic profiling on the preprocessed data using tools like Kraken2 and visualize the results using Krona.

```
# Example command to run Kraken2
kraken2 --db kraken2-db --paired --output kraken2_output.txt --report kraken2_report.txt \
        input_forward_paired.fastq.gz input_reverse_paired.fastq.gz
```

## 3. Assembly of the data into contigs
Use the metaSPAdes tool to assemble the preprocessed reads into contigs.

```
# Example command to run metaSPAdes
metaspades.py -1 input_forward_paired.fastq.gz -2 input_reverse_paired.fastq.gz -o metaspades_output
```

## 4. Create metagenome-assembled genomes (MAGs)
Use a tool like CheckM to identify high-quality MAGs from the assembled contigs.

```
# Example command to run CheckM
checkm lineage_wf -t 4 -x fa metaspades_output/contigs.fasta checkm_output
```

## 5. Annotation
Annotate the MAGs using tools like Prokka or eggNOG-mapper.

```
# Example command to run Prokka
prokka --outdir prokka_output --prefix sample_name metaspades_output/contigs.fasta
```

## Extra stuff

### Comparative metagenomics
Write a software pipeline that can be used to perform comparative metagenomics analysis between multiple samples.

### Machine learning in metagenomics
Write an R or Python package that can be used to apply machine learning techniques to metagenomics data, such as classification, clustering, or predictive modeling.
