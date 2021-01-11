---
title: "Parallelising freebayes with snakemake"
author: "Sanjay C Nagi"
date: "11/01/2021"
output: html_document
tags:
  - bioinformatics
  - variant calling
spotifyId: "314KmbRrSXcQHFujzdOr9v"
---

[`Freebayes`](https://github.com/freebayes/freebayes) is a bayesian haplotype-based variant caller, used widely in genomics. As with many variant callers, it is not readily parallelised, but can be done so by splitting the genome into smaller chunks, calling them separately, and subsequently combining the chunks together.

A wrapper for freebayes, [`freebayes-parallel`](https://github.com/freebayes/freebayes/blob/master/scripts/freebayes-parallel), does exactly this, making use of `gnu-parallel`. However, this approach has a couple of major limitations:

1)   When a chunk is completed, that cpu core will not move onto the next region until all cores have completed their respective chunk. This is particularly problematic in regions of variable coverage, and so one can attempt to split the genome into regions of roughly equal coverage. Unfortunately, this still results in many cores being unused for substantial periods of time.

2)   It is not possible to perform joint multi-sample calling with the freebayes-parallel wrapper (!!!)

I was implementing a `freebayes` variant calling step in a snakemake RNA-Sequencing pipeline I was writing (more on this later), and wanted to parallelise freebayes, without the above limitations. 

By using an extra snakemake wildcard, the index of each genome chunk, we can produce, and supply freebayes with different regions (bed) files, and then concatenate the output files later with `bcftools concat`. The benefit of this, is that snakemake will automatically run the next job when each chunk is complete, reducing overall computation time as compared with `freebayes-parallel`. Importantly, it also allows us to perform joint, multi-sample calling, which is one of the main benefits of using freebayes in the first place. 

To generate the regions, a separate snakemake rule runs an [R script](https://github.com/sanjaynagi/rna-seq-ir/blob/master/workflow/scripts/GenerateFreebayesParams.R) to read in the genome index (.fai) file, and output a bed file which breaks the genome into chunks of equal size. Finally, after concatenating the vcfs with `bcftools concat` it is also important to stream the output through `vcfuniq`, to ensure there are no duplicate calls at the region overlaps. 

I'll leave you with an old song I've enjoyed recently. Happy variant calling.

{% include spotifySong.html id=page.spotifyId %}
