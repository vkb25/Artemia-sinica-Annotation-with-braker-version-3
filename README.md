# Artemia-sinica-Annotation-with-braker-version-3[](https://pmc.ncbi.nlm.nih.gov/articles/PMC10312602/)

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
braker.pl --species=ArtemiaSinicaannv3 --genome=Artemia_sinica_genome_29_12_2021.fa.masked --prot_seq=proteins.fa --rnaseq_sets_ids=SRR15446616,SRR15446637,SRR15446638,SRR15446639,SRR15446642,SRR15446651,SRR15446664,SRR15446667,SRR15446668,SRR15446669,SRR15446670,SRR15446671,SRR15446672,SRR15446673,SRR15446674,SRR15446675,SRR15446676,SRR15446677,SRR15446678,SRR15446679,SRR15446680,SRR15446681,SRR15446682,SRR15446683 --CDBTOOLS_PATH=/path/cdbfasta/20230902/ --TSEBRA_PATH=/path/TSEBRA/bin/ --useexisting --gff3 --threads 20 --workingdir=/path/brakerv3_maskedrnd
```

The we fix the transcripts id 'feature doesn't contain any Parent and locus tag' using agat in two steps;

```
agat_convert_sp_gxf2gxf.pl -g braker.gff3 -o braker_agat.gff3

agat_convert_sp_gff2gtf.pl --gff braker_agat.gff3 -o braker_agat.gtf
```

in case you are interested in keeping only longest isoform in each gene, you can use this command line

`agat_convert_sp_gff2gtf.pl --gff braker_agatiso.gff3 -o braker_agatiso.gtf`


Then we get cds and aa of the annotation using getAnnoFastaFromJoingenes.py

`python getAnnoFastaFromJoingenes.py -g Artemia_sinica_genome_29_12_2021.fa -f braker_agat.gtf -o braker_agat`

Then we check for completeness using BUSCO as follows

```
module load anaconda3/2024.03_deb12
source /path/activate_anaconda3_2024.03_deb12.txt
conda activate busco
```

`busco --in braker_agat.aa -c 20 -l arthropoda -f --mode prot -o busco_brakagat`


We can also check the statistics of the annotation using this

`agat_sq_stat_basic.pl -i braker_agat.gff3 -g Artemia_sinica_genome_29_12_2021.fa -o braker_agat_statistics`


output of annotation statistics

```
Type (3rd column)       Number  Size total (kb) Size mean (bp)  % of the genome /!\Results are rounding to two decimal places
cds     86557   17398.50        201.01  1.02
exon    86557   17398.50        201.01  1.02
gene    12513   546127.83       43644.84        32.11
intron  71122   750016.76       10545.50        44.09
mrna    15435   767415.26       49719.16        45.11
start_codon     15430   46.19   2.99    0.00
stop_codon      15434   46.27   3.00    0.00
Total   303048  2098449.31      6924.48 123.36
```
