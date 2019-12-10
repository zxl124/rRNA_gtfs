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
To convert the file from UCSC format to Ensembl format, conversion tables from [ChromsomeMappings](https://github.com/dpryan79/ChromosomeMappings) were used. Chromosomes not in the conversion tables were ommitted.<br>
Additionally, `gene_biotype "rRNA"` were added to end of every line.
## Recommeded Use
There are many ways to use these files, of course. One way that I have tested and found quite effective is simply concatenating these GTF files to the original one, and running `featureCounts` with the options `-g gene_biotype -M -O --fraction --primary`. `-g gene_biotype` would tell `featureCounts` to tabulate reads by gene_biotype attribute; `-M -O --fraction` would tell `featureCounts` to count reads aligned to multiple locations and/or multiple genes, by assign them to target biotypes by fractions of reads, therefore not counting one read multiple times. `--primary` would tell `featureCounts` to count only primary alignments, not counting different alignments of the same read multiple times. The final tally would be a good approximation of biotype composition of the RNA-seq library.
```bash
cat GRCh38.gtf >> GRCh38_original.gtf
featureCounts -a GRCh38_original.gtf -g gene_biotype -M -O --fraction --primary -p -o sample_biotype.featureCounts.txt -s 0 sample.bam
```
