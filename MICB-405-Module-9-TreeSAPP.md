# Module 9 Worksheet
## TreeSAPP
#### *Axel Hauduc - 13 November 2020*

### Resources:
Original article: https://academic.oup.com/bioinformatics/advance-article/doi/10.1093/bioinformatics/btaa588/5868555

Original installation guide: https://github.com/hallamlab/TreeSAPP

Tutorial adapted from: https://github.com/hallamlab/TreeSAPP/wiki/PmoA-&-AmoA-reference-package-tutorial

GitHub for this tutorial: https://github.com/hauduc/MICB-405-2020W-T1/blob/master/MICB-405-Module-9-TreeSAPP.md

### Overview
Happy Friday! Today we will be working through a basic TreeSAPP workflow using example data from the TreeSAPP GitHub. As there is a lot to take in here, I encourage everyone to take a break after each step and review the supporting documentation to make sure you're understanding what you're doing at each step so you can better understand the output you'll be getting from TreeSAPP. 

Firstly, you will need to modify your conda config to set it to download the latest versions of each required software dependency. This is in contrast to Anvi'o, which required some outdated packages that were present in the bioconda and source-forge repositories:
```
conda config --set channel_priority false
```

Next, you will need to create a new conda environment for TreeSAPP. Make sure to ```conda deactivate``` out of the environment you're currently in.

```
conda create -n treesapp_cenv -c bioconda -c conda-forge treesapp
conda activate treesapp_cenv
```

You can review all the possible TreeSAPP sub-commands by simply running the command:
```
treesapp
```

Alright! Now that you have treesapp successfully installed (hopefully), you will need to get the raw data that will be used to create a reference package and a resulting marker contig map.

You'll now need to download the files that will be used to create a reference package. If you're unsure about what a reference package is, go to https://academic.oup.com/bioinformatics/advance-article/doi/10.1093/bioinformatics/btaa588/5868555 section 2 and review!
```
mkdir ~/treesapp_tutorial && cd ~/treesapp_tutorial

nit="https://raw.githubusercontent.com/hallamlab/RefPkgs/master/Nitrogen_metabolism/Nitrification"
guc="https://raw.githubusercontent.com/hallamlab"

wget ${nit}/XmoA/TIGR03080.faa
wget ${nit}/XmoA/ENOG5028JPK_EggNOGv5.faa
wget ${nit}/XmoA/p_amoA_FunGene9.5_isolates_C65S300_uclust99.faa
wget ${nit}/XmoA/uniprot-AmoA.fasta
wget ${nit}/XmoA/uniprot-EmoA.fasta
wget ${nit}/XmoA/uniprot-PxmA.fasta
wget ${guc}/TreeSAPP/master/dev_utils/TIGRFAM_seed_named.faa
wget ${guc}/TreeSAPP/master/dev_utils/TIGRFAM_info.tsv
```

### Creating a reference package
This section will rely on the submodule treesapp create which has been explained in greater detail here.

When building reference packages, it is always best to start with sequences that you can trust. Although at times you may have to settle for sequences that were automatically annotated based on sequence homology inferred by pairwise alignment, if you can source manually curated sequences -- such as those hosted by SwissProt -- always use those.

Fortunately for us, there are plenty of sources for curated PmoA & AmoA sequences. These were downloaded from TIGRFAM, EggNOG v5.0, and FunGene. For now we'll just use the TIGRFAM seed sequences and EggNOG sequences as these are best curated. Let's call this our seed reference package. There are 72 sequences in these fasta files (confirmed with grep -c "^>" *faa).

To speed things along we'll be using FastTree to infer the phylogeny and will skip bootstrapping with --fast --bootstraps 0. To remove potential redundant sequences, we'll cluster the candidate reference sequences at 97% similarity using the argument --similarity/-p. When clustering the candidate sequences TreeSAPP would normally ask which sequence to use for the representative of the cluster. This can be handy in cases when some sequences are better annotated and/or are especially important. To speed things up even more the flag --headless will prevent these requests. This command will take ~2 minutes to complete.

```
cat ENOG5028JPK_EggNOGv5.faa TIGR03080.faa > XmoA_seed.faa  # Create the input FASTA file
treesapp create --fast --bootstraps 0 --headless --overwrite --delete --cluster --trim_align -n 4 -m prot -p 0.97 --fastx_input XmoA_seed.faa -c XmoA --output XmoA_seed
```

The final reference package file is located in ./XmoA_seed/final_outputs/XmoA_build.pkl. This file contains all the individual components of a reference package (multiple sequence alignment, profile HMM, phylogenetic tree, taxonomic lineages) as well as some other data. These files were bundled up using the joblib Python library. They can be accessed individually using the submodule treesapp package.

You'll notice that you're prompted to edit the 'refpkg_code' attribute of the reference package once ```treesapp create``` finished:

```
# To integrate this package for use in TreeSAPP the following steps must be performed:
# 1. Replace the current refpkg_code 'Z1111' with:
# `treesapp package edit refpkg_code $code --overwrite --refpkg_path XmoA_seed/final_outputs/XmoA_build.pkl` where 
# $code is a unique identifier.
# 2. Copy XmoA_seed/final_outputs/XmoA_build.pkl to a directory containing other reference packages you want to analyse. 
# This may be in treesapp_venv/lib/python3.7/site-packages/treesapp//data/ or elsewhere
```

This code must be unique across all the reference packages you're using and follow the same format: first character is alphabetical, followed by four numbers. It's assigned the temporary string 'Z1111' but should be changed if you're using multiple reference packages. Let's change it to 'N0102' with:

```
treesapp package edit refpkg_code N0102 --overwrite --refpkg_path XmoA_seed/final_outputs/XmoA_build.pkl
```

If you plan on just using this one reference package then you don't really need to worry about changing it.

We can also modify the reference package's description while we're here:

```
treesapp package edit description "Alpha subunits of copper membrane monooxygenase enzymes" --overwrite --refpkg_path XmoA_seed/final_outputs/XmoA_build.pkl
```

### Classify new sequences with TreeSAPP assign
At this point you're able to use this reference package to classify sequences in any dataset. For this tutorial though, we're going to classify PmoA and AmoA sequences sourced from FunGene. The sequences are from genomes of isolates and, because of FunGene's quality annotation pipeline, can likely be trusted.

The command treesapp assign is perhaps the most popular command. Make sure to review what it does here: https://github.com/hallamlab/TreeSAPP/wiki/Classifying-sequences-with-treesapp-assign

For this command, we will use BMGE to trim the multiple sequence alignments prior to phylogenetic placement with the --trim_align flag. To use the PmoA and AmoA reference package we've built instead of any other reference packages that come with TreeSAPP we can use the argument --refpkg_dir with the path to a directory containing reference packages we want to use.

```
treesapp assign -n 4 -m prot --trim_align --refpkg_dir XmoA_seed/final_outputs/ --fastx_input p_amoA_FunGene9.5_isolates_C65S300_uclust99.faa --output p_amoA_FunGene9.5_isolates_assign/
```

The classification table contains detailed information corresponding to each classified query sequence. It is a tab-delimited table that can be read by a variety of tools. At this point, you can import and begin visualizing different components of the classifications in R.

Here are the fields and a brief description of each:

**Sample** is the prefix of the FASTA file storing query sequences, except when the --fpkm flag is used in which case the name is taken from the FASTQ file(s). This ensures the rows are unique in case multiple classification tables are concatenated. Note: the intervening headers would need to be removed, potentially with grep -v "^Sample".

**Query** is the name of the classified query sequence (e.g. contig or ORF name).

**Marker:** Name of the reference package (i.e. gene or protein name) the query sequence was classified as. If you are unfamiliar with this name, treesapp info -v or treesapp package view description with the path to the reference package can give you a brief description.

**Start_pos & End_pos:** This will give you the length of the query sequence that was aligned to the reference profile HMM and subsequently used for phylogenetic placement. This could be, and usually is, a subsequence of the full-length ORF or protein sequence.

**Taxonomy:** Taxonomic label assigned to the query sequence based on LCA alone. Should only be used for development and testing purposes.

**Abundance:** The abundance of the query sequence in the dataset. If --fpkm was not used it is set to 1, otherwise it is a fragments per kilobase per million mappable reads (FPKM) value.

**iNode:** The internal node number representing where on the reference phylogeny the query sequence was placed.

**LWR:** Likelihood weight ratio assigned to the query by RAxML-EPA.

**EvoDist:** Sum of all phylogenetic distances calculated by RAxML-EPA as well as the average node-to-tip distances.

**Distances:** A comma-separated list of the distances that summed to equal EvoDist

The output will be located in ```/treesapp_tutorial/p_amoA_FunGene9.5_isolates_assign/final_outputs``` under the name ```marker_contig.tsv```

### Analysis in R

Now, time to take a look at it on your own computer! SCP it to a new folder on your laptop, and create an R project in that folder using RStudio.

Load ```tidyverse```, import the tab-separated values file into R, and take a look at it by clicking on the R object or running ```view()```

Filter your R dataframe by unique taxonomies and take a look at the EvoDist values within those taxonomies. What does this tell you about the corresponding contigs aligned to each taxonomy? If you're unsure, make sure to take a quick look at what the original article has to say about evolutionary distances.

If you want, you can alse ```group_by()``` your dataframe by unique taxonomy and calculate mean EvoDist that way!


test2