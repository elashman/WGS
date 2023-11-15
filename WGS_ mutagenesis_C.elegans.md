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

module load bbmap

for f1 in *1_001.fastq.gz 
do f2=${f1%%1_001.fastq.gz}"2_001.fastq.gz" 
bbduk.sh threads=8 in1=$f1 in2=$f2 out1=fastq/$(basename $f1 .fastq.gz).trimmed.fastq.gz out2=fastq/$(basename $f2 .fastq.gz).trimmed.fastq.gz ref=../reference_genome/bbduk-fasta.fa ktrim=r k=23 mink=11 hdist=1 tpe tbo qtrim=rl trimq=15 
done


for f1 in *1_001.fastq.gz 
do f2=${f1%%1_001.fastq.gz}"2_001.fastq.gz" 
bbduk.sh threads=8 in1=$f1 in2=$f2 out1=fastq/$(basename $f1 .fastq.gz).trimmed.fastq.gz out2=fastq/$(basename $f2 .fastq.gz).trimmed.fastq.gz ref=../reference_genome/bbduk-fasta.fa ktrim=r k=23 mink=11 hdist=1 tpe tbo qtrim=rl trimq=15
done

for f1 in fastq/*.trimmed.fastq.gz 

# Step 3. 
Check the quality of the trimmed sequencing data using FastQC program

module load java
module load fastqc

for i in fastq/*fastq.gz
do
srun fastqc $i 
done

# Step 4.

Download the re ference genome => index it

bwa index Caenorhabditis_elegans.WBcel235.dna.toplevel.fa.gz

# Step 5. Allign the reads to the reference genome
module load bwa

for f1 in fastq/*1_001.trimmed.fastq.gz
do f2=${f1%%1_001.trimmed.fastq.gz}"2_001.trimmed.fastq.gz"
f3=${f1%%_S*trimmed.fastq.gz}""
echo $f3
bwa mem ../reference_genome/Caenorhabditis_elegans.WBcel235.dna.toplevel.fa.gz $f1 $f2 > $f3_align.bam
done


# Step 6. Sort and index BAM files
module load samtools 

for f1 in fastq/*_align.bam
do f3=${f1%%_S*trimmed.fastq.gz}""
samtools sort -T temp -O bam -o $f3_align_sorted.bam $f1
samtools index $f3_align_sorted.bam
done

samtools sort -T temp -O bam -o 177120_align_sorted.bam 177120_align.bam
samtools sort -T temp -O bam -o 177115_align_sorted.bam 177115_align.bam

samtools index 177120_align_sorted.bam
samtools index 177115_align_sorted.bam
