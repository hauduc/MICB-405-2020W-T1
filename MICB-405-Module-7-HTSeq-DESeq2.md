# Overview
In this tutorial, we will use DESeq2 to analyze the RNA-Seq data you aligned using STAR in the previous tutorial. This tutorial covers how to:  
1. Load counts data into R  
2. Set controls for DESeq2 by changing factor levels  
3. Run sanity checks to ensure your results make biological sense  
4. Filter differential expression tables by Padj and Log2 fold change  

# Setup
All of the data for this tutorial is located on the Orca1 server in `/projects/micb405/resources/DESeq2_tutorial/`. Use `scp` to copy the entire directory to your computer. 

Open RStudio and create a new R project (File > New Project). A window like this should pop up.

![](images/new.project1.png){width=50%}

Since we want all of our DESeq2 tutorial materials together, click on 'Existing Directory' and choose the `DESeq2_tutorial` directory that you copied from the server. 

# Scripting in R
One of the most important aspects of any bioinformatics is making sure that your code is easy to read and well documented. This means that scripts should be organized and you should use comments to split your code into different sections.

## Setting up your script
You should always begin an R script with a simple header:
```{r script header, eval = FALSE}
# Your name (e. g. Andrew Wilson)
# Title of your script (e.g. MICB405 DESeq2 tutorial)
# Date of your last update (e.g. 18 Oct 2019)
```

The next step of any R script is to load all of the packages that you plan to use:
```{r load packages, message=FALSE, warning=FALSE}
library(DESeq2)
library(tidyverse)
library(pheatmap)
library(RColorBrewer)
```

## Loading data
It is best practice to collect all input files in a folder (most often called "data") from which files will only be loaded but not overwritten.

Since there are multiple input files, we can use R to automatically generate paths to the files we want to open. All of the data files are in the same directory, so to save us some typing we can assign the path to that directory as a variable.
```{r set data directory, message=FALSE, warning=FALSE}
dir <- "data"
```

Each filename is generated from the sample name and a common suffix. In order to get a vector of sample names, we can use the `samples.csv` metadata file which we will need also to assign conditions to samples. When you are going to work on your own projects you would enter the filenames and the corresponding conditions yourself into a csv file.
```{r load metadata, message=FALSE}
#  file.path constructs filepath with the correct separator dependent of OS
sample_metadata <- read_csv(file.path(dir,"samples.csv"))
sample_metadata
```

As you can see, the metadata has 2 columns: sample names and conditions. We can generate filenames from `sample_metadata$sample` for each element in the vector (e.g. the first element is "`r sample_metadata$sample[1]`") by adding the directory "`r dir`" as prefix and ".htseq.out" as suffix.

```{r generate filepaths}
#  paste0 appends (i.e. adds together) character objects
files <- paste0(file.path(dir, sample_metadata$sample), ".htseq.out")
files
```

Then, as a quick test, you can run `all(file.exists(files))`, which should return `TRUE` to your console if the file paths are correct. First, `files.exists(files)` checks for each filename in `files` if that file  exists and returns `TRUE` or `FALSE`, resulting in a logical vector. If all files are present, then all elements in the logical vector will be `TRUE`.

```{r logical vector}
file.exists(files)
```
 The function `all` only returns `TRUE` if all elements of a logical vector are `TRUE`.
```{r check if all is there}
all(file.exists(files))
```

# Running DESeq2
DESeq2 requires that all of your data be in the shape of a SummarizedExperiment object. Since reshaping your input data with your own code would be complicated, there are functions within DESeq2 specific for the source of your counts file, which will prepare your data for use in DESeq2. Since we are using HTSeq to generate our counts data in this class, the function we need is called `DESeqDataSetFromHTSeqCount()`. One of the required arguments is `sampleTable`, which must be provided a data table with columns for "sampleName", "fileName", and for any of the independent variables you changed in your experiment (eg. sex, treatment, tissue). For that purpose, we can quickly build `sample_df` from the loaded metadata `sample_metadata` and the vector of file paths `files`, that we defined earlier.

The other required argument for `DESeqDataSetFromHTSeqCount()` is `design`. The design argument is a formula, designated by the tilde symbol `~`, and identifies which independent variable(s) you want to investigate. In our data, we have only a single independent variable, i.e. "condition". Experiments with multifactorial experimental design are a lot more complicated to investigate.

```{r DESeq dataset}
# Creates a new data table with the variables sampleName, fileName and condition
sample_df <- data.frame(sampleName = sample_metadata$sample,
                        fileName = files,
                        condition = sample_metadata$condition)

ddsHTSeq <- DESeqDataSetFromHTSeqCount(sampleTable = sample_df,
                                       design = ~ condition)
```

In a DESeq data set, the design variables are stored as a factor (a data type in R), which by default are organized alphabetically. Without further information, DESeq2 will use the condition that comes first alphabetically as reference. If you would be using "treated" and "untreated", for example, DESeq2 would incorrectly assume that the "treated" samples are the reference. To prevent any confusion, we can set the name used as reference explicitly with the argument `ref` inside of the `relevel()` function.

```{r Set controls}
## Set control condition using the relevel function
ddsHTSeq$condition <- relevel(ddsHTSeq$condition, ref = "control")
```

Now that the reference level has been set, we can run DESeq2 on the dataset.
```{r DESeq2, message=FALSE, warning=FALSE}
dds <- DESeq(ddsHTSeq)
```

# Sample Clustering
Because the distribution of RNA Seq data is highly skewed, with a few high abundance genes and many low abundance genes, it is helpful to transform our data to visualize clustering. DESeq2 comes with a function `rlog()`, which log-transforms your count data. After transformation, we can use PCA to identify which samples are more similar and if they group by one or more of the independent variables (in our case, we  have only a single variable that can take "control" or "treated").

```{r pca} 
rld <- rlog(dds)
plotPCA(rld, intgroup = "condition")
```

A distance matrix is another way to examine how samples cluster. The code chunk below calculates distances between each of the samples and then maps a colour palette (defined with `colorRampPalette`) onto it, so that more similar samples display darker colours.
```{r distance matrix}
sample_dists <- dist(t(assay(rld)))
sample_dist_matrix <- as.matrix(sample_dists)
colnames(sample_dist_matrix) <- NULL
colours <- colorRampPalette(rev(brewer.pal(9, "Blues")))(255)
pheatmap(sample_dist_matrix,
         clustering_distance_rows = sample_dists,
         clustering_distance_cols = sample_dists, 
         col = colours)
```

If the samples don't cluster together the way that you would expect them to, check for batch effects.

# Filtering and data export
Since our samples cluster according to the independent variable, we can move forward with our analysis. To examine our results, we can to save them to our global environment. DESeq2 calculates results for all combinations of levels in your experimental design. The `resultsNames()` function tells us the name of each result that DESeq2 has calculated, so we can choose which one to look at by specifying "name = " in `results()`. The `results()` function will output a matrix of all genes, including those with similar or differential expression between the two conditions.

```{r generate result table}
resultsNames(dds)
res <- results(dds, name = "condition_treatment_vs_control")
head(res)
```

Since we are only interested in genes which are differentially expressed, we can use the `subset()` function from base R to only keep results with an adjusted P value < 0.05, and then save the results to a CSV file.

```{r subset and export results, eval=FALSE}
significant_res <- subset(res, padj < 0.05)
write.csv(as.data.frame(significant_res), file = "treat_vs_control_05.csv")
```

If you want more information about DESeq2, you can access the package vignette by running:
```{r vignette, eval=FALSE}
vignette("DESeq2")
```