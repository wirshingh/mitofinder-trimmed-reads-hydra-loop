## Run Mitofinder on Hydra using trimmed reads in a loo

"
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
SAMPLEDIR="path to trimmed reads"  # Use double quotes for string assignment

# Loop through the R1 files
for GETSAMPLENAME in ${SAMPLEDIR}/*_R1_PE_trimmed.fastq.gz; do
    SAMPLENAME=$(basename "$GETSAMPLENAME" _R1_PE_trimmed.fastq.gz)
    
    # Create a directory for the current sample
    mkdir -p ./mitofinder_trimmedreads_All_Results/${SAMPLENAME}
    
    # Change to the sample's directory
    cd mitofinder_trimmedreads_All_Results/${SAMPLENAME} || exit 1  # Ensure we exit if cd fails

    # Run mitofinder with the specified options
    mitofinder \
        -j "${SAMPLENAME}_mitofinder_trimmedreads_results" \
        -o 5 \
        -r /scratch/nmnh_lab/macdonaldk/ref/mito_reference_(ref_SEE_BELOW)_*.gb \
        -1 "${SAMPLEDIR}/${SAMPLENAME}_R1_PE_trimmed.fastq.gz" \
        -2 "${SAMPLEDIR}/${SAMPLENAME}_R2_PE_trimmed.fastq.gz" \
        --new-genes

    # Return to the original directory
    cd ../.. || exit 1  # Ensure we exit if cd fails
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
for FINAL_GENES in ./*_Final_Results
do
cp "$FINAL_GENES"/*Final_Results/*final_genes_NT.fasta ./mitofinder_trimmedreads_Final_Genes
done
#
echo = `date` job $JOB_NAME done
"
