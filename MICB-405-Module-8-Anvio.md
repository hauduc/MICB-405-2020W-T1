# Module 8 Worksheet
## Anvio
#### *Axel Hauduc - 06 November 2020*
#### Based on Meren Lab Anvio Tutorial V2

### Resources:
Original article: https://peerj.com/articles/1319/

Metagenomics tutorial: http://merenlab.org/2016/06/22/anvio-tutorial-v2/

Installation tutorial: http://merenlab.org/2016/06/26/installation-v2/

### Overview
In this tutorial, we will use Anvio to visualize example metagenomic data. You will then be able to apply what you learned here to the Saanich metagenomic data should you choose the metagenomic project.

### Installation for Linux (local or server) & Windows WSL

If you haven't already, install Anaconda on your server profile or local machine:

https://www.anaconda.com/products/individual
```
mkdir ~/anvio_tutorial && cd ~/anvio_tutorial
conda config --set auto_activate_base false
conda config --set channel_priority false
conda create -y --name anvio-6.2 python=3.6
conda activate anvio-6.2
conda install -y -c conda-forge -c bioconda anvio==6.2
pip install "cherrypy>=3.0.8,<9.0.0"
```

### Installation for macOS (untested by Axel)

If you haven't already, install Anaconda on your server profile or local machine:

https://www.anaconda.com/products/individual
```
# first get a copy of the following file (if you get a "command not found"
# error for wget, you can either install it first, or manually download the
# file from that URL:
wget http://merenlab.org/files/anvio-conda-environments/anvio-environment-6.2.yml

# just to make sure there is not a v6.2 environment already: 
conda env remove --name anvio-6.2

# create a new v6.2 environment using the file you just downloaded:
conda env create -f anvio-environment-6.2.yml

# activate that environment
conda activate anvio-6.2
```

### Download data
```
wget https://raw.githubusercontent.com/meren/anvio/master/anvio/tests/sandbox/contigs.fa https://github.com/meren/anvio/raw/master/anvio/tests/sandbox/SAMPLE-01-RAW.bam https://github.com/meren/anvio/raw/master/anvio/tests/sandbox/SAMPLE-02-RAW.bam https://github.com/meren/anvio/raw/master/anvio/tests/sandbox/SAMPLE-03-RAW.bam

```

### Creating an anvi’o contigs database
An anvi’o contigs database will keep all the information related to your contigs: positions of open reading frames, k-mer frequencies for each contigs, where splits start and end, functional and taxonomic annotation of genes, etc. The contigs database is an essential component of everything related to anvi’o metagenomic workflow.

The following is the simplest way of creating a contigs database:

```
anvi-gen-contigs-database -f contigs.fa -o contigs.db -n 'A contigs database'
```

When you run this command, anvi-gen-contigs-database will, compute k-mer frequencies for each contig (the default is 4, but you can change it using --kmer-size parameter if you feel adventurous).

Soft-split contigs longer than 20,000 bp into smaller ones (you can change the split size using the --split-length). When the gene calling step is not skipped, the process of splitting contigs will consider where genes are and avoid cutting genes in the middle. For very very large assemblies this process can take a while, and you can skip it with --skip-mindful-splitting flag.

Identify open reading frames using Prodigal, the bacterial and archaeal gene finding program developed at Oak Ridge National Laboratory and the University of Tennessee. If you don’t want gene calling to be done, you can use the flag --skip-gene-calling to skip it. If you have your own gene calls, you can provide them to be used to identify where genes are in your contigs. All you need to do is to use the parameter --external-gene-calls (the format for external gene calls file is explained here).

Almost every anvi’o program comes with a help menu that explains available parameters in detail:

```
anvi-gen-contigs-database --help
```

Once you have your contigs database, you can start importing things into it, or directly go to the profiling step.

#### anvi-run-hmms

Although this is absolutely optional, you shouldn’t skip this step. Anvi’o can do a lot with hidden Markov models (HMMs provide statistical means to model complex data in probabilistic terms that can be used to search for patterns, which works beautifully in bioinformatics where we create models from known sequences, and then search for those patterns rapidly in a pool of unknown sequences to recover hits). To decorate your contigs database with hits from HMM models that ship with the platform (which, at this point, constitute multiple published bacterial single-copy gene collections), run this command:

```
anvi-run-hmms -c contigs.db
```

When you run this command (without any other parameters),

It will utilize multiple default bacterial single-copy core gene collections and identify hits among your genes to those collections using HMMER. If you have already run this once, and now would like to add an HMM profile of your own, that is easy. You can use --hmm-profile-dir parameter to declare where should anvi’o look for it. Or you can use the --installed-hmm-profile parameter to only run a specific default HMM profile on your contigs database.

Note that the program will use only one CPU by default, especially if you have multiple of them available to you, you should use the --num-threads parameter. It significantly improves the runtime, since HMMER is truly an awesome software.

If you are running COGs for the first time, you will need to set them up using:

```
anvi-setup-ncbi-cogs
anvi-run-ncbi-cogs -c contigs.db
```

### Profiling BAM files
It is time to initialize your BAM file, and create an anvi’o profile for your sample.

anvi-init-bam
Anvi’o requires BAM files to be sorted and indexed. In most cases the BAM file you get back from your mapping software will not be sorted and indexed. This is why we named the BAM file for our mock samples as SAMPLE-01-RAW.bam, instead of SAMPLE-01.bam.

If your BAM files already sorted and indexed (i.e., for each .bam file you have, there also is a .bam.bai file in the same directory), you can skip this step. Otherwise, you need to initialize your BAM files:

```
anvi-init-bam SAMPLE-01-RAW.bam -o SAMPLE-01.bam
```

But of course it is not fun to do every BAM file you have one by one. *Can you make a script that loops this functionality over your three BAM files?*

#### anvi-profile
In contrast to the contigs database, an anvi’o profile database stores sample-specific information about contigs. Profiling a BAM file with anvi’o using anvi-profile creates a single profile that reports properties for each contig in a single sample based on mapping results. Each profile database links to a contigs database, and anvi’o can merge single profiles that link to the same contigs database into merged profiles (which will be covered later).

In other words, the profiling step makes sense of each BAM file separately by utilizing the information stored in the contigs database. It is one of the most critical (and also most complex and computationally demanding) steps of the metagenomic workflow.

The simplest form of the command that starts the profiling looks like this:

```
anvi-profile -i SAMPLE-01.bam -c contigs.db
```

When you run anvi-profile it will:

Process each contig that is longer than 2,500 nts by default. You can change this value by using the --min-contig-length flag. But you should remember that the minimum contig length should be long enough for tetra-nucleotide frequencies to have enough meaningful signal. There is no way to define a golden number for minimum length that would be applicable to genomes found in all environments. We empirically chose the default to be 2,500, and have been happy with it. You are welcome to experiment, but we advise you to never go below 1,000. You also should remember that the lower you go, the more time it will take to analyze all contigs. You can use the –list-contigs parameter to have an idea how many contigs would be discarded for a given --min-contig-length parameter. If you have an arbitrary list of contigs you want to profile, you can use the flag --contigs-of-interest to ignore the rest.

Make up an output directory, and sample names for you. We encourage you to use the --output-dir parameter to tell anvi’o where to store your output files, and the --sample-name parameter to give a meaningful, preferably not-so-long sample name to be stored in the profile database. This name will appear almost everywhere, and changing it later will be a pain.

Processing of contigs will include,

The recovery of mean coverage, standard deviation of coverage, and the average coverage for the inner quartiles (Q1 and Q3) for a given contig. Profiling will also create an HD5 file where the coverage value for each nucleotide position will be kept for each contig for later use. While the profiling recovers all the coverage information, it can discard some contigs with very low coverage declared by --min-mean-coverage parameter (the default is 0, so everything is kept).

The characterization of single-nucleotide variants (SNVs) for every nucleotide position, unless you use --skip-SNV-profiling flag to skip it altogether (you will definitely gain a lot of time if you do that, but then, you know, maybe you shouldn’t). By default, the profiler will not pay attention to any nucleotide position with less than 10X coverage. You can change this behavior via --min-coverage-for-variability flag. Anvi’o uses a conservative heuristic to not report every position with variation: i.e., if you have 200X coverage in a position, and only one of the bases disagree with the reference or consensus nucleotide, it is very likely that this is due to a mapping or sequencing error, and anvi’o tries to avoid those positions. If you want anvi’o to report everything, you can use --report-variability-full flag. We encourage you to experiment with it, maybe with a small set of contigs, but in general you should refrain reporting everything (it will make your databases grow larger and larger, and everything will take longer for -99% of the time- no good reason).

Finally, because single profiles are rarely used for genome binning or visualization, and since the clustering step increases the profiling runtime for no good reason, the default behavior of profiling is to not cluster contigs automatically. However, if you are planning to work with single profiles, and if you would like to visualize them using the interactive interface without any merging, you can use the --cluster-contigs flag to initiate clustering of contigs. In this case anvi’o would use default clustering configurations for single profiles, and store resulting trees in the profile database. You do not need to use this flag if you are planning to merge multiple profiles (i.e., if you have more than one BAM file to work with, which will be the case for most people).

Every anvi’o profile that will be merged later must be generated with the same exact parameters and against the same contigs database. Otherwise, anvi’o will complain about it later, and likely nothing will get merged. Just saying.

### Working with anvi’o profiles
You have all your BAM files profiled! Did it take forever? Well, sorry about that. But now you are golden.

#### anvi-merge
The next step in the workflow is to to merge all anvi’o profiles.

This is the simplest form of the anvi-merge command:

```
anvi-merge SAMPLE-01/PROFILE.db \
           SAMPLE-02/PROFILE.db \
           SAMPLE-03/PROFILE.db \
           -o SAMPLES-MERGED \
           -c contigs.db \
           --enforce-hierarchical-clustering
```

When you run anvi-merge, it will attempt to create multiple clusterings of your splits using the default clustering configurations. Please take a quick look at the default clustering configurations for merged profiles –they are pretty easy to understand. By default, anvi’o will use euclidean distance and ward linkage algorithm to organize contigs; however, you can change those default values with the --distance and --linkage parameters (available options for distance metrics and linkage algorithms are listed in this release note). Hierarchical clustering results are necessary for comprehensive visualization and human guided binning; therefore, by default, anvi’o attempts to cluster your contigs using default configurations. You can skip this step by using --skip-hierarchical-clustering flag. But even if you don’t skip it, anvi’o will skip it for you if you have more than 20,000 splits, since the computational complexity of this process will get less and less feasible with increasing number of splits. That’s OK, though. There are many ways to recover from this. On the other hand, if you want to teach everyone who is the boss, you can force anvi’o try to cluster your splits regardless of how many of them are there by using --enforce-hierarchical-clustering flag. You have the power.


### Visualizing data on your laptop

#### anvi-interactive
Anvi’o interactive interface is one of the most sophisticated parts of anvi’o. In the context of the metagenomic workflow, the interactive interface allows you to browse your data in an intuitive way as it shows multiple aspects of your data, visualize the results of unsupervised binning, perform supervised binning, or refine existing bins.

The interactive interface of anvi’o is written from scratch, and can do much more than what is mentioned above. In fact, you don’t even need anvi’o profiles to visualize your data using the interactive interface. But since this is a tutorial for the metagenomic workflow, we will save you from these details. If you are interested in learning more, there are other resources that provide detailed descriptions of the anvi’o interactive interface and data formats it works with.

Most things you did so far (creating a contigs database, profiling your BAM files, merging them, etc) required you to work on a server. But anvi-interactive will require you to download the merged directory and your contigs databases to your own computer, because anvi-interactive uses a browser to interact with you.

This is the simplest way to run the interactive interface on your merged anvi’o profile:

```
anvi-interactive -p SAMPLES-MERGED/PROFILE.db -c contigs.db
```

While this command runs, Anvi'o will then act as an internal "server" which you can access by typing ```localhost:8080``` in your web browser, preferably Chrome. You may have a slighly different port number on your computer to make sure to pay attention to whatever the terminal gives you and input that.

Then press Draw to create an initial visualization.

Here is some additional information about the interactive interface (please see a full list of other options by typing anvi-interactive -h):

Storing visual settings: The interactive interface allows you to tweak your presentation of data remarkably, and store these settings in profile databases as “states”. If there is a state stored with the name default, or if you specify a state when you are running the program via the --state parameter, the interactive interface will load it, and proceeds to visualize the data automatically (without waiting for you to click the Draw button). States are simply JSON formatted data, and you can export them from or import them into an anvi’o profile database using the anvi-export-state and anvi-import-state programs. This way you can programmatically edit state files if necessary, and/or use the same state file for multiple projects.

Using additional data files: When you need to display more than what is stored in anvi’o databases for a project, you can pass additional data files to the interactive interface. If you have a newick tree as an alternative organization of your contigs, you can pass it using the --tree parameter, and it would be added to the relevant combo box. If you have extra layers to show around the tree (see Figure 1 in this publication as an example), you can use the --additional-layers parameter. Similarly, you can pass an extra view using the --additional-view parameter. Files for both additional layers and additional view are expected to be TAB-delimited text files and information is to be provided at the split-level (if you hate the way we do this please let us know, and we will be like “alright alright” and finally fix it). Please see the help menu for more information about the expected format for these files.

Setting a taxonomic level. When information about taxonomy is available in an anvi’o contigs database, anvi’o interactive interface will start utilizing it automatically and you will see a taxonomy layer in the interface for each of your contigs. By default the genus names will be used, however, you can change that behavior using the --taxonomic-level flag.

### Deliverable

Play around with different ways to visualize the data and save it as an SVG file, then submit that SVG file as your assignment.

Can you switch to a phylogram view? What additional information does this help display about your separate bins?
