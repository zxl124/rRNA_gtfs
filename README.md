## Background
In some common genome annotation GTF/GFF files, rRNA repeats are not properly marked. A detailed description/discussion of this problem can be seen [here](http://seqanswers.com/forums/showthread.php?t=41868). This can cause ineffective identification/filtering of rRNA reads in RNA-seq studies. To address this problem, a set of GTF files obtained from [UCSC table browser](https://genome.ucsc.edu/cgi-bin/hgTables) and modified versions of them are hosted in this repo to assist rRNA-related quality-check steps in RNA-seq data analysis pipelines.
## Source
Original GTF files were obtained from [UCSC table browser](https://genome.ucsc.edu/cgi-bin/hgTables) by selecting:
1. Genome and version
2. `Repeats` or `Variation or Repeats` under "group"
3. `RepeatMasker` under "track"
4. Enter `repClass does match rRNA` under "filter"
5. Choose `GTF` under "output format"
## Modification
Each line in the GTF file was duplicated with the third column changed from 'exon' to 'gene'. This is to increase compatiblity if users choose to count 'gene' instead of 'exon' using `featureCounts`. 
To convert the file from UCSC format to Ensembl format, conversion tables from [ChromsomeMappings](https://github.com/dpryan79/ChromosomeMappings) were used. Chromosomes not in the conversion tables were ommitted.<br>
Additionally, `gene_biotype "rRNA"` were added to end of every line.
## Recommeded Use
There are many ways to use these files, of course. One way that I have tested and found quite effective is simply concatenating these GTF files to the original one, and running `featureCounts` with the options `-g gene_biotype -M -O --fraction`. `-g gene_biotype` would tell `featureCounts` to tabulate reads by gene_biotype attribute; `-M -O --fraction` would tell `featureCounts` to count reads aligned to multiple locations and/or multiple genes, by assign them to target biotypes by fractions of reads, therefore not counting one read multiple times. The final tally would be a good approximation of biotype composition of the RNA-seq library.<br>
Optionally, you can choose to add `-t gene` to the `featureCounts` command. This will ask `featureCounts` to count reads overlapping with genes instead of exons. Counting only reads overlapping exons will result in bias against protein coding genes because they may have reads overlapping introns, which won't be counted, while other biotypes such as rRNA have little or no introns. This bias is stronger in samples with more intron content. For example, library prepared using rRNA depletion protocol generally has more pre-mRNA than libraries prepared with polyA selection protocol. So you might want to consider using the `-t gene` option for the former library type.
```bash
cat GRCh38.gtf >> GRCh38_original.gtf
featureCounts -a GRCh38_original.gtf -g gene_biotype -M -O --fraction -p -o sample_biotype.featureCounts.txt -s 0 sample.bam
```
## Test results
Test was done using two RNA-seq samples, one with rRNA depeltion, and one without. The FASTQ files were analyzed using a modified version of [nf-core/rnaseq](https://github.com/nf-core/rnaseq) pipeline version 1.4.2 using GRCh38 as reference. The analysis were done three times.
1. With original featureCounts code, almost no rRNA was detected.
![original result](/image/original.jpg)
2. With `-M -O --fraction` options, a little bit of rRNA was detected, but still very low.
![without rRNA GTF](/image/without_rRNA_gtf.jpg)
3. with both rRNA GTF file and `-M -O --fraction`, rRNA was detected correctly (close to expectations).
![with rRNA GTF](/image/with_rRNA_gtf.jpg)

Test data and MultiQC report of the test runs are available upon request.
