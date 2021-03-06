
# Call
## call gvcf from every patient

```
%%bash
env=gatk
patient=PATEINT

bam_directory=~/project/1-HCC/1-bam/
hc_directory=~/project/1-HCC/2-hc/

reference_genome=~/project/0-reference/2-GATK/hg19/human_g1k_v37_decoy.fasta

dbsnp=dbsnp_138.b37.vcf.gz

export PATH=$PATH:~/software/miniconda3/envs/r_env/bin

# run haplotypecaller to call germline variant for normal
gatk HaplotypeCaller \
    -R ${reference_genome} \
    -I ${bam_directory}/${patient}_normal.bam \
    --dbsnp ${gatk_bundle}/${dbsnp} \
    -bamout ${bam_directory}/${patient}_normal.bamout.bam
    -O ${hc_directory}/gvcf/${patient}_normal.hc.g.vcf.gz \
    -ERC GVCF
```

## combine

```
%%bash
env=gatk
patients=~/code/1-HCC/HCC_patient

bam_directory=~/project/1-HCC/1-bam/
hc_directory=~/project/1-HCC/2-hc/
pon_directory=~/project/1-HCC/3-pon/

gatk_bundle=~/project/0-reference/2-GATK/hg19

reference_genome=${gatk_bundle}/human_g1k_v37_decoy.fasta
region=${gatk_bundle}/chr.bed

export PATH=~/software/miniconda3/envs/r_env/bin:$PATH

gatk --java-options "-Xmx8g -Xms8g" GenomicsDBImport \
    -R ${reference_genome} \
    -L ${region} \
    --genomicsdb-workspace-path ${pon_directory}/pon/ \
    --reader-threads 5 \
    --batch-size 50 \
    --tmp-dir ~/tempdir \
    $(awk '{print "-V '${hc_directory}'/gvcf/"$0"_normal.hc.g.vcf.gz "}' ${patients})

gatk GenotypeGVCFs \
   -R ${reference_genome} \  
   -V gendb://${pon_directory}/pon/ \
   -O ${hc_directory}/merge.vcf.gz \
   --tmp-dir ~/tempdir
```


## VQSR
```
%%bash
env=gatk

hc_directory=~/project/1-HCC/2-hc/

# gatk bundle
gatk_bundle=~/project/0-reference/2-GATK/hg19
dbsnp=dbsnp_138.b37.vcf.gz
mills=Mills_and_1000G_gold_standard.indels.b37.vcf.gz
snp1000g=1000G_phase1.snps.high_confidence.b37.vcf.gz
hapmap=hapmap_3.3.b37.vcf.gz
omni=1000G_omni2.5.b37.vcf.gz
reference_genome=${gatk_bundle}/human_g1k_v37_decoy.fasta

export PATH=~/software/miniconda3/envs/r_env/bin:$PATH

# do VQSR for HaplotypeCaller
# snp VQSR
gatk VariantRecalibrator \
    -R ${reference_genome} \
    -V ${hc_directory}/merge.vcf.gz \
    --max-gaussians 8 \
    -resource:hapmap,known=false,training=true,truth=true,prior=15.0 ${gatk_bundle}/${hapmap} \
    -resource:omni,known=false,training=true,truth=false,prior=12.0 ${gatk_bundle}/${omni} \
    -resource:1000G,known=false,training=true,truth=false,prior=10.0 ${gatk_bundle}/${snp1000g} \
    -resource:dbsnp,known=true,training=false,truth=false,prior=6.0 ${gatk_bundle}/${dbsnp} \
    -an QD -an MQ -an MQRankSum -an ReadPosRankSum -an FS -an SOR -an DP \
    -mode SNP \
    --tranches-file ${hc_directory}/merge.snp.tranches \
    --rscript-file ${hc_directory}/merge.snp.plots.R \
    -O ${hc_directory}/merge.snp.recal 

gatk ApplyVQSR \
    -R ${reference_genome} \
    -V ${hc_directory}/merge.vcf.gz \
    --truth-sensitivity-filter-level 99.0 \
    --tranches-file ${hc_directory}/merge.snp.tranches \
    --recal-file ${hc_directory}/merge.snp.recal \
    -mode SNP \
    -O ${hc_directory}/merge.snp.vqsr.vcf.gz \

# indel vqsr
gatk VariantRecalibrator \
    -R ${reference_genome} \
    -V ${hc_directory}/merge.snp.vqsr.vcf.gz \
    --max-gaussians 4 \
    -resource:mills,known=false,training=true,truth=true,prior=12.0 ${gatk_bundle}/${mills} \
    -an DP -an QD -an FS -an SOR -an ReadPosRankSum -an MQRankSum \
    -mode INDEL \
    --tranches-file ${hc_directory}/merge.snp.indel.tranches \
    --rscript-file ${hc_directory}/merge.snp.indel.plots.R \
    -O ${hc_directory}/merge.snp.indel.recal 

gatk ApplyVQSR \
    -R ${reference_genome} \
    -V ${hc_directory}/merge.snp.vqsr.vcf.gz \
    --truth-sensitivity-filter-level 99.0 \
    --tranches-file ${hc_directory}/merge.snp.indel.tranches \
    --recal-file ${hc_directory}/merge.snp.indel.recal  \
    -mode INDEL \
    -O ${hc_directory}/merge.vqsr.vcf.gz
```