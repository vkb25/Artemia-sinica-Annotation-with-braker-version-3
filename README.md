# Artemia-sinica-Annotation-with-braker-version-3

### We need to softmasked the repetitive elements in the genome

##### First is to identify repeats de novo from your reference genome using RepeatModeler

Let's load the necessary modules to use

```
module load RepeatModeler

module load RepeatMasker

```

```
BuildDatabase -name Artemia_sinica_genome_29_12_2021 -engine ncbi Artemia_sinica_genome_29_12_2021.fasta

RepeatModeler -threads 10 -database Artemia_sinica_genome_29_12_2021
```

###### We then annotate the repeats using the library databases created above

`RepeatMasker -pa 30 -lib Artemia_sinica_genome_29_12_2021_renamed-families.fa -xsmall Artemia_sinica_genome_29_12_2021.fa`


###### We then annotate the genome using both RNA-Seq and protein data following this pipeline described here[](https://github.com/Gaius-Augustus/BRAKER)

The braker recommend using OrthoDB as basis for proteins.fa. The instructions on how to prepare the input OrthoDB proteins are documented here:[](https://github.com/gatech-genemark/ProtHint#protein-database-preparation). We used arthropoda protein sequences version 10 and we prepared as the input file as below


```

wget https://v100.orthodb.org/download/odb10_arthropoda_fasta.tar.gz

tar xvfz odb10_arthropoda_fasta.tar.gz

cat arthropoda/Rawdata/* > proteins.fasta

```

The pipeline ran as described here 

![nihpp-2023 06 10 544449v4-f0001](https://github.com/user-attachments/assets/a2de2198-2acc-4ef7-940b-a812d14761fc)


The actual run of braker in Artemia sinica annotation

```
braker.pl --species=ArtemiaSinicaannv3 --genome=Artemia_sinica_genome_29_12_2021.fa.masked --prot_seq=proteins.fa --rnaseq_sets_ids=SRR15446616,SRR15446637,SRR15446638,SRR15446639,SRR15446642,SRR15446651,SRR15446664,SRR15446667,SRR15446668,SRR15446669,SRR15446670,SRR15446671,SRR15446672,SRR15446673,SRR15446674,SRR15446675,SRR15446676,SRR15446677,SRR15446678,SRR15446679,SRR15446680,SRR15446681,SRR15446682,SRR15446683 --CDBTOOLS_PATH=/path/cdbfasta/20230902/ --TSEBRA_PATH=/nfs/scistore18/vicosgrp/vbett/Tools/TSEBRA/bin/ --useexisting --gff3 --threads 20 --workingdir=/path/brakerv3_maskedrnd
```

The we fix the trasncripts id your feature doesn't contain any Parent and locus tag using agat in two steps;

```
agat_convert_sp_gxf2gxf.pl -g braker.gff3 -o braker_agat.gff3

agat_convert_sp_gff2gtf.pl --gff braker_agat.gff3 -o braker_agat.gtf
```
