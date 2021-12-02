# kingfisher

follow github [repo](https://github.com/wwood/kingfisher-download)

## install

```
$ git clone https://github.com/wwood/kingfisher-download
$ cd kingfisher-download
$ conda env create -n kingfisher -f kingfisher.yml
$ conda activate kingfisher
$ cd bin
$ echo 'export PATH='$PWD':$PATH' >> ~/.bashrc
```

## aspera install
The aspera is needed in `ena-ascp`. If you don't prefer this download method, skip this step.
```
$ wget https://d3gcli72yxqn2z.cloudfront.net/connect_latest/v4/bin/ibm-aspera-connect_4.1.0.46-linux_x86_64.tar.gz
$ tar xzf ibm-aspera-connect_4.1.0.46-linux_x86_64.tar.gz
$ bash ibm-aspera-connect_4.1.0.46-linux_x86_64.sh
$ echo 'export PATH=~/.aspera/connect/bin:$PATH' >> ~/.bashrc
```


# use

The ERR, SRR, PRJNA code can be directly used here, thus you can get rid of the generating url and running wget step.

The `ena-ascp` and `ena-ftp` methods are likely to fail in mainland due to ebi banned by GFW and I recommend to use `aws-http`.

Example:

## download

specify `-f fastq.gz` to force it transform `fastq` to `fastq.gz`
```
# -p for PRJNA and SRR
$ kingfisher get -p PRJNA504942 -m aws-http prefetch -f fastq.gz --download-threads 16 -t 16

# -r for ERR
$ kingfisher get -r ERR1739691 -m ena-ascp ena-ftp -f fasta
```

## annotate
Download the annotation for later use.

```
$ kingfisher annotate -r ERR1739691
12/01/2021 04:26:20 PM INFO: Kingfisher v0.0.1-dev
12/01/2021 04:26:20 PM INFO: Querying NCBI esearch for 1 distinct accessions e.g. ERR1739691
12/01/2021 04:26:22 PM INFO: Querying NCBI efetch for 1 distinct IDs e.g. 4165047
run        | study_accession | Gbp   | library_strategy | library_selection | model               | sample_name | taxon_name
---------- | --------------- | ----- | ---------------- | ----------------- | ------------------- | ----------- | ----------
ERR1739691 | ERP017539       | 2.382 | WGS              | RANDOM            | Illumina HiSeq 2500 | MM1_1       | metagenome
12/01/2021 04:26:23 PM INFO: Kingfisher done.

$ kingfisher annotate -p PRJNA504942 -f tsv > annotation.tsv
12/01/2021 04:29:32 PM INFO: Kingfisher v0.0.1-dev
12/01/2021 04:29:47 PM INFO: Querying NCBI esearch for 364 distinct accessions e.g. SRR9139995
12/01/2021 04:29:48 PM INFO: Querying NCBI efetch for 364 distinct IDs e.g. 7941212
12/01/2021 04:30:03 PM INFO: Kingfisher done..

$ head -3 annotation.tsv
run     study_accession Gbp     library_strategy        library_selection       model   sample_name     taxon_name
SRR9330212      SRP198194       15.798  WCS     cDNA    Illumina HiSeq 3000     DSP116  Homo sapiens
SRR9326180      SRP198194       22.417  WCS     cDNA    Illumina HiSeq 3000     DST46   Homo sapiens
```
