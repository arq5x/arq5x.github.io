---
title: 2015 Improvements to GEMINI for rare disease research
category: blog
layout: post
post_author: Brent Pedersen
---
I've been in Aaron's lab for about 10 months now. We've made a lot of improvements
to gemini in that time. Some of these are:

+ [inheritance models support multigenerational pedigrees](https://github.com/brentp/inheritance).
+ compound het tool does phasing by inheritance to find true candidates.
+ massive speed improvements to querying (50X) and loading (2X)
+ conda-based install by Brad Chapman.
+ [improved variant prioritization](https://github.com/brentp/geneimpacts)
+ new snpEff and VEP versions are supported.

These are mostly driven by user-feedback. For example, in August, we went to the
UW Center For Mendelian Genomics workshop where Aaron gave extensive tutorials on
gemini and we also got to see experts talking about best-practices for mendelian
genetics research. In particular, Jessica Chong (who has given us much valuable
feedback on many aspects of gemini) noted that for mendelian disorders,
we expect that a causal variant should be rare in **every** population and had a
query that looked like this:

```
(aaf_esp_ea <= 0.005 or aaf_esp_ea is NULL) AND \
(aaf_esp_aa <= 0.005 or aaf_esp_aa is NULL) AND \
(aaf_1kg_amr <= 0.005 or aaf_1kg_amr is NULL) AND \
(aaf_1kg_eas <= 0.005 or aaf_1kg_eas is NULL) AND \
(aaf_1kg_sas <= 0.005 or aaf_1kg_sas is NULL) AND \
(aaf_1kg_afr <= 0.005 or aaf_1kg_afr is NULL) AND \
(aaf_1kg_eur <= 0.005 or aaf_1kg_eur is NULL) AND \
(aaf_adj_exac_afr <= 0.005 or aaf_adj_exac_afr is NULL) AND \
(aaf_adj_exac_amr <= 0.005 or aaf_adj_exac_amr is NULL) AND \
(aaf_adj_exac_eas <= 0.005 or aaf_adj_exac_eas is NULL) AND \
(aaf_adj_exac_fin <= 0.005 or aaf_adj_exac_fin is NULL) AND \
(aaf_adj_exac_nfe <= 0.005 or aaf_adj_exac_nfe is NULL) AND \
(aaf_adj_exac_sas <= 0.005 or aaf_adj_exac_sas is NULL) 
```

After seeing this, we made 2 changes.
+ We set the missing value for allele frequencies to -1 so that the or is NULL check is not required.
+ We added a new column which saves the maximum allele frequency for any of the above columns (except those from the
  fin population) or -1 if none are seen.

Those changes reduce the above query to:

```
max_aaf_all <= 0.005
```

That filter alone is extremely helpful in reducing the search space for mendelian disorders.

In 0.18.1 we also add(ed) a new column: `clinvar_gene_phenotype`. Which keeps a '|' delimited list
of any clinvar phenotypes of variants **in the same gene** as the variant. The other clinvar columns
that were also present in previous versions are reported only for the exact same variant.

As a result of these changes we were able to filter down to a **single variant** now known to be the
causal variant in a case at the University of Utah with this query:

```
gemini de_novo --filter "impact_severity != 'LOW' \
                         and max_aaf_all <= 0.005 \
                         and clinvar_gene_phenotype like '%pyruvate%'" \
                --columns "chrom,start,end,gene,impact,impact_severity" \
                $db
```

Removing any of those filters results in a sufficient number of candidates to make searching through
the list more duanting. Note the use of `clinvar_gene_phenotype` which must be lower-cased
and wrapped in '%' because a given gene could be associated with multiple phenotypes.

The query above runs in 3 seconds for a trio with nearly 8 million variants for a whole-genome dataset.

The `impact_severity` and `max_aaf_all` columns seem to be a good starting point for mendelian diseases,
regardless of the inheritance model. The above query alone is a good benchmark for the changes in usability
that we've made and we welcome comments on how to further improve the utility.

We're looking for additional ways to support variant::gene::phenotype lookups and also welcome feedback on
that [here](https://github.com/arq5x/gemini/issues/571).

Many of the other changes that we made just made things work more consistently (or correctly).

In 2016, we plan to add support for any organism (including hg38). This will require a new setup for loading annotations
which we'll achieve with [vcfanno](https://github.com/brentp/vcfanno). 
The other major change will be to allow more choices in database backends-- e.g. mysql and postrgres in addition to
sqlite.

