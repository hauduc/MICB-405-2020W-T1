# Module 6 Worksheet
## STAR
#### *Axel Hauduc - 23 October 2020*

### Resources:
Original paper: https://www.nature.com/articles/sdata2017185

Data source from original paper: https://www.ebi.ac.uk/arrayexpress/experiments/E-MTAB-6081/samples/

STAR documentation: https://github.com/alexdobin/STAR/blob/master/doc/STARmanual.pdf

### Step 1: Download mouse reference genome (FASTA, mm10) and gene annotation file (GTF)
```
mkdir ~/star && cd ~/star
wget ftp://ftp.ensembl.org/pub/release-84/fasta/mus_musculus/dna/Mus_musculus.GRCm38.dna.primary_assembly.fa.gz
wget ftp://ftp.ensembl.org/pub/release-84/gtf/mus_musculus/Mus_musculus.GRCm38.84.gtf.gz
```

### Step 2: Unzip just the fasta reference and gtf file, and make directories for your STAR index and for each of your sample files, which you will need to name
```
gunzip *{gtf,fa}.gz
mkdir STARIndex sample1 sample2 sample3 sample4 sample_n # Make sure to rename these based on your FASTQ files
```

### Step 3: Generate a STAR index based on your mm10 FASTA and your GTF files, **then proceed to Step 4 while it runs**
```
STAR --runMode genomeGenerate \
--genomeDir STARIndex \
--genomeFastaFiles Mus_musculus.GRCm38.dna.primary_assembly.fa \
--sjdbGTFfile Mus_musculus.GRCm38.84.gtf \
--sjdbOverhang 49 \
--runThreadN 16
```

### Step 4: Download healthy tissue RNAseq FASTQ files
#### Discuss among your group an idea for what you could investigate given the data you have available.
Download the FASTQ files that will allow you to accomplish this. Which ones did you choose, and for what purpose?
```
# In my case, I'm downloading 4 tissue types from the first individual mouse in order to observe differential gene expression in these areas.
# These are just an example! 
wget ftp://ftp.sra.ebi.ac.uk/vol1/fastq/ERR213/000/ERR2130640/ERR2130640.fastq.gz
wget ftp://ftp.sra.ebi.ac.uk/vol1/fastq/ERR213/009/ERR2130649/ERR2130649.fastq.gz
wget ftp://ftp.sra.ebi.ac.uk/vol1/fastq/ERR213/003/ERR2130623/ERR2130623.fastq.gz
wget ftp://ftp.sra.ebi.ac.uk/vol1/fastq/ERR213/004/ERR2130614/ERR2130614.fastq.gz
```

### Step 5: When the STAR index is ready, run STAR, outputting into a separate directory for each of the samples you wish to align
Since you won't have time for the whole process to run today, create a script that runs this STAR command for each of your samples and run it in a way so that it won't stop when you exit your shell.

Be sure that --readFilesIn, --outFileNamePrefix, and --outSAMattrRGline changes for each of your samples
```
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

### Next week:
You will hopefully have a set of BAM files that are ready to go next week that we will be able to analyze with HTSeq and DESeq2!

