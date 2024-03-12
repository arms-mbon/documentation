# **Processing ARMS data using the ARMS workflow**

For post-sequencing ARMS data processing, we use the pipeline PEMA (Pipeline for Environmental Metabarcoding Analysis), which has now been integrated in the LifeWatch ERIC IJI workflow on the Tesseract platform: [Access the workflow here](https://tesseract.lifewatch.dev/arms/workflow-information)

For operational details on executing a workflow and getting the outputs, refer to the tutorial offered by LifeWatch ERIC following [this link](https://training.lifewatch.eu/user-manuals-and-tutorials/resources/?category=17).

Upon initiating the workflow on Tesseract, selected sequences and accompanying metadata undergo steps to transform raw genetic data into taxonomic categorizations and screenings for invasive species. 

*Important information for the selection of sequences*

For some samples, Field/Area replicates and Sample/Technical replicates are available. You can find the replicate IDs for each MaterialSampleID in [replicates_list.csv](https://github.com/arms-mbon/documentation/blob/main/arms_in_tesseract/replicates_list.csv). Field/Area replicates are used when ARMS units are about about 10m or less apart (in the 3 dimensions) and are deployed and retrieved within a few days of each other. These ARMS units have different names (e.g. VH1 and VH2). Sample/technical replicates were used for some material samples with poor results from the first run on the sequencing, and so new sequencing was done on stored material (this information can also be seen in the column SequencingRunRepeat in the table where you select input sequences before running a workflow).

Additionally, you can find a list of all the sample (SAMEA) and run (ERR) accession numbers for the ARMS-MBON data in ENA in [ena_accession_numbers.xlsx](https://github.com/arms-mbon/data_workspace/blob/main/qualitycontrolled_data/combined/ena_accession_numbers.xlsx). All projects are linked under the ARMS-MBON project [PRJEB72316](https://www.ebi.ac.uk/ena/browser/view/prjeb72316) and there are different projects (different study accession numbers) for each country. The technical replicates (run 1 or run 2) are also indicated in this file.

Here you can find details about the key outputs of this workflow:

- **PEMA**: processing of raw DNA sequences
- **WORMS and WRIMS**: screening of invasive species
- **RvLab**: downstream ecological analyses

## **PEMA**

*Dive deeper into PEMA via their [Github page](https://github.com/hariszaf/pema).*

PEMA is the bioinformatic pipeline that ensure the processing of raw DNA sequences from selected ARMS samples. It runs under the workflow step call PEMARunner, and its current implemented version is 2.1.4.

Before using it, you need to set a few parameters to decide how the pipeline is going to process your samples. This is a **key** step of the workflow, to read more about it go to [this repository](https://github.com/arms-mbon/documentation/tree/main/arms_in_tesseract/PEMA_parameters).

On Tesseract, all PEMA-related outputs (i.e., intermediate files, final output, checkpoint files, and per-analysis parameters) are grouped in distinct subfolders per major steps. In the last subfolder, i.e., subfolder 8, the results are further split into folders per sample.

Here is what PEMA does:

### *1.Quality control and preprocessing of the raw sequences (.fastq files).*

[FastQC](https://github.com/s-andrews/FastQC) is performed to obtain an overall read quality summary (output file “1.qualityControl”)

*For ITS:* [Cutadapt](https://cutadapt.readthedocs.io/en/stable/) is used to remove the primers.

[Trimmomatic](http://www.usadellab.org/cms/?page=trimmomatic) trims the low quality in the last positions of the reads (“2.trimmingSequences”)

[Bayes-Hammer](https://cab.spbu.ru/software/spades/) finds and corrects errors in the reads (“3.adjustingSequences”)

[PANDAseq]( https://github.com/neufeld/pandaseq) merges forward and reverse reads (“4.mergingPairedEndFiles”)

[OBItools](https://pythonhosted.org/OBITools/welcome.html) groups the identical sequences and keep their abundances (“5.dereplicateSamples”) 

The [VSEARCH package](https://github.com/torognes/vsearch/releases/tag/v2.9.1) is then invoked for chimaera removal.
Only if the [SWARM algorithm](https://github.com/torognes/swarm) is selected in the parameters, chimaera removal will be performed after the ASV inference (see next section). 

### *2. ASVs inference/OTUs clustering.*

PEMA supports both OTU clustering (via the VSEARCH algorithm) and ASV inference (via SWARM) for all four marker genes. Make your method choice clear in the parameters file. 

Then, when [Swarm](https://github.com/torognes/swarm) is selected, the chimaeras are removed. 

The outputs of this step can be found in the file “7.mainOutput” in the zip file containing all PEMA outputs.

### *3.Taxonomic assignment.*

Alignment-based taxonomy assignment is then performed, with different algorithms and databases for the different marker genes.
[CREST classifier](https://github.com/lanzen/CREST) (for ITS and 18S) or [RDP Classifier](https://github.com/rdpstaff/classifier) (for COI - associated with Midori database) look for any short, matching sequence, or local alignments, with the reference databases. In the case of the 16S/18S rRNA marker genes, it should be specified if the taxonomy assignment method is alignment-based or phylogenetic-based (the latter one taking more time).

The output of this step can be found in “final_table.tsv” as well as in the files 7 and 8 from the zip file containing all PEMA outputs.

Important Information for PEMA v.2.1.4 Users: In this version of PEMA, for COI gene sequences, the taxonomic classification in these final_table and Extended_finalTable stops at the genus level. The species-level classification is not included in the Extended Final Tables. To obtain species-level classification for COI gene sequences, users should refer to the "tax_assignments" files. These files include detailed classifications beyond the genus level for each ASV provided in the Extended Final Tables.

### *4. In depth biodiversity analysis*

The last step of PEMA is biodiversity analysis using [phyloseq](http://joey711.github.io/phyloseq/index.html), an R package able to perform downstream ecological analysis on the taxonomically assigned OTUs or ASVs (α- and β-diversity analysis, taxonomic composition, statistical comparisons, and calculation of correlations between samples).

! HOWEVER this is not supported by the current version of the workflow. Therefore, the phyloseq parameter in the parameters file should be “**NO**”.


## **WORMS and WRIMS check**

Once the taxonomic classification has been performed, we have a list of ASVs/OTUs identified to the species level. 

The WORMS step of the workflow uses the WoRMS taxon match webservice to get the AphiaID from the [World Register of Marine Species](https://www.marinespecies.org/) for each identified species. AphiaIDs are Life Science Identifiers, which are unique and persistent IDs for each available name in the Aphia database which makes up the information in WoRMS. This step returns the WoRMS match (“yes” or “no”) and, if there’s a match, the scientific name and the aphiaID.

The [World Register of Introduced Marine Species](https://www.marinespecies.org/introduced/) records which marine species in WoRMS have been introduced by human activities to geographic areas outside their native range. 
Using the output from the previous step and the geographical location for the samples that those species were found at, the WRIMS taxon match webservice can check the known distribution of the species based on the AphiaID.
In the output of this step, each row is not only a unique species, but a unique species from a unique sampling event (i.e. from a unique input raw sequence taken from ENA).

## **RvLab**

[RvLab VRE](https://metadatacatalogue.lifewatch.eu/geonetwork/eng/catalog.search#/metadata/fdefdc26-14fe-4095-ba5f-e55903bc4008) is an R online environment created by LifeWatch ERIC to perform a variety of analyses, such as calculation of several biodiversity indices and the running of multivariate analyses. 

In the current version of the workflow, a multidimensional scaling is performed on the chosen samples (at least 3 samples needed) with the function Metamds. A plot is built, and the obtained values can be found in the log file.

NB.There must be enough data for the results to be considered. If data is insufficient, the stress value is likely to be nearly zero, and the results cannot be interpreted in that case.
