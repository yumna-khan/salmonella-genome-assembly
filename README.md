# salmonella-genome-assembly
## Introduction
Genome assembly reconstructs a complete DNA sequence from shorter sequencing reads, enabling the study of an organism’s genetic content (Koren et al., 2013). Since many sequencing platforms cannot read an entire chromosome in a single read, DNA is fragmented and sequenced multiple times to ensure sufficient coverage across the genome (Wick et al., 2023). However, repetitive regions, sequencing errors, and coverage gaps can complicate this process. When repeats are longer than reads, assemblies may become fragmented or misassembled, limiting the usefulness of the final genome for downstream analyses (Koren et al., 2013). This study aims to assemble the _Salmonella enterica_ genome using long read sequencing and evaluate the result by aligning it to a reference genome.

To address the challenges of repetitive regions, early assembly strategies utilized paired end sequencing to bridge gaps; however, these methods were often limited by fragment length and increased experimental complexity (Koren et al., 2013). Modern advances in long read platforms like Oxford Nanopore Technologies (ONT) and Pacific Biosciences (PacBio) have overcome these challenges by generating reads long enough to span repetitive regions (Wick et al., 2025b), enabling the complete assembly of bacterial genomes into single contigs per replicon (Wick et al., 2021). While assemblers like Canu and Flye are specifically designed for these datasets, long read assembly is not without limitations; different assemblers can produce varying results, including potential issues with missing plasmids, extra contigs, or incomplete circularization (Wick et al., 2025b).

Among long read assemblers, Flye and Canu represent two prominent yet distinct approaches. Flye utilizes repeat graphs to resolve unbridged repeats and generally offers superior speed and accuracy for bacterial ONT data compared to Canu, which, while highly contiguous, can be slower and less effective at resolving near identical repeats (Kolmogorov et al., 2019). Despite Flye’s strengths, achieving optimal base level accuracy in highly repetitive regions often requires post assembly polishing to correct residual errors. Following assembly, comparing the resulting consensus to a high quality reference genome is essential for evaluating both genomic differences and assembly quality. However, the assembly and reference genome can have different structures (such as inversions or genomic island movements), causing aligners to produce "clipped" or chimeric mappings (Magoc et al., 2013). While sequence alignment can reveal SNPs, indels, and structural variants (SVs), these differences may reflect true biological variation rather than technical errors (Magoc et al., 2013), making the selection of alignment and variant calling tools critical.

To address these challenges, Minimap2 provides high performance alignments optimized for long read data (Li et al., 2018). While it offers significant speed advantages over tools like NGMLR (Sedlazeck et al., 2018), this efficiency can occasionally reduce sensitivity in highly divergent genomic regions. To mitigate the risk of false positives, a dual validation strategy is ideal: read based tools like Clair3 offer high precision SNP detection (Wick et al., 2025a), but struggle with large structural variants and regions with low coverage or high complexity. Assembly based approaches like paftools provide superior precision for structural comparison over traditional pipelines like MUMmer (Li et al., 2018), but may miss small variants and can be affected by assembly errors. By utilizing both methods, researchers can simultaneously detect small scale variants and perform large scale structural validation. These findings are ultimately finalized through the Integrative Genomics Viewer (IGV), which allows for the inspection of complex regions and the verification of contig joins (Wick et al., 2023).

In this study, Oxford Nanopore reads will be assembled into a consensus genome for _Salmonella enterica_ using Flye. The resulting consensus will be aligned to an NCBI reference genome using Minimap2, followed by integrated variant calling and visualization.

## Methods
### Data Description
Raw ONT sequencing reads (FASTQ format) for _Salmonella enterica_ were obtained from the NCBI Sequence Read Archive (SRA) under accession SRR32410565. The dataset was generated using R10 chemistry, providing an estimated base quality of Q20+ and an N50 read length of 5–15 kb. For the reference based analyses, the complete genome assembly for _Salmonella enterica_ (ASM694v2) was retrieved from the NCBI RefSeq database in FASTA format.

### Quality Control
Initial read quality and length distributions were assessed using NanoPlot (v1.42.0) to determine the necessity of quality based filtering (De Coster et al., 2018). Following this assessment, raw reads were processed with Filtlong (v0.2.1) to remove short and low accuracy sequences. Filtering was performed using a minimum length threshold of 1,000 bp and a target weight of 90% to prioritize the highest quality reads, a strategy shown to improve the efficiency of downstream _de novo_ assembly (Wick et al., 2023).

### Genome Assembly
Reads were assembled using _de novo_ Flye (v2.9.6), a long read assembler optimized for error-prone ONT data (Kolmogorov et al., 2019), with the --nano-raw option, and a genome size of approximately 5 Mb (Megabases). Default parameters were used unless otherwise specified.

### Alignment
The assembled genome was aligned to the reference genome using Minimap2 (v2.30). The -x asm5 preset was used to optimize the alignment of closely related assemblies with a divergence of <5% (Li, 2018). For visualization and downstream read based analysis, raw reads were independently mapped to the reference using the -ax map-ont preset. Resulting SAM files were converted to compressed BAM format, sorted, and indexed using SAMtools (v1.19).

### Variant calling
Variants between the assembly and reference were identified using paftools.js. This processes PAF alignments to call SNPs and small indels, allowing direct comparison between the assembled and the reference genome (Li, 2018). To validate the assembly, variants were also called from the raw read alignments using Clair3 (v1.0.10).

### Visualization
Final validation was performed using the IGV (v2.19.7). The ASM694v2 reference, the Flye assembly BAM, and the read alignment BAM were loaded simultaneously. This allowed for the visual inspection of alignment consistency across the genome, the verification of contig junctions, and systematic evaluation of discrepancies between assembly and read based variant calls (Wick et al., 2023).





