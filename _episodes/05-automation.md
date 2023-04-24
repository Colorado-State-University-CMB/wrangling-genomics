---
title: "Automating a Variant Calling Workflow"
teaching: 30
exercises: 15
questions:
- "How can I make my workflow more efficient and less error-prone?"
objectives:
- "Write a shell script with multiple variables."
- "Incorporate a `for` loop into a shell script."
keypoints:
- "We can combine multiple commands into a shell script to automate a workflow."
- "Use `echo` statements within your scripts to get an automated progress update."
---

### Setup

~~~
# cd /projects/$USER/CM580A3-Intro-to-qCMB-2023/10_Alpine_HPC/02_scripts
git pull
cp ../.templates/align_and_vcf_template.sbatch ./align_and_vcf.sbatch
~~~
{: .bash}

There is no output. You maybe submit this script as-is to process a subsampled dataset.

### In an interactive compute session (ainteractive)



We will download and index the _E. coli_ genome in an interactive compute session using `align_and_vcf.sbatch`.
~~~
(base) [dcking@colostate.edu@login12 02_scripts]$ ainteractive
~~~
{: .bash}

~~~
ainteractive: submitting job... salloc --nodes=1 --ntasks=1  --partition=ainteractive --time=1:00:00  --bell srun --pty /bin/bash
salloc: Granted job allocation 1158992
salloc: Waiting for resource configuration
salloc: Nodes c3cpu-a7-u34-2 are ready for job
(base) [dcking@colostate.edu@c3cpu-a7-u34-2 02_scripts]$
~~~
{: .output}

Notice the arguments: ` --nodes=1 --ntasks=1  --partition=ainteractive --time=1:00:00`. The `ainteractive` sessions are meant to use minimal resources.  We will make changes to the script to get it running, but then leave ainteractive and submit the script using sbatch.

The initial part of the script only needs to run once, so we will comment it out after we run it.

~~~
### Download data ########################################################################################
### You must comment out lines in this box after the first time you run it. ##############################
echo -n "DOWNLOAD GENOME: "                                                                             #
mkdir -p $GENOME_DIR                                                                                    #   
echo "downloading..."                                                                                   #
curl -L -o $GENOME_DIR/$GENOME_FILE.gz $GENOME_URL                                                      #   
gunzip $GENOME_DIR/$GENOME_FILE.gz                                                                      #   
                                                                                                        #   
### Create indexes                                                                                      #
### You must comment out these lines after the first time you run it.                                   #
echo -n "INDEXING GENOME... "                                                                           #
bwa index $GENOME_DIR/$GENOME_FILE                                                                      #   
                                                                                                        #   
exit                                                                                                    #   
### end of download/index portion. Comment out (put a '#' at the beginning of each line ##################
##########################################################################################################
~~~
{: .bash}

~~~
# cd /projects/$USER/CM580A3-Intro-to-qCMB-2023/10_Alpine_HPC/02_scripts
bash align_and_vcf.sbatch
~~~
{: .bash}

~~~
ALIGN, COVERAGE AND VARIANT CALLING PIPELINE!
Starting... Sun Apr 23 22:11:42 MDT 2023
------------------------------------------------------------------
DOWNLOAD GENOME: downloading...
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100 1343k  100 1343k    0     0  1284k      0  0:00:01  0:00:01 --:--:-- 1285k
INDEXING GENOME... [bwa_index] Pack FASTA... 0.03 sec
[bwa_index] Construct BWT for the packed sequence...
[bwa_index] 0.70 seconds elapse.
[bwa_index] Update BWT... 0.02 sec
[bwa_index] Pack forward-only FASTA... 0.02 sec
[bwa_index] Construct SA from BWT and Occ... 0.30 sec
[main] Version: 0.7.17-r1188
[main] CMD: bwa index ../01_input/reference_genome/ecoli_rel606.fasta
[main] Real time: 1.141 sec; CPU: 1.058 sec
~~~
{: .output}


You must do three things:
 1. comment out the part in the box
 2. edit the script to use a threads argument:
    - example: command_name -t $SLURM_NTASKS
    - the commands that make use of an argument are `bwa`, `samtools`, and `bcftools`
 3. edit the script to use your trimmed fastq files
 4. Using sbatch (not ainteractive), see how fast you can get your script to finish by increasing ntasks

Look at the data being produced. Look at the log files being created. Figure out what it's doing.


END OF CM580A3 LESSON
END OF CM580A3 LESSON
END OF CM580A3 LESSON
END OF CM580A3 LESSON
END OF CM580A3 LESSON


