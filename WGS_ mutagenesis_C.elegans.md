# Step 1. run FastQC

module load java
module load fastqc

for i in *fastq.gz
do
srun fastqc $i 
done


