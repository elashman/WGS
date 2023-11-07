# Step 1. run FastQC

module load java
module load fastqc

for i in *fastq.gz
do
srun fastqc $i 
done

# moving files to own computer from cluster 

scp -r elashman@bea81.hpc.ista.ac.at:/nfs/scistore13/bonogrp/elashman/Documents/Thanh_DNAseq/*html  /Users/elashman/Documents/WGS_mutagenesis


