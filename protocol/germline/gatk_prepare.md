# environment prepare

## ~/.condarc
```
channels:
  - defaults
  - bioconda
  - r
```

## gatk
```
conda create -n gatk gatk4=4.2.0
conda activate gatk
conda install samtools
```
### version info
```
$ gatk --version
The Genome Analysis Toolkit (GATK) v4.2.0.0
HTSJDK Version: 2.24.0
Picard Version: 2.25.0

$ samtools --version
samtools 1.13
Using htslib 1.13
```

## R
```
conda create -n r_env r-base=3.6.1
conda activate r_env
conda install r-tidyverse r-devtools r-irkernel
```

# reference 

GATK bundle file list in ftp.broadinstitute.org (use an FTP client to view), username is gsapubftp-anonymous and password is blank.

A introduction is in [GATK Resource Bundle](https://github.com/bahlolab/bioinfotools/blob/master/GATK/resource_bundle.md)

## b37/hg19

```
mkdir -p ~/project/0-reference/2-GATK/hg19
cd ~/project/0-reference/2-GATK/hg19
nohup wget -c ftp://gsapubftp-anonymous@ftp.broadinstitute.org/bundle/b37/human_g1k_v37_decoy.fasta.gz
gunzip human_g1k_v37_decoy.fasta.gz
nohup wget -c ftp://gsapubftp-anonymous@ftp.broadinstitute.org/bundle/b37/1000G_phase1.snps.high_confidence.b37.vcf.gz &
nohup wget -c ftp://gsapubftp-anonymous@ftp.broadinstitute.org/bundle/b37/hapmap_3.3.b37.vcf.gz &
nohup wget -c ftp://gsapubftp-anonymous@ftp.broadinstitute.org/bundle/b37/1000G_omni2.5.b37.vcf.gz &
nohup wget -c ftp://gsapubftp-anonymous@ftp.broadinstitute.org/bundle/b37/Mills_and_1000G_gold_standard.indels.b37.vcf.gz &
nohup wget -c ftp://gsapubftp-anonymous@ftp.broadinstitute.org/bundle/b37/dbsnp_138.b37.vcf.gz &
```
## hg38

```
mkdir -p ~/project/0-reference/2-GATK/hg38
cd ~/project/0-reference/2-GATK/hg38
nohup wget -c ftp://gsapubftp-anonymous@ftp.broadinstitute.org/bundle/hg38/Homo_sapiens_assembly38.fasta.gz
gunzip Homo_sapiens_assembly38.fasta.gz
nohup wget -c ftp://gsapubftp-anonymous@ftp.broadinstitute.org/bundle/hg38/1000G_phase1.snps.high_confidence.hg38.vcf.gz
nohup wget -c ftp://gsapubftp-anonymous@ftp.broadinstitute.org/bundle/hg38/hapmap_3.3.hg38.vcf.gz
nohup wget -c ftp://gsapubftp-anonymous@ftp.broadinstitute.org/bundle/hg38/1000G_omni2.5.hg38.vcf.gz
nohup wget -c ftp://gsapubftp-anonymous@ftp.broadinstitute.org/bundle/hg38/Mills_and_1000G_gold_standard.indels.hg38.vcf.gz
nohup wget -c ftp://gsapubftp-anonymous@ftp.broadinstitute.org/bundle/hg38/dbsnp_146.hg38.vcf.gz
```

# index
```
%%bash
env=gatk
cd ~/project/0-reference/2-GATK/hg19

samtools faidx human_g1k_v37_decoy.fasta
gatk CreateSequenceDictionary -R human_g1k_v37_decoy.fasta -O human_g1k_v37_decoy.dict
awk '{print $1"\t0\t"$2}' human_g1k_v37_decoy.fasta.fai | head -25 > chr.bed

dbsnp=dbsnp_138.b37.vcf.gz
mills=Mills_and_1000G_gold_standard.indels.b37.vcf.gz
snp1000g=1000G_phase1.snps.high_confidence.b37.vcf.gz
hapmap=hapmap_3.3.b37.vcf.gz
omni=1000G_omni2.5.b37.vcf.gz

vcfs=(${dbsnp} ${mills} ${snp1000g} ${hapmap} ${omni})
for vcf in ${vcfs[@]}
do
    gatk IndexFeatureFile -I ${vcf}
done
```
