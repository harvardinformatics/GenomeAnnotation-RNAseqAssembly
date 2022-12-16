# GenomeAnnotation-RNAseqAssembly
This page provided a snakemake workflow for implementing alignment of RNA-seq Illumina short reads to genomes with [HISAT2](https://daehwankimlab.github.io/hisat2/) and [STAR](https://github.com/alexdobin/STAR), and assembling transcripts from RNA-seq alignments with [stringtie](https://github.com/gpertea/stringtie) and [scallop](https://github.com/Kingsford-Group/scallop). The current workflow settings are for unstranded RNA-seq libraries. Command line arguments should be added to rules depending upon whether libraries are "FR" oriented (e.g. ligation-stranded protocols) or "RF" oriented (e.g. dUTP-based). The naming of fastq files must match that output by [Trim_Galore]. Furthermore, parsing of those files extracts sample names assuming the fastq file names begin with a sample id followed by an underscore. Slight changes to rules and the SAMPLES variable command in the Snakefile may be necessary if sample ids are more complicated.

For STAR, we perform 2-pass mapping. There is an initial alignment step that generates bam files and splice site tables, indicating splice junctions found in the reads. We ignore the bam files generated in this 1st pass. In the second pass, we concatenate all the splice site tables from all samples into one splice table, and provide this for sample-level 2nd pass alignments. We do this to increase sensitivity with respect to possible splice sites, leveraging information across samples.

Prior to running the workflow, genome indices for HISAT2 and STAR must be built. This can be done as described below

#### Build HISAT2 genome index

We do this as follows

```bash
conda create -n hisat2 -c bioconda hisat2
source activate hisat2
hisat2-build -p 6 /PATH/TO/GENOME/genome.fasta IndexBaseName
conda deactivate
```
where IndexBaseName is simply the name you assign as the prefix for the various index files, and -p indicates the number of threads.

#### Build STAR genome index

```bash
conda create -n STAR -c bioconda STAR
source activate STAR
STAR --runMode genomeGenerate --genomeSAindexNbases 13 --genomeDir $(pwd) --genomeFastaFiles /PATH/TO/GENOME/genome.fasta --runThreadN 12
conda deactivate
```

The --genomeSAindexNbases argument is necessary for smaller genomes, otherwise the *genomeGenerate* tools will raise an exception. Consult the log files from the indexing step to see if STAR recommends changing the setting for this argument.

 
