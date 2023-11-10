# Step 1. run FastQC

module load java
module load fastqc

for i in *fastq.gz
do
srun fastqc $i 
done

# moving files to own computer from cluster 

scp -r elashman@bea81.hpc.ista.ac.at:/nfs/scistore13/bonogrp/elashman/Documents/Thanh_DNAseq/*html  /Users/elashman/Documents/WGS_mutagenesis


# STEP 2. Quality and adapter trimmimg with BBduk

~/Downloads/bbmap/bbduk.sh threads=8 \
in1=177115_S19_L004_R1_001.fastq.gz \
in2=177115_S19_L004_R2_001.fastq.gz \
out1=fastq/177115-trimmed.mate1.fastq.gz \
out2=fastq/177115-trimmed.mate2.fastq.gz \
ref=reference_genome/bbduk-fasta.fa \
ktrim=r k=23 mink=11 hdist=1 tpe tbo qtrim=rl trimq=15 

~/Downloads/bbmap/bbduk.sh threads=8 \
in1=177120_S14_L004_R1_001.fastq.gz \
in2=177120_S14_L004_R2_001.fastq.gz \
out1=fastq/177120-trimmed.mate1.fastq.gz \
out2=fastq/177120-trimmed.mate2.fastq.gz \
ref=reference_genome/bbduk-fasta.fa \
ktrim=r k=23 mink=11 hdist=1 tpe tbo qtrim=rl trimq=15 

for f1 in *1_001.fastq.gz
do
    f2=${f1%%1_001.fastq.gz}"2_001.fastq.gz"
    echo $f1
    echo $f2
    echo $(basename $f1 .fastq.gz).trimmed.fastq.gz \
    echo $(basename $f2 .fastq.gz).trimmed.fastq.gz \
done

module load bbmap
for f1 in *1_001.fastq.gz
do
    f2=${f1%%1_001.fastq.gz}"2_001.fastq.gz"
    bbduk.sh threads=8 \
in1=$f1 \
in2=$f2\
out1=fastq/$(basename $f1 .fastq.gz).trimmed.fastq.gz \
out2=fastq/$(basename $f2 .fastq.gz).trimmed.fastq.gz \
ref=../reference_genome/bbduk-fasta.fa \
ktrim=r k=23 mink=11 hdist=1 tpe tbo qtrim=rl trimq=15
done

