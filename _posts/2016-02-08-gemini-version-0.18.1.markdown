---
title: The future of GEMINI. 
category: blog
layout: post
comments: true
post_author: Aaron Quinlan
---

Synopsis
-------------
[GEMINI](http://gemini.readthedocs.org/en/latest/) is a software framework for exploring and prioritizing genetic variation in the the context of human disease. This post describes our software development efforts to extend GEMINI to other species and to current and future builds of the human genome.

A brief history of GEMINI
-------------------------

When I started my lab at the University of Virginia in 2011, I has the great fortune to start a collaboration with [Pat Concannon](http://diabetes.ufl.edu/research/our-team/patrick-concannon/) (now at the University of Florida) to study the genetic basis of hypersensitivity to ionizing radiation (IR). Thanks to years of effort by Pat and his longtime collaborator [Richard Gatti](http://people.healthsciences.ucla.edu/institution/personnel?personnel_id=8090), we sought out to identify new genes underlying IR among patients where were IR hypersensitive, yet had no loss of function variants in known IR sensitivity genes such as ATM or NBN. The basic plan was to exome sequence each of ~ 90 such patients, find the causal variants in each, and in so doing, find new DNA damage repair genes. **Simple, right**?

In hindsight, what we quickly found is now obvious. This is not simple at all. We started out with ad hoc scripts written by myself and Uma Paila, the first postdoctoral scientist in my then nascent lab. Because of an unfocused analysis approach that can be summarized as "VCF files plus a bunch of [bedtools](http://bedtools.readthedocs.org/en/latest/) commands", our efforts were scattered, difficult to reproduce, and therefore rather unproductive initially. I quickly realized that we desperately needed a standardized framework to integrate genetic variation with the wealth of genome annotations and reference databases that are so crucial for prioritizing genes and variants in the context of disease (e.g., imagine rare disease research without the [ExAC](http://exac.broadinstitute.org/) or [ClinVar](http://www.ncbi.nlm.nih.gov/clinvar/) resources!!!).

This now obvious insight led to the creation of our [GEMINI](http://gemini.readthedocs.org/en/latest/) software in 2011 ([manuscript](http://journals.plos.org/ploscompbiol/article?id=10.1371/journal.pcbi.1003153)). As outlined in the figure below, by placing genetic variants, sample phenotypes and genotypes, as well as genome annotations into an integrated database framework, GEMINI provides a simple, flexible, and powerful system for exploring genetic variation in the context of disease.

![gemini overview](/img/blog/gemini_overview.png){: .img-responsive .img-centered}

We ended up using the GEMINI framework to identify our first new gene underlying IR sensitivity (MTPAP; [manuscript](http://www.ncbi.nlm.nih.gov/pubmed/24651433)). We have been working pretty much non-stop on GEMINI since 2012 and it has benefited from substantial contributions from our user community. Notable effort comes from [Uma Paila](https://github.com/udp3f), [Brad Chapman](https://github.com/chapmanb), and [Rory Kirchner](https://github.com/roryk) (all authors on the original paper), as well as [Bjorn Gruning](https://github.com/bgruening), [Daniel Gaston](https://github.com/dgaston), and [Colby Chiang](https://github.com/cc2qe). We have also benefited from tremendously valuable feedback and contributions from [Jessica Chong](https://twitter.com/jxchong) and the entire [University of Washington Center of Mendelian Genomics](http://uwcmg.org/), which uses GEMINI as part of the analysis pipeline to solve Mendelian disorders.

Where are we now?
-----------------
Last June, I moved my lab to the University of Utah to be a part of the USTAR Center for Genetic Discovery, along with Mark Yandell, Lynn Jorde and Gabor Marth. I was incredibly fortunate to convince [Brent Pedersen](https://github.com/brentp) to join the lab. Brent has done yeoman's work to overhaul GEMINI to be faster, leaner, and more powerful. 


Flexible VCF annotation
-----------------------------------------------
The first exciting advance that Brent has developed is a very powerful new tool called [vcfanno](https://github.com/brentp/vcfanno), which allows one to annotate a VCF file with one or more annotations from many diverse annotation files in BED, VCF, GFF, or BAM format. One specifies a simple configuration file that defines which annotation files should be used and which fields or operations should used from those annotations files to yield new annotations in the INFO field of the resulting VCF file (for more details about this process, see the vcfanno [documentation](https://github.com/brentp/vcfanno) and our previous blog [post](http://127.0.0.1:4000/blog/2016/01/08/combine-ExAC-w-vcfanno.html). For example, the following figure illustrates how vcfanno converts a "naked" VCF file to a VCF where each variant is fully-dressed with the alternate allele frequency (``AF``) and the number of heterozygotes (``AC_Het``) from the [ExAC VCF](ftp://ftp.broadinstitute.org/pub/ExAC_release/release0.3/), as well as computing the mean GERP score observed for bases overlapping each variant. [Vcfanno](https://github.com/brentp/vcfanno) automatically updates the header and packs the new annotations into the INFO fields of the resulting VCF.

![gemini overview](/img/blog/vcfanno-overview.png){: .img-responsive .img-centered}

Creating a GEMINI database from an annotated VCF 
-------------------------------------------------
More recently, we have created [vcf2db](https://github.com/quinlan-lab/vcf2db), a new tool that takes advantage of [SQLAlchemy](http://www.sqlalchemy.org/) to allow a GEMINI-compatible database to be created DRECTLY FROM A VCF FILE! Why am I so excited about this? Well, the beauty is that so long as one can annotate one's VCF with the relevant annotations using vcfanno, one can create a GEMINI database for subsequent analysis. The vcf2db script uses the data types defined for the INFO fields in the VCF file's header to infer what the corresponding data types should be for the annotation columns that will be created in the database file. In other words, the VCF header defines, in part, the database schema! 

As a quick example, consider the ``exac_aaf``, ``exac_num_het``, and ``gerp_mean`` attributes that vcfanno places in the INFO field of the example figure above. Vcfanno also defines those attributes in VCF header as a Float, Integer, and Float, respectively. The vc2db script will therefore create three new columns in the GEMINI database's ``variants`` table called ``exac_aaf``, ``exac_num_het``, and ``gerp_mean`` using the  datatypes that are appropriate to the database backend that you are using. This is where SQLAlchemy comes in --- currently, GEMINI only reads a SQLite database, but in the future, it will work with MySQL, PostgreSQL, and other databases supported by SQLAlchemy.

And here is really exciting part: since vcf2db doesn't care about how the VCF was annotated (so long as it meets the VCF [specification](https://samtools.github.io/hts-specs/)), it can create GEMINI databases for ANY SPECIES OR ANY GENOME BUILD!. Of course, not all batteries are included, since one needs to collect the relevant annotations for your species or build of interest, but once you have done that, you are off and running!


Connecting the dots to support other species and genome builds
---------------------------------------------------------------
In summary, by decoupling the annotation of a VCF file from the loading of the GEMINI database, we are very close to a new future for GEMINI that provides support for non-human species and provides support for researchers using GRCh38 (and beyond) of the human genome. Our focus over the coming months with be to simplify and integrate all of this flexibility directly into GEMINI. For example, we plan to provide vcfanno configuration files that are pre-populated for different species and builds. This will simplify the hardest aspect of all of this --- finding, downloading, and standardizing all of the annotations that are germane to one's analysis. Stay tuned, as we are very excited about the future of GEMINI. Now to find funding to further support its future...

