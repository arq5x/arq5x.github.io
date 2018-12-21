---
title: A map of constrained coding regions (CCRs) in the human genome. 
category: blog
layout: post
comments: true
post_author: Aaron Quinlan
---

> **Once in a while you get shown the light  
In the strangest of places if you look at it right.**  
> - Jerry Garcia and Robert Hunter, *Scarlet Begonias*


> **The unseen enemy is always the most fearsome.**
>
> - George R.R. Martin, *A Clash of Kings*

>

![ccr overview](/img/blog/constraint_concept.png){: .img-responsive}

>

Summary
==========
In this post, I provide background and an overview of our recent manuscript, entitled [A map of constrained coding regions in the human genome](https://www.nature.com/articles/s41588-018-0294-6.epdf?author_access_token=N8RkVYcavtplcSSE2KJkNdRgN0jAjWel9jnR3ZoTv0OG-o3UTB17cpoBs8B6XMBCl-5E0ZvpOii0iPl_hRSMGjWfkG4em7gjy95eV4bkTb0AF-E_dj3obeJfaadTja3aj9hUh1xk_BIztWgJScFx8w%3D%3D). Briefly, we studied genetic variation detected among >120,000 human exomes from version 2.0.1 of the [Genome Aggregation Database](http://gnomad.broadinstitute.org) (gnomAD) to reveal focal coding regions that are *constrained* owing to an atypical dearth of variation (e.g., in the cartoon above). These "constrained coding regions" (CCRs) are inferred to be under strong purifying selection and are enriched for known pathogenic variants. **Perhaps the most intriguing aspect of this map of CCRs is the fact than many of the most constrained regions lie within genes lacking a prior disease association. These regions hold the promise of new disease gene discovery in the context of developmental disorders.**

>

What are the most constrained regions of the human genome?
============================================================
A longstanding interest in human genetics is identifying the subset of our genome that is the most essential to life and healthy development. Generally speaking, such regions should be under the highest purifying selection and should therefore exhibit decreased nucleotide diversity. In the case of protein-coding genes, especially strong "constraint" should be observed against protein-altering (i.e., missense, stop-gain, frameshift, etc.) variants. Indeed, this concept underlies the motivation behind recent, "gene-wide" metrics of constraint such as the [Residual Variation Intolerance Score](https://journals.plos.org/plosgenetics/article?id=10.1371/journal.pgen.1003709) (RVIS) and the more recent [probability of Loss-of-function Intolerance](https://www.nature.com/articles/nature19057) (pLI) score. While these metrics have proven incredibly useful to studies of rare disease, single, gene-wide metrics inherently cannot describe the regional variation in constraint that exists within each protein-coding gene. **Identifying focal regions of constraint is the motivation of our [manuscript](https://www.nature.com/articles/s41588-018-0294-6.epdf?author_access_token=N8RkVYcavtplcSSE2KJkNdRgN0jAjWel9jnR3ZoTv0OG-o3UTB17cpoBs8B6XMBCl-5E0ZvpOii0iPl_hRSMGjWfkG4em7gjy95eV4bkTb0AF-E_dj3obeJfaadTja3aj9hUh1xk_BIztWgJScFx8w%3D%3D).** In the post below, I highlight the history of the manuscript, the key results and data files, as well as our views on important future studies. 

>

Background
==========

Sometime in late 2010, as I was nervously preparing to start my own research group (I cannot even begin to convey how utterly scared I was), I saw a talk describing the [NHLBI Exome Sequencing Project's](https://esp.gs.washington.edu/drupal/) goal to sequence ~6,000 human exomes. It quickly dawned on me (and likely many others) that we could leverage the variation found in these exomes to infer coding regions that are under purifying selection from the *absence* of variation in these samples. Multiple fun discussions with the brilliant [Bill Pearson](http://www.people.virginia.edu/~wrp/) helped to focus the research. Soon after, [Jim Havrilla](https://twitter.com/jim_havrilla) joined my lab and quickly latched on to this idea for his doctoral thesis. 

**Fast forward EIGHT years**. This kernel of an idea catalyzed the creation of our [map of constrained coding regions in the human genome](https://www.nature.com/articles/s41588-018-0294-6.epdf?author_access_token=N8RkVYcavtplcSSE2KJkNdRgN0jAjWel9jnR3ZoTv0OG-o3UTB17cpoBs8B6XMBCl-5E0ZvpOii0iPl_hRSMGjWfkG4em7gjy95eV4bkTb0AF-E_dj3obeJfaadTja3aj9hUh1xk_BIztWgJScFx8w%3D%3D). This work was led throughout by Jim Havrilla, but benefited from clever ideas, slick code, and advice from [Brent Pedersen](https://twitter.com/brent_p) and [Ryan Layer](https://twitter.com/ryanlayer).

![survivorship](https://upload.wikimedia.org/wikipedia/commons/thumb/9/98/Survivorship-bias.png/640px-Survivorship-bias.png)

As we describe in the manuscript's introduction, this idea is based upon the concept of [survival bias](https://en.wikipedia.org/wiki/Survivorship_bias), which is pervasive in science and is most famously demonstrated in the work of Abraham Wald and the Statistical Research Group (SRG) during WWII. Here's the scenario. Allied planes are being shot down left and right and military leadership obviously wants to slow this down. However, metal is scarce. Moreover, while adding metal further protects the plane, it also makes it less maneuverable and less fuel efficient. This is a classic optimization problem - how could they use the least amount of metal while maximizing defenses? The story goes that the SRG was presented with data describing the bullet hole patterns observed from hundreds of planes that returned from their sorties (for example, the hypothetical figure that I borrowed from Wikipedia below).

Military leadership is said to have interpreted this data to imply that armor should be placed where the bullet holes are the densest (this is where we are being shot!). **Wald famously disagreed. He realized that the observed data were biased by the fact that they came solely from the planes that *returned* (survived).** He reasoned that armor should instead be placed where the bullets are *not*, as these regions are likely where the shot down planes had taken damage; that is, **constrained flying regions**.


>

Brief Summary
=============

Similarly, we used survival bias to identify constrained (i.e., under strong purifying selection) coding regions (CCRs) in the human genome from the *absence* of protein-changing variation among >120,000 human exomes.

I like analogies. A lot. So, here is a video describing another based upon rainfall on a sidewalk.

>

<iframe width="640" height="360" src="https://www.youtube.com/embed/KGJYfw_V0wo" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

>

The analogy of unexpected patterns of constraint being revealed by thousands of raindrops (genetic variants) led to the concept for the cover of the current issue of Nature Genetics!

>

![ng cover](/img/blog/ng_cover_JAN19-web_1.png){: .img-responsive }

>

Key Results
=============

As detailed in the manuscript, we identified constrained coding regions as segments of protein-coding genes that lack even a single protein-altering variant among the >120,000 exomes in [Genome Aggregation Database](http://gnomad.broadinstitute.org) (gnomAD). Whereas the average density of such variation in gnomAD is approximately 1 in 7 coding bases, the most constrained coding regions (e.g., at or above the 99th percentile) **often lack protein-altering variants for more than  100 bases**. For example, the red regions below reflect CCRs we identified at the 95th percentile and higher in *KCNQ2* and *TNNT2*.

>

<img width="640" src="https://images.readcube-cdn.com/publishers/nature/figures/a088277ecf9b0a14920c7935ca2a690c3a216096dce064ce7de9481c828b5ae0/1.jpg">

>

As a positive control, we demonstrate that, the most constrained coding regions are enriched for pathogenic variants in [ClinVar](https://www.ncbi.nlm.nih.gov/clinvar/) that are known to underlie rare human disease phenotypes. For example, one of the most constrained regions is a 274 coding base pair region devoid of protein-altering variation in *SCN8A*. The 4 exons comprising this CCR encode the bulk of the ion transport domain. Below is a screen shot from our [CCR browser](https://s3.us-east-2.amazonaws.com/ccrs/ccr.html) built using [IGV.js](https://github.com/igvteam/igv.js/wiki). The image is a bit difficult to make out, so can view the region directly with [this link](https://s3.us-east-2.amazonaws.com/ccrs/ccr.html#12:52,179,723-52,189,901). Dark red regions reflect CCRs at or above the 99th percentile.

![scn8a](/img/blog/scn8a.png){: .img-responsive }

Similarly, we find that CCRs complement other variant prioritization tools for the interpretation of *de novo* mutation in the context of rare disease. We argue that *de novo* mutations lying within the most constrained (e.g., 99th percentile and higher) coding regions are likely to be involved in developmental phenotypes. In fact, while it didn't make it into the manuscript, **almost all of the pathogenic mutations we identified in our [recent study](https://www.ncbi.nlm.nih.gov/pubmed/30109124) of early infantile epileptic encephalopathy lie within CCRs at or above the 95th (and most above the 99th) percentile.**

We therefore argue that *de novo* mutations lying within regions of the highest constraint are of particular interest in the context of developmental disorders. An important caveat, however, is that mutations lying within less constrained regions should be ignored, owing to the fact than many pathogenic alleles lie within regions with dense variation (e.g., BRCA1).  

Another fun result is the observation that intra-species constraint often complements inter-species conservation measurements; that is, conservation does not always predict intra-species constraint. Furthermore, we identify a subset of protein domain families that harbor the greatest constraint. From a high level, these domains typically interact with DNA or modify chromatin. Most detail about constraint in these domains is provided in a preprint from [Boukas et al](https://www.biorxiv.org/content/early/2018/05/07/219097).

>


New Disease Genes?
==================
Given the enrichment of high CCRs for known pathogenic variants, the result we are most excited by is the fact that many highly (>99%) constrained regions lie within genes lacking prior disease association. Certainly some of these are false positives. **However, we hypothesize that some of these regions reflect strong purifying selection and when mutated, they will lead to developmental phenotypes, or even embryonic lethality.**

We are excited to explore these regions in future studies and hope the map of constrained coding regions will help to guide future research in the community and empower mutation interpretation in rare disease studies. We are excited to see that this is indeed already happening (see [Jensen et al](https://www.biorxiv.org/content/biorxiv/early/2018/11/02/459958.full.pdf), [Wray et al](https://www.sciencedirect.com/science/article/pii/S0092867418307141), and [Boukas et al](https://www.biorxiv.org/content/early/2018/05/07/219097))!

>

Caveats
=======
The elegance of our approach to identifying constrained coding regions (CCRs) is that it is very simple. However, it is intentionally very strict, as we wanted to minimize false positive predictions. Admittedly, "breaking" constrained regions based upon the presence of a single protein-altering variant in gnomAD may lead to false negatives; that is, larger regions of constraint disrupted by a single variant. We emphasize that the map we have created reveals constrained regions under a dominant model, and is not well-suited to recessive constraint. Finally, as powerful as gnomAD is, it is primarily comprised of variation from individuals of European ancestry. As such, it is unclear to what degree our map properly models constrained regions in other ancestries. 

>

The future
===========
What next?  In the next couple of years, a truly massive number of human genomes will be sequenced. Furthermore, thanks to gnomAD and other efforts, there is an exciting commitment to data sharing in human genomics. Therefore, our hope is that this research, as well as [similar ideas](https://www.biorxiv.org/content/early/2017/06/12/148353) from [Kaitlin Samocha](https://twitter.com/ksamocha), launches new approaches to isolating the critical regions of our genome. We anticipate that variation from many more human genomes will increase the resolution and accuracy of regions predicted to be under strong purifying selection. Similarly, we are rapidly approaching datasets including genome-wide variation from >100,000 genomes thanks to [gnomAD]((http://gnomad.broadinstitute.org)), [Genomics England](https://www.genomicsengland.co.uk/), [TopMED](https://www.nhlbiwgs.org/), and the [Centers for Common Disease Genomics](https://www.genome.gov/27563570/centers-for-common-disease-genomics-ccdg/). These datasets hold the intriguing promise of modeling human constraint in non-coding regions of our genome. 

**We will continue to update our map of CCRs with future versions of gnomAD and perhaps other resources. We also look forward to new approaches that will arise and are eager to continue our research in this area. Stay tuned.**

>

Gratitude
=========
We are proud [research parasites](http://researchparasite.com/). This study would have been impossible without the truly heroic efforts of the team behind the [Genome Aggregation Database](http://gnomad.broadinstitute.org), and its predecessor, the [Exome Aggregation Consortium](http://exac.broadinstitute.org). We are incredibly grateful to [Daniel MacArthur](https://twitter.com/dgmacarthur), [Konrad Karczewski](https://twitter.com/konradjk), [Monkol Lek](https://twitter.com/theFourier2k) and the rest of the team. **Assembling this resource has been invaluable to human genetics. Thank you.** We also thank Kyle Vogan (Nature Genetics) and the anonymous referees that provided invaluable feedback during the review process. Their feedback and the subsequent discussions helped to greatly improve the clarity and accuracy of our manuscript.

>

Constrained Coding Region (CCR) Resources
==========================================
- [Documentation](https://quinlan-lab.github.io/ccr/)
- [The CCR browser](https://s3.us-east-2.amazonaws.com/ccrs/ccr.html) built using IGV.js)
- [Autosomal Constrained Coding Regions in BED format](https://s3.us-east-2.amazonaws.com/ccrs/ccrs/ccrs.autosomes.v2.20180420.bed.gz)
- [X Chromosome Constrained Coding Regions in BED format](https://s3.us-east-2.amazonaws.com/ccrs/ccrs/ccrs.xchrom.v2.20180420.bed.gz)
- [Autosomal CCRs >=90th percentile in BED format](https://s3.us-east-2.amazonaws.com/ccrs/ccrs/ccrs.autosomes.90orhigher.v2.20180420.bed.gz)
- [X Chromosome CCRs >=90th percentile in BED format](https://s3.us-east-2.amazonaws.com/ccrs/ccrs/ccrs.xchrom.90orhigher.v2.20180420.bed.gz)

