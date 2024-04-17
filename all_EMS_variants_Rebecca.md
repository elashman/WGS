mimodd vcf-filter 177115_177120_mimodd_variants_dp10.vcf --sample 177115 177120 --gt 1/1,1/0 0/0 -o 177115_variants_Rebecca.vcf 
mimodd annotate 177115_variants_Rebecca.vcf WBcel235.105 -o 177115_variants_Rebecca_anno.vcf
mimodd varreport 177115_variants_Rebecca_anno.vcf -o 177115_variants_Rebecca_report.html -f html
