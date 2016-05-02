---
title: A species and build agnostic version of gemini. 
category: blog
layout: post
comments: true
post_author: Brent Pedersen
---

Synopsis
-------------
This post describes how to leverage functionality now availability in GEMINI to study genetic variation from any build of the human genome, *as well as any other species*.

Gemini Lite
-----------
As we've described in a previous [post](http://quinlanlab.org/blog/2016/02/08/gemini-version-0.18.1.html), we have been working on making gemini much more flexible
so that researchers have much greater control over the genome annotations that are integrated into the resulting database. We have also tried to provide fine scale control over which attributes from each annotation file are added (e.g., which INFO field(s) do you want to include from the ExAC or ClinVar VCF files?).

Our solution depends upon an entirely new workflow, consisting of the following steps:

+ First, use [vt](https://github.com/atks/vt) to decompose and normalize your VCF
+ Then, annotate the gene and transcript impact with annotate with VEP and/or snpEff.
+ Next, annotate the resulting VCF with any additional annotations using [vcfanno](https://github.com/brentp/vcfanno). This step effectively replaces the fixed, hard-coded annotations that GEMINI previously created for a VCF.
+ Finally, load the completely annotated VCF with [vcf2b](https://github.com/quinlan-lab/vcf2db), the new loader that I have developed. As we described in our [previous post](http://quinlanlab.org/blog/2016/02/08/gemini-version-0.18.1.html), `vcf2db` uses the INFO fields in the VCF to drive which columns are created in the GEMINI database. In this workflow, new INFO tags are added by `vcfanno` for each annotation file and attribute.
+ We emphasize that neither the `vcfanno` nor `vcf2db` steps make any assumptions about the species or genome build. Therefore, this is a *general solution to variant annotation and exploration*.

At this point, we have some additional work to do with helping to vet quality annotation
files for various species about build, but it's possible to use this
*improved* workflow today to support.

Below, I'll demonstrate its use on human (GRCh37) annotations but it
can just as easily be used for any other build (e.g., GRCh38) or non-human species. 
All that is required is the researcher collect the annotation files relevant to the 
species and build of interest and create a `vcfanno` configuration file that dictates 
the exact annotation files and attributes that are desired. 

We recognize that curating annotation files is a challenge itself --- we will discuss our general solution to this problem in a future post.

Setup
-----

We assume that you have a working gemini installation. First, you'll need to download
a `vcfanno` binary for your system. Even if you have a previous version of `vcfanno`, you'll
need to download the version `0.0.11-beta` pre-release
[here](https://github.com/brentp/vcfanno/releases). If you're on 64-bit linux
you can grab [just the binary](https://github.com/brentp/vcfanno/releases/download/v0.0.11-beta/vcfanno).

You'll also need the new loader vcf2db:

    git clone https://github.com/quinlan-lab/vcf2db
    cd vcf2db
    conda install -y gcc snappy # install the C library for snappy
    conda install -c nlesc python-snappy=0.5
    pip install -r requirements.txt


Annotation
----------

Here, we assume that you've already annotated with VEP or snpEff and
that vcfanno-0.0.11-beta is on your path as `vcfanno`.
You'll then need to identify the path in which you installed the gemini 
annotation files. It is a directory that contains the ExAC VCF, 
the clinvar VCF, and the other annotation files used by gemini. The example `vcfanno` configuration file that we use here assumes that you have already installed gemini
annotation files. You can easily add new files to this configuration file, or replace
them with the annotations relevant to your genome build or species. This merely serves
as a guide.

Here is an example of an annotation entry for the ClinVar VCF in the `rare-disease.conf` configuration file we will provide to `vcfanno`:

    [[annotation]]
    file="clinvar_20160203.tidy.vcf.gz"
    fields=["CLNSIG", "CLNDBN"]
    names=["clinvar_pathogenic", "clinvar_disease_name"]
    ops=["self", "self"]
    
    # convert 5 to 'pathogenic', 255 to 'unknown', etc.
    [[postannotation]]
    fields=["clinvar_pathogenic"]
    op="lua:clinvar_sig(clinvar_pathogenic)"
    name="clinvar_sig"
    type="String"

The first "annotation" block tells `vcfanno` to extract the `CLNSIG` and `CLNDBN` attributes from the 
``clinvar_20160203.tidy.vcf.gz`` file. These columns will be created as INFO attributes 
in the resulting VCF and renamed as `clinvar_pathogenic` and `clinvar_disease_name`. The
second "postannotation" block creates a `derived INFO attribute called `clinvar_sig`, 
whose value is computed by a Lua function `clinvar_sig()` provided in `rare-disease.lua`.
This function converts ClinVar pathogenicity codes to human readable descriptions.

In the following code block, we download the complete `vcfanno` configuration and Lua files,
then run `vcfanno` on your input VCF using these configurations. Note that the `$GEMINI_ANNO`
environment variable should be set to the location of the installed gemini annotation files.
You will see some error messages, but those simply occur when a variant in the VCF doesn't overlap with an annotation in the annotation files.
:

    wget https://raw.githubusercontent.com/brentp/vcfanno/master/example/rare-disease.conf
    wget https://raw.githubusercontent.com/brentp/vcfanno/master/example/rare-disease.lua
    GEMINI_ANNO=/path/to/gemini/annos
    
    vcfanno -p 4 -base-path $GEMINI_ANNO -lua rare-disease.lua \
            rare-disease.conf $VCF \
            | bgzip -c > anno.vcf.gz
   
Loading
-------

Now, assuming that you have a ped file describing the samples in your study, you can load the VCF from the above annotation steps using the `vcf2db.py` script. By default, this creates a SQLite database, but you may also create a MySQL or Postgres database (see the [docs](https://github.com/quinlan-lab/vcf2db). Note: if you have gemini from github, you can remove the `--legacy-compression` tag to get a nice speedup in both loading and querying along with a decrease in database size):

	PED=/path/to/ped/file
    python vcf2db.py --legacy-compression anno.vcf.gz $PED my.db


Querying
--------

At this point, the database is gemini-compatible though some of the columns are named differently. For rare disease research, a useful query is:

    gemini auto_rec \
        --filter "max_aaf_all < 0.005 AND impact_severity != 'LOW'" \
    	--columns "chrom,start,end,gene,impact,impact_severity,clinvar_pathogenic,clinvar_disease    _name" \
        my.db


Performance
-----------

The new loader can load about 1350 variants / second on a single-core. This means it can load
an **exome trio with 20K variants in about 15 seconds.**

I ran a test load on a dataset we have with 16.4 million variants in 78 samples. It loads with
a single core in about 3 hours and the database is half the size of the one currently produced
by gemini (that took much longer with multiple CPUs to load).

The querying is also dramatically faster because of the new genotype storage format used
by the new loader and supported in gemini master branch. Running `de novo` for a single trio
out of the family of the 78 runs in 10 seconds on the new database vs 47 seconds before. This
is a worst-case scenario for gemini and we'll have further improvements here soon.

These will also translate to more modest improvements in query speed for small databases with
a few samples.


Conclusion
----------

Note that while we have used human and current gemini annotations as an example, this
will work for any diploid organism provided that:

+ It can be annotated with VEP or snpEff
+ There are useful annotations available to apply with vcfanno

We are very eager to get feedback about this workflow for other species and genome builds (e.g., GRCh38). In a future post, we will demonstrate:

+ How to use the new compression scheme we are working on for gemini
+ How to store sample genotypes, depths, in their own tables to allow custom querying by genotype in SQL syntax.
+ A new system for reproducible, community-curated, genomics data management.
