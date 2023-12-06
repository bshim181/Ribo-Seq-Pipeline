# Ribo-Seq-Pipeline
Creator: Bohoon Shim 
Server: Hosted on Slurm (Kraken) 
Functionality: Handles Ribosome Profiling Reads(Single End Fastq Files) to predict potentially translating open reading frames. 

*Current Pipeline is compatible with the following specific versions of software:*

```
bowtie2                   2.5.1
fastx_toolkit             0.0.14
gtfparse                  1.2.1
python                    3.10.12 
star                      2.7.10b
tqdm                      4.66.1
bedtools                  2.30.0 
R                         4.1
```

# Step 1. Preparation Stage 

## Reference Files 

  1. *Primary Assembly / Gene Annotation File* : Only tested on Gencode annotations
  2. Build rRNA Reference file

### rRNA Reference 

A good resource for rRNA sequences is RNAcentral, a database of non-coding RNA from multiple databases such as Rfam and RDP (Ribosomal Database Project).
Download and merge RNAcentral FASTA files:

```
#Convert multi-line sequences to single-line (using fasta_formatter from FASTX-Toolkit):
wget ftp://ftp.ebi.ac.uk/pub/databases/RNAcentral/releases/12.0/sequences/rnacentral_species_specific_ids.fasta.gz

#unzip
gzip -cd rnacentral_species_specific_ids.fasta.gz \
  | fasta_formatter -w 0 \
  | gzip \
  > rnacentral.nowrap.fasta.gz

#Remove empty lines, replace spaces with underscores, and keep just ribosomal sequences:
gzip -cd rnacentral.nowrap.fasta.gz \
  | sed '/^$/d' \
  | sed 's/\s/_/g' \
  | grep -E -A 1 "ribosomal_RNA|rRNA" \
  | grep -v "^--$" \
  | gzip \
  > rnacentral.ribosomal.nowrap.fasta.gz

#Set a variable for the species of interest. For example:
species="homo_sapiens"
species="mus_musculus"
species="drosophila_melanogaster"

Extract species-specific ribosomal sequences:
zcat rnacentral.ribosomal.nowrap.fasta.gz \
  | grep -A 1 -F -i "${species}" \
  | grep -v "^--$" \
  | fasta_formatter -w 80 \
  > rRNA.${species}.fa
```
Code is taken from https://github.com/igordot/genomics/blob/master/workflows/rrna-ref.md


# Step 2. Setting up Ribotricer and RibORF

Setting up Ribotricer 

```
conda create -n ribotricer_env -c bioconda ribotricer
conda activate ribotricer_env
```

```
ribotricer prepare-orfs --gtf {GTF} --fasta {FASTA} --prefix output_directory/file_name.tsv
```

Setting up RibORF

```
wget https://github.com/zhejilab/RibORF/tree/master/RibORF.2.0
RibORF=source_dir/RibORF.2.0
```

# Step 3: 


     
     
