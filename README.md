# README

This repository contains the dataflows, infiles, and scripts I used in the paper Porter and Hajibabaei, 2021 "Profile hidden Markov model sequence analysis can help remove putative pseudogenes from DNA barcoding and metabarcoding datasets".

Infiles and scripts can be downloaded from https://github.com/terrimporter/PorterHajibabaei2021_pseudogene/releases .

## Simulations and Analyses

### Part A

1. Retrieve a set of COI gene sequences from BOLD: 

a. BOLD data releases can be downloaded from https://v3.boldsystems.org/index.php/datarelease .  The output of this script is a set of tsv (tab separated values) files.

```linux
zsh getBOLDdataReleases.sh releases.txt
```

b. Retrieved tsv files can be parsed to retrieve the nucleotide and amino acid sequences in FASTa format.  In the script, edit to include the path to the directory where the BOLD tsv files are located.  You will need the names.dmp and nodes.dmp files from the taxdmp.zip files that can be downloadd from NCBI at https://ftp.ncbi.nlm.nih.gov/pub/taxonomy/ .  In the script, edit to include the path to names.dmp and nodes.dmp.  The output of this script is bold.nt.fasta and bold.aa.fasta .

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

## Scripts


## References

Porter, T.M., Hajibabaei, M. (2021) Profile hidden Markov model sequence analysis can help remove putative pseudogenes from DNA barcoding and metabarcoding datasets.  BioRxiv.

Last updated: January 25, 2021
