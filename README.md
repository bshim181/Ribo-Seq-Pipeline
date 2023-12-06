# Ribo-Seq-Pipeline
Creator: Bohoon Shim 
Server: Hosted on Kraken 
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
Pipeline Overview 


# Step 1. Preparation Stage 

## Reference Files 

  1. *Primary Assembly / Gene Annotation File* : Only tested on Gencode annotations
     
     
