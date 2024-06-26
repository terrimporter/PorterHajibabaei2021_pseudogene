# README

This repository describes dataflows, and provides key infiles, and scripts that I used in the paper Porter and Hajibabaei, 2021 "Profile hidden Markov model sequence analysis can help remove putative pseudogenes from DNA barcoding and metabarcoding datasets".

Infiles and scripts can be downloaded from https://github.com/terrimporter/PorterHajibabaei2021_pseudogene/releases .

## Overview of methods

[Pseudogene filtering method 1 - ORFfinder](#Pseudogene-filtering-method-1---ORFfinder)  
[Pseudogene filtering method 2 - ORFfinder + HMM profile analysis](#Pseudogene-filtering-method-2---ORFfinder-+-profile-analysis)  

## Overview of simulations and analyses

[Part A - Simulate DNA barcode datasets](#Part-A---Simulate-DNA-barcode-datasets)   
[Part B - Simulate metabarcode datasets](#Part-B---Simulate-metabarcode-datasets)   
[Part C - Filter pseudogenes from a real freshwater benthos COI metabarcode dataset](#Part-C---Filter-pseudogenes-from-a-real-freshwater-benthos-COI-metabarcode-dataset)  

## Scripts used to generate figures in the manuscript

[Scripts to generate figures in the manuscript](#Scripts-to-generate-figures-in-the-manuscript)

### Pseudogene filtering method 1 - ORFfinder

In this method, ORFfinder was used to translate nucleotide sequences into open reading frames (ORFs) (nt) and the longest ORF was retained as described in the manuscript.  The lengths of the resulting ORFs were assessed and the smallest and longest ORFs were excluded as putative pseudogenes if they were outliers. 

The cutoff for short outliers :

25th percentile - (1.5 * interquartile range)

The cutoff for long outliers :

75th percentile + (1.5 * interquartile range)

For assessing a DNA barcode dataset, I recommend running ORFfinder from the command line.  ORFfinder can be downloaded from NCBI at https://ftp.ncbi.nlm.nih.gov/genomes/TOOLS/ORFfinder/linux-i64/ .  

```linux
# to get ORFs (nt)
ORFfinder -in file.fasta -ml 30 -g 5 -s 2 -n true -strand plus -out file.nt.fasta -outfmt 1
```

The longest ORF for each sequence can then be parsed from the ORFfinder output.

For assessing a COI metabarcode dataset,  ORFfinder pseodugene removal has been integrated into SCVUC COI metabarcode pipeline v4.1.0 available from https://github.com/Hajibabaei-Lab/SCVUC_COI_metabarcode_pipeline/releases/tag/v4.1.0.  This snakemake pipeline begins with raw paired-end COI Illumina reads.

### Pseudogene filtering method 2 - ORFfinder + HMM profile analysis

This method is similar to method 1 above, except that the longest ORFs were queried against a COI HMM profile using hmmscan (HMMER) as described in the manuscript.  The sequence bit scores were assessed and the smallest were excluded as putativ pseudogenes if they were outliers (as described sbove for short outliers).

For assessing a DNA barcode dataset, I recommend running ORFfinder and HMMER at the command line.  HMMER is available from http://hmmer.org/ .  A COI HMM profile is available as a part of the SCVUC COI metabarcode pipeline v4.3.0 https://github.com/Hajibabaei-Lab/SCVUC_COI_metabarcode_pipeline/releases/tag/v4.3.0.  All 5 files starting with the prefix bold.hmm need to be downloaded.  Starting with the longest ORFs (aa) from ORFfinder, hmmscan can be run.

```linux
# to get ORFs (aa)
ORFfinder -in file.fasta -ml 30 -g 5 -s 2 -n true -strand plus -out file.aa.fasta -outfmt 2

# parse out longest ORFs from ORFfinder output

# compare longest ORFs to a COI profile
hmmscan --tblout hmmer.out bold.hmm longest.orfs.nt.fasta
```

### Part A - Simulate DNA barcode datasets

1. Retrieve a set of COI gene sequences from BOLD:  
a. BOLD data releases can be downloaded from https://v3.boldsystems.org/index.php/datarelease .  The output of this script is a set of tsv (tab separated values) files.
    ```linux
    zsh getBOLDdataReleases.sh releases.txt
    ```  
b. Retrieved tsv files can be parsed to retrieve the nucleotide and amino acid sequences in FASTA format.  In the script, edit to include the path to the directory where the BOLD tsv files are located.  You will need the names.dmp and nodes.dmp files from the taxdmp.zip files that can be downloadd from NCBI at https://ftp.ncbi.nlm.nih.gov/pub/taxonomy/ .  In the script, edit to include the path to names.dmp and nodes.dmp.  The output of this script is bold.nt.fasta and bold.aa.fasta .

```linux
perl parse_BOLD_data_releases3.plx
```

2. Retrieve a set of COI pseudogene sequences from the NCBI nucleotide database:  

a. In the script, edit to include your own email address.  The output of this script is pseudogenes.fasta .

```linux
perl ebot_entrez_get_fasta.plx
```

b. This script will convert the FASTA format from the NCBI into a strict FASTA format.  The output of this script is pseudogenes.fasta.strict .

```linux
perl messedup_fasta_to_strict_fasta.plx < pseudogenes.fasta > pseudogenes.fasta.strict
```

3. Cross check sequences from BOLD and the NCBI nucleotide database and keep sequences if COI gene and pseudogenes can be found for the same species.  The output of this script is a set of nucleotide and amino acid sequences for species with both COI gene and pseudogene sequences.

```linux
perl parse_pseudogenes.plx pseudogenes.fasta.strict bold.nt.fasta bold.aa.fasta
```

These files can now be used to assess length, GC contact, and to test pseudogene filtering methods.

4. Phylogenetic analyses:

a. Nucleotide sequence files are available here at ~/PartA_DNA_barcode_simulation/phylogram/nt .  These sequences were obtained from BOLD and NCBI as described above.  Amino acid sequence files are also available here at ~/PartA_DNA_barcode_simulation/phylogram/aa .  COI amino acid sequences were obtained from BOLD as described above.  COI pseudogene sequeces were generated by translating the nucleotide sequences from the NCBI nucleotide database with ORFfinder and retaining the longest sequences.  These files can be used with TRANALIGN to generate a codon alignment and processed with phylip (EMBOSS) to generate phylograms as described in the manuscript.

5. dN/dS analyses:

a. The nucleotide gene sequences from all 10 species were combined into a single file, as were the pseudogene sequences.  The same was done for the amino acid sequnces.  These files are available here at ~/PartA_DNA_barcode_simulation/dNdS .  These files can be used with TRANALIGN to generate a codon alignments that can be analyzed in [R] to assess dN/dS ratios as described in the manuscript.

### Part B - Simulate metabarcode datasets

1. Use the COI gene sequences from BOLD to create a perturbed community datasets.  

a. The output of this script will be a dataset with 100,000 randomly chosen sequences (mutated_0.fasta), a dataset where some of the sequences have been pseudogenized by inserting GC -> AT point mutations to reduce GC content (mutated_1.fasta), and a dataset where some of the sequences have been pseudogenized by inserting frameshift mutations by introducing indels (mutated_2.fasta).  The parameters for how sequencs are picked and pseudogenized are at the top of the file and can be edited: the number of sequences to sample [100000], the proportion of sequences to pseudogenize [0.19], percentage of bases targeted to reduce GC content [0.025], percentage of bases to introduce an indel [0.025].  The outfile mutated_0.fasta is a set of control sequences with no pseudogenes simulated.  

```linux
perl mutate.plx bold.nt.fasta
```

The output of this script can be used to assess length and GC content as well as for testing with two different pseudogene filtering methods.

b. The output of this script will be datasets with sequences as described above but they will be half-length (~ 300bp).  This script is meant to be used with the fasta files from above.

```linux
perl chop_in_half.plx mutated_0.fasta
```

The output of this script can be used to assess length and GC content as well as for testing with two diffrent pseudogene filtering methods.

c. The output of this script will be datasets with sequences as described above but with double the proportion of pseudogenes.

```linux
perl mutate2.plx bold.nt.fasta
```

The output of this script can be used to assess length and GC content as well as for testing with two diffrent pseudogene filtering methods.

d. The output of this script will be datasets with sequences as described above but with half the proportion of pseudogenes.

```linux
perl mutate3.plx bold.nt.fasta
```

The output of this script can be used to assess length and GC content as well as for testing with two diffrent pseudogene filtering methods.

### Part C - Filter pseudogenes from a real freshwater benthos COI metabarcode dataset

The dataset used for this analysis was originally published by Hajibabaei et al., 2019.  This is a COI freshwater benthos metabarcode dataset where 6 different primer sets were used so the amplicons span differnt portions of the COI barcode region.

Raw reads are available from the NCBI SRA # PRJNA545426 .

1. Process raw reads using the standard pipeline SCVUC v4.3.0 that uses ORFfinder + HMM profile analysis to remove putative pseudogenes.

2. Process raw reads with a modified pipeline that KEEPS rare reads, i.e., skips the rare sequence removal steps.  The modified files needed to run this pipeline are available here at ~/PartC_freshwater_benthos_COI_metabarcoding_real_example/SCVUC-4.3.0-keep_rare .

3. Process raw reads with a modified pipeline that KEEPS noisy sequences, i.e., skips the denoising step.  The modified file needed to run this pipeline is available here at ~/PartC_freshwater_benthos_COI_metabarcoding_real_example/SCVUC-4.3.0-keep_noise .

4. Process raw reads with a modified pipeline that KEEPS chimeric sequences, i.e., skips the chimera removal step.  The modified file needed to run this pipeline is available here at ~/PartC_freshwater_benthos_COI_metabarcoding_real_example/SCVUC-4.3.0-keep_chimeras .

### Scripts to generate figures in the manuscript

Fig 2 - The R script Fig2_PartA.R uses the infiles gene_test.fasta and pseudogene_test.fasta to calculate GC content; the infiles gene_test.nt.fasta.filtered and pseudogene_tets.nt.fasta.filtered to calculate the length of the longest ORF from ORFfinder; and gene_test.txt and pseudogene_test.txt to calculate bitscores from HMMER3.

Fig 3 - The R script Fig3_PartB.R uses the infiles mutated_0.nt.fasta.filtered, mutated_1.nt.fasta.filtered, and mutated_2.nt.fasta.filtered to calculate the length of the longest ORFs from ORFfinder; and mutated_0.txt, mutated_1.txt, and mutated_2.txt to calculate bitscores from HMMER3.

## References

Hajibabaei, M., Porter, T.M., Wright, M., Rudar, J. (2019) COI metabarcoding primr choice affects richness and recovery of indicator taxa in freshwater systems.  PLoS ONE, 14(9): e0220953.

Porter, T.M., Hajibabaei, M. (2021) Profile hidden Markov model sequence analysis can help remove putative pseudogenes from DNA barcoding and metabarcoding datasets.  BioRxiv.

Last updated: March 25, 2021
