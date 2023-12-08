# Ribo-Seq-Pipeline
Creator: Bohoon Shim 

Server: Hosted on Slurm (Kraken) 

Functionality: Handles Ribosome Profiling Reads(Single End Fastq Files) to predict potentially translating open reading frames. 

Ribo-seq protocol which the pipeline supports can be found here: https://www.sciencedirect.com/science/article/pii/S1046202316303292?via%3Dihub

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

## Setting up your Conda Environment for the Pipeline

```
# using .yml file 
conda env create -f environment.yml

# using pip
pip install -r requirements.txt

# using Conda
conda create --name ribo_seq --file requirements.txt
```

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

## Setting up Ribotricer in your conda environment 
Ribotricer: https://github.com/smithlabcode/ribotricer

```
conda create -n ribotricer_env -c bioconda ribotricer
conda activate ribotricer_env
```

Generate Ribotricer ORF Index file with your gtf file and primary assembly

```
START_CODONS="ATG,CTG,GTG,TTG,ACG"
ribotricer prepare-orfs --gtf {GTF} --fasta {FASTA} --prefix output_directory/file_name.tsv
--min_orf_length 21 --start_codons $START_CODONS
```

## Setting up RibORF
Download source codes from RibORF repository into your source directory. 
RibORF Github: https://github.com/zhejilab/RibORF/tree/master/RibORF.2.0

```
wget https://github.com/zhejilab/RibORF/tree/master/RibORF.2.0
RibORF=source_dir/RibORF.2.0
```

Once Ribotricer list of ORFs/Index has been generated, we will convert it to RibORF Format. 

  1. Download genePredToGTF script from UCSC and generate genePred format of your gtf file
     ```
     wget http://hgdownload.cse.ucsc.edu/admin/exe/linux.x86_64/gtfToGenePred
     chmod +x gtfToGenePred
     ./gtfToGenePred target.gtf target.genePred
     ```
  2. Convert Ribotricer Index to RibORF Index Format. This converts list of all possible ORFs into GenePred Format 
     ```
     python3 utils/transform_index.py --gtf gtf --gene_pred gene_pred --index ribotricer_index --out_dir ${out_dir}/ribORF_index.txt
     ```

# Step 3: Main nuORF Prediction Script 

Run the Prediction Script in your source directory 
In the Initial Run, It will automatically generate STAR and Bowtie Index based on your reference file input 

```
  -r : Run_name
  -o : Output_directory
  -f : Fastq_directory
  -s : Source_directory (where your main script is located)
  -k : Skip to specific steps (1~14) 
  -t : Run_Type (Ligation_Free Protocol or Traditional Protocol)
  --samplesheet: TSV File (Column 1: Sample Specific Adapters / Column 2: Sample Fastq Basenames)

  Skip Parameters

  #0 = From the start 
  #1 = From cutadapt and umi_extraction  
  #2 = From rRNA removal with bowtie 
  #3 = From STAR Alignment
  #4 = Quality Checking with Ribotish 
  #5 = RiboORF Codon Periodicity Plot 
  #6 = Search for offset parameters for each sample 
  #7 = Sam file offset correction for RibORF Prediction
  #8 = Ribotricer ORF Prediction
  #9 = RibORF ORF Prediction 
  #10 = ORF Prediction to Bed Conversion 
  #11 = Merging RibORF and Ribotricer Predictions 
  #12 = Remove duplicate ORFs based on start and end position
  #13 = Calculate ORF TPM
  #14 = Filter for non-annotated ORFs (non-canonical open reading frames) 

  Reference Files
  --rrna : rRNA_Reference_File
  --reference : Primary Assembly
  --gene_annotation : GTF File
  --ribotricer_index : Ribotricer Index
  --gene_pred : GenePred Format of GTF
  --ribORF_Index : RibORF Index

  ./prediction.sh -r run_name -o output -f fastq -s source -k 0 -t Traditional --samplesheet sample.tsv --rrna --reference --gene_annotation --ribotricer_index --gene_pred --ribORF_index

```

<img width="980" alt="Screenshot 2023-12-08 at 8 20 58 AM" src="https://github.com/bshim181/Ribo-Seq-Pipeline/assets/53489568/be7ec1c9-ed1f-423c-93bb-14ee905af62d">

     
     
