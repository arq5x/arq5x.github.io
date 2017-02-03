% samtools Tutorial
% Aaron Quinlan
% November 20, 2013


Synopsis
========

Our goal is to work through examples that demonstrate how to 
explore, process and manipulate SAM and BAM files with the `samtools`
software package.

For future reference, use the samtools [documentation](http://www.htslib.org/doc/).


\


Installing samtools
===================
Follow these steps:

    cd ~
    # optional. you may already have a src directory
    mkdir src
    cd ~/src
    git clone https://github.com/samtools/htslib
    git clone https://github.com/samtools/samtools
    cd samtools
    make
    cp samtools ~/bin


Setup
=====
Create a new directory from your home directory called "samtools-demo".

    cd ~
    mkdir samtools-demo

Navigate into that directory.

    cd samtools-demo

Download the example gzipped SAM file I have provided.

    curl https://s3.amazonaws.com/samtools-tutorial/sample.sam.gz > sample.sam.gz
    gzip -d sample.sam.gz


The samtools help
==================
To bring up the help, just type

    samtools

As you can see, there are multiple "subcommands" and for samtools to
work you must tell it which subcommand you want to use. Examples:

    samtools view
    samtools sort
    samtools depth


Converting SAM to BAM with samtools "view"
==========================================
To do anything meaningful with alignment data from BWA or other aligners (which produce text-based SAM output), we need to first convert the SAM to its binary counterpart, BAM format. The binary format is much easier for computer programs to
work with. However, it is consequently very difficult for humans to read. More on that later.

To convert SAM to BAM, we use the `samtools view` command. We must specify that our input is in SAM format (by default it expects BAM) using the -S option. We must also say that we want the output to be BAM (by default it produces BAM) with
the -b option. Samtools follows the UNIX convention of sending its output to the UNIX STDOUT, so we need to use a redirect operator (">") to create a BAM file from the output. 

    samtools view -S -b sample.sam > sample.bam

Now, we can use the samtools view command to convert the BAM to SAM so we mere mortals can read it.

    samtools view sample.bam | head


samtools "sort"
=================
When you align FASTQ files with all current sequence aligners, the
alignments produced are in random order with respect to their position
in the reference genome. In other words, the BAM file is in the order
that the sequences occurred in the input FASTQ files.

    samtools view sample.bam | head

Doing anything meaningful such as calling variants or visualizing
alignments in IGV) requires that the BAM is further manipulated. It must be sorted such that the alignments occur in "genome order". That is, ordered positionally based upon their alignment coordinates on each chromosome.

    samtools sort sample.bam -o sample.sorted.bam

Now let's check the order:

    samtools view sample.sorted.bam | head

Notice anything different about the coordinates of the alignments?

\


samtools "index"
================

Indexing a genome sorted BAM file allows one to quickly extract alignments
overlapping particular genomic regions. Moreover, indexing is required by genome viewers such as IGV so that the viewers can quickly display alignments in each genomic region to which you navigate.

    samtools index sample.sorted.bam

This will create an additional "index" file. List (`ls`) the contents of the current directory and look for the new index file.

    ls 

Now, let's exploit the index to extract alignments from the 33rd megabase
of chromosome 1. To do this, we use the samtools `view` command, which we will give proper treatment in the next section. For now, just do it without understanding. No really. Do it.

    samtools view sample.sorted.bam 1:33000000-34000000

How many alignments are there in this region?

    samtools view sample.sorted.bam 1:33000000-34000000 | wc -l


\


samtools "view"
================

The samtools `view` command is the most versatile tool in the samtools package. It's main function, not surprisingly, is to allow you to convert the binary (i.e., easy for the computer to read and process) alignments in the BAM file view to text-based SAM alignments that are easy for *humans* to read and process.


Scrutinize some alignments
--------------------------
Let us start by inspecting the first five alignments in our BAM in detail.

    samtools view sample.sorted.bam | head -n 5


Let's make the FLAG more readable
---------------------------------
Let us start by inspecting the first five alignments in our BAM in detail.

    samtools view -X sample.sorted.bam | head -n 5

You can use the detailed help to get a better sense of what each character in the human "readable" FLAG means. See section 6 of the Notes.

    samtools view -?


Count the total number of alignments.
-------------------------------------
    samtools view sample.sorted.bam | wc -l


Inspect the header.
--------------------
The "header" in a BAM file records important information regarding the 
reference genome to which the reads were aligned, as well as other information about how the BAM has been processed. One can ask the `view`
command to report solely the header by using the `-H` option.

    samtools view -H sample.sorted.bam


Capture the FLAG.
-------------------------------------
As we discussed earlier, the FLAG field in the BAM format encodes several key pieces of information regarding how an alignment aligned to the reference genome. We can exploit this information to isolate specific types of alignments that we want to use in our analysis.

For example, we often want to call variants solely from paired-end sequences that aligned "properly" to the reference genome. Why?

To ask the `view` command to report solely "proper pairs" we use the `-f` option and ask for alignments where the second bit is true (proper pair is true).

    samtools view -f 0x2 sample.sorted.bam

How many *properly* paired alignments are there?

    samtools view -f 0x2 sample.sorted.bam | wc -l

Now, let's ask for alignments that are NOT properly paired.  To do this, we use the `-F` option (note the capitalization to denote "opposite").

    samtools view -F 0x2 sample.sorted.bam

How many *improperly* paired alignments are there?

    samtools view -F 0x2 sample.sorted.bam | wc -l

Does everything add up?



Other options.
====================
There are many other options for the `view` command.  This is merely meant to be an appetizer.  Please look through the help, as well as visit the samtools [documentation](http://www.htslib.org/doc/) site.

