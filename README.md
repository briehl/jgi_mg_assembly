
# JGI Metagenome Assembly in KBase
---

A [KBase](https://kbase.us) module generated by the [KBase SDK](https://github.com/kbase/kb_sdk).

This module implements the JGI metagenome assembly pipeline in KBase. This implementation is still pending some testing and refinement, but works as follows:

### Current limitations
This version only works on Paired End Illumina reads. single end reads or reads from other platforms will likely fail in strange or spectacular ways.

### Input:
A single paired end reads library composed of sequences read from an Illumina machine.

### Output:
An assembled set of contigs, as an Assembly object, along with a brief report on the results.

### Computational Steps:
1. **BBTools RQCFilter** (see the KBase-wrapped [BBTools](https://github.com/kbaseapps/BBTools) or the [App Catalog page](https://appdev.kbase.us/#appcatalog/app/BBTools/RQCFilter/beta) for details).
This produces a library of reads that pass an automated quality control, removes reads that match a standard set of contaminants, trims adapters and linkers.
2. **BFC version r181** ([paper](https://www.ncbi.nlm.nih.gov/pubmed/25953801), [Github](https://github.com/lh3/bfc))
This corrects sequencing errors on short reads generated by Illumina sequencing technology. Uses parameters `-1 -k 21 -t 10`
3. **seqtk** ([Github](https://github.com/lh3/seqtk))
Seqtk is a toolkit for managing FASTA and FASTQ files. It's used here to remove orphan single end reads from the paired end library, after filtering and error correction. Used with the `dropse` subcommand.
4. **SPAdes version 3.12.0** ([paper](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC3342519/), [website](http://cab.spbu.ru/software/spades/))
This does the work of assembling the reads. Uses parameters `--only-assembler -k 33,55,77,99,127 --meta -t 32 -m 2000`. Therefore, this
uses metaSPAdes with k-mers values of 33, 55, 77, 99, and 127. A caveat here is that if your reads (after filtering and correcting) have an
average length that's less than any of those k-mer values, that k-mer value won't be used. E.g., if your corrected reads have an average length
of 100, the k=127 pass will be skipped.
5. **BBTools stats version 38.19** ([website](https://jgi.doe.gov/data-and-tools/bbtools/))
Two more tools are used from BBTools at the end. The first is `stats.sh` to build the assembly statistics. Run with flag `format=6`.
6. **BBTools bbmap version 38.19**
BBMap is finally used at the end to build a coverage map of reads mapped onto the finalized assembly, and generate coverage statistics. Uses default parameters, except for the flag `ambiguous=random`.

### Next steps
1. Add flexibility to auto-detect and adapt to single end reads. This mainly means doing the detection at app start, and adjusting a few pipeline parameters in the process.
2. Add an option to use Quast to show a deeper report on assembly quality.
3. Add an option to use Qualimap to show a deeper report on the generated reads mapping to assembly.
