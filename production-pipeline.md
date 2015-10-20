The Production GBS Pipeline
===========================

This describes a wish-list of features and planning of new or existing tools
and pipelines. This isn't what we're currently using, as many steps are
currently only ideas.

This has been prepared by Megan Supple and Kevin Murray.


Overview
--------

We take Illumina sequencing runs, de-multiplex them to separate files for each
sample. We then perform a bunch of QC steps on these reads, which we store on
disk permanently as interleaved FASTQ files. These are analyses in a variety of
combinations using GBS-specific variant calling pipelines.


These steps and the tools used are summarised in the following table, and
detailed in the following sections.

| Step                | Tool(s)     | Implementation Status/Issues      |
| ------------------- | ----------- | --------------------------------- |
| De-multiplexing     | axe         | [Implemented][Axe], pre-publication |
| Read QC             | gbsqc       | [Implemented][gbsqc], needs review |
| *In-silico* gel cut | N/A         | Needs custom tool, ANU will implement |
| Contamination Check | Various/New | Many tools available, few suit. See below |
| Sample QC           | N/A         | Needs custom tool, ANU has ideas  |
| Variant Calling     | stacks      | Has annoying limitations          |
| Variant Calling     | TASSEL      | UNEAK deprecated                  |


De-multiplexing
---------------

Requirements:

The following features are required in a demultiplexer for our GBS data:

- Combinatorial barcodes
- Barcodes of differing length
- Interleaved output

Options:

- Axe: Fast, supports combinatorial barcodes of differing length
- flexbar: Slow, seems inaccurate, doesn't handle differing length barcodes

Our choice:

- We implemented Axe as we couldn't find anything that suited our needs


Read QC
-------

Kevin has a C++ library to do various read QC things, and has made a tool
called [gbsqc](https://github.com/kdmurray91/libqcpp) specifically for GBS
reads.

Steps:

- Measure read qualities *a la* FastQC
- Trim/Merge: Merge reads from fragments between ~100bp and ~190bp (with 100bp
  reads) into a single read, trim adaptors from reads w/ fragment size &lt;
  100bp. Uses a Needleman-Wunsch global alignment between the two reads. (NB:
  this requires paired end reads, and by our calculation, 100bp paired end is
  optimal, though any reasonable length would work)
- Windowed Quality Trimmer: very similar to
  [sickle](https://github.com/najoshi/sickle). Removes low base quality reads.
- Measure read qualities *a la* FastQC post-QC for comparison.

The single-tool solution seems to be an improvement on the multi-tool pipeline
both in terms of performance and simplicity.


*In silico* gel cuts
====================

This "tool" is throughly in air-quotes for now. The rationale and basic idea
follows.

The GBS protocol has a weakness in that the size of a fragment is tied to its
location in the genome. Therefore, if the size range analyses is different
between analyses you end up assaying different sub-sections of the genome. For
*de novo* uses, this is a pretty bad thing. Fixing this *in vitro* is
relatively difficult when there are many people making libraries. So, an *in
silico* method would be better.

By merging read pairs into fragments, we can tell the exact length of the
fragment, and therefore select the size range of fragments computationally.
This means that we can always assay the same size range, even across lanes with
different actual size ranges.


Contamination Check
===================

There may be many unintended things in your reads. Checking for contamination
(any reads not from your species of interest) is worthwhile. However, many
tools rely on either massive databases (often loaded in RAM), or blasting
assembled contigs against massive databases (which we don't have at this
point).

This needs some thought. One possible option is to ignore contamination for now
and blast the tags as assembled by e.g. stacks. Then, you can either, a), go
all in and run Kraken or similar on the reads to remove contaminants, or b),
just remove the loci that look like contaminants.

On the other hand, things like quick GC plots or K-mer composition scores could
show you if you have a large problem with contamination. This is made difficult
by the bacterial and mitochondrial genomes. And would only work at the kingdom
level.

A slightly related note would be the filtering of low complexity reads, i.e.
reads with polymeric repeats (AAAAAA, ATATAT etc.). This is not too difficult
from a compositional basis, one for example can filter low entropy reads out.


Sample filtering
================

We would like a way of filtering out bad samples before any variant calling.
Samples with too few reads, or with either to much contamination or low
complexity reads.


Variant Calling
===============

Nearly all of our GBS is *de novo*, so we need a reference-free method of
assembling tags. This basically means a choice between ustacks and tassel's
UNEAK. There are issues with both:

### Uneak Issues:

 - Limited to 64bp (well, only looks at 64bp)
 - Limited to 1 SNV per 64bp locus
 - Fairly monocot-specific filtering (developed for switchgrass by maize
   people)
 - Arcane hacks required to accept pre-demulitplexed or pre-qc'd files
 - Doesn't output a proper VCF

### Ustacks issues:

 - Doesn't handle reads of varying lengths.
 - Doesn't output a proper VCF or a list of **all** assembled tag pairs

<!-- References -->

[Axe]: https://github.com/kdmurray91/axe
[gbsqc]: https://github.com/kdmurray91/libqcpp
