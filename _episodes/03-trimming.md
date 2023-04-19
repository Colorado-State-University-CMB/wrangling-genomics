---
title: "Trimming and Filtering"
teaching: 30
exercises: 25
questions:
- "How can I get rid of sequence data that does not meet my quality standards?"
objectives:
- "Clean FASTQ reads using Trimmomatic."
- "Select and set multiple options for command-line bioinformatic tools."
- "Write `for` loops with two variables."
keypoints:
- "The options you set for the command-line tools you use are important!"
- "Data cleaning is an essential step in a genomics workflow."
---

# Cleaning reads

In the previous episode, we took a high-level look at the quality
of each of our samples using FastQC. We visualized per-base quality
graphs showing the distribution of read quality at each base across
all reads in a sample and extracted information about which samples
fail which quality checks. Some of our samples failed quite a few quality metrics used by FastQC. This does not mean,
though, that our samples should be thrown out! It is very common to have some quality metrics fail, and this may or may not be a problem for your downstream application. For our variant calling workflow, we will be removing some of the low quality sequences to reduce our false positive rate due to sequencing error.

We will use a program called
[Trimmomatic](http://www.usadellab.org/cms/?page=trimmomatic) to
filter poor quality reads and trim poor quality bases from our samples.

## Trimmomatic options

Trimmomatic has a variety of options to trim your reads. If we run the following command, we can see some of our options.

~~~
# don't need to be in a particular directory, but need env:
# conda activate qc-trim
trimmomatic
~~~
{: .bash}

Which will give you the following output:
~~~
Usage: 
       PE [-version] [-threads <threads>] [-phred33|-phred64] [-trimlog <trimLogFile>] [-summary <statsSummaryFile>] [-quiet] [-validatePairs] [-basein <inputBase> | <inputFile1> <inputFile2>] [-baseout <outputBase> | <outputFile1P> <outputFile1U> <outputFile2P> <outputFile2U>] <trimmer1>...
   or: 
       SE [-version] [-threads <threads>] [-phred33|-phred64] [-trimlog <trimLogFile>] [-summary <statsSummaryFile>] [-quiet] <inputFile> <outputFile> <trimmer1>...
   or: 
       -version
~~~
{: .output}

This output shows us that we must first specify whether we have paired end (`PE`) or single end (`SE`) reads.
Next, we specify what flag we would like to run. For example, you can specify `threads` to indicate the number of
processors on your computer that you want Trimmomatic to use. In most cases using multiple threads (processors) can help to run the trimming faster. These flags are not necessary, but they can give you more control over the command. The flags are followed by positional arguments, meaning the order in which you specify them is important. 
In paired end mode, Trimmomatic expects the two input files, and then the names of the output files. These files are described below. While, in single end mode, Trimmomatic will expect 1 file as input, after which you can enter the optional settings and lastly the name of the output file.

| option    | meaning |
| ------- | ---------- |
|  \<inputFile1>  | Input reads to be trimmed. Typically the file name will contain an `_1` or `_R1` in the name.|
| \<inputFile2> | Input reads to be trimmed. Typically the file name will contain an `_2` or `_R2` in the name.|
|  \<outputFile1P> | Output file that contains surviving pairs from the `_1` file. |
|  \<outputFile1U> | Output file that contains orphaned reads from the `_1` file. |
|  \<outputFile2P> | Output file that contains surviving pairs from the `_2` file.|
|  \<outputFile2U> | Output file that contains orphaned reads from the `_2` file.|

Two important parameters that we'll use to keep our data organized in our directory structure:

| option    | meaning |
| ------- | ---------- |
|  -basein \<inputBase> | This will automatically prepend our *input* file names with a value. Ours is `-basein ../01_input/untrimmed_fastq/`|
|  -baseout \<outputBase> | This will automatically prepend our *output* file names with a value. Ours is `-baseout ../03_output`|

The last thing trimmomatic expects to see is the trimming parameters:

| step   | meaning |
| ------- | ---------- |
| `ILLUMINACLIP` | Perform adapter removal. |
| `SLIDINGWINDOW` | Perform sliding window trimming, cutting once the average quality within the window falls below a threshold. |
| `LEADING`  | Cut bases off the start of a read, if below a threshold quality.  |
|  `TRAILING` |  Cut bases off the end of a read, if below a threshold quality. |
| `CROP`  |  Cut the read to a specified length. |
|  `HEADCROP` |  Cut the specified number of bases from the start of the read. |
| `MINLEN`  |  Drop an entire read if it is below a specified length. |
|  `TOPHRED33` | Convert quality scores to Phred-33.  |
|  `TOPHRED64` |  Convert quality scores to Phred-64. |

We will use only a few of these options and trimming steps in our
analysis. It is important to understand the steps you are using to
clean your data. For more information about the Trimmomatic arguments
and options, see [the Trimmomatic manual](http://www.usadellab.org/cms/uploads/supplementary/Trimmomatic/TrimmomaticManual_V0.32.pdf).

However, a complete command for Trimmomatic will look something like the command below. **This command is an example and will not work, as we do not have the files it refers to**:

~~~
# only an example command
trimmomatic PE -threads 4 SRR_1056_1.fastq SRR_1056_2.fastq  \
              SRR_1056_1.trimmed.fastq SRR_1056_1un.trimmed.fastq \
              SRR_1056_2.trimmed.fastq SRR_1056_2un.trimmed.fastq \
              ILLUMINACLIP:SRR_adapters.fa SLIDINGWINDOW:4:20
~~~
{: .bash}

In this example, we have told Trimmomatic:

| code   | meaning |
| ------- | ---------- |
| `PE` | that it will be taking a paired end file as input |
| `-threads 4` | to use four computing threads to run (this will speed up our run) |
| `SRR_1056_1.fastq` | the first input file name |
| `SRR_1056_2.fastq` | the second input file name |
| `SRR_1056_1.trimmed.fastq` | the output file for surviving pairs from the `_1` file |
| `SRR_1056_1un.trimmed.fastq` | the output file for orphaned reads from the `_1` file |
| `SRR_1056_2.trimmed.fastq` | the output file for surviving pairs from the `_2` file |
| `SRR_1056_2un.trimmed.fastq` | the output file for orphaned reads from the `_2` file |
| `ILLUMINACLIP:SRR_adapters.fa`| to clip the Illumina adapters from the input file using the adapter sequences listed in `SRR_adapters.fa` |
|`SLIDINGWINDOW:4:20` | to use a sliding window of size 4 that will remove bases if their phred score is below 20 |




## Running Trimmomatic

Now we will run Trimmomatic on our data. But first, explore the directory pointed to by `$CONDA_PREFIX` to get data that was installed with trimmomatic.

### Exploring your conda environment/installation

> ## Understanding more about conda 
> We know that we've created conda environments and added software to them,
> but we don't *see* what conda is doing.
> 
> One way to look is to explore the directories where conda does its work.
{: .callout}

This *section doesn't require being in a specific directory,* because all commands will deal with **absolute paths**. 

Absolute paths fully describe the 
location of a file or directory, and are valid regardless of your current working directory. You can identify an absolute path because it always starts with a `/`. We have mainly dealt with *relative paths* so far, which can contain slashes, but never at the very beginning.

~~~
# don't need to be in a particular directory, but need env:
# conda activate qc-trim
echo $CONDA_PREFIX
~~~
{: .bash}

David's output
~~~
/projects/.colostate.edu/dcking/software/anaconda/envs/qc-trim
~~~
{: .output}

This is a subdirectory of my /projects directory that is just for the environment qc-trim.  All programs data I installed into this environment
will be in here.  Let's explore it. *Yours may differ for some packages*

~~~
# don't need to be in a particular directory, but need env:
# conda activate qc-trim
ls $CONDA_PREFIX
~~~
{: .bash}

David's output
~~~
bin              conda-meta  etc    include  legal  libexec  opt      share  var                          x86_64-conda-linux-gnu
compiler_compat  conf        fonts  jmods    lib    man      release  ssl    x86_64-conda_cos7-linux-gnu
~~~
{: .output}

Installed programs are in `bin`, whereas installed data is typical in `share`.

~~~
# don't need to be in a particular directory, but need env:
# conda activate qc-trim
ls $CONDA_PREFIX/share
~~~
{: .bash}

*My `share` directory*:

~~~
aclocal          dbus-1  fontconfig  glib-2.0  info      locale  tabset    trimmomatic         xml
bash-completion  doc     gettext     icu       licenses  man     terminfo  trimmomatic-0.39-2  zoneinfo
~~~
{: .output}

There are two `trimmomatic` directories, but `trimmomatic` is just a link (alias) to `trimmomatic-0.39-2`.

~~~
# don't need to be in a specific directory
total 568
drwxr-xr-x.  2 dcking@colostate.edu erinnishgrp@colostate.edu  149 Apr 16 20:01 aclocal
drwxr-xr-x.  3 dcking@colostate.edu erinnishgrp@colostate.edu   29 Apr 16 19:58 bash-completion
drwxr-xr-x.  2 dcking@colostate.edu erinnishgrp@colostate.edu   59 Apr 16 19:58 dbus-1
drwxr-xr-x.  6 dcking@colostate.edu erinnishgrp@colostate.edu   91 Apr 16 19:58 doc
drwxr-xr-x.  3 dcking@colostate.edu erinnishgrp@colostate.edu   28 Apr 16 20:01 fontconfig
drwxr-xr-x.  3 dcking@colostate.edu erinnishgrp@colostate.edu   21 Apr 16 19:58 gettext
drwxr-xr-x.  6 dcking@colostate.edu erinnishgrp@colostate.edu   97 Apr 16 19:58 glib-2.0
drwxr-xr-x.  3 dcking@colostate.edu erinnishgrp@colostate.edu   22 Apr 16 20:01 icu
drwxr-xr-x.  2 dcking@colostate.edu erinnishgrp@colostate.edu  213 Apr 16 20:01 info
drwxr-xr-x.  4 dcking@colostate.edu erinnishgrp@colostate.edu   53 Apr 16 19:57 licenses
drwxr-xr-x. 27 dcking@colostate.edu erinnishgrp@colostate.edu  509 Apr 16 20:01 locale
drwxr-xr-x.  8 dcking@colostate.edu erinnishgrp@colostate.edu  128 Apr 16 20:01 man
drwxr-xr-x.  2 dcking@colostate.edu erinnishgrp@colostate.edu   91 Apr 16 19:57 tabset
drwxr-xr-x. 44 dcking@colostate.edu erinnishgrp@colostate.edu  798 Apr 16 19:57 terminfo
lrwxrwxrwx.  1 dcking@colostate.edu erinnishgrp@colostate.edu   18 Apr 16 19:58 trimmomatic -> trimmomatic-0.39-2
drwxr-xr-x.  3 dcking@colostate.edu erinnishgrp@colostate.edu  224 Apr 16 19:58 trimmomatic-0.39-2
drwxr-xr-x.  4 dcking@colostate.edu erinnishgrp@colostate.edu   52 Apr 16 20:01 xml
drwxr-xr-x. 19 dcking@colostate.edu erinnishgrp@colostate.edu 1590 Apr 16 19:57 zoneinfo
~~~
{: .output}

Explore `trimmomatic`

~~~
# don't need to be in a particular directory, but need env:
# conda activate qc-trim
ls $CONDA_PREFIX/share/trimmomatic
~~~
{: .bash}

~~~
adapters  build_env_setup.sh  conda_build.sh  LICENSE  metadata_conda_debug.yaml  trimmomatic  trimmomatic.jar
~~~
{: .output}

~~~
# don't need to be in a particular directory, but need env:
# conda activate qc-trim
ls $CONDA_PREFIX/share/trimmomatic/adapters
~~~
{: .bash}

~~~
NexteraPE-PE.fa  TruSeq2-PE.fa  TruSeq2-SE.fa  TruSeq3-PE-2.fa  TruSeq3-PE.fa  TruSeq3-SE.fa
~~~
{: .output}

**WE FOUND IT**
The adapter sequences are just short fasta format sequences
~~~
# don't need to be in a particular directory, but need env:
# conda activate qc-trim
cat $CONDA_PREFIX/share/trimmomatic/adapters/NexteraPE-PE.fa 
~~~
{: .bash}

~~~
>PrefixNX/1
AGATGTGTATAAGAGACAG
>PrefixNX/2
AGATGTGTATAAGAGACAG
>Trans1
TCGTCGGCAGCGTCAGATGTGTATAAGAGACAG
>Trans1_rc
CTGTCTCTTATACACATCTGACGCTGCCGACGA
>Trans2
GTCTCGTGGGCTCGGAGATGTGTATAAGAGACAG
>Trans2_rc
CTGTCTCTTATACACATCTCCGAGCCCACGAGAC
~~~
{: .output}


We are going to run Trimmomatic on one of our paired-end samples. 
While using FastQC we saw that Nextera adapters were present in our samples. 
The adapter sequences came with the installation of trimmomatic, so we will first copy these sequences into our current directory.

~~~
# cd /projects/$USER/CM580A3-Intro-to-qCMB-2023/10_Alpine_HPC
cd ../01_input/untrimmed_fastq
cp $CONDA_PREFIX/share/trimmomatic/adapters/NexteraPE-PE.fa .
ls
~~~
{: .bash}

~~~
NexteraPE-PE.fa        SRR2584863_2.fastq     SRR2584866_1.fastq.gz  SRR2589044_1.fastq     SRR2589044_2.fastq.gz
SRR2584863_1.fastq     SRR2584863_2.fastq.gz  SRR2584866_2.fastq     SRR2589044_1.fastq.gz
SRR2584863_1.fastq.gz  SRR2584866_1.fastq     SRR2584866_2.fastq.gz  SRR2589044_2.fastq
~~~
{: .output}

New command template (this is not the actual command)
~~~ 
trimmomatic PE -threads 4 SRR_1056_1.fastq SRR_1056_2.fastq  \
              SRR_1056_1.trimmed.fastq SRR_1056_1un.trimmed.fastq \
              SRR_1056_2.trimmed.fastq SRR_1056_2un.trimmed.fastq \
              ILLUMINACLIP:SRR_adapters.fa SLIDINGWINDOW:4:20
~~~
{: .bash}

 - We will: use a sliding window of size 4 that will remove bases if their
phred score is below 20 (like in our example above). 
 - ... also discard any reads that do not have at least 25 bases remaining after
this trimming step. 
 - Three additional pieces of code are also added to the end of the ILLUMINACLIP step. 
   - These three additional numbers (2:40:15) tell Trimmomatic how to handle sequence matches to the Nextera adapters. 
   - A detailed explanation of how they work is advanced for this particular lesson. 
 - For now we will use these numbers as a default and recognize they are needed to for Trimmomatic
to run properly. 

This command will take a few minutes to run. ***(faster on Alpine)***

Make a script called `trimmomatic.sbatch`.

~~~
#!/usr/bin/env bash
#SBATCH --nodes=1
#SBATCH --ntasks=1
#SBATCH --time=0:10:00
#SBATCH --partition=amilan
#SBATCH --qos=normal
#SBATCH --job-name=trimmomatic

source /curc/sw/anaconda3/latest

conda activate qc-trim

adapters_fa=../01_input/NexteraPE-PE.fa
IN=../01_input/untrimmed_fastq
OUT=../03_output/trimmed_reads
mkdir -p $OUT

trimmomatic PE \
  $IN/SRR2589044_1.fastq.gz      $IN/SRR2589044_2.fastq.gz \
   $OUT/SRR2589044_1.trim.fastq.gz $OUT/SRR2589044_1un.trim.fastq.gz \
   $OUT/SRR2589044_2.trim.fastq.gz $OUT/SRR2589044_2un.trim.fastq.gz \
   SLIDINGWINDOW:4:20 \
   MINLEN:25 \
   ILLUMINACLIP:$adapters_fa:2:40:15
~~~
{: .bash}

Run the script:
~~~
sbatch trimmomatic.sbatch
~~~
{: .bash}

~~~
Submitted batch job 1141244
~~~
{: .output}
 
> ## Using variables and string functions to generalize a script
> You can generalize your script by replacing explicit values in your
> code with that of a variable. You set the variable before using it, so, 
> when you want to change your script to work on a different value, that value
> only needs to be changed in one place. 
> ~~~
> 
> filename=SRR2589044_1.fastq.gz # want to remove "_1.fastq.gz" 
> accession=$(basename $filename _1.fastq.gz}
>                                   
> echo "processing accession $accession" 
> # prints: processing accession SRR2589044
> # replace all previous occurrences of SRR2589044 with ${accession}
> trimmomatic PE \
>   $IN/${accession}_1.fastq.gz      $IN/${accession}_2.fastq.gz \
>   $OUT/${accession}_1.trim.fastq.gz $OUT/${accession}_1un.trim.fastq.gz \
>   $OUT/${accession}_2.trim.fastq.gz $OUT/${accession}_2un.trim.fastq.gz \
>   SLIDINGWINDOW:4:20 \
>   MINLEN:25 \
>   ILLUMINACLIP:$adapters_fa:2:40:15
>~~~
>{: .bash}
> You have to do `${accession}` to separate the variable name from the trailing underscores (i.e. `${accession}_`) 
>
{: .callout}

Run the script:
~~~
sbatch trimmomatic.sbatch
~~~
{: .bash}

~~~
Submitted batch job 1141303
~~~
{: .output}


The output from trimmomatic is will be in a log file (slurm-1141303.out) like this:

~~~
TrimmomaticPE: Started with arguments:
 SRR2589044_1.fastq.gz SRR2589044_2.fastq.gz SRR2589044_1.trim.fastq.gz SRR2589044_1un.trim.fastq.gz SRR2589044_2.trim.fastq.gz SRR2589044_2un.trim.fastq.gz SLIDINGWINDOW:4:20 MINLEN:25 ILLUMINACLIP:NexteraPE-PE.fa:2:40:15
Multiple cores found: Using 2 threads
Using PrefixPair: 'AGATGTGTATAAGAGACAG' and 'AGATGTGTATAAGAGACAG'
Using Long Clipping Sequence: 'GTCTCGTGGGCTCGGAGATGTGTATAAGAGACAG'
Using Long Clipping Sequence: 'TCGTCGGCAGCGTCAGATGTGTATAAGAGACAG'
Using Long Clipping Sequence: 'CTGTCTCTTATACACATCTCCGAGCCCACGAGAC'
Using Long Clipping Sequence: 'CTGTCTCTTATACACATCTGACGCTGCCGACGA'
ILLUMINACLIP: Using 1 prefix pairs, 4 forward/reverse sequences, 0 forward only sequences, 0 reverse only sequences
Quality encoding detected as phred33
Input Read Pairs: 1107090 Both Surviving: 885220 (79.96%) Forward Only Surviving: 216472 (19.55%) Reverse Only Surviving: 2850 (0.26%) Dropped: 2548 (0.23%)
TrimmomaticPE: Completed successfully
~~~
{: .output}

> ## Exercise
>
> Use the output from your Trimmomatic command to answer the
> following questions.
>
> 1) What percent of reads did we discard from our sample?
> 2) What percent of reads did we keep both pairs?
>
>> ## Solution
>> 1) 0.23%
>> 2) 79.96%
> {: .solution}
{: .challenge}

You may have noticed that Trimmomatic automatically detected the
quality encoding of our sample. It is always a good idea to
double-check this or to enter the quality encoding manually.

We can confirm that we have our output files:

~~~
ls ../03_output/trimmed_reads/SRR2589044* -l -h
~~~
{: .bash}

~~~
-rw-r--r--. 1 dcking@colostate.edu dckingpgrp@colostate.edu  94M Apr 18 21:28 ../03_output/trimmed_reads/SRR2589044_1.trim.fastq.gz
-rw-r--r--. 1 dcking@colostate.edu dckingpgrp@colostate.edu  18M Apr 18 21:28 ../03_output/trimmed_reads/SRR2589044_1un.trim.fastq.gz
-rw-r--r--. 1 dcking@colostate.edu dckingpgrp@colostate.edu  91M Apr 18 21:28 ../03_output/trimmed_reads/SRR2589044_2.trim.fastq.gz
-rw-r--r--. 1 dcking@colostate.edu dckingpgrp@colostate.edu 271K Apr 18 21:28 ../03_output/trimmed_reads/SRR2589044_2un.trim.fastq.gz
~~~
{: .output}


We have just successfully run Trimmomatic on one of our FASTQ files!
However, there is some bad news. Trimmomatic can only operate on
one sample at a time and we have more than one sample. The good news
is that we can use a `for` loop to iterate through our sample files
quickly! 

~~~
#!/usr/bin/env bash
#SBATCH --nodes=1
#SBATCH --ntasks=1
#SBATCH --time=0:10:00
#SBATCH --partition=amilan
#SBATCH --qos=normal
#SBATCH --job-name=trimmomatic

source /curc/sw/anaconda3/latest

conda activate qc-trim

adapters_fa=../01_input/untrimmed_fastq/NexteraPE-PE.fa
IN=../01_input/untrimmed_fastq
OUT=../03_output/trimmed_reads
mkdir -p $OUT
for infile in $IN/*_1.fastq.gz
do
   accession=$(basename $infile _1.fastq.gz)
   echo "processing accession $accession" 
   trimmomatic PE \
       $IN/${accession}_1.fastq.gz      $IN/${accession}_2.fastq.gz \
       $OUT/${accession}_1.trim.fastq.gz $OUT/${accession}_1un.trim.fastq.gz \
       $OUT/${accession}_2.trim.fastq.gz $OUT/${accession}_2un.trim.fastq.gz \
       SLIDINGWINDOW:4:20 \
       MINLEN:25 \
       ILLUMINACLIP:$adapters_fa:2:40:15
done
~~~
{: .bash}


Go ahead and run the for loop. It should take a few minutes for
Trimmomatic to run for each of our six input files. Once it is done
running, take a look at your directory contents. You will notice that even though we ran Trimmomatic on file `SRR2589044` before running the for loop, there is only one set of files for it. Because we matched the ending `_1.fastq.gz`, we re-ran Trimmomatic on this file, overwriting our first results. That is ok, but it is good to be aware that it happened.

~~~
# cd /projects/$USER/CM580A3-Intro-to-qCMB-2023/10_Alpine_HPC/02_scripts
cd ../03_output/trimmed_fastq
ls
~~~
{: .bash}

~~~
SRR2584863_1.trim.fastq.gz    SRR2584863_2un.trim.fastq.gz  SRR2584866_2.trim.fastq.gz    SRR2589044_1un.trim.fastq.gz
SRR2584863_1un.trim.fastq.gz  SRR2584866_1.trim.fastq.gz    SRR2584866_2un.trim.fastq.gz  SRR2589044_2.trim.fastq.gz
SRR2584863_2.trim.fastq.gz    SRR2584866_1un.trim.fastq.gz  SRR2589044_1.trim.fastq.gz    SRR2589044_2un.trim.fastq.gz
~~~
{: .output}

## make use of multiple threads

Notice the addition of `-threads`, passing it the value of $SLURM_NTASKS.

You can change --ntasks=2. Try 4, or 6. 

~~~
#!/usr/bin/env bash
#SBATCH --nodes=1
#SBATCH --ntasks=2
#SBATCH --time=0:10:00
#SBATCH --partition=amilan
#SBATCH --qos=normal
#SBATCH --job-name=trimmomatic
source /curc/sw/anaconda3/latest
conda activate qc-trim
adapters_fa=../01_input/untrimmed_fastq/NexteraPE-PE.fa
IN=../01_input/untrimmed_fastq
OUT=../03_output/trimmed_reads
mkdir -p $OUT
for infile in $IN/*_1.fastq.gz
do
   accession=$(basename $infile _1.fastq.gz)
   echo "processing accession $accession" 
   trimmomatic PE \
       -threads $SLURM_NTASKS\
       $IN/${accession}_1.fastq.gz      $IN/${accession}_2.fastq.gz \
       $OUT/${accession}_1.trim.fastq.gz $OUT/${accession}_1un.trim.fastq.gz \
       $OUT/${accession}_2.trim.fastq.gz $OUT/${accession}_2un.trim.fastq.gz \
       SLIDINGWINDOW:4:20 \
       MINLEN:25 \
       ILLUMINACLIP:$adapters_fa:2:40:15
done
~~~
{: .bash}

## optional - array version


~~~
#!/usr/bin/env bash
#SBATCH --nodes=1
#SBATCH --ntasks=4
#SBATCH --time=0:02:00
#SBATCH --partition=amilan
#SBATCH --qos=normal
#SBATCH --job-name=trimmomatic
# to submit this script, you do sbatch --array=0-2 trimmomatic.sbatch
source /curc/sw/anaconda3/latest
conda activate qc-trim
adapters_fa=../01_input/untrimmed_fastq/NexteraPE-PE.fa
IN=../01_input/untrimmed_fastq
OUT=../03_output/trimmed_reads
mkdir -p $OUT
infiles=( $IN/*_1.fastq.gz )
infile=${infiles[$SLURM_ARRAY_TASK_ID]}
accession=$(basename $infile _1.fastq.gz)
echo "processing accession $accession" 
trimmomatic PE \
   -threads $SLURM_NTASKS\
   $IN/${accession}_1.fastq.gz      $IN/${accession}_2.fastq.gz \
   $OUT/${accession}_1.trim.fastq.gz $OUT/${accession}_1un.trim.fastq.gz \
   $OUT/${accession}_2.trim.fastq.gz $OUT/${accession}_2un.trim.fastq.gz \
   SLIDINGWINDOW:4:20 \
   MINLEN:25 \
   ILLUMINACLIP:$adapters_fa:2:40:15
~~~
{: .bash}

~~~
sbatch --array=0-2 trimmomatic.sbatch
~~~
{: .bash}

~~~
Submitted batch job 1141362
~~~
{: .output}


Checking job status:

~~~
sq
~~~
{: .bash}

Array job pending:

~~~
Tue Apr 18 22:01:59 2023
             JOBID PARTITION     NAME     USER ST       TIME  NODES NODELIST(REASON)
     1141362_[0-2]    amilan trimmoma dcking@c PD       0:00      1 (Priority)
~~~
{: .output}

Array job running:

~~~
Tue Apr 18 22:02:19 2023
             JOBID PARTITION     NAME     USER ST       TIME  NODES NODELIST(REASON)
         1141362_0    amilan trimmoma dcking@c  R       0:09      1 c3cpu-c11-u1-3
         1141362_1    amilan trimmoma dcking@c  R       0:09      1 c3cpu-c11-u1-3
         1141362_2    amilan trimmoma dcking@c  R       0:09      1 c3cpu-c11-u1-3
~~~
{: .output}

Comparison between single job, 4-6 threads, and array job, 4 threads:

~~~
sa
~~~
{: .bash}

~~~
1141352      trimmomat+          4  COMPLETED      0:0   00:02:07   00:10:00 2023-04-18T21:51:20 2023-04-18T21:51:43 2023-04-18T21:53:50 
1141360      trimmomat+          6  COMPLETED      0:0   00:01:58   00:10:00 2023-04-18T21:54:19 2023-04-18T21:55:17 2023-04-18T21:57:15 
1141362_0    trimmomat+          4  COMPLETED      0:0   00:00:44   00:02:00 2023-04-18T22:01:15 2023-04-18T22:02:10 2023-04-18T22:02:54 
1141362_1    trimmomat+          4  COMPLETED      0:0   00:01:00   00:02:00 2023-04-18T22:01:15 2023-04-18T22:02:10 2023-04-18T22:03:10 
1141362_2    trimmomat+          4  COMPLETED      0:0   00:00:29   00:02:00 2023-04-18T22:01:15 2023-04-18T22:02:10 2023-04-18T22:02:39 
~~~
{: .output}

## Maintain data integrity with checksums


~~~
md5sum * > checksum.md5
cat checksum.md5
~~~
{: .bash}

~~~
ada4ddd4e2f02671786491a793d3b380  SRR2584863_1.trim.fastq.gz
50045bcaab602667a95f53870002e68d  SRR2584863_1un.trim.fastq.gz
7c22ca7cd60030160f6425d9e6bc75ea  SRR2584863_2.trim.fastq.gz
c6314804acac3a013f951d6fb6fe2d52  SRR2584863_2un.trim.fastq.gz
9bf83b31d7882f637f57d0ecac2e288f  SRR2584866_1.trim.fastq.gz
57ed6d9cf50697693127c76d41ec074e  SRR2584866_1un.trim.fastq.gz
e6cd3bdc17af0b5779fb3f40ff8e280e  SRR2584866_2.trim.fastq.gz
84ad1a8cd661253c25a653c3cb5b364b  SRR2584866_2un.trim.fastq.gz
e029c09c402be9e8d9703ebd74c6b5b0  SRR2589044_1.trim.fastq.gz
c5d43e3d4ae406b34019cf290c795f9e  SRR2589044_1un.trim.fastq.gz
7d29399f60affb38e3439d3132857363  SRR2589044_2.trim.fastq.gz
375e9041a385a39571ac0672e4ed764c  SRR2589044_2un.trim.fastq.gz
~~~
{: .output}

~~~
md5sum -c checksum.md5
~~~
{: .bash}

~~~
SRR2584863_1.trim.fastq.gz: OK
SRR2584863_1un.trim.fastq.gz: OK
SRR2584863_2.trim.fastq.gz: OK
SRR2584863_2un.trim.fastq.gz: OK
SRR2584866_1.trim.fastq.gz: OK
SRR2584866_1un.trim.fastq.gz: OK
SRR2584866_2.trim.fastq.gz: OK
SRR2584866_2un.trim.fastq.gz: OK
SRR2589044_1.trim.fastq.gz: OK
SRR2589044_1un.trim.fastq.gz: OK
SRR2589044_2.trim.fastq.gz: OK
SRR2589044_2un.trim.fastq.gz: OK
~~~
{: .output}


