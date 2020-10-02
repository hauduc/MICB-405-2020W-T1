# Module 4 Worksheet
## Samtools & Bcftools
#### *Axel Hauduc - 02 October 2020*

## Breakout Room Session
### Main Excercise
1. Create a new SAM file for the downsampled FASTQ files /projects/micb405/data/bordetella/F01_R1_1M.fastq and /projects/micb405/data/bordetella/F01_R2_1M.fastq aligned against your bordetella reference genome using BWA in a script.
2. Run ```samtools flagstat``` on your resulting SAM file. What do the different lines mean?
3. Next, create a script that processes the resulting SAM into an indexed, sorted BAM with duplicates removed.
4. For sorting and duplicate removal, what is a "sanity check" step you could perform on the output (hint... what function within ```samtools``` can allow you to check your binary BAM file by eye) to verify that reads have been sorted (easier), or that duplicates have been removed (harder)? What would you be looking for to confirm that your commands worked as intended?
5. Then, add a line in your script to call variants using ```bcftools```!

## More in-depth information here:
http://www.htslib.org/doc/samtools.html
http://www.htslib.org/doc/bcftools.html
