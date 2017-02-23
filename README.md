Document created 22 Feb 2017

# K-mer counting in tetraploid potato

The goal of this analysis is to identify high-copy kmers that will be used to assemble consensus repeats of tetraploid potato.
See Koch et al (2014) doi: https://doi.org/10.1093/nar/gku210 for reference on repeat assembly from NGS reads.

###Dataset: Tetraploid potato clones Atlantic and Superior were sequenced by the Buell lab (Michigan State University) and retrieved from EMBL ENA. Libraries were randomly sheared
and sequenced on an Illumina HiSeq 2500. Other details of library prep are not available from ENA metadata.

Diploid potato clones IVP-101 and PL-4 were sequenced by the Comai lab 150PE Illumina HiSeq 4000. Libraries made with random shearing to avg 350bp with Covaris E220,
library prep done with KAPA Hyper prep (4 cycles of library amplification) using half-scale reactions for all steps except library amplification.

Tetraploid clone Alca Tarma was also sequenced by the Comai lab using the same library prep and sequencing conditions described above. 

##1. Download metadata from European Nucleotide Archive URL: http://www.ebi.ac.uk/ena/data/view/PRJNA287438 (local/isner)

Select the following columns:
study_accession
sample_accession
secondary_sample_accession
experiment_accession
run_accession
scientific_name
instrument_model
library_layout
read_count
experiment_title
experiment_alias
fastq_bytes
fastq_md5
fastq_ftp

md5sum of metadata file:
```md5 PRJNA287438.txt```
MD5 (PRJNA287438.txt) = 3182e86b4f7fabf9fd27138b35f4bf1c

##2. Copy metadata to isner (UC Davis Genome Center cluster) and download fastq files for Atlantic and Superior WGS reads. Download takes ~6 days.
```grep "WGS" PRJNA287438.txt | cut -f 14 | tr ";" "\n" | xargs wget 2>&1 | tee -a wget_log```
Downloaded: 12 files, 108G in 6d 5h 50m 30s (209 KB/s)

Check to make sure md5 checksums match expected in metadata file
```md5sum *.gz > md5sum_batch_tetraploid_wget.txt```

Files match, we can proceed.

##3. Interleave and pool reads (isner):

```gunzip *.gz```

Get read length for each file:

```
interleaveSwitcher-2.py -f SRR2069940_1.fastq -r SRR2069940_2.fastq -o SRR2069940.fastq 
interleaveSwitcher-2.py -f SRR2070067_1.fastq -r SRR2070067_2.fastq -o SRR2070067.fastq
interleaveSwitcher-2.py -f SRR2070065_1.fastq -r SRR2070065_2.fastq -o SRR2070065.fastq
interleaveSwitcher-2.py -f SRR2070063_1.fastq -r SRR2070063_2.fastq -o SRR2070063.fastq
interleaveSwitcher-2.py -f SRR2069942_1.fastq -r SRR2069942_2.fastq -o SRR2069942.fastq
interleaveSwitcher-2.py -f SRR2069941_1.fastq -r SRR2069941_2.fastq -o SRR2069941.fastq
```

Get read length for each interleaved file

```for i in *.fastq ; do echo $i ; head -n 2 $i | tail -n 1 | awk 'NR==1{print length ($0)}' ; done```

SRR2069940.fastq
150
SRR2069941.fastq
150
SRR2069942.fastq
150
SRR2070063.fastq
100
SRR2070065.fastq
100
SRR2070067.fastq
150

Pool according to clone (Atlantic vs. Superior)

Superior:
```cat SRR2069940.fastq SRR2069941.fastq SRR2069942.fastq SRR2070067.fastq > superior-pooled.fastq```

Atlantic:
```cat SRR2070063.fastq SRR2070065.fastq > atlantic-pooled.fastq```

##4. Tidy up working directory (isner)

```
mkdir download_metadata WGS_interleaved_unpooled WGS_uninterleaved_unpooled WGS_pooled
mv wget_log md5sum_batch_tetraploid_wget.txt download_metadata/
mv SRR2069940_2.fastq SRR2069941_2.fastq SRR2069942_2.fastq SRR2070063_2.fastq SRR2070065_2.fastq SRR2070067_2.fastq SRR2069940_1.fastq SRR2069941_1.fastq SRR2069942_1.fastq SRR2070063_1.fastq SRR2070065_1.fastq SRR2070067_1.fastq WGS_uninterleaved_unpooled/
gzip WGS_uninterleaved_unpooled/*.fastq
mv SRR*.fastq WGS_interleaved_unpooled
mv atlantic-pooled.fastq  superior-pooled.fastq WGS_pooled
```

##5. Extract 30-mers from pooled with Jellyfish (definitely isner)

```module load jellyfish/2.2.3/```

```
jellyfish count atlantic-pooled.fastq -m 30 -s 100M -t 10 -L 2 -o atlantic-pooled-30mer-count.jf
jellyfish histo atlantic-pooled-30mer-count.jf > histo-atlantic-pooled-30mer.txt
```

![alt text][logo]

[logo]: https://github.com/kramundson/Potato-repeat-assembly/images/atlantic30.png "Atlantic 30mer counts"

```
jellyfish count superior-pooled.fastq -m 30 -s 100M -t 10 -L 2 -o superior-pooled-30mer-count.jf
jellyfish histo superior-pooled-30mer-count.jf > histo-superior-pooled-30mer.txt
```

![alt text][logo2]

[logo2]: https://github.com/kramundson/Potato-repeat-assembly/images/superior30.png "Superior 30mer counts"

As a comparison, do for 150PE reads of two haploid inducers, PL-4 and IVP-101, as well as another tetraploid, AlcaTarma.
This data was generated in-house and is described in the README.md file in the same path as each read file

```
ln -s /cato2pool/backup-share/kramundson/Potato_snp/data/161014_J00113_0191_AHFWKKBBXX_run377_2016-10-14_KA_1_2/demultiplex-only/KA2.fq ./
ln -s /cato2pool/backup-share/kramundson/Potato_snp/data/161014_J00113_0191_AHFWKKBBXX_run377_2016-10-14_KA_1_2/demultiplex-only/KA1.fq ./
ln -s /home/kramundson/Potato_snp/170203_J00113_0227_BHFWLYBBXX_run413_2017-02-03_H821P_Amundson/demultiplex-only/IVP-101.fq ./
```

AlcaTarma:
```
jellyfish count KA1.fq -m 30 -s 100M -t 4 -L 2 -o AlcaTarma-30mer-count.jf
jellyfish histo AlcaTarma-30mer-count.jf > histo-AlcaTarma-30mer.txt
```

![alt text][logo3]

[logo3]: https://github.com/kramundson/Potato-repeat-assembly/images/alcatarma30.png "AlcaTarma 30mer counts"

PL-4:
```
jellyfish count KA2.fq -m 30 -s 100M -t 4 -L 2 -o PL4-30mer-count.jf
jellyfish histo PL4-30mer-count.jf > histo-PL4-30mer.txt
```

![alt text][logo4]

[logo4]: https://github.com/kramundson/Potato-repeat-assembly/images/pl430.png "PL-4 30mer counts"

IVP-101:
```
jellyfish count IVP-101.fq -m 30 -s 100M -t 4 -L 2 -o IVP101-30mer-count.jf
jellyfish histo IVP101-30mer-count.jf > histo-IVP101-30mer.txt
```

![alt text][logo5]
[logo5]: https://github.com/kramundson/Potato-repeat-assembly/images/IVP10130.png "IVP-101 30mer counts"

##6. Compare histograms above with k=15 (isner)

Atlantic

```
jellyfish count atlantic-pooled.fastq -m 15 -s 100M -t 4 -L 2 -o atlantic-pooled-15mer-count.jf
jellyfish histo atlantic-pooled-15mer-count.jf > histo-atlantic-pooled-15mer.txt
```

![alt text][logo6]

[logo6]: https://github.com/kramundson/Potato-repeat-assembly/images/atlantic15.png "Atlantic 15mer counts"

Superior

```
jellyfish count atlantic-pooled.fastq -m 15 -s 100M -t 4 -L 2 -o superior-pooled-15mer-count.jf
jellyfish histo superior-pooled-15mer-count.jf > histo-superior-pooled-15mer.txt
```

![alt text][logo7]

[logo7]: https://github.com/kramundson/Potato-repeat-assembly/images/superior15.png "Superior 15mer counts"

AlcaTarma
```
jellyfish count KA1.fq -m 15 -s 100M -t 4 -L 2 -o AlcaTarma-15mer-count.jf
jellyfish histo AlcaTarma-15mer-count.jf > histo-AlcaTarma-15mer.txt
```

![alt text][logo8]

[logo8]: https://github.com/kramundson/Potato-repeat-assembly/images/alcatarma15.png "Alca Tarma 15mer counts"

IVP-101
```
jellyfish count IVP-101.fq -m 15 -s 100M -t 4 -L 2 -o IVP101-15mer-count.jf
jellyfish histo IVP101-15mer-count.jf > histo-IVP101-15mer.txt
```

![alt text][logo9]

[logo9]: https://github.com/kramundson/Potato-repeat-assembly/images/IVP10115.png

PL-4
```
jellyfish count KA2.fq -m 15 -s 100M -t 4 -L 2 -o PL4-15mer-count.jf
jellyfish histo PL4-15mer-count.jf > histo-PL4-15mer.txt
```

![alt text][logo10]

[logo10]: https://github.com/kramundson/Potato-repeat-assembly/images/pl415.png

Looks like filtering at 30nt will provide a cleaner set of high-copy kmers. Based on the high-coverage Atlantic data, I expect the homozygote peak to be dispersed.

TODO: determine filtering criteria

Another interesting question: Just how heterozygous is potato? At what value of k does the 15-mer type histogram start to look like the 30-mer type histogram?
Might this change be gradual or abrupt?
