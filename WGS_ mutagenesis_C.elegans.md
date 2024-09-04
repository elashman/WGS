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

for f1 in *R1_001.fastq.gz; do f2=${f1%%1_001.fastq.gz}"2_001.fastq.gz"; bbduk.sh threads=8 in1=$f1 in2=$f2 out1=${f1%%.fastq.gz}.trimmed.fastq.gz out2=${f2%%.fastq.gz}.trimmed.fastq.gz ref=../../reference_genome/bbduk-fasta.fa ktrim=r k=23 mink=11 hdist=1 tpe tbo qtrim=rl trimq=15 ; done

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

bwa mem reference_genome/Caenorhabditis_elegans.WBcel235.dna.toplevel.fa.gz fastq/177115-trimmed.mate1.fastq.gz fastq/177115-trimmed.mate2.fastq.gz > 177115_align.bam
bwa mem reference_genome/Caenorhabditis_elegans.WBcel235.dna.toplevel.fa.gz fastq/177120-trimmed.mate1.fastq.gz fastq/177120-trimmed.mate2.fastq.gz > 177120_align.bam



# Step 6. Sort and index BAM files
module load samtools 

for f1 in *_align.bam
do f3=${f1%%_align.bam}"_align_sorted.bam"
samtools sort -T temp -O bam -o $f3 $f1
samtools index $f3
done

samtools sort -T temp -O bam -o 177120_align_sorted.bam 177120_align.bam
samtools sort -T temp -O bam -o 177115_align_sorted.bam 177115_align.bam

samtools index 177120_align_sorted.bam
samtools index 177115_align_sorted.bam

for f1 in *_align.bam
do echo $f1
f3=${f1%%_align.bam}"_align_sorted.bam"
echo $f3
done

# Step 7. Remove Duplicates
module load picard/2.27.5
module load samtools

for f1 in *align_sorted.bam
do f2=${f1%%_align_sorted.bam}"_align_sorted_deletedup_metrics.txt"
f3=${f1%%_align_sorted.bam}"_align_sorted_deletedup.bam"
java -jar $PICARD MarkDuplicates VALIDATION_STRINGENCY=LENIENT REMOVE_DUPLICATES=TRUE I=$f1 O=$f3 M=$f2
samtools index $f3
done

# Step 8. Asssign Read Group to reads in each file
module load picard/2.27.5

java -jar $PICARD AddOrReplaceReadGroups I=177115_align_sorted_deletedup.bam O=177115_align_sorted_deletedup_header.bam RGID=0 RGSM=177115 RGLB=unknown RGPL=ILLUMINA RGPU=unknown
java -jar $PICARD AddOrReplaceReadGroups I=177120_align_sorted_deletedup.bam O=177120_align_sorted_deletedup_header.bam RGID=1 RGSM=177120 RGLB=unknown RGPL=ILLUMINA RGPU=unknown

java -jar $PICARD AddOrReplaceReadGroups I=177115__align_mimodd_sorted.bam O=177115_align_mimodd_sorted_deletedup_header.bam RGID=0 RGSM=177115 RGLB=unknown RGPL=ILLUMINA RGPU=unknown
java -jar $PICARD AddOrReplaceReadGroups I=177120__align_mimodd_sorted.bam O=177120_align_mimodd_sorted_deletedup_header.bam RGID=1 RGSM=177120 RGLB=unknown RGPL=ILLUMINA RGPU=unknown

# check that the groups were assigned: 
samtools view -H  177120_align_sorted_deletedup_header.bam

# Step 9. Variant calling 
module load python/3.6 

mimodd varcall ../reference_genome/mimodd_Caenorhabditis_elegans.WBcel235.dna.toplevel.fa 177115_align_mimodd_sorted_deletedup_header.bam 177120_align_mimodd_sorted_deletedup_header.bam -o 177115_177120_mimodd_calls_deletedup.bcf --verbose

# Step 10. Variants extraction
module load python/3.6 

for f1 in *_mimodd_calls_deletedup.bcf
do f3=${f1%%_mimodd_calls_deletedup.bcf}"_mimodd_calls_deletedup.vcf"
mimodd varextract $f1 -o $f3
done

# Step 11. Variant filtering. 

mimodd vcf-filter 177115_177120_mimodd_calls_deletedup.vcf -s 177115 177120 --dp 10 10 -o 177115_177120_mimodd_variants_dp10.vcf

# Step 12. VAC mapping

mimodd map VAC 177115_177120_mimodd_variants_dp10.vcf -m 177115 -c 177120 -o 177115_177120_calls_deletedup_dp10_linkage_VAC.txt -p 177115_177120_calls_deletedup_dp10_linkage_VAC.pdf --no-scatter

# Copy to computer 

scp -r elashman@bea81.hpc.ista.ac.at:/nfs/scistore13/bonogrp/elashman/Documents/Thanh_DNAseq/*txt /Users/elashman/Documents
scp -r elashman@bea81.hpc.ista.ac.at:/nfs/scistore13/bonogrp/elashman/Documents/Thanh_DNAseq/*pdf /Users/elashman/Documents

# Step 13. Identify the causative mutation using mapping information
mimodd vcf-filter 177115_177120_mimodd_variants_dp10.vcf --region I%20dna:chromosome%20chromosome:WBcel235:I:1:15072434:1%20REF:5500000-9500000 --sample 177115 177120 --gt 1/1 0/0 -o 177115_177120_peak_variants.vcf
scp -r elashman@bea81.hpc.ista.ac.at:/nfs/scistore13/bonogrp/elashman/Documents/Thanh_DNAseq/177115_177120_peak_variants.vcf  /Users/elashman/Documents/


