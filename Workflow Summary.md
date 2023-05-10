# Discovering Novel Viruses

## Introduction

The global virome is enormous and only a fraction of it has been characterized and annotated. This guide will outline steps to characterize and annotate an unknown virus isolate.

#### Prerequisites
- Viral RdRP Sequence or specifically the [palmprint](https://peerj.com/articles/14055/) of the virus

The palmprint is a sub-sequence of a virus' RdRP Protein and this domain contains 3 sites: motif A, B and C. 

#### Initial Checks
Run your palmprint through Serratus' [palmID search](https://serratus.io/palmid)

This will align your palmprint against the palmprints of millions of viruses in palmDB

Secondly, check your virus sequence against the "known" viruses through a program called [NCBI BLAST](https://blast.ncbi.nlm.nih.gov/Blast.cgi). Run BLASTp or Protein BLAST on your protein sequence and check the significant alignments. 
A run through BLAST can look like: 

![image](https://user-images.githubusercontent.com/72664748/236889680-e2f12bd2-84cf-499b-9f89-cb549cf11afe.png)

In this case, the top hit is the VP1 protein from a Changuinola virus. This protein is a part of an annotated genome and that genome is present in refseq. So this virus is not really "unknown" and your isolate is just another instance of this virus. 

However, if there are no exact matches, but some distantly similar sequences, then you should continue on in your search.
Head to the SRA Table in serratus to see if your virus isolate is present in the SRA database. For example: 

![image](https://user-images.githubusercontent.com/72664748/236889847-41325bf6-2f71-4b70-b318-55a13097f7cf.png)

In this case, your virus' original source is this SRA hit that can be seen here. You can search up the SRR identifier in the [NCBI Database](https://www.ncbi.nlm.nih.gov/) to find out more information on this specific sample. For example your search can look like this:

![image](https://user-images.githubusercontent.com/72664748/236890023-d208a793-34c8-41a3-ab21-fd2604179191.png)

#### So your virus is really unknown

If your virus is not found in the SRA database or in palmDB, then your virus is truly unknown and hasn't been uplaoded to any database before. So, what you should do is gather the reads in .fastq format that you must have gotten from the seqeuncer and head to the next step: Assembly

## Assemble the Reads to Create a Library for the Virus

Current sequencing technologies cannot read the whole genome at once. It reads small pieces of mean length between 50–300 bases (next-generation sequencing/short reads). These short pieces are called **reads**. 

Once we have small pieces of the genome, we have to combine (assemble) them together based on their overlapping information and build the complete genome. This process is called **assembly**.

Special software tools called **assemblers** are used to assemble these reads according to how they overlap, in order to generate continuous strings called **contigs**. These contigs can be the whole genome itself, or parts of the genome

![image](https://user-images.githubusercontent.com/72664748/236890077-e12adf60-7ea4-46ca-a3e5-d8cb36123053.png)


### De novo vs Reference Based Assembly

De Novo sequence assemblers assemble genomes without the use of a reference genome. Basically, if the genome is new and the virus sequence is unique, De Novo will most likely be required. 

A good De Novo assembler is rnaSPAdes. 

The other type if Reference Based Assembly which assemble by mapping sequences to reference genomes. 


Here is the google colab I created for running rnaSPAdes on paired-end reads from an Illumina sequencer.

[Step 1: Assembling the Library for your Virus](https://colab.research.google.com/drive/1NZbx3vpFlDFZPJPhTcmxORp15rRNlxBE)

### Finding the Assembled Genome in the Assembled Library

Then align the entire library against the initial sequence that you got to find the largest possible sequence that matches. This could potentially be the entire genome of the unknown virus. 

You should probably use [diamond2](https://github.com/bbuchfink/diamond) for aligning your original sequence to the assembled transcripts that were outputted by the assembler. This will give you possibly a larger sequence of the genome or it might just give you your whole genome. 

You then need to make sure that this is the entire genome by finding the 3' and 5' signatures on both ends.
If on ORF finder you get full open reading frames which don't get interrupted by the end of the contig, then you can be pretty sure that it is the full genome.

# Genome Annotation

### Find the domains in the the virus

Search InterPro with your ORFs to find out the potential proteins that are being created. Also, BLAST each ORF to confirm the type of protein being created. 

### Create a Genome Map for your Virus

Using R packages such as gggenes or ggenomes, create a genome map for the virus and overlay the interpro domains over that map to see exactly which parts of the ORF hit against the InterPro database. 

This notebook is written in R and would require an R kernel to run. 
I installed R on my personal machine and ran this notebook with a Jupyter Notebook.

Here is the notebook that I used to create this map: [Gene Map of Virus](https://colab.research.google.com/drive/1b-i40r0qWz5dmEQjR0qu93AzhPQyw4nN)

An example of a genome map is this:
![image](https://user-images.githubusercontent.com/72664748/236890198-4646ea9b-31b4-45df-9223-e1bc7cf0bcd0.png)

These maps are benefitial to create as they are great for reference and for quickly seeing which ORFs are present in your virus along with which parts of the ORF are well conserved and are predicted by InterPro.

### Throw the ORFs into AlphaFold

Run each ORF in the [AlphaFold colab](https://colab.research.google.com/github/sokrypton/ColabFold/blob/main/AlphaFold2.ipynb#scrollTo=kOblAo-xetgx) and you will get the structural prediction of the protein. 

For example this is the structural prediction for an RdRP protein. 
<img width="815" alt="ORF 19 Rdrp Alpha Fold" src="https://user-images.githubusercontent.com/72664748/236890265-dcbd4bbb-7de6-4f88-87b9-fbd8ec33d4cd.png">

### Analyse the UTRs of the genome 

Run the UTRs of the genome through RFAM and look for any structural RNA that pop up. You can check in "related" sequences and see if it's there too. You can throw the UTRs through mfold to see if the related seqeunces in RFAM have overall conserved structures. If they do, you can be pretty condifdent that they will have similar function, and have evovled together.

### Map the raw reads to the assembled transcripts


There are many aligners to use. BWA is an example. The resulting SAM file can then be converted to a smaller binary BAM file using samtools. Then you need to index the BAM file and get a .bai file. Throw the BAM file into IGV to view the read alignemnts. Find your reference genome that you used to align the reads in IGV

Colabs for this process:

Step 2: [Aligning the Raw Reads (.fastq file) to the Assembled Transcripts](https://colab.research.google.com/drive/11mefOUcLpIBAhg1CrBenrXt5oVN_zNuH)

Step 3: [Converting the SAM File to a BAM file and then Indexing the BAM File](https://colab.research.google.com/drive/1zwB4g57EhOXpTO67j7upU-1gUiASqLQ6)

Some of the reasons why people map raw reads to assembled transcripts is:
1. Measure the level of an RNA molecule (gene expression)
2. Measure if there exist variations in RNA sequences (variant calling)
3. Measure if there exists the presence/absence of a particular species of RNA (pathogen detection)
4. Measure the level of evidence supporting the existence of an assembled contig

We are doing mapping, for number 4. 

In IGV inspect the ends of the molecule to find the 5' and the 3' signature. The 5' signature would be all positive sense reads. 
This is an example of the genome in IGV mapped to the raw reads. The ends taper off and create a nice poisson distribution. Here is what it could look like:

![image](https://user-images.githubusercontent.com/72664748/236890514-40fac810-59ba-41c4-92a3-282215043721.png)

This is the 5' end of the genome and the colours represent the type of strand. Having all positive strands near the 5' end is a telltale sign for an end of molecule.

![image](https://user-images.githubusercontent.com/72664748/236890601-0f03de47-95b3-41a5-8201-f3f8f867fb35.png)


## Phylogenetics

Start by Running the RdRP ORF of the virus through NCBI's BLAST

![image](https://user-images.githubusercontent.com/72664748/236890999-f128ae5d-de6e-4611-bbce-31dc735b4cc7.png)

Get the full genomes of each of these viruses. Start by going to its SRA source and then going to its dbsource

![image](https://user-images.githubusercontent.com/72664748/236891059-666f551c-4baf-4ceb-8217-ea37193a607b.png)

Then extract the ORF of every full genome you find and place them into different fasta files. 

So you should have different files for each protein (in fasta format) from each virus that was there in the RdRP BLAST. For example one file for every RdRP protein, one file for each Protein X and so on and so forth.

### Run MSA on each of those protein files

Run that file through a sequence aligner, like MUSCLE. 

[Colab for running MUSCLE](https://colab.research.google.com/drive/132HF_TOOorRY1LW7tsdxPkwqBP4H_wSL#scrollTo=1-hgKww-ZbiT)

### Create phylogenies for each of those alignments

Run those aligned files through IQTree2 to get the phylogenies for each gene. 

[Colab for running IQTree](https://colab.research.google.com/drive/1bTnBE6j__bQVUjlGIqIb05WVVnwgVlDD)

## View the Phylogenies in Dendroscope and compare them

![image](https://user-images.githubusercontent.com/72664748/236891187-44a4f675-eb83-45b4-a2ef-1283e307428e.png)

For example this is comparison of the trees for the RdRP proteins and the Protease Proteins. The virus that I want to characterize and annotate is nicknamed soledSmilo.
