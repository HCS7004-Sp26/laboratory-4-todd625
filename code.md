## Login in OSC, either by using SSH in your terminal, a command line instanc in your web browser, or using VSCode from OSC
ssh todd625@pitzer.osu.edu

## Check where you are and make sure you create your working directory in the right place
```shell
cd /fs/scratch/PAS3260/Fiona/
mkdir Lab4
```

## Recursively add Jonathan files into my Lab 4
```shell
cp -r /fs/scratch/PAS3260/Jonathan/Lab_4/* .
```

## Go to your working directory and execute:
```shell
cd /fs/scratch/PAS3260/Fiona/Lab4/Lab_4
ls -l
ls -Llh
ls -lh
```

### What do you see? -l has the full values for size written out, but -Llh and -lh are human readable so the values are shortened and have units.
## There is a md5 file, what are md5 files for? md5 is used to check data integrity (make sure downloaded file is the same as the original)
## Let's work with this file:
```shell
md5sum -c Lab5.md5 > Checking.txt
grep "FAIL" Checking.txt
## Parenthesis on integrity checking
touch myDNA.fasta
nano myDNA.fasta
md5sum myDNA.fasta
# Inspect the files
```
## Now, let's explore the files a little:
```shell
head ref_Run1CB3334.fasta
head Rice.gff3
head Run1CB3334.fastq
head Run1CB3334.vcf
head Run1CB3334.fastq.gz
head Run1CB3334.bam
```
### Do you see something weird?
## For `Run1CB3334.fastq.gz` try:
```shell
zcat Run1CB3334.fastq.gz | head
```

## Let's check the fastq file in a little more detail
```shell
head -100 Run1CB3334.fastq | less
```

### How would you count the number of reads in the fastq file?
```shell
wc -l Run1CB3334.fastq
```
# Is it? Try:
zcat Run1CB3334.fastq.gz | echo "$((`wc -l` / 4))"
```
Why the difference?
### Let's download some data from NCBI
```
module load sratoolkit/2.10.7
# What is sratoolkit?
fastq-dump --gzip --split-files --readids --origfmt ERR3638927
```
(a faster option is fasterq-dump, how can you look for information about this command?)
```shell
zcat ERR3638927.fastq.gz | head
```
## Let's download a SAM file
```shell
sam-dump --output-file ERR3638927.sam ERR3638927
```
To manipulate the SAM file we will need samtools:
```shell
module load samtools/1.10
```
Then, we can make a sorted BAM file:
```shell
samtools view -bS ERR3638927.sam | samtools sort -o ERR3638927_sorted.bam
```
Index it:
```shell
samtools index ERR3638927_sorted.bam
```
Make it a FASTA file:
```shell
samtools fasta ERR3638927_sorted.bam > ERR3638927.fasta
head ERR3638927.fasta
```
And index it:
```shell
samtools faidx ERR3638927.fasta
```
## Finally, let's check the VCF file
```shell
head -100 Run1CB3334.vcf | less
module load vcftools/0.1.16
vcftools --vcf Run1CB3334.vcf
vcftools --vcf Run1CB3334.vcf --maf 0.05
vcftools --vcf Run1CB3334.vcf --maf 0.05 --recode --out Run1CB3334_Filtered_maf05
vcftools --vcf Run1CB3334_Filtered_maf05.recode.vcf
```
## Can we check the VCF file using a container as in Lab 3?
```shell
module unload vcftools/0.1.16
vcftools --help
apptainer pull vcftools.sif docker://quay.io/biocontainers/vcftools:0.1.16--pl5321hdcf5f25_9
apptainer exec vcftools.sif vcftools --help
apptainer exec vcftools.sif vcftools --vcf Run1CB3334.vcf --maf 0.05 --recode --out Run1CB3334_Filtered_maf05_from_container
apptainer exec vcftools.sif vcftools --vcf Run1CB3334_Filtered_maf05_from_container.recode.vcf
```
## How could we use a container of samtools?
```shell
module unload samtools/1.10
samtools --help
apptainer pull samtools.sif docker://
apptainer exec samtools.sif samtools --help
```
Followign the same structure as VCFtools, define the commands to sort and index fasta and BAM files using the samtools container
