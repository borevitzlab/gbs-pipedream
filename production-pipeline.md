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

| Step                | Tool(s)     | Implementation Status             |
| ------------------- | ----------- | --------------------------------- |
| De-multiplexing     | axe         | [Implemented][Axe], pre-publication |
| Read QC             | gbsqc       | [Implemented][gbsqc], needs review |
| *In-silico* gel cut | N/A         | Needs custom tool, ANU will implement |
| Sample QC           | N/A         | Needs custom tool, ANU has ideas  |
| Variant Calling     | stacks      | Has annoying limitations          |
| Variant Calling     | TASSEL      | UNEAK deprecated                  |


De-multiplexing
---------------

Requirements:

The following features are required:

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
  100bp. Uses a Needleman-Wunsch global alignment between the two reads.
- Windowed Quality Trimmer: very similar to
  [sickle](https://github.com/najoshi/sickle). Removes low base quality reads.
- Measure read qualities *a la* FastQC post-QC for comparison.

The single-tool solution seems to be an improvement on the multi-tool pipeline
both in terms of performance and simplicity.


*In silico* gel cuts
====================

The GBS protocol has a weakness in that the size of a fragment is tied to its
location in the genome. Therefore, if the size range analyses is different
between analyses you end up assaying different sub-sections of the genome. For
*de novo* uses, this is a pretty bad thing. Fixing this *in vitro* is
relatively difficult when there are many people making libraries

<!-- References -->

[Axe]: https://github.com/kdmurray91/axe
[gbsqc]: https://github.com/kdmurray91/libqcpp
