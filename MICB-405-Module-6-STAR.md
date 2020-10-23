# Module 6 Worksheet
## STAR
#### *Axel Hauduc - 23 October 2020*

## Resources:
### Original paper: https://www.nature.com/articles/sdata2017185
### Data source from original paper: https://www.ebi.ac.uk/arrayexpress/experiments/E-MTAB-6081/samples/

## Downloading mouse reference and gene annotations
```
wget ftp://ftp.ensembl.org/pub/release-84/fasta/mus_musculus/dna/Mus_musculus.GRCm38.dna.primary_assembly.fa.gz
wget http://ftp.ensembl.org/pub/release-84/gtf/mus_musculus/Mus_musculus.GRCm38.84.gtf.gz
```

## Download healthy tissue RNAseq FASTQ files
```
# In my case, I'm downloading 4 tissue types from the first individual mouse in order to observe differential gene expression in these areas. 
wget ftp://ftp.sra.ebi.ac.uk/vol1/fastq/ERR213/000/ERR2130640/ERR2130640.fastq.gz
wget ftp://ftp.sra.ebi.ac.uk/vol1/fastq/ERR213/009/ERR2130649/ERR2130649.fastq.gz
wget ftp://ftp.sra.ebi.ac.uk/vol1/fastq/ERR213/003/ERR2130623/ERR2130623.fastq.gz
wget ftp://ftp.sra.ebi.ac.uk/vol1/fastq/ERR213/004/ERR2130614/ERR2130614.fastq.gz
```
## Unzip just the fasta reference and gtf file, and make directories for your STAR index and sample files
```
gunzip *{gtf,fa}.gz
mkdir STARIndex sample1 sample2 sample3 sample4 sample_n
```
## 
```
STAR --runMode genomeGenerate \
--genomeDir STARIndex \
--genomeFastaFiles Mus_musculus.GRCm38.dna.primary_assembly.fa \
--sjdbGTFfile Mus_musculus.GRCm38.84.gtf \
--sjdbOverhang 49 \
--runThreadN 16
```

## Run the STAR aligner, outputting into your 
STAR --genomeDir STARIndex/ \
--readFilesIn sample1.fastq.gz \
--outFileNamePrefix sample1/sample1.fastq.gz \
--runThreadN 8 \
--limitBAMsortRAM 60000000000 \
--outSAMattrRGline ID:sample1.fastq.gz SM:sample1.fastq.gz \
--outBAMsortingThreadN 8 \
--outSAMtype BAM SortedByCoordinate \
--outSAMunmapped Within \
--outSAMstrandField intronMotif \
--readFilesCommand zcat \
--chimSegmentMin 20 \
--genomeLoad NoSharedMemory
```