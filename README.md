# Salmonella enterica Genome Assembly
# General Overview
# Introduction
Genome assembly reconstructs a complete DNA sequence from shorter sequencing reads, enabling the study of an organism’s genetic content (Koren et al., 2013). Since many sequencing platforms cannot read an entire chromosome in a single read, DNA is fragmented and sequenced multiple times to ensure sufficient coverage across the genome (Wick et al., 2023). Despite advances in sequencing technologies, genome assembly remains challenging due to repetitive regions, sequencing errors, and uneven coverage. In particular, repeats longer than individual reads can result in fragmented or misassembled genomes, limiting their utility for downstream comparative and functional analyses (Koren et al., 2013). 

These challenges are especially relevant for bacterial pathogens such as _Salmonella enterica_, a leading cause of foodborne illness in the United States (Han et al., 2024). The severity of these infections varies significantly between strains, driven largely by the unique genetic makeup of specific serovars. Characterizing these genetic features is essential for understanding pathogen evolution, antimicrobial resistance, and adaptation to environmental pressures (Han et al., 2024). Therefore, this study aims to reconstruct the _Salmonella enterica_ genome using long-read sequencing and to evaluate assembly quality through comparison with a reference genome.

To address the challenges of repetitive regions, early assembly strategies utilized paired end sequencing to bridge gaps; however, these methods were often limited by fragment length and increased experimental complexity (Koren et al., 2013). Modern advances in long read platforms like Oxford Nanopore Technologies (ONT) and Pacific Biosciences (PacBio) have overcome these challenges by generating reads long enough to span repetitive regions (Wick et al., 2025b), enabling the complete assembly of bacterial genomes into single contigs per replicon (Wick et al., 2021). While assemblers like Canu and Flye are specifically designed for these datasets, long read assembly is not without limitations; different assemblers can produce varying results, including potential issues with missing plasmids, extra contigs, or incomplete circularization (Wick et al., 2025b).

Among long read assemblers, Flye and Canu represent two prominent yet distinct approaches. Flye utilizes repeat graphs to resolve unbridged repeats and generally offers superior speed and accuracy for bacterial ONT data compared to Canu, which, while highly contiguous, can be slower and less effective at resolving near identical repeats (Kolmogorov et al., 2019). Despite Flye’s strengths, achieving optimal base level accuracy in highly repetitive regions often requires post assembly polishing to correct residual errors. Following assembly, comparing the resulting consensus to a high quality reference genome is essential for evaluating both genomic differences and assembly quality. However, the assembly and reference genome can have different structures (such as inversions or genomic island movements), causing aligners to produce "clipped" or chimeric mappings (Magoc et al., 2013). While sequence alignment can reveal SNPs, indels, and structural variants (SVs), these differences may reflect true biological variation rather than technical errors (Magoc et al., 2013), making the selection of alignment and variant calling tools critical.

To address these challenges, Minimap2 provides high performance alignments optimized for long read data (Li et al., 2018). While it offers significant speed advantages over tools like NGMLR (Sedlazeck et al., 2018), this efficiency can occasionally reduce sensitivity in highly divergent genomic regions. To mitigate the risk of false positives, a dual validation strategy is ideal: read based tools like Clair3 offer high precision SNP detection (Wick et al., 2025a), but struggle with large structural variants and regions with low coverage or high complexity. By utilizing both methods, researchers can simultaneously detect small scale variants and perform large scale structural validation. These findings are ultimately finalized through the Circos plot and bar plot, which allows for the inspection of overall assembly regions and the structural variants per gene.

In this study, Oxford Nanopore reads will be assembled into a consensus genome for _Salmonella enterica_ using Flye. The resulting consensus will be aligned to an NCBI reference genome using Minimap2, followed by integrated variant calling and visualization.


# Methods
## 1. Data Description
Raw ONT sequencing reads (FASTQ format) for _Salmonella enterica_ were obtained from the NCBI Sequence Read Archive (SRA) under accession SRR32410565. The dataset was generated using R10 chemistry, providing an estimated base quality of Q20+ and an N50 read length of 5–15 kb. For the reference based analyses, the complete genome assembly for _Salmonella enterica_ (ASM694v2) was retrieved from the NCBI RefSeq database in FASTA and GTF/GFF format.

## 2. Quality Control
Initial read quality and length distributions were assessed using NanoPlot (v1.42.0) to determine the necessity of quality based filtering (De Coster et al., 2018). Given the high read quality (Q20+) and consistent length distribution, additional filtering was unlikely to improve assembly accuracy. Moreover, strict filtering can disproportionately remove low coverage or shorter reads originating, potentially reducing assembly completeness.

## 3. Genome Assembly
Reads were assembled using de novo Flye (v2.9.6), a long read assembler optimized for error-prone ONT data (Kolmogorov et al., 2019); with the use of the `--nano-hq` option, and a genome size of approximately 5 Mb (Megabases) (`--genome-size 5m`). Note, `--nano-hq` was chosen over `--nano-raw` due to the high quality reads of Q20+. Moreover, QUAST (v5.2.0) was selected to assess contiguity, consensus accuracy, and large scale structural consistency with the reference genome, which is critical for evaluating the reliability of downstream variant and structural analyses (Gurevich et al., 2013). 

## 4. Alignment
Two alignment steps were performed to assess different aspects of data quality and assembly accuracy. The raw Oxford Nanopore data was aligned to the reference genome using Minimap2 (v2.28) with the `-ax map-ont` preset, which is optimized for the error profile of ONT reads, and is recommended for raw ONT read alignment (Li et al., 2018). Resulting SAM files were converted to compressed BAM format, sorted, and indexed using SAMtools (v1.22.1). Alignment statistics, including mapping rate, depth of coverage, and read level quality metrics, were obtained using SAMtools flagstat and depth to confirm sufficient coverage for assembly. An average depth of **154x** was observed.

Furthermore, to verify structural variation and assembly analysis, the assembled genome was aligned to the reference genome using Minimap2 with the preset `-ax asm5`. This preset was selected because it is optimized for alignment of assembled sequences with low divergence (≤5%) (E Pearce, 2021). 


## 5. Variant calling
To compute variant calling, the raw read alignments used Clair3 (v1.1.2). The contigs were generated using the BAM file, and the `--platform ont` parameter was chosen to specify the nanopore data. In addition, the `--include_all_ctgs` option was selected to ensure that all contigs from the Flye assembly were considered, including any small or low coverage contigs for downstream comparative analyses.

## 6. Visualization
Final validation was performed using Circos (v0.69-9) and bar plots generated in R (v4.5.2). Circos was used in combination with MUMmer (v4.0.0rc1) and BCFtools (v1.19) to assess structural relationships between the assembled contigs and the reference genome. Alignments were generated using Nucmer and processed with `show-coords`, filtering for high confidence matches with ≥90% sequence identity and alignment lengths ≥10 kb (Jain, 2018). These thresholds were chosen to retain meaningful alignments while excluding short or low identity matches that may represent repetitive or ambiguous regions. 

Reference and assembly sequence lengths were indexed using samtools faidx to generate karyotype files required for Circos visualization. Circos plots were configured to emphasize genome wide synteny, and additional plots were produced to compare filtered versus unfiltered alignments as well as to visualize the genomic distribution of SNPs and indels.

Lastly, a barplot was created to view specific genes containing structural variants of indels or SNPs. This analysis was performed in R using the reference genome annotation (GFF) and variant calls (VCF). To improve interpretability, only genes containing two or more variants were included, allowing emphasis on genes with potentially meaningful levels of variation.

# Results

**Table 1:** Summary of sequencing, assembly, and alignment quality
| Category                    | Metric                     | Value |
| --------------------------- | -------------------------- | ----- |
| **Sequencing (NanoPlot)**   | Total yield (Mb)           | ~809  |
|                             | Estimated coverage (x)     | >150x |
|                             | Median read quality (Q)    | 23.7  |
| **Assembly (Flye / QUAST)** | Number of contigs          | 3     |
|                             | Total assembly length (Mb) | ~4.7  |
|                             | N50 (Mb)                   | ~4.6  |
|                             | Mismatches per 100 kbp     | 27    |
| **Alignment (SAMtools)**    | Reads mapped (%)           | 94.3  |
|                             | Average depth (×)          | ~154  |

**Table 2:** Genes variants counts, location, and function
| Gene               | Genomic location       | SNP count | Indel count | Functional annotation                                    |
| ------------------ | ---------------------- | --------- | ----------- | -------------------------------------------------------- |
| **pSLT108 (traT)** | pSLT virulence plasmid | 755      | 41         | Outer membrane lipoprotein involved in surface exclusion |
| **STM1022**        | Chromosome             | 103      | 68         | Chaperone-associated protein involved in stress response |



<img width="3000" height="3000" alt="Image" src="https://github.com/user-attachments/assets/c2a59926-7fb1-4bf9-879f-2e15a4bb0a81" />

**Figure 1:** Circos visualization of alignments between the assembled contigs and the _Salmonella enterica_ reference genome (ASM694v2) after **filtering**. The reference chromosome is shown in grey, with assembled contigs shown in blue and orange. 



<img width="3000" height="3000" alt="Image" src="https://github.com/user-attachments/assets/7e3e69c0-3d10-4270-aeb0-b253244e9878" />
**Figure 2:** Circos visualization of unfiltered alignments between the assembled contigs and the _Salmonella enterica_ reference genome (ASM694v2). The reference is shown in grey, with contigs shown in blue, orange, and red. 


<img width="3000" height="3000" alt="Image" src="https://github.com/user-attachments/assets/8475304f-c524-43a7-93e2-7e95ed22f1a8" />
**Figure 3:** Circos plot showing the distribution of SNPs (blue) and indels (red) relative to the Salmonella enterica reference genome (grey).


<img width="960" height="488" alt="Image" src="https://github.com/user-attachments/assets/20ac8116-bdee-40e2-8d3c-bc19abf982ee" />
**Figure 4:** Bar plot showing the number of SNPs and indels per gene. Only genes containing two or more variants are shown to emphasize loci with elevated sequence variation.


# Discussion


# References
De Coster, W., D’Hert, S., Schultz, D. T., Cruts, M., & Van Broeckhoven, C. (2018). NanoPack: visualizing and processing long-read sequencing data. Bioinformatics, 34(15), 2666–2669. https://academic.oup.com/bioinformatics/article/34/15/2666/4934939

Kolmogorov, M., Yuan, J., Lin, Y., & Pevzner, P. A. (2019). Assembly of long, error-prone reads using repeat graphs. Nature Biotechnology, 37(5), 540–546. https://doi.org/10.1038/s41587-019-0072-8

Koren, S., Harhay, G. P., Smith, T. P., Bono, J. L., Harhay, D. M., Mcvey, S. D., Radune, D., Bergman, N. H., & Phillippy, A. M. (2013). Reducing assembly complexity of microbial genomes with single-molecule sequencing. Genome Biology, 14(9), R101. https://doi.org/10.1186/gb-2013-14-9-r101

Li, H. (2018). Minimap2: pairwise alignment for nucleotide sequences. Bioinformatics, 34(18), 3094–3100. https://academic.oup.com/bioinformatics/article/34/18/3094/4994778

Magoc, T., Pabinger, S., Canzar, S., Liu, X., Su, Q., Puiu, D., Tallon, L. J., & Salzberg, S. L. (2013). GAGE-B: an evaluation of genome assemblers for bacterial organisms. Bioinformatics, 29(14), 1718–1725. https://doi.org/10.1093/bioinformatics/btt273

Sedlazeck, F. J., Rescheneder, P., Smolka, M., Fang, H., Nattestad, M., Von Haeseler, A., & Schatz, M. C. (2018b). Accurate detection of complex structural variations using single-molecule sequencing. Nature Methods, 15(6), 461–468. https://doi.org/10.1038/s41592-018-0001-7

Wick, R. R., Judd, L. M., Cerdeira, L. T., Hawkey, J., Méric, G., Vezina, B., Wyres, K. L., & Holt, K. E. (2021). Trycycler: consensus long-read assemblies for bacterial genomes. Genome Biology, 22(1), 266. https://doi.org/10.1186/s13059-021-02483-z

Wick, R. R., Judd, L. M., & Holt, K. E. (2023). Assembling the perfect bacterial genome using Oxford Nanopore and Illumina sequencing. PLoS Computational Biology, 19(3), e1010905. https://doi.org/10.1371/journal.pcbi.1010905

Wick, R. R., Judd, L. M., Stinear, T. P., & Monk, I. R. (2025a). Are reads required? High-precision variant calling from bacterial genome assemblies. Access Microbiology, 7(5). https://doi.org/10.1099/acmi.0.001025.v3

Wick, R. R., P Howden, B., & P Stinear, T. (2025b, August 28). Autocycler: long-read consensus assembly for bacterial genomes. Oxford Academic. https://academic.oup.com/bioinformatics/article/41/9/btaf474/8242761
