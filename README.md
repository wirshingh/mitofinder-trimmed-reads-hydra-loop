## Run Mitofinder on Hydra using trimmed reads in a loop
### Job Summary

This job will run mitofinder in a loop on Hydra using trimmed reads.
User will need to provide paths to trimmed reads and a reference data set (see 'To Run this Job' below).
Results will be in 4 directories where the job file is run. 

 1. "mitofinder_trimmedreads_All_Results" will contain all results for each sample
 2. "mitofinder_trimmedreads_Final_Results" will contain only the final genes assembled,
 relevant annotation files for GenBank/Geneious, and log files.
 3. "mitofinder_trimmedreads_Final_Genes" will contain only the final genes assembled in 
 fasta format.
 4. "mitofinder_final_mito_contigs" will contain all assembled mitochondrial contigs. These can be used for use in other annotation programs like MITOS.


```
#!/bin/sh
# ----------------Parameters---------------------- #
#$ -S /bin/sh
#$ -pe mthread 9
#$ -q mThM.q
#$ -l mres=72G,h_data=8G,h_vmem=8G,himem
#$ -cwd
#$ -j y
#$ -N mf_test
#$ -o mf_test.log
#
# ----------------Modules------------------------- #
module load bioinformatics/mitofinder
# ----------------Your Commands------------------- #
#
echo + `date` job $JOB_NAME started in $QUEUE with jobID=$JOB_ID on $HOSTNAME
echo + NSLOTS = $NSLOTS
#

#============================================================================
# CONFIGURATION
#============================================================================

# Define the directory containing the trimmed reads
SAMPLEDIR="full path to trimmed reads directory"  

# Define the project's base directory. This is where the results will go.
SAMPLEDIR_BASE="Full path to base directory"

#============================================================================
# PART 1 - Run Mitofinder in a loop using trimmed reads
#============================================================================

# Create the results directory
mkdir -p ${SAMPLEDIR_BASE}/mitofinder_trimmedreads_All_Results

# Loop through the R1 files
for GETSAMPLENAME in ${SAMPLEDIR}/*_R1_PE_trimmed.fastq.gz; do
    SAMPLENAME=$(basename "$GETSAMPLENAME" _R1_PE_trimmed.fastq.gz)
    
    # Create a directory for the current sample
    mkdir -p ${SAMPLEDIR_BASE}/mitofinder_trimmedreads_All_Results/${SAMPLENAME}
    
    # Change to the sample's directory
    cd ${SAMPLEDIR_BASE}/mitofinder_trimmedreads_All_Results/${SAMPLENAME} || exit 1  # Ensure exit if cd fails

    # Run mitofinder with the specified options
    mitofinder \
        -j "${SAMPLENAME}_mitofinder_trimmedreads_results" \
        -o 5 \
        -r "full path to reference file" \
        -1 "${SAMPLEDIR}/${SAMPLENAME}_R1_PE_trimmed.fastq.gz" \
        -2 "${SAMPLEDIR}/${SAMPLENAME}_R2_PE_trimmed.fastq.gz" \
        --new-genes

    # Return to the base directory
    cd ${SAMPLEDIR_BASE} || exit 1  # Ensure exit if cd fails
done
#
#=================================================================================
# PART2 - These are extra steps that will copy the most important files from the results directories and group them together
#=================================================================================

mkdir -p ${SAMPLEDIR_BASE}/mitofinder_trimmedreads_Final_Results
for GETSAMPLENAME in ${SAMPLEDIR}/*_R1_PE_trimmed.fastq.gz
do
SAMPLENAME=$(basename "$GETSAMPLENAME" _R1_PE_trimmed.fastq.gz)
cp -r ${SAMPLEDIR_BASE}/*trimmedreads_All_Results/${SAMPLENAME}/*_results/*_Final_Results ${SAMPLEDIR_BASE}/mitofinder_trimmedreads_Final_Results
cp    ${SAMPLEDIR_BASE}/*trimmedreads_All_Results/${SAMPLENAME}/*.log ${SAMPLEDIR_BASE}/mitofinder_trimmedreads_Final_Results
done
#
mkdir -p ${SAMPLEDIR_BASE}/mitofinder_trimmedreads_Final_Genes
for FINAL_GENES in ${SAMPLEDIR_BASE}/*trimmedreads_Final_Results
do
cp "$FINAL_GENES"/*Final_Results/*final_genes_NT.fasta ${SAMPLEDIR_BASE}/mitofinder_trimmedreads_Final_Genes
done
#
echo "Results copied to directory 'mitofinder_trimmedreads_Final_Results'"
echo "Final Genes copied to directory 'mitofinder_trimmedreads_Final_Genes'"

#===============================================================================
# PART3 - Copy all assembled mitocontigs into a single directory
#===============================================================================

# Make a new directory for copied final mitocontigs
mkdir -p ${SAMPLEDIR_BASE}/mitofinder_final_mito_contigs

# Loop that will get sample names from the R1 trimmed reads file and copy final mitocontigs
for GETSAMPLENAME in ${SAMPLEDIR}/*_R1_PE_trimmed.fastq.gz
do
    SAMPLENAME=$(basename "$GETSAMPLENAME" _R1_PE_trimmed.fastq.gz)
    
    # Check and copy *_contig_*.fasta files
    if ls ${SAMPLEDIR_BASE}/*trimmedreads_All_Results/${SAMPLENAME}/*_results/*_Final_Results/*_contig_*.fasta 1> /dev/null 2>&1; then
        cp ${SAMPLEDIR_BASE}/*trimmedreads_All_Results/${SAMPLENAME}/*_results/*_Final_Results/*_contig_*.fasta ${SAMPLEDIR_BASE}/mitofinder_final_mito_contigs
    fi
    
    # Check and copy *_contig.fasta files
    if ls ${SAMPLEDIR_BASE}/*trimmedreads_All_Results/${SAMPLENAME}/*_results/*_Final_Results/*_contig.fasta 1> /dev/null 2>&1; then
        cp ${SAMPLEDIR_BASE}/*trimmedreads_All_Results/${SAMPLENAME}/*_results/*_Final_Results/*_contig.fasta ${SAMPLEDIR_BASE}/mitofinder_final_mito_contigs
    fi
done

# Remove *_genes_* files
rm -f ${SAMPLEDIR_BASE}/mitofinder_final_mito_contigs/*_genes_*

echo "All mitochondrial contigs copied to directory 'mitofinder_final_mito_contigs''"
echo "DONE"
echo = `date` job $JOB_NAME done

```


### To Run this Job
The trimmed reads files need to end in '_R1_PE_trimmed.fastq.gz' (forward) and '_R1_PE_trimmed.fastq.gz' (reverse) for the job to work. Alternatively, the job file can be edited to match the trimmed reads file names accordingly.

These items need to be added in the script:

1. SAMPLEDIR="path to trimmed reads"

  After the '=' paste the full path to the trimmed reads.

2. SAMPLEDIR_BASE="Full path to base directory"

   After the '=' paste the full path to the base directory. This is where the results will go.

3. In Part 1 for the mitofinder flag -o write the digit for the genetic code (see GENETIC CODES below)

4. In Part 1 for the mitofinder flag -r include path to refrence data set in GenBank format (.gb). Or, premade reference data sets can be used. See "REFERENCE DATABASES" below.

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

 ### Extra Script
 ### List and Count All Genes Assembled for Each Sample

 Run this job file in the base directory of the Mitofinder run after running Mitofinder.

 ```
# /bin/sh
# ----------------Parameters---------------------- #
#$ -S /bin/sh
#$ -q sThC.q
#$ -l mres=7G,h_data=7G,h_vmem=7G
#$ -cwd
#$ -j y
#$ -N mitofinder_list_and_count_genes
#$ -o mitofinder_list_and_count_genes.log
#
# ----------------Modules------------------------- #
#
# ----------------Your Commands------------------- #
#
echo + `date` job $JOB_NAME started in $QUEUE with jobID=$JOB_ID on $HOSTNAME
#
# Clear or create the output file before starting the loop
output_file="mitofinder_gene_list_and_counts_for_each_sample.txt"
> "$output_file"

# Loop through all relevant FASTA files
for filename in ./mitofinder_trimmedreads_Final_Genes/*.fasta; do
    # --- ADDED: Print the current file to the terminal ---
    echo "Processing: $filename"
    
    # Extract the sample name from the filename
    samplename=$(basename "$filename" _genes_NT.fasta)
    echo "$samplename" >> "$output_file"

    # Process each line of the input file
    while IFS= read -r line; do
        # Extract the text after '@'
        after_at=$(echo "$line" | awk -F'@' '{print $2}')
        # Append the result to the output file
        if [ -n "$after_at" ]; then
            echo "$after_at" >> "$output_file"
        fi
    done < "$filename"
done

# Initialize variables for the counting phase
current_fasta=""
line_count=0
lines_after=()
temp_file=$(mktemp)

# Move collected data to the temporary file for further processing
mv "$output_file" "$temp_file"
> "$output_file"

# Process the temporary file to add counts
while IFS= read -r line; do
    if [[ $line == *_final ]]; then
        # If there is an existing entry, write it out
        if [ -n "$current_fasta" ]; then
            echo "$current_fasta total genes = $line_count" >> "$output_file"
            for l in "${lines_after[@]}"; do
                echo "$l" >> "$output_file"
            done
            echo >> "$output_file"
        fi
        # Reset for the new sample
        current_fasta="$line"
        line_count=0
        lines_after=()
    else
        lines_after+=("$line")
        ((line_count++))
    fi
done < "$temp_file"

# Handle the final entry in the file
if [ -n "$current_fasta" ]; then
    echo "$current_fasta total genes = $line_count" >> "$output_file"
    for l in "${lines_after[@]}"; do
        echo "$l" >> "$output_file"
    done
fi

# Clean up
rm "$temp_file"
echo "------------------------------------------"
echo "Results successfully saved to '$output_file'"
#
echo = `date` job $JOB_NAME done

```



 

