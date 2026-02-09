# _Salmonella enterica_ Genome Assembly
## General Overview
This project reconstructs the genome of _Salmonella enterica_ using Oxford Nanopore long-read sequencing and evaluates assembly quality, alignment accuracy, and genomic variation. The analysis focuses on both chromosomal and plasmid sequences, highlighting differences in sequence conservation, structural variation, and gene level mutations.

## Introduction
Genome assembly reconstructs a complete DNA sequence from shorter sequencing reads, enabling the study of an organism’s genetic content (Koren et al., 2013). Since many sequencing platforms cannot read an entire chromosome in a single read, DNA is fragmented and sequenced multiple times to ensure sufficient coverage across the genome (Wick et al., 2023). Despite advances in sequencing technologies, genome assembly remains challenging due to repetitive regions, sequencing errors, and uneven coverage. In particular, repeats longer than individual reads can result in fragmented or misassembled genomes, limiting their utility for downstream comparative and functional analyses (Koren et al., 2013). 

These challenges are especially relevant for bacterial pathogens such as _Salmonella enterica_, a leading cause of foodborne illness in the United States (Han et al., 2024). The severity of these infections varies significantly between strains, driven largely by the unique genetic makeup of specific serovars. Characterizing these genetic features is essential for understanding pathogen evolution, antimicrobial resistance, and adaptation to environmental pressures (Han et al., 2024). Therefore, this study aims to reconstruct the _Salmonella enterica_ genome using long-read sequencing and to evaluate assembly quality through comparison with a reference genome.

To address the challenges of repetitive regions, early assembly strategies utilized paired end sequencing to bridge gaps; however, these methods were often limited by fragment length and increased experimental complexity (Koren et al., 2013). Modern advances in long read platforms like Oxford Nanopore Technologies (ONT) and Pacific Biosciences (PacBio) have overcome these challenges by generating reads long enough to span repetitive regions (Wick et al., 2025b), enabling the complete assembly of bacterial genomes into single contigs per replicon (Wick et al., 2021). While assemblers like Canu and Flye are specifically designed for these datasets, long read assembly is not without limitations; different assemblers can produce varying results, including potential issues with missing plasmids, extra contigs, or incomplete circularization (Wick et al., 2025b).

Among long read assemblers, Flye and Canu represent two prominent yet distinct approaches. Flye utilizes repeat graphs to resolve unbridged repeats and generally offers superior speed and accuracy for bacterial ONT data compared to Canu, which, while highly contiguous, can be slower and less effective at resolving near identical repeats (Kolmogorov et al., 2019). Despite Flye’s strengths, achieving optimal base level accuracy in highly repetitive regions often requires post assembly polishing to correct residual errors. Following assembly, comparing the resulting consensus to a high quality reference genome is essential for evaluating both genomic differences and assembly quality. However, the assembly and reference genome can have different structures (such as inversions or genomic island movements), causing aligners to produce "clipped" or chimeric mappings (Magoc et al., 2013). While sequence alignment can reveal SNPs, indels, and structural variants (SVs), these differences may reflect true biological variation rather than technical errors (Magoc et al., 2013), making the selection of alignment and variant calling tools critical.

To address these challenges, Minimap2 provides high performance alignments optimized for long read data (Li et al., 2018). While it offers significant speed advantages over tools like NGMLR (Sedlazeck et al., 2018), this efficiency can occasionally reduce sensitivity in highly divergent genomic regions. To mitigate the risk of false positives, a dual validation strategy is ideal: read based tools like Clair3 offer high precision SNP detection (Wick et al., 2025a), but struggle with large structural variants and regions with low coverage or high complexity. By utilizing both methods, researchers can simultaneously detect small scale variants and perform large scale structural validation. These findings are ultimately finalized through the Circos plot and bar plot, which allows for the inspection of overall assembly regions and the structural variants per gene.

In this study, Oxford Nanopore reads will be assembled into a consensus genome for _Salmonella enterica_ using Flye. The resulting consensus will be aligned to an NCBI reference genome using Minimap2, followed by integrated variant calling and visualization.


## Methods
### 1. Data Description
Raw ONT sequencing reads (FASTQ format) for _Salmonella enterica_ were obtained from the NCBI Sequence Read Archive (SRA) under accession SRR32410565. The dataset was generated using R10 chemistry, providing an estimated base quality of Q20+ and an N50 read length of 5–15 kb. For the reference based analyses, the complete genome assembly for _Salmonella enterica_ (ASM694v2) was retrieved from the NCBI RefSeq database in FASTA and GTF/GFF format.

### 2. Quality Control
Initial read quality and length distributions were assessed using NanoPlot (v1.42.0) to determine the necessity of quality based filtering (De Coster et al., 2018). Given the high read quality (Q20+) and consistent length distribution, additional filtering was unlikely to improve assembly accuracy. Moreover, strict filtering can disproportionately remove low coverage or shorter reads originating, potentially reducing assembly completeness.

### 3. Genome Assembly
Reads were assembled using de novo Flye (v2.9.6), a long read assembler optimized for error-prone ONT data (Kolmogorov et al., 2019); with the use of the `--nano-hq` option, and a genome size of approximately 5 Mb (Megabases) (`--genome-size 5m`). Note, `--nano-hq` was chosen over `--nano-raw` due to the high quality reads of Q20+. Moreover, QUAST (v5.2.0) was selected to assess contiguity, consensus accuracy, and large scale structural consistency with the reference genome, which is critical for evaluating the reliability of downstream variant and structural analyses (Gurevich et al., 2013). 

### 4. Alignment
Two alignment steps were performed to assess different aspects of data quality and assembly accuracy. The raw Oxford Nanopore data was aligned to the reference genome using Minimap2 (v2.28) with the `-ax map-ont` preset, which is optimized for the error profile of ONT reads, and is recommended for raw ONT read alignment (Li et al., 2018). Resulting SAM files were converted to compressed BAM format, sorted, and indexed using SAMtools (v1.22.1). Alignment statistics, including mapping rate, depth of coverage, and read level quality metrics, were obtained using SAMtools flagstat and depth to confirm sufficient coverage for assembly. An average depth of **154x** was observed.

Furthermore, to verify structural variation and assembly analysis, the assembled genome was aligned to the reference genome using Minimap2 with the preset `-ax asm5`. This preset was selected because it is optimized for alignment of assembled sequences with low divergence (≤5%) (E Pearce, 2021). 

### 5. Variant calling
To compute variant calling, the raw read alignments used Clair3 (v1.1.2). The contigs were generated using the BAM file, and the `--platform ont` parameter was chosen to specify the nanopore data. In addition, the `--include_all_ctgs` option was selected to ensure that all contigs from the Flye assembly were considered, including any small or low coverage contigs for downstream comparative analyses.

### 6. Visualization
Final validation was performed using Circos (v0.69-9) and bar plots generated in R (v4.5.2). Circos was used in combination with MUMmer (v4.0.0rc1) and BCFtools (v1.19) to assess structural relationships between the assembled contigs and the reference genome. Alignments were generated using Nucmer and processed with `show-coords`, filtering for high confidence matches with ≥90% sequence identity and alignment lengths ≥10 kb (Jain, 2018). These thresholds were chosen to retain meaningful alignments while excluding short or low identity matches that may represent repetitive or ambiguous regions. 

Reference and assembly sequence lengths were indexed using samtools faidx to generate karyotype files required for Circos visualization. Circos plots were configured to emphasize genome wide synteny, and additional plots were produced to compare filtered versus unfiltered alignments as well as to visualize the genomic distribution of SNPs and indels.

Lastly, a barplot was created to view specific genes containing structural variants of indels or SNPs. This analysis was performed in R using the reference genome annotation (GFF) and variant calls (VCF). To improve interpretability, only genes containing two or more variants were included, allowing emphasis on genes with potentially meaningful levels of variation.

## Results

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

**Figure 3:** Circos plot showing the distribution of SNPs (blue) and indels (red) relative to the _Salmonella enterica_ reference genome (grey).


<img width="960" height="488" alt="Image" src="https://github.com/user-attachments/assets/20ac8116-bdee-40e2-8d3c-bc19abf982ee" />

**Figure 4:** Bar plot showing the number of SNPs and indels per gene. Only genes containing two or more variants are shown to emphasize loci with elevated sequence variation.


## Discussion
### 1. Sequencing Quality and Data Evaluation
To begin, quality control was first performed to evaluate whether filtering of the raw long read data was necessary prior to assembly. The NanoPlot report indicated a total yield of approximately 809 Mb, which significantly exceeds the ~4.7–5.0 Mb genome size typically reported for _Salmonella enterica_ (Stevens et al., 2018). This level of sequencing output corresponds to >150x coverage, providing redundancy for error correction during assembly. The median read quality score (Q23.7) exceeds the commonly accepted threshold for high quality long read data, suggesting a low per base error rate. Moreover, the similarity between pre and post filtering summary statistics indicates that the dataset contained few low quality or excessively short reads. As a result, additional filtering was unnecessary, and would not improve assembly quality and may reduce coverage, particularly for low copy elements such as plasmids.

### 2. Genome Assembly Assessment
De novo assembly of the unfiltered reads using Flye resulted in three contigs, which were evaluated using QUAST. Two contigs aligned well to the reference genome, while one contig remained unaligned. Unaligned contigs in bacterial assemblies may arise from low complexity regions, residual sequencing artifacts, or plasmid or mobile elements that differ significantly from the reference (Schiavone et al., 2013). Despite this, the total assembly length of ~4.7 Mb closely matches the expected genome size of _Salmonella enterica_, indicating minimal contamination or missing sequence. In addition, the high N50 value (~4.6 Mb) suggests that nearly the entire chromosome was assembled into a single contiguous sequence. This level of contiguity reflects both the long read sequencing strategy and sufficient depth to resolve repetitive regions that often fragment into short reads. In addition, the low mismatch rate (27 mismatches per 100 kbp) supports a high consensus accuracy and indicates that base level errors were effectively corrected during assembly.

### 3. Alignment Quality and Coverage
To further assess data integrity, raw reads were aligned to the reference genome and evaluated using flagstat and average depth metrics. A mapping rate of 94.3% indicates minimal read loss during alignment. The presence of secondary and supplementary alignments likely reflects repetitive regions, structural variation, or split alignments across insertion sequences, which are common in bacterial genomes (Pang et al., 2013).

Moreover, the average sequencing depth of ~154x exceeds the typical 60–80x coverage often considered sufficient for bacterial variant detection (Pang et al., 2013). While higher coverage does not necessarily improve accuracy beyond a certain point, it increases confidence in variant calls and supports detection of low frequency variants. The absence of duplicate reads or QC failed alignments further confirms the overall quality of the dataset.

### 4. Whole Genome Alignment
Whole genome alignment of the assembled contigs to the reference genome was performed with and without filtering to assess the impact of alignment. In the filtered alignment (Figure 1), two contigs map strongly to NC_003197.2, the chromosome of _Salmonella enterica_ serovar Typhimurium LT2. This genetic consistency reflects the nature of the _Salmonella_ core genome, which is generally stable among isolates belonging to the same serovar, defined by shared surface antigens (Luo et al., 2012). The near total coverage of the reference genome confirms a highly accurate assembly with minimal risk of large-scale errors, justifying its use for downstream analysis. 

A notable gap in the filtered alignment corresponds to NC_003277.2, the pSLT virulence plasmid. This plasmid was excluded under filtering conditions due to length and/or quality thresholds, highlighting a common challenge in genome assembly workflows. Plasmids often have lower coverage, distinct nucleotide composition, or higher repeat content compared to chromosomes, making them more susceptible to removal during filtering (Robertson, 2023). When filtering was removed (Figure 2), the plasmid sequences were retained, resulting in a noisier alignment with multiple overlapping regions. While this complicates interpretation, it also reveals repetitive and mobile genetic elements. Such elements are frequently associated with virulence and antimicrobial resistance and are a key source of genomic diversity in _Salmonella_ (Robertson, 2023).

### 5. Structural Variations
As shown in Figure 3, genomic variability is much higher in plasmids than in the chromosome. Unlike the stable chromosomal core, plasmids are highly dynamic and undergo constant genetic exchange. This fluidity enables the bacteria to quickly gain new traits, such as host adaptation and drug resistance, through gene acquisition and rearrangement (Robertson, 2023).

To quantify sequence level variation, SNPs and indels were summarized per gene (Figure 4). The majority of variants were concentrated in genes located on the pSLT plasmid, with several genes containing high SNP counts (>400–700 SNPs), while most genes contained few or no variants. Gene pSLT108 showed the highest SNP (755 SNPs), whereas STM1022 exhibited the highest number of indels (68 indels). This distribution is consistent with known bacterial mutation patterns, where SNPs occur more frequently than indels, with an estimated ratio of 0.5–2 indels per 10 SNPs in prokaryotic genomes (Felten et al., 2017). The lack of indels shows that the genome is under pressure to avoid frameshift mutations, which are usually harmful to protein function.

Moreover, the pSLT108 gene is located on the pSLT virulence plasmid and is commonly annotated as traT. This gene encodes an outer membrane lipoprotein involved in surface exclusion, a mechanism that prevents the entry of redundant plasmids into cells already harboring pSLT (Chen et al., 2025). SNPs within coding regions may be synonymous (silent) or nonsynonymous (missense) (Loo et al., 2020). While synonymous mutations do not alter the amino acid sequence, missense mutations can influence protein structure, stability, or activity. The high SNP density observed in pSLT108 may therefore reflect adaptive fine tuning of plasmid associated functions, allowing sequence diversification without compromising essential plasmid maintenance or virulence roles.

In contrast, STM1022 is annotated as a chaperone associated gene involved in protein folding and stability under stress conditions (Pullinger et al., 2008). Insertions and deletions within coding regions are more likely to result in frameshift mutations, which alter the downstream reading frame and frequently produce truncated or nonfunctional proteins (Williams et al., 2013). The accumulation of indels in STM1022 suggests that this gene may tolerate loss-of-function mutations or be under relaxed selective pressure, potentially reducing stress tolerance or virulence without being lethal to the bacterium (Lin et al., 2017).

### 6. Conclusion
Overall, the observed variant profile aligns with biological expectations for _Salmonella enterica_ and reinforces the importance of variant calling in bacterial genomics; where core chromosomal genes remain conserved while plasmid-encoded and accessory genes accumulate greater sequence variation. Such analyses are essential for understanding evolutionary relationships, tracking transmission, and identifying genetic determinants of antimicrobial resistance (Hall et al., 2024).

## References
Chen, N., Bukys, A., Lundgren, C. a. K., Deme, J. C., Sayyed, H. E., Kapanidis, A. N., Lea, S. M., & Berks, B. C. (2025). Structure of the conjugation surface exclusion protein TraT. Communications Biology, 8(1), 1702. https://doi.org/10.1038/s42003-025-09102-8

De Coster, W., D’Hert, S., Schultz, D. T., Cruts, M., & Van Broeckhoven, C. (2018). NanoPack: visualizing and processing long-read sequencing data. Bioinformatics, 34(15), 2666–2669. https://academic.oup.com/bioinformatics/article/34/15/2666/4934939

E Pearce, M., C. Langridge C, G., & Lauer D, A. C. (2021, September). An evaluation of the species and subspecies of the genus Salmonella with whole genome sequence data: Proposal of Type Strains and epithets for Novel S. Enterica Subspecies VII, VIII, IX, X and XI. ScienceDirect. https://www.sciencedirect.com/science/article/pii/S0888754321002810

Felten, A., Nova, M. V., Durimel, K., Guillier, L., Mistou, M., & Radomski, N. (2017). First gene-ontology enrichment analysis based on bacterial coregenome variants: insights into adaptations of Salmonella serovars to mammalian- and avian-hosts. BMC Microbiology, 17(1), 222. https://doi.org/10.1186/s12866-017-1132-1

Gurevich, A., Saveliev, V., Vyahhi, N., & Tesler, G. (2013). QUAST: quality assessment tool for genome assemblies. Bioinformatics, 29(8), 1072–1075. https://doi.org/10.1093/bioinformatics/btt086

Hall, M. B., Wick, R. R., Judd, L. M., Nguyen, A. N., Steinig, E. J., Xie, O., Davies, M., Seemann, T., Stinear, T. P., & Coin, L. (2024). Benchmarking reveals superiority of deep learning variant callers on bacterial nanopore sequence data. eLife, 13. https://doi.org/10.7554/elife.98300

Han, J., Aljahdali, N., Zhao, S., Tang, H., Harbottle, H., Hoffmann, M., Frye, J. G., & Foley, S. L. (2024). Infection biology of Salmonella enterica. EcoSal Plus, 12(1), eesp00012023. https://doi.org/10.1128/ecosalplus.esp-0001-2023

Jain, C., Rodriguez-R, L. M., Phillippy, A. M., Konstantinidis, K. T., & Aluru, S. (2018). High throughput ANI analysis of 90K prokaryotic genomes reveals clear species boundaries. Nature Communications, 9(1), 5114. https://doi.org/10.1038/s41467-018-07641-9

Kolmogorov, M., Yuan, J., Lin, Y., & Pevzner, P. A. (2019). Assembly of long, error-prone reads using repeat graphs. Nature Biotechnology, 37(5), 540–546. https://doi.org/10.1038/s41587-019-0072-8

Koren, S., Harhay, G. P., Smith, T. P., Bono, J. L., Harhay, D. M., Mcvey, S. D., Radune, D., Bergman, N. H., & Phillippy, A. M. (2013). Reducing assembly complexity of microbial genomes with single-molecule sequencing. Genome Biology, 14(9), R101. https://doi.org/10.1186/gb-2013-14-9-r101

Loo, S. L., Ong, A., Kyaw, W., Thibaut, L. M., Lan, R., & Tanaka, M. M. (2020). Nonsynonymous Polymorphism Counts in Bacterial Genomes: a Comparative Examination. Applied and Environmental Microbiology, 87(1). https://doi.org/10.1128/aem.02002-20

Li, H. (2018). Minimap2: pairwise alignment for nucleotide sequences. Bioinformatics, 34(18), 3094–3100. https://academic.oup.com/bioinformatics/article/34/18/3094/4994778

Lin, M., Whitmire, S., Chen, J., Farrel, A., Shi, X., & Guo, J. (2017). Effects of short indels on protein structure and function in human genomes. Scientific Reports, 7(1), 9313. https://doi.org/10.1038/s41598-017-09287-x

Luo, Y., Kong, Q., Yang, J., Mitra, A., Golden, G., Wanda, S., Roland, K. L., Jensen, R. V., Ernst, P. B., & Curtiss, R. (2012). Comparative genome analysis of the High pathogenicity Salmonella typhimurium strain UK-1. PLoS ONE, 7(7), e40645. https://doi.org/10.1371/journal.pone.0040645

Magoc, T., Pabinger, S., Canzar, S., Liu, X., Su, Q., Puiu, D., Tallon, L. J., & Salzberg, S. L. (2013). GAGE-B: an evaluation of genome assemblers for bacterial organisms. Bioinformatics, 29(14), 1718–1725. https://doi.org/10.1093/bioinformatics/btt273

Pang, S., Octavia, S., Feng, L., Liu, B., Reeves, P. R., Lan, R., & Wang, L. (2013). Genomic diversity and adaptation of Salmonella enterica serovar Typhimurium from analysis of six genomes of different phage types. BMC Genomics, 14(1), 718. https://doi.org/10.1186/1471-2164-14-718

Pullinger, G. D., Dziva, F., Charleston, B., Wallis, T. S., & Stevens, M. P. (2008). Identification of Salmonella enterica Serovar Dublin-Specific Sequences by Subtractive Hybridization and Analysis of Their Role in Intestinal Colonization and Systemic Translocation in Cattle. Infection and Immunity, 76(11), 5310–5321. https://doi.org/10.1128/iai.00960-08

Robertson, J., Schonfeld, J., Bessonov, K., Bastedo, P., & Nash, J. H. E. (2023). A global survey of Salmonella plasmids and their associations with antimicrobial resistance. Microbial Genomics, 9(5). https://doi.org/10.1099/mgen.0.001002

Sedlazeck, F. J., Rescheneder, P., Smolka, M., Fang, H., Nattestad, M., Von Haeseler, A., & Schatz, M. C. (2018b). Accurate detection of complex structural variations using single-molecule sequencing. Nature Methods, 15(6), 461–468. https://doi.org/10.1038/s41592-018-0001-7

Schiavone, A.; Pugliese, N.; Samarelli, R.; Cumbo, C.; Minervini, C.F.; Albano, F.; Camarda, A. Factors Affecting the Quality of Bacterial Genomes Assemblies by Canu after Nanopore Sequencing. Appl. Sci. 2022, 12, 3110. https://doi.org/10.3390/app12063110

Stevens, M. J. A., Zurfluh, K., Althaus, D., Corti, S., Lehner, A., & Stephan, R. (2018). Complete and Assembled Genome Sequence of Salmonella enterica subsp. enterica Serotype Senftenberg N17-509, a Strain Lacking Salmonella Pathogen Island 1. Genome Announcements, 6(12). https://doi.org/10.1128/genomea.00156-18

Wick, R. R., Judd, L. M., Cerdeira, L. T., Hawkey, J., Méric, G., Vezina, B., Wyres, K. L., & Holt, K. E. (2021). Trycycler: consensus long-read assemblies for bacterial genomes. Genome Biology, 22(1), 266. https://doi.org/10.1186/s13059-021-02483-z

Wick, R. R., Judd, L. M., & Holt, K. E. (2023). Assembling the perfect bacterial genome using Oxford Nanopore and Illumina sequencing. PLoS Computational Biology, 19(3), e1010905. https://doi.org/10.1371/journal.pcbi.1010905

Wick, R. R., Judd, L. M., Stinear, T. P., & Monk, I. R. (2025a). Are reads required? High-precision variant calling from bacterial genome assemblies. Access Microbiology, 7(5). https://doi.org/10.1099/acmi.0.001025.v3

Wick, R. R., P Howden, B., & P Stinear, T. (2025b, August 28). Autocycler: long-read consensus assembly for bacterial genomes. Oxford Academic. https://academic.oup.com/bioinformatics/article/41/9/btaf474/8242761

Williams, L. E., & Wernegreen, J. J. (2013). Sequence Context of indel mutations and their effect on protein evolution in a bacterial endosymbiont. Genome Biology and Evolution, 5(3), 599–605. https://doi.org/10.1093/gbe/evt033

Wick, R. R., P Howden, B., & P Stinear, T. (2025b, August 28). Autocycler: long-read consensus assembly for bacterial genomes. Oxford Academic. https://academic.oup.com/bioinformatics/article/41/9/btaf474/8242761
