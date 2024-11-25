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

The busco statistics of final braker gene prediction following Tsebra gene selection

```
C:70.1%[S:54.6%,D:15.5%],F:9.7%,M:20.2%,n:1013
```
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


BRAKER produces several important output files in the working directory.

braker.gtf: Final gene set of BRAKER. This file may contain different contents depending on how you called BRAKER in ETPmode: 
    Final gene set of BRAKER consisting of genes predicted by AUGUSTUS and GeneMark-ETP that were combined and filtered by TSEBRA.
    otherwise: Union of augustus.hints.gtf and reliable GeneMark-ES/ET/EP predictions (genes fully supported by external evidence). 
In --esmode, this is the union of augustus.ab_initio.gtf and all GeneMark-ES genes. Thus, this set is generally more sensitive (more genes correctly predicted) and can be less specific (more false-positive predictions can be present). *This output is not necessarily better than augustus.hints.gtf*, and it is not recommended to use it if BRAKER was run in ESmode.


In this annotation, we find augustus.hints.gtf to be better than braker.gtf (the output of the combined and filtered by TSEBRA though we ran the braker in ETP mode) as shown by the high busco score relative to the final output. Therefore I will recommend to use augustus.hints.gtf since AUGUSTUS is trained on 'high-confindent' genes (genes with very high extrinsic evidence support) from the GeneMark-ETP prediction and a set of genes is predicted by AUGUSTUS. 

First we filter those less than 100bp in ORF

`agat_sp_filter_by_ORF_size.pl --gff augustus_hintsagat.gff3 -o augustus_hintsagatORF.gff3`

then we filter out those with incomplete gene models 

`agat_sp_filter_incomplete_gene_coding_models.pl --gff augustus_hintsagatORF3_sup100.gff --fasta Artemia_sinica_genome_29_12_2021.fa -o augustus_hintsagatORF3_sup100compgene.gff`

Fix overlapping genes if any

`agat_sp_fix_overlaping_genes.pl -f augustus_hintsagatORF3_sup100compgene.gff -o augustus_hintsagatORF3_sup100compgenefix.gff3`

statistics of the annotation

```
Type (3rd column)       Number  Size total (kb) Size mean (bp)  % of the genome /!\Results are rounding to two decimal places
cds     121633  28650.80        235.55  1.68
exon    121633  28999.84        238.42  1.70
five_prime_utr  9       299.24  33249.22        0.02
gene    24664   915488.32       37118.40        53.82
intron  94849   1059071.07      11165.86        62.26
mrna    26798   1087721.87      40589.67        63.95
start_codon     26788   80.36   3.00    0.00
stop_codon      26793   80.38   3.00    0.00
three_prime_utr 5       49.80   9960.20 0.00
Total   443172  3120441.69      7041.15 183.44
```




```
C:80.4%[S:72.7%,D:7.7%],F:9.5%,M:10.1%,n:1013
```


# We are now running the annotation using the soft-masked genome and RNA sequencing data and compare the output. 

```
braker.pl --species=ArtemiaSinicaannexp --genome=Artemia_sinica_genome_29_12_2021.fa.masked --rnaseq_sets_ids=SRR15446616,SRR15446637,SRR15446638,SRR15446639,SRR15446642,SRR15446651,SRR15446664,SRR15446667,SRR15446668,SRR15446669,SRR15446670,SRR15446671,SRR15446672,SRR15446673,SRR15446674,SRR15446675,SRR15446676,SRR15446677,SRR15446678,SRR15446679,SRR15446680,SRR15446681,SRR15446682,SRR15446683 --CDBTOOLS_PATH=/path/cdbfasta/20230902/ --TSEBRA_PATH=/path/TSEBRA/bin/ --useexisting --gff3 --threads 20 --workingdir=/path/brakerv3_starmaskedexp
```

![braker1](https://github.com/user-attachments/assets/4ee7aaf0-9e42-4b3f-94d4-1ec8df2c8275)

Refining the annotation output

`agat_convert_sp_gxf2gxf.pl -g braker.gff3 -o braker_agat.gff3`

First we filter those less than 100bp in ORF

`agat_sp_filter_by_ORF_size.pl --gff braker_agat.gff3 -o braker_agatORF.gff3`


then we filter out those with incomplete gene models 

`agat_sp_filter_incomplete_gene_coding_models.pl --gff braker_agatORF3_sup100.gff --fasta Artemia_sinica_genome_29_12_2021.fa -o braker_agatORF3_sup100compgene.gff`


fix overlapping genes if any

`agat_sp_fix_overlaping_genes.pl -f braker_agatORF3_sup100compgene.gff -o braker_agatORF3_sup100compgenefix.gff3`

Get getAnnoFasta From Joingenes

`python getAnnoFastaFromJoingenes.py -g Artemia_sinica_genome_29_12_2021.fa -f braker_agatORF3_sup100compgenefix.gtf -o braker_agatORF3_sup100compgenefix`


annotation statistics

```
Type (3rd column)       Number  Size total (kb) Size mean (bp)  % of the genome /!\Results are rounding to two decimal places
cds     130595  30069.42        230.25  1.77
exon    130595  30218.09        231.39  1.78
five_prime_utr  13      101.57  7813.31 0.01
gene    25035   763676.41       30504.35        44.90
intron  100938  921720.07       9131.55 54.19
mrna    29675   951789.48       32073.78        55.95
start_codon     29662   88.99   3.00    0.01
stop_codon      29670   89.01   3.00    0.01
three_prime_utr 5       47.10   9421.00 0.00
Total   476188  2697800.15      5665.41 158.60
```

#### The busco score for the final annotation output

`C:79.3%[S:67.8%,D:11.5%],F:10.3%,M:10.4%,n:1013`

