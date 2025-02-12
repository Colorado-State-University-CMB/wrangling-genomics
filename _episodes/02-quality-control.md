---
title: "Assessing Read Quality"
teaching: 30
exercises: 20
questions:
- "How can I describe the quality of my data?"
objectives:
- "Explain how a FASTQ file encodes per-base quality scores."
- "Interpret a FastQC plot summarizing per-base quality across all reads."
- "Use `for` loops to automate operations on multiple files."
keypoints:
- "Quality encodings vary across sequencing platforms."
- "`for` loops let you perform the same set of operations on multiple files with a single command."
---

# Bioinformatic workflows

When working with high-throughput sequencing data, the raw reads you get off of the sequencer will need to pass
through a number of  different tools in order to generate your final desired output. The execution of this set of
tools in a specified order is commonly referred to as a *workflow* or a *pipeline*.

An example of the workflow we will be using for our variant calling analysis is provided below with a brief
description of each step.

![workflow](../img/variant_calling_workflow.png)


1. Quality control - Assessing quality using FastQC
2. Quality control - Trimming and/or filtering reads (if necessary)
3. Align reads to reference genome
4. Perform post-alignment clean-up
5. Variant calling

These workflows in bioinformatics adopt a plug-and-play approach in that the output of one tool can be easily
used as input to another tool without any extensive configuration. Having standards for data formats is what
makes this feasible. Standards ensure that data is stored in a way that is generally accepted and agreed upon
within the community. The tools that are used to analyze data at different stages of the workflow are therefore
built under the assumption that the data will be provided in a specific format.

# Starting with data

Often times, the first step in a bioinformatic workflow is getting the data you want to work with onto a computer where you can work with it. If you have outsourced sequencing of your data, the sequencing center will usually provide you with a link that you can use to download your data. Today we will be working with publicly available sequencing data.

We are studying a population of *Escherichia coli* (designated Ara-3), which were propagated for more than 50,000 generations in a glucose-limited minimal medium. We will be working with three samples from this experiment, one from 5,000 generations, one from 15,000 generations, and one from 50,000 generations. The population changed substantially during the course of the experiment, and we will be exploring how with our variant calling workflow.

The data are paired-end, so we will download two files for each sample. We will use the [European Nucleotide Archive](https://www.ebi.ac.uk/ena) to get our data. The ENA "provides a comprehensive record of the world's nucleotide sequencing information, covering raw sequencing data, sequence assembly information and functional annotation." The ENA also provides sequencing data in the fastq format, an important format for sequencing reads that we will be learning about today.

We will:
 1. Create a subdirectory in 01_input
 2. Create a script directory
 3. Write a download script

It will take about 15 minutes to download the files.

Here we are using the `-p` option for `mkdir`. This option allows `mkdir` to create the new directory, even if one of the parent directories does not already exist. It also supresses errors if the directory already exists, without overwriting that directory. 

~~~
# cd /projects/$USER/CM580A3-Intro-to-qCMB-2023/10_Alpine_HPC

mkdir -p 01_input/untrimmed_fastq/
mkdir -p 02_scripts
mkdir -p 03_output
cd 02_scripts
~~~
{: .bash}

## WE WILL NOW CREATE AND RUN OUR FIRST BATCH SCRIPT

It is not always necessary to download data as its own job, but we'll do so
to practice our scripting.

~~~
# cd /projects/$USER/CM580A3-Intro-to-qCMB-2023/10_Alpine_HPC/02_scripts
nano Downloader.sbatch
~~~
{: .bash}

**Start your script with the following lines**

~~~
#!/usr/bin/env bash
#SBATCH --nodes=1
#SBATCH --ntasks=1
#SBATCH --time=0:20:00
#SBATCH --partition=amilan
#SBATCH --qos=normal
#SBATCH --job-name=download-fastq
~~~
{: .bash}

Save the file now to create it. (you may exit out of nano) 

Breakdown of the above script header:

|Code|Description|
|----|-----------|
|#!/usr/bin/env bash| Called `shebang`. It tells the system what kind of script this is.|
|#SBATCH --nodes=1| We request one node.|
|#SBATCH --ntasks=1| We request one CPU.|
|#SBATCH --time=0:10:00| Actually only 5 minutes the way we're going to do it. |
|#SBATCH --partition=amilan| An Alpine-specific parameter for a standard job. |
|#SBATCH --qos=normal|Quality Of Service- normal|
|#SBATCH --job-name=download-fastq|This name will appear in some reports|

Complete the script by pasting these lines at the end. That's all for now, ***but do not run yet!***

Note: the original instructions downloaded from EBI, but the connection was no longer working.
These files are served courtesy of the Osborne Nishimura Lab.

> ## Exercise
>
>  Open the script in ondemand's visual editor.
>
> You should be able to find Downloader.sbatch in the Dashboard File browser under the /projects/youreid@colostate.edu section.
>
> Once you find it, look for an option to open the file to edit it.
>
>> ## Solution
>>
>> Once navigating to your `/projects` file listing, click CM580A3-Intro-to-qCMB-2023, 
>>
>> then click 10_Alpine_HPC
>>
>> then click 02_scripts
>>
>> Between the Name and Size of the file is a clickable box with 3 dots, in there is an option to edit the file.
>>
>> It will open in a new tab with a text editor.
> {: .solution}
{: .challenge}

Add the following to your script:

~~~
cd ../01_input/untrimmed_fastq
curl -O http://129.82.125.224:34/CM580A3/10_Alpine_HPC/SRR2589044/SRR2589044_1.fastq.gz
curl -O http://129.82.125.224:34/CM580A3/10_Alpine_HPC/SRR2589044/SRR2589044_2.fastq.gz
curl -O http://129.82.125.224:34/CM580A3/10_Alpine_HPC/SRR2584863/SRR2584863_1.fastq.gz
curl -O http://129.82.125.224:34/CM580A3/10_Alpine_HPC/SRR2584863/SRR2584863_2.fastq.gz
curl -O http://129.82.125.224:34/CM580A3/10_Alpine_HPC/SRR2584866/SRR2584866_1.fastq.gz
curl -O http://129.82.125.224:34/CM580A3/10_Alpine_HPC/SRR2584866/SRR2584866_2.fastq.gz

gunzip *.gz
~~~
{: .bash}

Make sure you save and quit (CTRL-X)

We will make use of relative paths, including the parent directory  ".." to keep our file organization straight. Here we did that with the first `cd`.


### Submitting the script and checking its status

~~~
# cd /projects/$USER/CM580A3-Intro-to-qCMB-2023/10_Alpine_HPC/02_scripts
sbatch Downloader.sbatch
~~~
{: .bash}

You will get a job ID if the submission was successful.

~~~
Submitted batch job 1129890
~~~
{: .output}


#### Using slurm commands to check the job status

**The aliases were not working for me properly yesterday (Alpine is in growing pains).** See 
Jobs->Active Jobs tab in the on-demand interface for a report that does appear to be working.

|Alias|Command|Description|
|-----|-------|-----------|
|sq|squeue -u $USER|squeue gives you a report of what you have running and what is waiting in the job queue|
|sa|sacct -X --format JobID,JobName,AllocCPUS,State,ExitCode,Elapsed,TimeLimit,Submit,Start,End|sacct is a job report|



Mine *seemed* to wait in the queue for a long time:


~~~
sq
~~~
{: .bash}

~~~
Sun Apr 16 14:47:35 2023
             JOBID PARTITION     NAME     USER ST       TIME  NODES NODELIST(REASON)
           1129890    amilan download dcking@c PD       0:00      1 (Priority)
~~~
{: .ouput}

BUT, it turned out that sacct and squeue were not working properly, so...

Use the Jobs->Active Jobs tab in the on-demand interface.


Once the job is running, the log file will be created and updated as it goes along. My job's number was 1130135,
so my log file was slurm-1130135.out.

~~~
# cd /projects/$USER/CM580A3-Intro-to-qCMB-2023/10_Alpine_HPC/02_scripts
ls
~~~
{: .bash}

~~~
Downloader.sbatch  slurm-1130135.out
~~~
{: .output}


Let's look at the log file using `cat` (your job number will be different from mine):

~~~
cat slurm-1130135.out 
~~~
{: .bash}

~~~
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  123M  100  123M    0     0  82.1M      0  0:00:01  0:00:01 --:--:-- 82.1M
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  127M  100  127M    0     0  98.8M      0  0:00:01  0:00:01 --:--:-- 98.8M
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  174M  100  174M    0     0  5158k      0  0:00:34  0:00:34 --:--:-- 5135k
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  182M  100  182M    0     0  5467k      0  0:00:34  0:00:34 --:--:-- 5493k
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  308M  100  308M    0     0  97.5M      0  0:00:03  0:00:03 --:--:-- 97.5M
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  295M  100  295M    0     0   108M      0  0:00:02  0:00:02 --:--:--  108M
~~~
{: .output}


---


# Quality control

We will now assess the quality of the sequence reads contained in our fastq files.

![workflow_qc](../img/var_calling_workflow_qc.png)
## Details on the FASTQ format

Although it looks complicated (and it is), we can understand the
[fastq](https://en.wikipedia.org/wiki/FASTQ_format) format with a little decoding. Some rules about the format
include...

|Line|Description|
|----|-----------|
|1|Always begins with '@' and then information about the read|
|2|The actual DNA sequence|
|3|Always begins with a '+' and sometimes the same info in line 1|
|4|Has a string of characters which represent the quality scores; must have same number of characters as line 2|

We can view the first complete read in one of the files our dataset by using `head` to look at
the first four lines.

~~~
# cd /projects/$USER/CM580A3-Intro-to-qCMB-2023/10_Alpine_HPC/02_scripts
cd  ../01_input/untrimmed_fastq
head -n 4 SRR2584863_1.fastq
~~~
{: .bash}

~~~
@SRR2584863.1 HWI-ST957:244:H73TDADXX:1:1101:4712:2181/1
TTCACATCCTGACCATTCAGTTGAGCAAAATAGTTCTTCAGTGCCTGTTTAACCGAGTCACGCAGGGGTTTTTGGGTTACCTGATCCTGAGAGTTAACGGTAGAAACGGTCAGTACGTCAGAATTTACGCGTTGTTCGAACATAGTTCTG
+
CCCFFFFFGHHHHJIJJJJIJJJIIJJJJIIIJJGFIIIJEDDFEGGJIFHHJIJJDECCGGEGIIJFHFFFACD:BBBDDACCCCAA@@CA@C>C3>@5(8&>C:9?8+89<4(:83825C(:A#########################
~~~
{: .output}

Line 4 shows the quality for each nucleotide in the read. Quality is interpreted as the
probability of an incorrect base call (e.g. 1 in 10) or, equivalently, the base call
accuracy (e.g. 90%). To make it possible to line up each individual nucleotide with its quality
score, the numerical score is converted into a code where each individual character
represents the numerical quality score for an individual nucleotide. For example, in the line
above, the quality score line is:

~~~
CCCFFFFFGHHHHJIJJJJIJJJIIJJJJIIIJJGFIIIJEDDFEGGJIFHHJIJJDECCGGEGIIJFHFFFACD:BBBDDACCCCAA@@CA@C>C3>@5(8&>C:9?8+89<4(:83825C(:A#########################
~~~
{: .output}

The numerical value assigned to each of these characters depends on the
sequencing platform that generated the reads. The sequencing machine used to generate our data
uses the standard Sanger quality PHRED score encoding, using Illumina version 1.8 onwards.
Each character is assigned a quality score between 0 and 41 as shown in
the chart below.

~~~
Quality encoding: !"#$%&'()*+,-./0123456789:;<=>?@ABCDEFGHIJ
                   |         |         |         |         |
Quality score:    01........11........21........31........41
~~~
{: .output}

Each quality score represents the probability that the corresponding nucleotide call is
incorrect. This quality score is logarithmically based, so a quality score of 10 reflects a
base call accuracy of 90%, but a quality score of 20 reflects a base call accuracy of 99%.
These probability values are the results from the base calling algorithm and depend on how
much signal was captured for the base incorporation.

Looking back at our read:

~~~
@SRR2584863.1 HWI-ST957:244:H73TDADXX:1:1101:4712:2181/1
TTCACATCCTGACCATTCAGTTGAGCAAAATAGTTCTTCAGTGCCTGTTTAACCGAGTCACGCAGGGGTTTTTGGGTTACCTGATCCTGAGAGTTAACGGTAGAAACGGTCAGTACGTCAGAATTTACGCGTTGTTCGAACATAGTTCTG
+
CCCFFFFFGHHHHJIJJJJIJJJIIJJJJIIIJJGFIIIJEDDFEGGJIFHHJIJJDECCGGEGIIJFHFFFACD:BBBDDACCCCAA@@CA@C>C3>@5(8&>C:9?8+89<4(:83825C(:A#########################
~~~
{: .output}

we can now see that there is a range of quality scores, but that the end of the sequence is
very poor (`#` = a quality score of 2).

> ## Exercise
>
> What is the last read in the `SRR2584863_1.fastq ` file? How confident
> are you in this read?
>
>> ## Solution
>> ~~~
>> $ tail -n 4 SRR2584863_1.fastq
>> ~~~
>> {: .bash}
>>
>> ~~~
>> @SRR2584863.1553259 HWI-ST957:245:H73R4ADXX:2:2216:21048:100894/1
>> CTGCAATACCACGCTGATCTTTCACATGATGTAAGAAAAGTGGGATCAGCAAACCGGGTGCTGCTGTGGCTAGTTGCAGCAAACCATGCAGTGAACCCGCCTGTGCTTCGCTATAGCCGTGACTGATGAGGATCGCCGGAAGCCAGCCAA
>> +
>> CCCFFFFFHHHHGJJJJJJJJJHGIJJJIJJJJIJJJJIIIIJJJJJJJJJJJJJIIJJJHHHHHFFFFFEEEEEDDDDDDDDDDDDDDDDDCDEDDBDBDDBDDDDDDDDDBDEEDDDD7@BDDDDDD>AA>?B?<@BDD@BDC?BDA?
>> ~~~
>> {: .output}
>>
>> This read has more consistent quality at its end than the first
>> read that we looked at, but still has a range of quality scores,
>> most of them high. We will look at variations in position-based quality
>> in just a moment.
>>
> {: .solution}
{: .challenge}

At this point, lets validate that all the relevant tools are installed. Remember how we set up the conda environments?

Activate your conda environment for fastqc in order to access the program. `conda activate qc-trim`

~~~
$ fastqc -h
            FastQC - A high throughput sequence QC analysis tool

SYNOPSIS

        fastqc seqfile1 seqfile2 .. seqfileN

    fastqc [-o output dir] [--(no)extract] [-f fastq|bam|sam]
           [-c contaminant file] seqfile1 .. seqfileN

DESCRIPTION

    FastQC reads a set of sequence files and produces from each one a quality
    control report consisting of a number of different modules, each one of
    which will help to identify a different potential type of problem in your
    data.

    If no files to process are specified on the command line then the program
    will start as an interactive graphical application.  If files are provided
    on the command line then the program will run with no user interaction
    required.  In this mode it is suitable for inclusion into a standardised
    analysis pipeline.

    The options for the program as as follows:

    -h --help       Print this help file and exit

    -v --version    Print the version of the program and exit

    -o --outdir     Create all output files in the specified output directory.
                    Please note that this directory must exist as the program
                    will not create it.  If this option is not set then the
                    output file for each sequence file is created in the same
                    directory as the sequence file which was processed.

    --casava        Files come from raw casava output. Files in the same sample
                    group (differing only by the group number) will be analysed
                    as a set rather than individually. Sequences with the filter
                    flag set in the header will be excluded from the analysis.
                    Files must have the same names given to them by casava
                    (including being gzipped and ending with .gz) otherwise they
                    will not be grouped together correctly.

    --nano          Files come from naopore sequences and are in fast5 format. In
                    this mode you can pass in directories to process and the program
                    will take in all fast5 files within those directories and produce
                    a single output file from the sequences found in all files.

    --nofilter      If running with --casava then don't remove read flagged by
                    casava as poor quality when performing the QC analysis.

    --extract       If set then the zipped output file will be uncompressed in
                    the same directory after it has been created.  By default
                    this option will be set if fastqc is run in non-interactive
                    mode.

    -j --java       Provides the full path to the java binary you want to use to
                    launch fastqc. If not supplied then java is assumed to be in
                    your path.

    --noextract     Do not uncompress the output file after creating it.  You
                    should set this option if you do not wish to uncompress
                    the output when running in non-interactive mode.

    --nogroup       Disable grouping of bases for reads >50bp. All reports will
                    show data for every base in the read.  WARNING: Using this
                    option will cause fastqc to crash and burn if you use it on
                    really long reads, and your plots may end up a ridiculous size.
                    You have been warned!

    -f --format     Bypasses the normal sequence file format detection and
                    forces the program to use the specified format.  Valid
                    formats are bam,sam,bam_mapped,sam_mapped and fastq

    -t --threads    Specifies the number of files which can be processed
                    simultaneously.  Each thread will be allocated 250MB of
                    memory so you shouldn't run more threads than your
                    available memory will cope with, and not more than
                    6 threads on a 32 bit machine

    -c              Specifies a non-default file which contains the list of
    --contaminants  contaminants to screen overrepresented sequences against.
                    The file must contain sets of named contaminants in the
                    form name[tab]sequence.  Lines prefixed with a hash will
                    be ignored.

    -a              Specifies a non-default file which contains the list of
    --adapters      adapter sequences which will be explicity searched against
                    the library. The file must contain sets of named adapters
                    in the form name[tab]sequence.  Lines prefixed with a hash
                    will be ignored.

    -l              Specifies a non-default file which contains a set of criteria
    --limits        which will be used to determine the warn/error limits for the
                    various modules.  This file can also be used to selectively
                    remove some modules from the output all together.  The format
                    needs to mirror the default limits.txt file found in the
                    Configuration folder.

   -k --kmers       Specifies the length of Kmer to look for in the Kmer content
                    module. Specified Kmer length must be between 2 and 10. Default
                    length is 7 if not specified.

   -q --quiet       Supress all progress messages on stdout and only report errors.

   -d --dir         Selects a directory to be used for temporary files written when
                    generating report images. Defaults to system temp directory if
                    not specified.

BUGS

    Any bugs in fastqc should be reported either to simon.andrews@babraham.ac.uk
    or in www.bioinformatics.babraham.ac.uk/bugzilla/
~~~
{: .bash}


## Assessing quality using FastQC
In real life, you will not be assessing the quality of your reads by visually inspecting your 
FASTQ files. Rather, you will be using a software program to assess read quality and 
filter out poor quality reads. We will first use a program called [FastQC](http://www.bioinformatics.babraham.ac.uk/projects/fastqc/) to visualize the quality of our reads. 
Later in our workflow, we will use another program to filter out poor quality reads. 


FastQC has a number of features which can give you a quick impression of any problems your
data may have, so you can take these issues into consideration before moving forward with your
analyses. Rather than looking at quality scores for each individual read, FastQC looks at
quality collectively across all reads within a sample. The image below shows one FastQC-generated plot that indicates
a very high quality sample:

![good_quality](../img/good_quality1.8.png)

The x-axis displays the base position in the read, and the y-axis shows quality scores. In this
example, the sample contains reads that are 40 bp long. This is much shorter than the reads we
are working with in our workflow. For each position, there is a box-and-whisker plot showing
the distribution of quality scores for all reads at that position. The horizontal red line
indicates the median quality score and the yellow box shows the 1st to
3rd quartile range. This means that 50% of reads have a quality score that falls within the
range of the yellow box at that position. The whiskers show the absolute range, which covers
the lowest (0th quartile) to highest (4th quartile) values.

For each position in this sample, the quality values do not drop much lower than 32. This
is a high quality score. The plot background is also color-coded to identify good (green),
acceptable (yellow), and bad (red) quality scores.

Now let's take a look at a quality plot on the other end of the spectrum.

![bad_quality](../img/bad_quality1.8.png)

Here, we see positions within the read in which the boxes span a much wider range. Also, quality scores drop quite low into the "bad" range, particularly on the tail end of the reads. The FastQC tool produces several other diagnostic plots to assess sample quality, in addition to the one plotted above.

## Running FastQC

We will now assess the quality of the reads that we downloaded. First, make sure you are still in the `untrimmed_fastq` directory

~~~
# cd /projects/$USER/CM580A3-Intro-to-qCMB-2023/10_Alpine_HPC
cd 01_input/untrimmed_fastq/
~~~
{: .bash}

> ## Exercise
>
>  How big are the files?
> (Hint: Look at the options for the `ls` command to see how to show
> file sizes.)
>
>> ## Solution
>>
>> ~~~
>> $ ls -l -h
>> ~~~
>> {: .bash}
>>
>> ~~~
>> -rw-rw-r-- 1 dcuser dcuser 545M Jul  6 20:27 SRR2584863_1.fastq
>> -rw-rw-r-- 1 dcuser dcuser 183M Jul  6 20:29 SRR2584863_2.fastq.gz
>> -rw-rw-r-- 1 dcuser dcuser 309M Jul  6 20:34 SRR2584866_1.fastq.gz
>> -rw-rw-r-- 1 dcuser dcuser 296M Jul  6 20:37 SRR2584866_2.fastq.gz
>> -rw-rw-r-- 1 dcuser dcuser 124M Jul  6 20:22 SRR2589044_1.fastq.gz
>> -rw-rw-r-- 1 dcuser dcuser 128M Jul  6 20:24 SRR2589044_2.fastq.gz
>> ~~~
>> {: .output}
>>
>> There are six FASTQ files ranging from 124M (124MB) to 545M.
>>
> {: .solution}
{: .challenge}


# Write a job script for `fastqc`

One of the most important challenges is organizing 
your directory structure for a workflow. 
The following organization conveniently
separates input files from scripts, and 
generated output. You may find a different
 organization suits you better, but we have set it up into 01_intput, 02_scripts, 03_output.

Verify that your directory structure looks right:

~~~
# cd /projects/$USER/CM580A3-Intro-to-qCMB-2023/10_Alpine_HPC
ls
~~~
{: .bash}

~~~
01_input  02_scripts  03_output  CM580A3_Alpine_setup.sh  README.md
~~~
{: .output}

Above, 01_input, 02_scripts, and 03_output are directories created with `mkdir`.

As with the download script, care must be taken to make sure the pathnames
are specified properly. 

For example, if we are inside 02_scripts, 
then we refer to 01_input as ../01_input. 
Likewise, we refer to 03_output as ../03_output
This will be clear in the arguments we
use with the fastq command. 

---

To begin writing our script, go to 
02_scripts. 

```
# cd /projects/$USER/CM580A3-Intro-to-qCMB-2023/10_Alpine_HPC
cd 02_scripts
```

Create a new file called fastqc.sbatch. You can use `nano` in the terminal, or create it with the ondemand file browser.

This time, we are going to use more than one CPU. We'll use a ***special variable*** called `$SLURM_NTASKS` to make use of it in the script.

~~~
#!/usr/bin/env bash
#SBATCH --nodes=1
#SBATCH --ntasks=2
#SBATCH --time=0:20:00
#SBATCH --partition=amilan
#SBATCH --qos=normal
#SBATCH --job-name=fastqc

source /curc/sw/anaconda3/latest

conda activate qc-trim

mkdir -p ../03_output/fastqc_reports

fastqc -o ../03_output/fastqc_reports -t $SLURM_NTASKS ../01_input/untrimmed_fastq/*.fastq

~~~
{: .bash}

Save your script and submit using `sbatch`

~~~
# cd /projects/$USER/CM580A3-Intro-to-qCMB-2023/10_Alpine_HPC/02_scripts
sbatch fastqc.sbatch
~~~
{: .bash}

~~~
Submitted job 1234567
~~~
{: .output}

The log file will continually update with the 
progress of the analysis. It will start like this:

~~~
Started analysis of SRR2584863_1.fastq
Approx 5% complete for SRR2584863_1.fastq
Approx 10% complete for SRR2584863_1.fastq
Approx 15% complete for SRR2584863_1.fastq
Approx 20% complete for SRR2584863_1.fastq
Approx 25% complete for SRR2584863_1.fastq
Approx 30% complete for SRR2584863_1.fastq
Approx 35% complete for SRR2584863_1.fastq
Approx 40% complete for SRR2584863_1.fastq
Approx 45% complete for SRR2584863_1.fastq
~~~
{: .output}

In total, it should take about five minutes for FastQC to run on all
six of our FASTQ files (or *will* it?????). When the analysis completes, your prompt
will return. So your screen will look something like this:

~~~
Approx 80% complete for SRR2589044_2.fastq.gz
Approx 85% complete for SRR2589044_2.fastq.gz
Approx 90% complete for SRR2589044_2.fastq.gz
Approx 95% complete for SRR2589044_2.fastq.gz
Analysis complete for SRR2589044_2.fastq.gz
$
~~~
{: .output}



~~~
# cd /projects/$USER/CM580A3-Intro-to-qCMB-2023/10_Alpine_HPC/03_output
ls fastqc_repports/
~~~
{: .bash}

~~~
SRR2584863_1_fastqc.html  SRR2584863_2_fastqc.zip   SRR2584866_2_fastqc.html  SRR2589044_1_fastqc.zip
SRR2584863_1_fastqc.zip   SRR2584866_1_fastqc.html  SRR2584866_2_fastqc.zip   SRR2589044_2_fastqc.html
SRR2584863_2_fastqc.html  SRR2584866_1_fastqc.zip   SRR2589044_1_fastqc.html  SRR2589044_2_fastqc.zip
~~~
{: .output}

For each input FASTQ file, FastQC has created a `.zip` file and a

`.html` file. The `.zip` file extension indicates that this is 
actually a compressed set of multiple output files. We will be working
with these output files soon. The `.html` file is a stable webpage
displaying the summary report for each of our samples.

We want to keep our data files and our results files separate, so notice that we
we had the output files write into a new directory within our `03_output/` directory, using the `-o` argument to fastqc.


Now we can navigate into this results directory and do some closer
inspection of our output files. *BUT* we have to download the html (or zip) files
because they won't render in the ondemand browser. 

Go to /projects/$USER/CM580A3-Intro-to-qCMB-2023/10_Alpine_HPC/03_output in the ondemand file browser.


## Viewing the FastQC results

You can download the html files one at a time, or by checking each box. I recommend saving them all in a new directory.
Once you have, you can open any of the html files to get the full report for that sample.


## Decoding the other FastQC outputs
We have now looked at quite a few "Per base sequence quality" FastQC graphs, but there are nine other graphs that we have not talked about! Below we have provided a brief overview of interpretations for each of these plots. For more information, please see the FastQC documentation [here](https://www.bioinformatics.babraham.ac.uk/projects/fastqc/Help/) 


+ [**Per tile sequence quality**](https://www.bioinformatics.babraham.ac.uk/projects/fastqc/Help/3%20Analysis%20Modules/12%20Per%20Tile%20Sequence%20Quality.html): the machines that perform sequencing are divided into tiles. This plot displays patterns in base quality along these tiles. Consistently low scores are often found around the edges, but hot spots can also occur in the middle if an air bubble was introduced at some point during the run.
+ [**Per sequence quality scores**](https://www.bioinformatics.babraham.ac.uk/projects/fastqc/Help/3%20Analysis%20Modules/3%20Per%20Sequence%20Quality%20Scores.html): a density plot of quality for all reads at all positions. This plot shows what quality scores are most common.
+ [**Per base sequence content**](https://www.bioinformatics.babraham.ac.uk/projects/fastqc/Help/3%20Analysis%20Modules/4%20Per%20Base%20Sequence%20Content.html): plots the proportion of each base position over all of the reads. Typically, we expect to see each base roughly 25% of the time at each position, but this often fails at the beginning or end of the read due to quality or adapter content.
+ [**Per sequence GC content**](https://www.bioinformatics.babraham.ac.uk/projects/fastqc/Help/3%20Analysis%20Modules/5%20Per%20Sequence%20GC%20Content.html): a density plot of average GC content in each of the reads.
+ [**Per base N content**](https://www.bioinformatics.babraham.ac.uk/projects/fastqc/Help/3%20Analysis%20Modules/6%20Per%20Base%20N%20Content.html): the percent of times that 'N' occurs at a position in all reads. If there is an increase at a particular position, this might indicate that something went wrong during sequencing.
+ [**Sequence Length Distribution**](https://www.bioinformatics.babraham.ac.uk/projects/fastqc/Help/3%20Analysis%20Modules/7%20Sequence%20Length%20Distribution.html): the distribution of sequence lengths of all reads in the file. If the data is raw, there is often on sharp peak, however if the reads have been trimmed, there may be a distribution of shorter lengths.
+ [**Sequence Duplication Levels**](https://www.bioinformatics.babraham.ac.uk/projects/fastqc/Help/3%20Analysis%20Modules/8%20Duplicate%20Sequences.html): A distribution of duplicated sequences. In sequencing, we expect most reads to only occur once. If some sequences are occurring more than once, it might indicate enrichment bias (e.g. from PCR). If the samples are high coverage (or RNA-seq or amplicon), this might not be true.
+ [**Overrepresented sequences**](https://www.bioinformatics.babraham.ac.uk/projects/fastqc/Help/3%20Analysis%20Modules/9%20Overrepresented%20Sequences.html): A list of sequences that occur more frequently than would be expected by chance.
+ [**Adapter Content**](https://www.bioinformatics.babraham.ac.uk/projects/fastqc/Help/3%20Analysis%20Modules/10%20Adapter%20Content.html): a graph indicating where adapater sequences occur in the reads.
+ [**K-mer Content**](https://www.bioinformatics.babraham.ac.uk/projects/fastqc/Help/3%20Analysis%20Modules/11%20Kmer%20Content.html): a graph showing any sequences which may show a positional bias within the reads.

## Working with the FastQC text output

Now that we have looked at our HTML reports to get a feel for the data,
let's look more closely at the other output files. Go back to the tab
open to the terminal.
~~~
# cd /projects/$USER/CM580A3-Intro-to-qCMB-2023/10_Alpine_HPC/03_output/fastqc_untrimmed_reads
ls
~~~
{: .bash}

~~~
SRR2584863_1_fastqc.html  SRR2584866_1_fastqc.html  SRR2589044_1_fastqc.html
SRR2584863_1_fastqc.zip   SRR2584866_1_fastqc.zip   SRR2589044_1_fastqc.zip
SRR2584863_2_fastqc.html  SRR2584866_2_fastqc.html  SRR2589044_2_fastqc.html
SRR2584863_2_fastqc.zip   SRR2584866_2_fastqc.zip   SRR2589044_2_fastqc.zip
~~~
{: .output}

Our `.zip` files are compressed files. They each contain multiple
different types of output files for a single input FASTQ file. To
view the contents of a `.zip` file, we can use the program `unzip`
to decompress these files. Let's try doing them all at once using a
wildcard.


We can't do `unzip *.zip` like we did `gunzip *.gz` because it doesn't support multiple files on the command line.
Therefore, we'll use a `for` loop like we learned in the Shell Genomics lesson to iterate through all of
our `.zip` files and unzip them. Let's see what that looks like and then we will 
discuss what we are doing with each line of our loop.

~~~
# cd /projects/$USER/CM580A3-Intro-to-qCMB-2023/10_Alpine_HPC/03_output/fastqc_reports
for filename in *.zip
> do
> unzip $filename
> done
~~~
{: .bash}

In this example, the input is six filenames (one filename for each of our `.zip` files).
Each time the loop iterates, it will assign a file name to the variable `filename`
and run the `unzip` command.
The first time through the loop,
`$filename` is `SRR2584863_1_fastqc.zip`.
The interpreter runs the command `unzip` on `SRR2584863_1_fastqc.zip`.
For the second iteration, `$filename` becomes
`SRR2584863_2_fastqc.zip`. This time, the shell runs `unzip` on `SRR2584863_2_fastqc.zip`.
It then repeats this process for the four other `.zip` files in our directory.


When we run our `for` loop, you will see output that starts like this:

~~~
Archive:  SRR2589044_2_fastqc.zip
   creating: SRR2589044_2_fastqc/
   creating: SRR2589044_2_fastqc/Icons/
   creating: SRR2589044_2_fastqc/Images/
  inflating: SRR2589044_2_fastqc/Icons/fastqc_icon.png
  inflating: SRR2589044_2_fastqc/Icons/warning.png
  inflating: SRR2589044_2_fastqc/Icons/error.png
  inflating: SRR2589044_2_fastqc/Icons/tick.png
  inflating: SRR2589044_2_fastqc/summary.txt
  inflating: SRR2589044_2_fastqc/Images/per_base_quality.png
  inflating: SRR2589044_2_fastqc/Images/per_tile_quality.png
  inflating: SRR2589044_2_fastqc/Images/per_sequence_quality.png
  inflating: SRR2589044_2_fastqc/Images/per_base_sequence_content.png
  inflating: SRR2589044_2_fastqc/Images/per_sequence_gc_content.png
  inflating: SRR2589044_2_fastqc/Images/per_base_n_content.png
  inflating: SRR2589044_2_fastqc/Images/sequence_length_distribution.png
  inflating: SRR2589044_2_fastqc/Images/duplication_levels.png
  inflating: SRR2589044_2_fastqc/Images/adapter_content.png
  inflating: SRR2589044_2_fastqc/fastqc_report.html
  inflating: SRR2589044_2_fastqc/fastqc_data.txt
  inflating: SRR2589044_2_fastqc/fastqc.fo
~~~
{: .output}

The `unzip` program is decompressing the `.zip` files and creating
a new directory (with subdirectories) for each of our samples, to
store all of the different output that is produced by FastQC. There

are a lot of files here. The one we are going to focus on is the 
`summary.txt` file. 


If you list the files in our directory now you will see:

~~~
SRR2584863_1_fastqc       SRR2584866_1_fastqc       SRR2589044_1_fastqc
SRR2584863_1_fastqc.html  SRR2584866_1_fastqc.html  SRR2589044_1_fastqc.html
SRR2584863_1_fastqc.zip   SRR2584866_1_fastqc.zip   SRR2589044_1_fastqc.zip
SRR2584863_2_fastqc       SRR2584866_2_fastqc       SRR2589044_2_fastqc
SRR2584863_2_fastqc.html  SRR2584866_2_fastqc.html  SRR2589044_2_fastqc.html
SRR2584863_2_fastqc.zip   SRR2584866_2_fastqc.zip   SRR2589044_2_fastqc.zip
~~~
{:. output}

The `.html` files and the uncompressed `.zip` files are still present,
***but now we also have a new directory for each of our samples.*** We can 
see for sure that it is a directory if we use the `-F` flag for `ls`. 

~~~
# cd /projects/$USER/CM580A3-Intro-to-qCMB-2023/10_Alpine_HPC/03_output/fastqc_reports
ls -F
~~~
{: .bash}

~~~
SRR2584863_1_fastqc/      SRR2584866_1_fastqc/      SRR2589044_1_fastqc/
SRR2584863_1_fastqc.html  SRR2584866_1_fastqc.html  SRR2589044_1_fastqc.html
SRR2584863_1_fastqc.zip   SRR2584866_1_fastqc.zip   SRR2589044_1_fastqc.zip
SRR2584863_2_fastqc/      SRR2584866_2_fastqc/      SRR2589044_2_fastqc/
SRR2584863_2_fastqc.html  SRR2584866_2_fastqc.html  SRR2589044_2_fastqc.html
SRR2584863_2_fastqc.zip   SRR2584866_2_fastqc.zip   SRR2589044_2_fastqc.zip
~~~
{: .output}

Let's see what files are present within one of these output directories.

~~~
# cd /projects/$USER/CM580A3-Intro-to-qCMB-2023/10_Alpine_HPC/03_output/fastqc_reports
ls -F SRR2584863_1_fastqc/
~~~
{: .bash}

~~~
fastqc_data.txt  fastqc.fo  fastqc_report.html	Icons/	Images/  summary.txt
~~~
{: .output}

Use `less` to preview the `summary.txt` file for this sample.

~~~
# cd /projects/$USER/CM580A3-Intro-to-qCMB-2023/10_Alpine_HPC/03_output/fastqc_reports
less SRR2584863_1_fastqc/summary.txt
~~~
{: .bash}

~~~
PASS    Basic Statistics        SRR2584863_1.fastq
PASS    Per base sequence quality       SRR2584863_1.fastq
PASS    Per tile sequence quality       SRR2584863_1.fastq
PASS    Per sequence quality scores     SRR2584863_1.fastq
WARN    Per base sequence content       SRR2584863_1.fastq
WARN    Per sequence GC content SRR2584863_1.fastq
PASS    Per base N content      SRR2584863_1.fastq
PASS    Sequence Length Distribution    SRR2584863_1.fastq
PASS    Sequence Duplication Levels     SRR2584863_1.fastq
PASS    Overrepresented sequences       SRR2584863_1.fastq
WARN    Adapter Content SRR2584863_1.fastq
~~~
{: .output}

The summary file gives us a list of tests that FastQC ran, and tells
us whether this sample passed, failed, or is borderline (`WARN`). Remember, to quit from `less` you must type `q`.

## Documenting our work

We can make a record of the results we obtained for all our samples

by concatenating all of our `summary.txt` files into a single file 
using the `cat` command. We will call this `fastqc_summaries.txt` and move
it to `fastqc_summaries.txt`.

~~~
# cd /projects/$USER/CM580A3-Intro-to-qCMB-2023/10_Alpine_HPC/03_output/fastqc_reports
cat */summary.txt > ../fastqc_summaries.txt
cd ..
ls
~~~
{: .bash}

~~~
fastqc_summaries.txt  fastqc_untrimmed_reads
~~~
{: .output}

> ## Exercise
>
> Which samples failed at least one of FastQC's quality tests? What
> test(s) did those samples fail?
>
>> ## Solution
>>
>> We can get the list of all failed tests using `grep`.
>>
>> ~~~
>> # cd /projects/$USER/CM580A3-Intro-to-qCMB-2023/10_Alpine_HPC/03_output/
>> grep FAIL fastqc_summaries.txt
>> ~~~
>> {: .bash}
>>
>> ~~~
>> FAIL    Per base sequence quality       SRR2584863_2.fastq.gz
>> FAIL    Per tile sequence quality       SRR2584863_2.fastq.gz
>> FAIL    Per base sequence content       SRR2584863_2.fastq.gz
>> FAIL    Per base sequence quality       SRR2584866_1.fastq.gz
>> FAIL    Per base sequence content       SRR2584866_1.fastq.gz
>> FAIL    Adapter Content SRR2584866_1.fastq.gz
>> FAIL    Adapter Content SRR2584866_2.fastq.gz
>> FAIL    Adapter Content SRR2589044_1.fastq.gz
>> FAIL    Per base sequence quality       SRR2589044_2.fastq.gz
>> FAIL    Per tile sequence quality       SRR2589044_2.fastq.gz
>> FAIL    Per base sequence content       SRR2589044_2.fastq.gz
>> FAIL    Adapter Content SRR2589044_2.fastq.gz
>> ~~~
>> {: .output}
>>
> {: .solution}
{: .challenge}



# Other notes  -- optional 


> ## Quality encodings vary
>
> Although we have used a particular quality encoding system to demonstrate interpretation of 
> read quality, different sequencing machines use different encoding systems. This means that, 
> depending on which sequencer you use to generate your data, a `#` may not be an indicator of 
> a poor quality base call.
>
> This mainly relates to older Solexa/Illumina data,
> but it is essential that you know which sequencing platform was
> used to generate your data, so that you can tell your quality control program which encoding
> to use. If you choose the wrong encoding, you run the risk of throwing away good reads or
> (even worse) not throwing away bad reads!
{: .callout}


> ## Same symbols, different meanings
>
> Here we see `>` being used as a shell prompt, whereas `>` is also
> used to redirect output.
> Similarly, `$` is used as a shell prompt, but, as we saw earlier,
> it is also used to ask the shell to get the value of a variable.
>
> If the *shell* prints `>` or `$` then it expects you to type something,
> and the symbol is a prompt.
>
> If *you* type `>` or `$` yourself, it is an instruction from you that
> the shell should redirect output or get the value of a variable.
{: .callout}
