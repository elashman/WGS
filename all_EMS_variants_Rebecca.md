mimodd vcf-filter 177115_177120_mimodd_variants_dp10.vcf --sample 177115 177120 --gt 1/1,1/0 0/0 -o 177115_variants_Rebecca.vcf 

# Chromosome names were long like "V%20dna:chromosome%20chromosome:WBcel235:V:1:20924180:1%20REF". Were changed to shorter ones like "V". Otherwise the annotation tool was not working.


mimodd annotate 177115_variants_Rebecca.vcf WBcel235.105 -o 177115_variants_Rebecca_anno.vcf

mimodd varreport 177115_variants_Rebecca_anno.vcf -o 177115_variants_Rebecca_report.html -f html
