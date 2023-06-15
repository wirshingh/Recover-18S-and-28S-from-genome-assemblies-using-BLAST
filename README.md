## Script for extracting specific loci (18S and 28S) from genomic assemblies 
Copy the entire script below into a text editor (BBedit) and save as "extractnucribo.sh"

```
#extractnucribo.sh
#!/bin/sh

#DESCRIPTION
#This script will extract nuclear ribosomal 18S and 28S (or whatever loci are provided
#as a reference) from assemlies of genome data.

#~~~~~~~~~~~~~~~~~~~~~~~~~~~~

#READ THIS BEFORE YOU USE
#This script is intended to run on local computers.
#For it to work, the following things are needed.

#First, the filename of the fasta file with reference sequences MUST end 
#with ".ribodb.fasta". This file contains the 18S and 28S reference sequences, 
#usually downloaded from GenBank.

#Second, the assembly files can end with "_L002_contigs.fasta", the output of Spades.
#If your assemblies have a different file name ending, modify the filename in the 
#script below in Step 2.
#For example, change the "*_L002_contigs.fasta" in line 57 to match your assembly 
#file names. It MUST be a wildcard unique to only and all the assembly files, and
#have the sample name in front of the wildcard.

#Third, the programs blast (ncbi-blast-2.13.0+.dmg) 
#found at https://ftp.ncbi.nlm.nih.gov/blast/executables/blast+/LATEST/
#and seqtk (https://github.com/lh3/seqtk) must be installed on the computer used.
#NOTE: IF SEQTK IS NOT INSTALLED TO BIN, USE THE FULL PATH THE SEQTK EXECUTABLE 
#In Line 81, change the path to the location of seqtk.

#~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

#TO RUN SCRIPT 
#This script must be in the same directory as the reference fasta files (*.ribodb.fasta)
#and the assemblies (some form of *_L002_contigs.fasta). 
#Type "sh extractnucribo.sh"
#The extracted 18S and 28S sequences will be in a folder called "nuc_ribo_outputs" 
#Depending on the amount of assemblies run, it will take a few to several minutes
#to complete.

#~~~~~~~~~~~~~~~~~~~~~~~~~~~~


#Step 1 - Make a custum local blast database with makeblastdb
#This step creates a blast database from the reference fasta files provided.

for filename in *.ribodb.fasta
do
basename=$(basename ${filename} )
makeblastdb -in ${filename} -dbtype nucl -out blastdbout
done


#Step 2 - blast your contigs to the local database using blastn
#This step takes the provided assemblies and blasts them to the database created in step 1

for filename in *_L002_contigs.fasta
do
basename=$(basename ${filename} _L002_contigs.fasta)
blastn -db blastdbout -query ${filename}  -outfmt 6 -evalue 1e-5 -out ${basename}.txt 
done


#Step 3 - Create a list of all unique hits
#This step uses the output from step 2, and creates a list of the unique sequence hits.

for filename in *.txt
do
basename=$(basename ${filename} .txt)
cut -f1 ${filename} | sort | uniq > ${basename}.lst
done


#Step 4 - Pull the unique blast hit sequences from the contig files using seqtk
#This step uses the information in the list of unique hits from step 3, and pulls out
#the sequences from the contig files. These are the 18S and 28S sequences.

for filename in *_L002_contigs.fasta
do
basename=$(basename ${filename} _L002_contigs.fasta)
#[USE FULL PATH TO SEQTK IF NOT IN BIN] 
seqtk subseq ${basename}_L002_contigs.fasta ${basename}.lst > ${basename}.unique.fasta
done


#Step 5 - Rename extracted unique sequences with sample names and delete unneeded files
#This step renames the extracted 18S and 28S sequences with sample names of the assembly
#file and deletes the unneeded intermediate files.

for filename in *.unique.fasta
do
basename=$(basename ${filename} .unique.fasta)
sed "s+>+>${basename}_+g" "$filename" >> ${basename}_nucRibos.fasta
done

rm blastdbout* *.lst *.txt *.unique.fasta;

mkdir nuc_ribo_outputs;

mv *nucRibos.fasta nuc_ribo_outputs;
```
