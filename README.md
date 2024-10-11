## Run Mitofinder on Hydra using trimmed reads in a loop
### Job Summary

This job will run mitofinder in a loop on Hydra using trimmed reads.
User will need to provide paths to trimmed reads and a reference data set (see 'To Run this Job' below).
Results will be in 3 directories where the job file is run:
 1. "mitofinder_trimmedreads_All_Results" will contain all results for each sample
 2. "mitofinder_trimmedreads_Final_Results" will contain only the final genes assembled,
 relevant annotation files for GenBank/Geneious, and log files.
 3. "mitofinder_trimmedreads_Final_Genes" will contain only the final genes assembled in 
 fasta format.

```
#!/bin/sh
# ----------------Parameters---------------------- #
#$ -S /bin/sh
#$ -pe mthread 4
#$ -q mThM.q
#$ -l mres=48G,h_data=12G,h_vmem=12G,himem
#$ -cwd
#$ -j y
#$ -N mf_trimmedreads
#$ -o mf_trimmedreads.log
#
# ----------------Modules------------------------- #
module load bioinformatics/mitofinder
# ----------------Your Commands------------------- #
#
echo + `date` job $JOB_NAME started in $QUEUE with jobID=$JOB_ID on $HOSTNAME
echo + NSLOTS = $NSLOTS
#
#!/bin/bash

# Create the results directory
mkdir -p mitofinder_trimmedreads_All_Results

# Define the directory containing the trimmed reads
SAMPLEDIR="path to trimmed reads"  

# Loop through the R1 files
for GETSAMPLENAME in ${SAMPLEDIR}/*_R1_PE_trimmed.fastq.gz; do
    SAMPLENAME=$(basename "$GETSAMPLENAME" _R1_PE_trimmed.fastq.gz)
    
    # Create a directory for the current sample
    mkdir -p ./mitofinder_trimmedreads_All_Results/${SAMPLENAME}
    
    # Change to the sample's directory
    cd mitofinder_trimmedreads_All_Results/${SAMPLENAME} || exit 1  # Ensure exit if cd fails

    # Run mitofinder with the specified options
    mitofinder \
        -j "${SAMPLENAME}_mitofinder_trimmedreads_results" \
        -o 5 \
        -r "path to reference data set.gb" \
        -1 "${SAMPLEDIR}/${SAMPLENAME}_R1_PE_trimmed.fastq.gz" \
        -2 "${SAMPLEDIR}/${SAMPLENAME}_R2_PE_trimmed.fastq.gz" \
        --new-genes

    # Return to the original directory
    cd ../.. || exit 1  # Ensure exit if cd fails
done
#
#
#These are extra steps that will copy the most important files from the results directories and group them together
#
mkdir -p mitofinder_trimmedreads_Final_Results
for GETSAMPLENAME in ${SAMPLEDIR}/*_R1_PE_trimmed.fastq.gz
do
SAMPLENAME=$(basename "$GETSAMPLENAME" _R1_PE_trimmed.fastq.gz)
cp -r ./*trimmedreads_All_Results/${SAMPLENAME}/*_results/*_Final_Results ./mitofinder_trimmedreads_Final_Results
cp    ./*trimmedreads_All_Results/${SAMPLENAME}/*.log ./mitofinder_trimmedreads_Final_Results
done
#
mkdir mitofinder_trimmedreads_Final_Genes
for FINAL_GENES in ./*trimmedreads_Final_Results
do
cp "$FINAL_GENES"/*Final_Results/*final_genes_NT.fasta ./mitofinder_trimmedreads_Final_Genes
done
#
echo = `date` job $JOB_NAME done
```


### To Run this Job
The trimmed reads files need to end in '_R1_PE_trimmed.fastq.gz' (forward) and '_R1_PE_trimmed.fastq.gz' (reverse) for the job to work. Alternatively, the job file can be edited to match the trimmed reads file names accordingly.

These items need to be added in the script:

SAMPLEDIR="path to trimmed reads"

After the '=' paste the path to the trimmed reads.

For flag -o write the digit for the genetic code (see GENETIC CODES below)

For flag -r include path to refrence data set in GenBank format (.gb). Or, premade reference data sets can be used. See "REFERENCE DATABASES" below.

GENETIC CODES
 1. The Standard Code 
 2. The Vertebrate Mitochondrial Code 
 3. The Yeast Mitochondrial Code 
 4. The Mold, Protozoan, and Coelenterate Mitochondrial Code and the
     Mycoplasma/Spiroplasma Code
 5. The Invertebrate Mitochondrial Code
 6. The Ciliate, Dasycladacean and Hexamita Nuclear Code 
 9. The Echinoderm and Flatworm Mitochondrial Code 
 10. The Euplotid Nuclear Code 
 11. The Bacterial, Archaeal and Plant Plastid Code 
 12. The Alternative Yeast Nuclear Code 
 13. The Ascidian Mitochondrial Code 
 14. The Alternative Flatworm Mitochondrial Code 
 16. Chlorophycean Mitochondrial Code 
 21. Trematode Mitochondrial Code 
 22. Scenedesmus obliquus Mitochondrial Code 
 23. Thraustochytrium Mitochondrial Code 
 24. Pterobranchia Mitochondrial Code 
 25. Candidate Division SR1 and Gracilibacteria Code


FLAGS

 -j Sequence ID to be used throughout the process

 -o is the genetic code to use. See 'Genetic Codes' list above.
 
 -1 is the path to the R1 PE trimmed file
 
 -2 is the path to the R2 PE trimmed file
 
 --new-genes denotes that some of the genes in the reference database are not
 one of the "official" genes as determined by mitofinder.

 REFERENCE DATABASES
 
 -r is the path to the reference database in genbank (.gb) format. Custom reference databases can be used. Alternatively, premade references can be found in /scratch/nmnh_lab/macdonaldk/ref/mito_reference_(ref)_*.gb:
 Replace (ref) with one of the taxon groups below
 "Annelida", "Arthropoda", "Bryozoa", "Cnidaria", "Ctenophora", "Echinodermata", 
 "Mollusca", "Nemertea", "Porifera", "Tunicata", "Vertebrata" or , "Metazoa"

