---
title: "Hands-On: Loading Our Input Dataset Onto Galaxy"
exercises: 60
objectives:
- "Locate a sequencing dataset from NCBI based on info from a publication"
- "Download sequencing metadata in bulk from NCBI"
- "Load data from the NCBI SRA into the environment"
keypoints:
- "Data can be found on NCBI using the a variety of identifiers for Bioproject, Biosample and SRA"
- "Galaxy is an open source tool for conducting bioinformatic analyses"
- "FASTQ files generally store raw sequencing data"
---

> ## Prerequisites
> This tutorial assumes that you have done the following: 
> * Signed up for a Galaxy account and can log in.
> * Have taken a look at the Galaxy Environment 101 Tutorial and are familiar with the concepts therein. 
> * Have familiarized yourself with the steps of variant-calling and related file-formats (you can always do this as you progress through the tutorial as needed!). 
{: .prereq}



## Obtaining data from the NCBI Short Read Archive

First we need to identify some samples of interest. The Sequence Read Archive (SRA) is the primary archive of unassembled reads operated by the US National Institutes of Health (NIH). SRA is a great place to get the sequencing data that underlie publications and studies. We know where to find the data because Science (and in fact, most publications) require authors to deposit their data in a public database and give us the information we need to find it on the database. 

The data availability statement can be found in the acknowledgements section of Lemieux et al. 2020: 

<a href="{{ page.root }}/fig/Lemieux_Data_Statement.png">
  <img src="{{ page.root }}/fig/Lemieux_Data_Statement.png" alt="Data availability statement from Lemieux et al. 2020" />
</a>

As you can see, we now know we can find sequencing data related to this study under `BioProject PRJNA622837`. As defined by the NCBI, a BioProject is "a collection of biological data related to a single initiative, originating from a single organization or from a consortium", so they generally contain multiple sequencing data sets. The NCBI has much a much more [in-depth description of BioProjects](https://www.ncbi.nlm.nih.gov/books/NBK169438/) on its webpage, but their general structure can be summarized below: 

<img src="{{ page.root }}/fig/Bioproject_Structure.png" alt="Schematic of an NCBI BioProject">

> ## "Hands-On: Get Metadata from NCBI SRA"
> 1. Go to NCBI’s SRA page by pointing your browser to `https://www.ncbi.nlm.nih.gov/sra`
> 2. Perform a search using the Bioproject ID from the paper: `BioProject PRJNA622837`
> <img src="{{ page.root }}/fig/SRA_Bioproject_Search.png" alt="SRA search toolbar filled in with Bioproject ID from above">
> 3. The web page will show a large number of available SRA datasets related to project described in Lemieux et al. 2020 (at the time of writing there were 22,173). 
> 4. Download metadata describing these datasets by:
>   * clicking on **Send to**: dropdown
>   * Selecting `File`
>   * Changing **Format** to `RunInfo`
>   * Clicking **Create file**. The screen should now look like this: 
> <img src="{{ page.root }}/fig/SRA_Download_RunInfo_Menu.png" alt="SRA Download RunInfo Menu Options">
> 5. Click "Create File" to download the `SraRunInfo.csv` file to a place on your computer where you will be able to find it easily for the next step.
{: .challenge}


## Getting Information About Individual Samples

So, what is in the the SRA Run Info file?

The file we just downloaded is **not** sequencing data itself. Rather, it is **metadata** describing the properties of sequencing reads. We could actually open this file in a program like Excel, and if we did so, we could see columns of information like **Release Date**, **Sequencing Platform**, and **Sample Name** as reported by the researchers. In this study every accession corresponds to an individual patient whose samples were sequenced. We will filter this list down to just a few sequenced samples that will be used in the remainder of this tutorial. 

This is all very valuable information - for example, if we want to know more about the sample that became the SRA sequencing dataset `SRS8612691` we can use the **Sample Name** column to retrieve information about when the sample was actually collected. You can search for a **Sample Name** in the general NCBI database by using the **All Databases** option in the toolbar: 

<img src="{{ page.root }}/fig/NCBI_Sample_SearchBar.png" alt="Searching NCBI By Sample Name">

We can see that the sample that was sequenced to become the SRA sequencing dataset `SRS8612691` was collected on January 4th, 2021 and was collected as part of routine surveillance: 

<img src="{{ page.root }}/fig/NCBI_SampleSearch_Results.png" alt="Results of Searching NCBI By A Sample Name">


## Processing and Filtering Metadata on Galaxy

> ## Hands-On: Upload `SRARunInfo.csv` onto Galaxy
> 1. If you haven't already, log in to your Galaxy account. Create a new, empty history called something like "Variant Calling Tutorial".
> 2. In the tools panel, click the **Upload Data** button: 
>
> <img src="{{ page.root }}/fig/Galaxy_upload_button.png" alt="Galaxy upload button">
> 
> 3. Find `SRARunInfo.csv` on your computer, and drag it into the "Drop Files Here" space in the dialog box that pops up.
> 4. Click the **Start** button to begin the upload. 
> 5. At this point, you can press the **Close** button. In a several seconds, `SRARunInfo.csv` should be ready to look at in the History panel. 
> 6. You can now look at the content of this file by clicking on the eye icon <span class="glyphicon glyphicon-eye-open"></span>. You will see that this file contains a lot of information about individual SRA accessions.
> 
> How large is the sequencing file associated with the accession number `SRR14114810`? 
> 
> > ## Solution
> >
> > 272 MB. If you open the file by clicking the eye icon, you should be able to find `SRR14114810` in the **Run** column, and then look over to see 272 in the **size_MB** column. 
> >
> {: .solution}
> **Note:** If you ever want a smaller summary of the data, you can also click the name of of item in your history, which should produce something like this: 
> 
> <img src="{{ page.root }}/fig/Galaxy_Pencil_Summary.png" alt="Data summary produced by clicking the name of a data set in your in Galaxy History" height="300">
> 
{: .challenge}



The Galaxy servers MIGHT powerful enough to process all 22,000+ datasets, but to make this tutorial bearable we need to selected a smaller, but still interesting, subset. In particular, we are interested in samples from early in the ongoing Covid-19 pandemic, so we will be choosing samples collected in April and May of 2020, which I have chosen using the "Collection Date" information retrieved by as described above. 

**We are going to focus on the following two sequencing data sets:** 
* Run Number `SRR12733957`: A sequencing run from a sample collected on April 6th, 2020.
* Run Number `SRR11954102`: A sequencing run from a sample collected on May 2nd, 2020.

> ## Warning: Don't get caught by the wrong "Cut"! 
> <span class="glyphicon glyphicon-warning-sign"></span> WARNING: There are two cut tools in Galaxy due to historical reasons. For this tutorial we are assuming you are using the named EXACTLY <button type="button" class="btn btn-outline-tool" style="pointer-events: none"> Cut columns from a table (cut) </button>  The other tool follows a similar logic but with a different interface. <span class="glyphicon glyphicon-warning-sign"></span> 
> 
{: .callout}

> ## Hands-On: Creating a subset of data
>
> 1. Find the <button type="button" class="btn btn-outline-tool" style="pointer-events: none"> Select lines that match an expression  </button> tool in the **Filter and Sort** section of the tool panel. You may find that Galaxy has an overwhelming amount of tools installed. To find a specific tool type the tool name in the tool panel search box to find the tool.
> 
> 2. Make sure the SraRunInfo.csv dataset we just uploaded is listed in the <span class="glyphicon glyphicon-file"></span> **Select lines from** field of the tool form.
>
> 3. In the **Pattern** field enter the following expression: `SRR12733957|SRR11954102`. The "&#124;" symbol (called a "pipe") means "or". So we are telling this tool to find lines containing `SRR12733957` OR `SRR11954102`.
> 4. Click the `Execute` button.
> 5. Once the this has been executed, you should have a file with a total of three lines. If you look at the file (<span class="glyphicon glyphicon-eye-open"></span>), you will see that one of the two lines has been duplicated to take the place of the "header" line (which is fine for now). 
> 6. Cut the first column from the file using the <button type="button" class="btn btn-outline-tool" style="pointer-events: none"> Cut columns from a table (cut) </button>  tool, which you will find in **Text Manipulation** section of the tool pane. 
> 7. Make sure the dataset produced by the previous step is selected in the **File to cut** field of the tool form.
> 8. Change **Delimited by** to `Comma`.
> 9. In **List of fields** drop-down menu select `Column: 1`. This is telling the tool to only return the first column of our data.
> 10. Hit `Execute`.  This will produce a text file with just two lines:
> ~~~
> SRR12733957
> SRR11954102
> ~~~
> {: .output}
{: .challenge}

## Downloading the actual sequence data

So, now we have a file that contains just the two accession numbers for the sequencing data sets that we want. This is the perfect input for tools in Galaxy that can use this information to download big sequencing data sets straight from the NCBI SRA directly into your Galaxy environment without ever putting the big files on your computer. 

> ## Hands-On: Getting data from SRA
> 1. <button type="button" class="btn btn-outline-tool" style="pointer-events: none"> Faster Download and Extract Reads in FASTQ </button> with the following parameters: 
> - **Select Input Type**: `List of SRA Accession, one per line`. 
> - The input parameter <span class="glyphicon glyphicon-file"></span> **select input type** should point the output of <button type="button" class="btn btn-outline-tool" style="pointer-events: none"> Cut columns from a table (cut) </button>. 
> 2. Click the `Execute` button. This will run the tool, which retrieves the sequence read read datasets and places them into your Galaxy environment. <span class="glyphicon glyphicon-time"></span> Note that this step can take a few minutes, so this might be a good time get get a fresh cup of coffee!
> 3. <span class="glyphicon glyphicon-eye-open"></span> Take a look at the entries that were created in your history panel: 
> - `Pair-end data (fasterq-dump)`: Contains Paired-end datasets (if available)
> - `Single-end data (fasterq-dump)`:  Contains Single-end datasets (if available)
> - `Other data (fasterq-dump)`:  Contains Unpaired datasets (if available)
> - `fasterq-dump log`: Contains Information about the tool execution
>
> Is the data we downloaded **single-end** or **paired-end** data? 
> > ## Solution
> > Our data is **paired-end**! If you click each of the data files generated that are now in your history, you will see that `Single-end data (fasterq-dump)` is an empty list, meaning that the tool did not download any files of that type. On the other hand, `Pair-end data (fasterq-dump)` is a "list of pairs with 2 items", each of which is a FASTQ-formatted file if you look inside: 
> > <img src="{{ page.root }}/fig/List_of_Pairs.png" alt="Preview of paired-end data downloaded by fasterq-dump tool">
> > 
> {: .solution}
{: .challenge}

`Pair-end data (fasterq-dump)`, `Single-end data (fasterq-dump)` and `Other data (fasterq-dump)` are actually collections of datasets. Collections in Galaxy are logical groupings of datasets that reflect the semantic relationships between them in the experiment / analysis. In this case the tool creates separate collections for paired-end reads, single reads, and other (any other type of file). See the [Galaxy Collections tutorial](https://training.galaxyproject.org/training-material/topics/galaxy-interface/tutorials/collections/tutorial.html) and watch [Galaxy tutorial videos](https://www.youtube.com/playlist?list=PLNFLKDpdM3B9UaxWEXgziHXO3k-003FzE) (with names beginning with “Dataset Collections”) for more information.

Explore the collections by first clicking on the collection name in the history panel. This takes you inside the collection and shows you the datasets in it. You can then navigate back to the outer level of your history.

Once `fasterq` finishes transferring data (all boxes are green / done), we are ready to analyze it.

> ## SRA Upload Workaround: Getting data from a Galaxy History
> **When to use this tool**: If you have waited for more than 12 hours for these FASTQ files to successfully load into your Galaxy history using the <button type="button" class="btn btn-outline-tool" style="pointer-events: none"> Faster Download and Extract Reads in FASTQ </button> tool , you can transfer these files from an existing history I set up for this week. 
> 
> 1. Navigate to `https://usegalaxy.org/u/faes.biof521/h/biof521sp22week2tutorialfiles` (the URL will depend on the exact history you are trying to import). 
> 2. In the top right-hand corner, click the big **"+"** button for the option to import this history into your Galaxy account. 
> <img src="{{ page.root }}/fig/History_Import_TopCorner.png" alt="History Import Button in Top Corner" width="400" height="200">
> 3. Once that is done (it should just take a few moments), Galaxy will automatically take you back to your home screen, with the imported history now active. 
> 4. Now you can go to the `View All Histories` option, and you should see this history side by side with your older histories. 
> <img src="{{ page.root }}/fig/ViewAllHistories.png" alt="The View All Histories Option on Galaxy"  width="400">
> 5. Drag and drop only the files that you need (in this case, probably just the `paired-end` collection into the history you set up for this tutorial. 
> 6. Once you are done with that, make sure to click the **Switch To** tab above your Week 2 Tutorial history to make that one active before pressing the **Home** icon on the top bar to return to your usual Galaxy view.
{: .solution}


## What does our paired-end data actually look like?  

It is common to prepare pair-end and mate-pair sequencing libraries. This is highly beneficial for a number of applications discussed, such as those discussed in Week 2's **Genome Assembly** module. For now let’s just briefly discuss what these are and how they manifest themselves in FASTQ form.

<img src="{{ page.root }}/fig/pairedend_matepair.png" alt="Diagram of paired-end and mate-pair sequencing data">

In **paired-end** sequencing (left) the actual ends of rather short DNA molecules (less than 1kb) are determined, while for **mate-pair** sequencing (right) the ends of long molecules are joined and prepared in special sequencing libraries. In these mate pair protocols, the ends of long, size-selected molecules are connected with an internal adapter sequence (i.e. linker, yellow) in a circularization reaction. The circular molecule is then processed using restriction enzymes or fragmentation. Fragments are enriched for the linker and outer library adapters are added around the two combined molecule ends. The internal adapter can then be used as a second priming site for an additional sequencing reaction in the same orientation or sequencing can be performed from the second adapter, from the reverse strand. (From “Understanding and improving high-throughput sequencing data production and analysis”, Ph.D. dissertation by Martin Kircher)

**Thus in both cases (paired-end and mate-pair) a single physical piece of DNA (or RNA in the case of RNA-seq) is sequenced from two ends and so generates two reads**. These can be represented as separate files (two FASTQ files with first and second reads) or a single file were reads for each end are interleaved (discussed later). Each of our data sets is a pair of FASTQ files, with each data set having a file for "Forward" read and "Reverse" reads. 

For example, the first two reads of the `SRR11954102` data set are: 

**Forward** 
~~~
@SRR11954102.1 1 length=101
ACGGGGGGGCTTACCATCTGGCCCCAGTGCTGCAATGATACCGCGAGACCCACGCTCACCGGCTCCAGATTTATCAGCAATAAACCAGCCAGCCGGAAGGG
+SRR11954102.1 1 length=101
FFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFF
@SRR11954102.2 2 length=101
GCATTAAGCGCGGCGGGTGTGGTGGTTACGCGCAGCGTGACCGCTACACTTGCCAGCGCCCTAGCGCCCGCTCCTTTCGCTGTCTTCCCTTCCTTTCTCGC
+SRR11954102.2 2 length=101
FFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFF,FFFFFFFFFFFFFFFFFFF
~~~ 
{: .output}

**Reverse** 
~~~
@SRR11954102.1 1 length=101
AACTCGCCTTGATCGTTGGGAACCGGAGCTGAATGAAGCCATACCAAACGACGAGCGTGACACCACGATGCCTGTAGCAATGGCAACAACGTTGCGCAAAC
+SRR11954102.1 1 length=101
FFFFFFF:FFFFFFFFFFFFF:FFFFFFFFFFFFF:FFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFF:FFFFFFFFF
@SRR11954102.2 2 length=101
CCCTTTAGGGTTCCGATTTAGTGCTTTACGGGGAAAGCCGGCGAACGTGGCGAGAAAGGAAGGGAAGAAAGCGAAAGGAGCGGGCGCTAGGGCGCTGGCAA
+SRR11954102.2 2 length=101
FF:FFFFFFFFFFFFFFFFFF:FFFFFFFFF:FFFF:FFFFFFFFFFF:FFFFFFFFFFFFFFFFFF:FFFFFFFFFFFFFFFFFFFFFF:FFFFFFFFFF
~~~
{: .output}
