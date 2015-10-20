The Production GBS Pipeline
===========================

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
specifically for GBS reads.

Steps:

- Measure read qualities *a la* FastQC
- Trim/Merge: Merge reads from fragments between ~100bp and ~190bp (with 100bp
  reads) into a single read, trim adaptors from reads w/ fragment size &lt;
  100bp. Uses a Needleman-Wunsch global alignment between the two reads.
- Windowed Quality Trimmer: very similar to
  [sickle](https://github.com/najoshi/sickle). Removes low base quality reads.

<!-- References -->

[Axe]: https://github.com/kdmurray91/axe
[gbsqc]: https://github.com/kdmurray91/libqcpp
