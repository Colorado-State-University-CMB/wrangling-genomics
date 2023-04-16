---
layout: lesson
root: .
---


# Alpine High Performance Computer

**Large-scale cluster computer (SUPERCOMPUTER).**

Alpine is the University of Colorado Boulder Research Computing’s third-generation high performance computing (HPC) cluster. Alpine is a heterogeneous compute cluster currently composed of hardware provided from University of Colorado Boulder, Colorado State University, and Anschutz Medical Campus. Alpine currently offers **317** compute nodes and a total of **18,080** cores.

 - Core: a CPU
 - Node: like your computer, but with 24 to 64 processors
 - RAM: 
   - Each CPU has (256GB or more on special jobs)
   - High memory jobs can request almost 1 Tb 
 - Can run multiple jobs in parallel
 - Free up your own computer 
 - Don't need to be logged in

**What for?**

 - Next-gen sequence analysis
 - Large-scale modeling (deep learning)
 - Render farm for animation
 - Alternative to AWS

[readthedocs on Alpine](https://curc.readthedocs.io/en/latest/clusters/alpine/index.html)

## Directory organization and space

 - home directory: /home/$USER - 5Gb
 - project directory: /projects/$USER - 250Gb
 - scratch directory: /scratch/alpine/$USER - 1Tb (**FILES DELETED AFTER 90 DAYS**) 

 - Config files go in home directory. 
 - Workflows, code, important data files go in project directory. 
 - Temporary data files go in scratch directory

IF YOU FILL UP YOUR HOME DIRECTORY, YOUR ACCOUNT MAY BECOME UNUSABLE


## Command line workflows and High Performance Computing

A lot of genomics analysis is done using command-line tools for three reasons:   
1) you will often be working with a large number of files, and working through the command-line rather than 
through a graphical user interface (GUI) allows you to automate repetitive tasks,    
2) you will often need more compute power than is available on your personal computer, and 
connecting to and interacting with remote computers requires a command-line interface, and    
3) you will often need to customize your analyses, and command-line tools often enable more 
customization than the corresponding GUI tools (if in fact a GUI tool even exists).   

In a [previous lesson](http://www.datacarpentry.org/shell-genomics/), you learned how to use the bash shell to interact with your computer through a command line interface. In this 
lesson, you will be applying this new knowledge to carry out a common genomics workflow - identifying variants among sequencing samples 
taken from multiple individuals within a population. We will be starting with a set of sequenced reads (`.fastq` files), performing
some quality control steps, aligning those reads to a reference genome, and ending by identifying and visualizing variations among these
samples. 

As you progress through this lesson, keep in mind that, even if you aren't going to be doing this same workflow in your research, 
you will be learning some very important lessons about using command-line bioinformatic tools. What you learn here will enable you to 
use a variety of bioinformatic tools with confidence and greatly enhance your research efficiency and productivity.

> ## Prerequisites
>
> This lesson assumes a working understanding of the bash shell. If you haven't already completed the [Shell Genomics](http://www.datacarpentry.org/shell-genomics/) lesson, and aren't familiar with the bash shell, please review those materials
> before starting this lesson.
>
> This lesson also assumes some familiarity with biological concepts, including the structure of DNA, nucleotide abbreviations, and the 
> concept of genomic variation within a population. 
>
> ~~This lesson uses data hosted on an Amazon Machine Instance (AMI). Workshop participants will be given information on how
> to log-in to the AMI during the workshop. Learners using these materials for self-directed study will need to set up their own
> AMI. Information on setting up an AMI and accessing the required data is provided on the [Genomics Workshop setup page](http://www.datacarpentry.org/genomics-workshop/setup.html).~~
>
> We're using Alpine instead (see setup) 
>
{: .prereq}
