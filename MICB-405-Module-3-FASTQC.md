# Module 3 Worksheet
## FASTQC
#### *Axel Hauduc - 25 September 2020*

## Breakout Room Session
### Main Excercise - Part 1
Navigate to ```/projects/micb405/data/Ebola```. You should see paired-end FASTQ files. Let's say these files just came off the sequencer, and you want to take a quick look ensure nothing went awry during the sequencing process. You're not interested in the exact stats, but you still want to take a look at all the modules in the graphical HTML report.

Perform FASTQC on these files, outputting the results to somewhere in your home directory.

Next, ```scp``` just the .html files to your computer. (Hint: remember how * refers to any name of a file within the folder. How might you modify this to refer to files *ending* in ".html"?)

Now, find the resulting pair of HTML files somewhere on your computer. There should be two. Open with the web browser of your choice!

### Main Excercise - Part 2
Take a look at the different modules in the FASTQC output. Check out "Per base sequence quality", "Per sequence quality scores", and "Per base sequence content"
Describe why are these considered acceptable by FASTQC.
For "Per base sequence content", what would you need to change for FASTQC to flag that module with a warning?

Take a look at "Per sequence GC content". What went wrong here? Would shifting this distribution so that the mean matched that of the theoretical mean be sufficient for FASTQC to flag it as acceptable?



### Bonus

Look at "Sequence Duplication Levels". This pattern is very distinct, and it's highly unusual for a whole-genome sequencing (DNA) run. However, let's say you know that this sequencing run was done perfectly. What kind of genetic or epigenetic material could have been sequenced here, rather than the simple DNA that FASTQC expects, which causes some of the same sequences to appear many times?


### More information on FastQC!
https://rtsf.natsci.msu.edu/genomics/tech-notes/fastqc-tutorial-and-faq/

https://dnacore.missouri.edu/PDF/FastQC_Manual.pdf
