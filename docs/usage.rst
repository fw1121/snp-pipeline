.. _usage-label:

========
Usage
========

.. highlight:: bash

The SNP Pipeline is run from the Unix command line.  The pipeline consists of a collection
of shell scripts and python scripts:

    * copy_snppipeline_data.py : copies supplied example data to a work directory
    * prepReference.sh : indexes the reference genome
    * alignSampleToReference.sh : aligns samples to the reference genome
    * prepSamples.sh : finds variants in each sample
    * create_snp_matrix.py : creates a matrix of SNPs across all samples

Step-by-Step Example Workflow Based on Lamda Virus Test Data Provided with Code
-------------------------------------------------------------------------------

Step 1 - Gather data::

    # The SNP Pipeline distribution includes sample data organized as shown below:
    snppipeline/data/lambdaVirusInputs/reference/lambda_virus.fasta
    snppipeline/data/lambdaVirusInputs/samples/sample1/sample1_1.fastq
    snppipeline/data/lambdaVirusInputs/samples/sample1/sample1_2.fastq
    snppipeline/data/lambdaVirusInputs/samples/sample2/sample2_1.fastq
    snppipeline/data/lambdaVirusInputs/samples/sample2/sample2_2.fastq
    snppipeline/data/lambdaVirusInputs/samples/sample3/sample3_1.fastq
    snppipeline/data/lambdaVirusInputs/samples/sample3/sample3_2.fastq
    snppipeline/data/lambdaVirusInputs/samples/sample4/sample4_1.fastq
    snppipeline/data/lambdaVirusInputs/samples/sample4/sample4_2.fastq

Step 2 - Prep work::

    # Copy the supplied test data to a work area:
    cd test
    copy_snppipeline_data.py lambdaVirusInputs testLambdaVirus
    cd testLambdaVirus
    # Create files of sample directories and fastQ files:
    ls -d --color=never samples/* > sampleDirectoryNames.txt
    rm sampleFullPathNames.txt
    cat sampleDirectoryNames.txt | while read dir; do echo $dir/*.fastq >> sampleFullPathNames.txt; done
    # Determine the number of CPU cores in your computer
    NUMCORES=$(grep -c ^processor /proc/cpuinfo)

Step 3 - Prep the reference::

    prepReference.sh reference/lambda_virus

Step 4 - Align the samples to the reference::

    # Align each sample, one at a time, using all CPU cores
    cat sampleFullPathNames.txt | xargs --max-args=2 --max-lines=1 alignSampleToReference.sh $NUMCORES reference/lambda_virus

Step 5 - Prep the samples::

    # Process the samples in parallel using all CPU cores
    cat sampleDirectoryNames.txt | xargs -n 1 -P $NUMCORES prepSamples.sh reference/lambda_virus
        
Step 6 - Run snp pipeline (samtools pileup in parallel and combine alignment and pileup to
generate snp matrix)::

    create_snp_matrix.py -d ./ -f sampleDirectoryNames.txt -r reference/lambda_virus.fasta -l snplist.txt -a snpma.fasta -i True

Step 7 - View the results:

Upon successful completion of the pipeline, the snplist.txt file should have 163 entries.  The SNP Matrix 
can be found in snpma.fasta::

    ls -l snplist.txt
    ls -l snpma.fasta


Step-by-Step Example Workflow Based on S. Agona Data Downloaded from SRA
------------------------------------------------------------------------
TODO: do this


Step-by-Step Workflow - General Case
------------------------------------

Step 1 - Gather data:

You will need the following data:

* Reference genome
* Fastq input files for multiple samples

Organize the data into separate directories for each sample as well as the reference.  One possible
directory layout is shown below.  Note the mix of paired and unpaired samples::

    ./myProject/reference/my_reference.fasta
    ./myProject/samples/sample1/sampleA.fastq
    ./myProject/samples/sample2/sampleB.fastq
    ./myProject/samples/sample3/sampleC_1.fastq
    ./myProject/samples/sample3/sampleC_2.fastq
    ./myProject/samples/sample4/sampleD_1.fastq
    ./myProject/samples/sample4/sampleD_2.fastq

Step 2 - Prep work::

    # Optional step: Copy your input data to a safe place:
    cp -r myProject myProjectClean
    # The SNP pipeline will generate additional files into the reference and sample directories
    cd myProject
    # Create files of sample directories and fastQ files:
    ls -d --color=never samples/* > sampleDirectoryNames.txt
    rm sampleFullPathNames.txt
    cat sampleDirectoryNames.txt | while read dir; do echo $dir/*.fastq >> sampleFullPathNames.txt; done
    # Determine the number of CPU cores in your computer
    NUMCORES=$(grep -c ^processor /proc/cpuinfo)

Step 3 - Prep the reference::

    # Note: do not specify the .fasta file extension here
    prepReference.sh reference/my_reference

Step 4 - Align the samples to the reference::

    # Align each sample, one at a time, using all CPU cores
    cat sampleFullPathNames.txt | xargs --max-args=2 --max-lines=1 alignSampleToReference.sh $NUMCORES reference/my_reference

Step 5 - Prep the samples::

    # Process the samples in parallel using all CPU cores
    cat sampleDirectoryNames.txt | xargs -n 1 -P $NUMCORES prepSamples.sh reference/my_reference

Step 6 - Run snp pipeline (samtools pileup in parallel and combine alignment and pileup to
generate snp matrix)::

    create_snp_matrix.py -d ./ -f sampleDirectoryNames.txt -r reference/my_reference.fasta -l snplist.txt -a snpma.fasta -i True

Step 7 - View the results:

Upon successful completion of the pipeline, the snplist.txt file contains the variants found in each sample.  The SNP Matrix 
can be found in snpma.fasta::

    ls -l snplist.txt
    ls -l snpma.fasta


create_snp_matrix.py Command Syntax
------------------------------------
Help for the SNP Pipeline command-line arguments can be found with the --help parameter::

    create_snp_matrix.py  --help


