SRST2
====

Short Read Sequence Typing for Bacterial Pathogens

This program is designed to take Illumina sequence data, a MLST database and/or a database
of gene sequences (e.g. resistance genes, virulence genes, etc) and report the presence of
STs and/or reference genes.



Current release - v0.1.3 - Feb 6, 2014


Dependencies:

python (v2.7.5), scipy

bowtie2 v2.1.0     http://bowtie-bio.sourceforge.net/bowtie2/index.shtml

SAMtools v0.1.18   https://sourceforge.net/projects/samtools/files/samtools/0.1.18/ (NOTE 0.1.19 DOES NOT WORK)


Updates in v0.1.3

- Fixed a bug that occurred while trying to type genes from a user-supplied database (see issue #5, thanks to Scott Long)

- Fixed a bug in gene detection reporting - genes are now correctly reported by cluster, rather than by gene symbol (see issue #7)

- Added maximum divergence option for reporting (--max_divergence), default is now to report only hits with <10% divergence from the database (see issue #8)

- added parameter to pass to bowtie2 parameter '-u N' to stop mapping after the first N reads. Default behaviour remains to map all reads. However, for large read sets (e.g. >100x), extra reads do not help and merely increase the time taken for mapping and scoring, and you may want to limit to the first million reads (100x of a 2 Mbp genome) using '--stop_after 1000000'.



Authors - Michael Inouye, Harriet Dashnow, Bernie Pope, Kathryn Holt (University of Melbourne)
		
How to cite - Please check back for news on the paper. In the meantime, please cite "SRST2 - Short Read Sequence Typing for Bacterial Pathogens, http://katholt.github.io/srst2/"
		
Problems? - Email drkatholt@gmail.com. For updates, join the srst2 google group https://groups.google.com/forum/#!forum/srst2.


Installation
====
1 - Install dependencies first

2 - Get and install the code

Option 1:

Download the zip or tarball at the top left of this page and unpack it

Then install using pip (http://www.pip-installer.org/) or easy_install. 
You may need to use sudo to install centrally.

    pip install srst2-0.1.0-beta/

or

    easy_install srst2-0.1.0-beta/

Option 2:

Clone the git repository:

    git clone https://github.com/katholt/srst2
    
and then install with pip:

    pip install srst2/
    
OR do both at once:

    sudo pip install git+https://github.com/katholt/srst2

3 - Test that the programs are installed properly

    srst2 --version

    getmlst.py -h

    scores_vs_expected.py -h

    slurm_srst2.py -h

The downloaded directory also contains things that might be useful for srst2 users:

data/resistance.fasta contains a preliminary resistance sequence database that can be used for identifying resistance genes

database_clustering/ contains scripts and instructions for formatting of gene databases for use with srst2

example.txt contains a tutorial/example on running srst2 using public data


Basic usage - MLST
====

1 - Gather your input files:

(i) sequence reads (this example uses paired reads in gzipped fastq format, see below for options)

(ii) a fasta sequence database to match to. For MLST, this means a fasta file of all allele sequences. If you want to assign STs, you also need a tab-delim file which defines the ST profiles as a combination of alleles. You can retrieve these files automatically from pubmlst.org/data/ using the script provided:

getmlst.py --species "Escherichia coli"

2 - Run MLST:

srst2 --input_pe strainA_1.fastq.gz strainA_2.fastq.gz --output strainA_test --log 
	--mlst_db Escherichia_coli.fasta --mlst_definitions ecoli.txt

3 - Check the outputs:

(i) MLST results are output in: "strainA_test__mlst__Escherichia_coli__results.txt"

```
Sample  ST      adk     fumC    gyrB    icd     mdh     purA    recA    mismatches      uncertainty     depth

strainA     152     11      63      7       1       14      7       7                       25.8319955826
```

Basic usage - Resistance genes
====

1 - Gather your input files:

(i) sequence reads (this example uses paired reads in gzipped fastq format, see below for options)

(ii) a fasta sequence database to match to. For resistance genes, this means a fasta file of all the resistance genes/alleles that you want to screen for, clustered into gene groups. A suitable database, which combines sequences from ResFinder and CARD, is distributed with SRST2 (data/resistance.fasta).

2 - Run gene detection:

srst2 --input_pe strainA_1.fastq.gz strainA_2.fastq.gz --output strainA_test --log --gene_db resistance.fasta

3 - Check the outputs:

(i) Gene detection results are output in: "strainA_test__genes__resistance__results.txt"

```
Sample  aadA    dfrA    sul2    tet(B)

strainA     aadA1-5 dfrA1_1 sul2_2  tet(B)_4
```

All usage options
====

```
srst2 -h

SRST2 - Short Read Sequence Typer (v2)

optional arguments:
  -h, --help            show this help message and exit

  --version             show version number and exit 
  
  --input_se INPUT_SE [INPUT_SE ...]
                        Single end read file(s) for analysing (may be gzipped)
                        
  --input_pe INPUT_PE [INPUT_PE ...]
                        Paired end read files for analysing (may be gzipped)
                        
  --forward FORWARD     Designator for forward reads (only used if NOT in
                        MiSeq format sample_S1_L001_R1_001.fastq.gz; otherwise
                        default is _1, i.e. expect forward reads as
                        sample_1.fastq.gz)
                        
  --reverse REVERSE     Designator for reverse reads (only used if NOT in
                        MiSeq format sample_S1_L001_R2_001.fastq.gz; otherwise
                        default is _2, i.e. expect forward reads as
                        sample_2.fastq.gz)
                        
  --read_type {q,qseq,f}
                        Read file type (for bowtie2; default is q=fastq; other
                        options: qseq=solexa, f=fasta)
                        
  --mlst_db MLST_DB     Fasta file of MLST alleles (optional)
  
  --mlst_delimiter MLST_DELIMITER
                        Character(s) separating gene name from allele number
                        in MLST database (default "-", as in arcc-1)
                        
  --mlst_definitions MLST_DEFINITIONS
                        ST definitions for MLST scheme (required if mlst_db
                        supplied and you want to calculate STs)
                        
  --gene_db GENE_DB [GENE_DB ...]
                        Fasta file/s for gene databases (optional)
                        
  --no_gene_details     Switch OFF verbose reporting of gene typing
                                                
  --min_coverage MIN_COVERAGE
                        Minimum %coverage cutoff for gene reporting 
                        (default 90)
                        
  --max_divergence MAX_DIVERGENCE
                        Maximum %divergence cutoff for gene reporting 
                        (default 10)
                        
  --min_depth MIN_DEPTH
                        Minimum mean depth to flag as dubious allele call
                        (default 5)
                        
  --min_edge_depth MIN_EDGE_DEPTH
                        Minimum edge depth to flag as dubious allele call
                        (default 2)
                        
  --prob_err PROB_ERR   Probability of sequencing error (default 0.01)
  
  --stop_after STOP_AFTER
                        Stop mapping after this number of reads have been 
                        mapped (otherwise map all)
                        
  --other OTHER         Other arguments to pass to bowtie2
  
  --mapq MAPQ           Samtools -q parameter (default 1)
  
  --baseq BASEQ         Samtools -Q parameter (default 20)
  
  --output OUTPUT       Output file prefix
  
  --log            	Switch ON logging to file (otherwise log to stdout)
  
  --save_scores         Switch ON verbose reporting of all scores
  
  --use_existing_pileup
                        Use existing pileups if available, otherwise they will
                        be generated
                        
  --use_existing_scores
                        Use existing scores files if available, otherwise they
                        will be generated
                        
  --prev_output PREV_OUTPUT [PREV_OUTPUT ...]
                        SRST2 results files to compile (any new results from
                        this run will also be incorporated)
```

Input read formats and options
====

Any number of readsets can be provided using --input_se (for single end reads) and/or --input_pe (for paired end reads). You can provide both types of reads at once. Note however that if you do this, each readset will be typed one by one (in serial). So if it takes 2 minutes to type each read set, and you provide 10 read sets, it will take 20 minutes to get your results. The better way to proces lots of samples quickly is to give each one its own srst2 job (e.g. submitted simultaneously to your job scheduler or server); then compile the results into a single report using "srst2 --prev_output *results.txt --output all". That way each readset's 2 minutes of analysis is occurring in parallel on different nodes, and you'll get your results for all 10 samples in 2 minutes rather than 20.

### Read formats 
Reads can be in any format readable by bowtie2. The format is passed on to the bowtie2 command via the --read_type flag in srst2. The default format is fastq (passed to bowtie 2 as q); other options are qseq=solexa, f=fasta. So to use fasta reads, you would need to tell srst2 this via '--read_type f'.

Reads may be gzipped.

### Read names 
Srst2 can parse Illumina MiSeq reads files; we assume that files with names in the format 'XXX_S1_L001_R1_001.fastq.gz' and 'XXX_S1_L001_R2_001.fastq.gz' are the forward and reverse reads from a sample named 'XXX'. So, you can simply use 'srst2 --input_pe XXX_S1_L001_R1_001.fastq.gz XXX_S1_L001_R2_001.fastq.gz' and srst2 will recognise these as forward and reverse reads of a sample named XXX. If you have single rather than paired MiSeq reads, you would use 'srst2 --input_se XXX_S1_L001_R1_001.fastq.gz'.

### Paired reads
If you have paired reads that are named in some way other than the Illumina MiSeq format, e.g. from the SRA or ENA public databases, you need to tell srst2 how to pass these to bowtie2.
bowtie2 requires forward and reverse reads to be supplied in separate files, e.g strainA_1.fastq.gz and strainA_2.fastq.gz. srst2 attempts to sort reads supplied via --input_pe into read pairs, based on the suffix (_1, _2 in this example) that occurs before the file extension (.fastq.gz in this example). So if you supplied --input_pe strainA_1.fastq.gz strainB_1.fastq.gz strainA_2.fastq.gz strainB_2.fastq.gz, srst2 would sort these into two pairs (strainA_1.fastq.gz, strainA_2.fastq.gz) and (strainB_1.fastq.gz, strainB_2.fastq.gz) and pass each pair on to bowtie2 for mapping. By default, the suffixes are assumed to be "_1" for forward reads and "_2" for reverse reads, but you can tell srst2 if you have other conventions, via --forward and --reverse. E.g. if your files were named strainA_read1.fastq.gz and strainA_read2.fastq.gz, you would use these commands: --input_pe strainA_read1.fastq.gz strainA_read2.fastq.gz --forward _read1 --reverse _read2. 

### Sample names
Sample names are taken from the first part of the read file name (before the suffix if you have paired reads). E.g. 'strainA_1.fastq.gz' is assumed to belong to a sample called "strainA"; 'strainB_C_1.fastq.gz" would be assumed to belong to a sample called "strainB_C". These sample names will be used to name all output files, and will appear in the results files.


MLST Database format
====
MLST databases are specified by allele sequences, and a profiles table. These can be downloaded from the public databases, ready to use with srst2, using the provided script getmlst.py (see above).

### Allele sequences file, fasta format. 

--mlst_db alleles.fasta

This should contain ALL allele sequences for the MLST scheme; i.e. if there are 7 loci in the scheme, then the sequences of all alleles from all 7 loci should appear together in this file. If you have one file per locus, just cat these together first. If you use getmlst.py, this is done for you.

--mlst_delimiter '-'

The names of the alleles (i.e. the fasta headers) are critical for a functioning MLST scheme and therefore for correct calling of STs. There are two key components to every fasta header: the name of the locus (e.g. in E. coli these are adk, fumC, gyrB, icd, mdh, purA, recA) and the number assigned to the allele (1, 2, 3 etc). These are usually separated by a delimiter like '-' or '_'; e.g. in the E. coli scheme, alleles are named adk-1, fumC-10, etc. For ST calling to work properly, srst2 needs to know what the delimiter is. By default, we assume it is '-' as this is the most common; however some schemes use '_' (e.g. in the C. difficile scheme, the first allele is 'adk_1', so you would need to set --mlst_delimiter '_' on the srst2 command line). If you use getmlst.py, it will remind you of this and try to guess for you what the most likely delimiter is.

### MLST definitions file, tab delimited format.

--mlst_definitions

This is the file that tells you the ST number that is assigned to known combinations of alleles. Column 1 is the ST, and subsequent columns are the loci that make up the scheme. The names of these loci must match the allele names in the sequences database, e.g. adk, fumC, gyrB, icd, mdh, purA, recA in the E. coli scheme. If you download data from pubmlst this should not be a problem. Sometimes there are additional columns in this file, e.g. a column assigning STs to clonal complexes. srst2 will ignore any columns that don't match gene names found in the allele sequences file.

Gene databases
====

In addition to MLST, srst2 can do gene/allele detection. This works by mapping reads to each of the reference sequences in a fasta file(s) (provided through --gene_db) and reporting details of all genes that are covered above a certain level (--min_coverage, 90% by default). 

If the input database contains different alelles of the same gene, srst2 can report just the best matching allele for that gene (much like with MLST we report the best matching allele for each locus in the scheme). This makes the output manageable, as you will get one column per gene/locus (e.g. blaCTX-M) which reports the specific allele that was called in each sample (e.g. blaCTX-M-15 in sample A, blaCTX-M-13 in sample B).

We have provided a preliminary set of resistance genes in the /data directory of srst2, this is based on the ResFinder database and CARD. The fasta file (data/resistance.fasta) is ready for use with srst2.

You can however format any sequence set for screening with srst2. See instructions at the bottom of this page.

Output files
====

### MLST results

If MLST sequences and profiles were provided, STs will be printed in tab-delim format to a file called "[outputprefix]__mlst__[db]__results.txt", e.g.: "strainArun1__mlst__Escherichia_coli__results.txt".

The format looks like this:
```
Sample  ST      adk     fumC    gyrB    icd     mdh     purA    recA    mismatches      uncertainty     depth

strainA     1502    6       63      7       1       14      7       7                       12.3771855735
```

Each locus has a column in which the best scoring allele number is printed. 

\* indicates the best scoring allele has >=1 mismatch (SNP or indel, according to majority rules consensus of the aligned reads vs the allele sequence). Details of the mismatches are given in the mismatches column. This often means you have a novel allele. 

? indicates uncertainty in the result because the best scoring allele has some low-depth bases; either the the first or last 2 bases of the allele had <N reads mapped, or a truncation was called in which neigbhbouring bases were coverd with <N reads, or the average depth across the whole allele was <X. N is set by the parameter --min_edge_depth (default 2), X is set by --min_depth (default 5). The source of the uncertainty is printed to the uncertainty column. 

\- indicates that no allele could be assigned (generally means there were no alleles that were >90% covered by reads) 

If the combination of these alleles appears in the ST definitions file provided by --mlst_definitions, this ST will be printed in the ST column. "NF" indicates the allele combination was not found; "ND" indicates ST calculations were not done (because no ST definitions were provided). Here, * next to the ST indicates that there were mismatches against at least one of the alleles. This suggests that you have a novel variant of this ST rather than a precise match to this ST. ? indicates that there was uncertainty in at least one of the alleles. In all cases, the ST is calculated using the best scoring alleles, whether or not there are mismatches or uncertainty in those calls.

The *mismatches* column gives details of any mismatches (defined by majority rules consensus of the aligned reads vs the allele sequence) against the top scoring allele that is reported in the corresponding locus column. Possibilities are: (i) snps, adk-1/1snp indicates there was 1 SNP against the adk-1 allele; (ii) indels, adk-1/2indel indicates there were 1 indels (insertion or deletion calls in the alignment); (iii) holes, adk-1/5holes indicates there were 5 sections of the allele sequence that were not covered in the alignment and pileup (e.g. due to truncation at the start or end of the gene, or large deletions within the gene).

The *uncertainty* column gives details of parts of the top scoring alleles for which the depth of coverage was too low to give confidence in the result, this may be zero or any number up to the specified cutoff levels set via --min_edge_depth and --min_depth. Possibilities are considered in this order: (i) edge depth, adk-1/edge1.0 indicates that the mean read depth across either the first 2 or last 2 bases of the assigned allele was 1.0; this is monitored particularly because coverage at the ends of the allele sequences is dependent on bowtie2 to properly map reads that overhang the ends of the allele sequences, which is not as confident as when the whole length of a read maps within the gene (reported if this value is below the cutoff specified (default --min_edge_depth 2), low values can be interpreted as indicating uncertainty in the result as we can’t confidently distinguish alleles that differ at these low-covered bases); (ii) truncations, adk-1/del1.0 indicates that a truncation or large deletion was called for which the neighbouring 2 bases were covered to depth 1.0, this can be interpreted as indicating there is only very weak evidence for the deletion, as it is likely just due to random decline in coverage at this point (reported if this value is below the cutoff specified (default --min_edge_depth 2); (iii) average depth, adk-1/depth3.5 indicates that the mean read depth across the length of the assigned allele was 3.5 (reported if this value is below the cutoff specified, which by default is --min_depth 5)

The *depth* column indicates the mean read depth across the length of all alleles which were assigned a top scoring allele number (i.e. excluding any which are recorded as '-'). So if there are 7 loci with alleles called, this number represents the mean read depth across those 7 loci. If say, 2 of the 7 alleles were not called (recorded as ‘-’), the mean depth is that of the 5 loci that were called.

------------
### Gene typing

Gene typing results files report the details of sequences provided in fasta files via --genes_db that are detected above the minimum %coverage theshold set by --min_coverage (default 90).

Two output files are produced:

1 - A detailed report, [outputprefix]__fullgenes__[db]__results.txt, with one row per gene per sample:

```
Sample  DB      gene    allele  coverage        depth   diffs   uncertainty     cluster seqid   annotation

strainA     resistance      dfrA    dfrA1_1 100.0   6.79368421053           edge1.5 590     137     

strainA     resistance      aadA    aadA1-5 100.0   10.6303797468                   162     1631    

strainA     resistance      sul2    sul2_9  100.0   79.01992966                     265     1763    

strainA     resistance      blaTEM  blaTEM-1_5      100.0   70.8955916473                   258     1396    

strainA     resistance      tet(A)  tet(A)_4        97.6666666667   83.5831202046   28holes edge0.0 76      1208    

strainB     resistance      strB    strB1   100.0   90.0883054893                   282     1720    

strainB     resistance      strA    strA4   100.0   99.0832298137                   325     1142 
```

*coverage* indicates the % of the gene length that was covered (if clustered DB, then this is the highest coverage of any members of the cluster)

*uncertainty* is as above

2 - A tabulated summary report of samples x genes, [outputprefix]__genes__[db]__results.txt:

```
Sample  aadA    blaTEM  dfrA    strA    strB    sul2    tet(A)

strainA     aadA1-5 blaTEM-1_5      dfrA1_1?        -   -   sul2_9  tet(A)_4*?

strainB     -     -      -        strA4   strB1   -  -
```

The first column indicates the sample name, all other columns report the genes/alleles that were detected in the sample set. If multiple samples were input, or if previous outputs were provided for compiling results, then all the genes detected in ANY of the samples will have their own column in this table.

If you were using a clustered gene database (such as the resistance.fasta database provided), the name of each cluster (i.e. the basic gene symbol) will be printed in the column headers, while specific alleles will be printed in the sample rows.

\* indicates mismatches

? indicates uncertainty due to low depth in some parts of the gene

\- indicates the gene was not detected (> %coverage threshold, --min_coverage 90)

------------
### Combined results

If more then one database is provided for typing (via --mlst_db and/or --gene_db), or if previous results are provided for merging with the current run which contain data from >1 database (via --prev_output), then an additional table summarizing all the database results is produced. This is named "[outputprefix]__compiledResults.txt" and is a combination of the MLST style table plus the tabulated gene summary (file 2 above).

```
Sample  ST      adk     fumC    gyrB    icd     mdh     purA    recA    mismatches      uncertainty     depth   aadA    blaTEM  dfrA    strA    strB    sul2    tet(A)

sampleA     152*     11      63*      7       1       14      7       7                       21.3139900892   aadA1-5 blaTEM-1_5      dfrA1_1?        strA4   strB1   sul2_9  tet(A)_4*?
```

------------
### Mapping results

The bowtie2 alignment of reads to each input database is stored in [outputprefix]__[sample].[db].sorted.bam and the samtools pileup of the alignment is stored in [outputprefix]__[sample].[db].pileup. 

If you used --save_scores, a table of scores for each allele in the database is printed to [outputprefix]__[sample].[db].scores.


More basic usage examples
====

Run single read sets against MLST database and resistance database

srst2 --input_pe pool11_tag2_1.fastq.gz pool11_tag2_2.fastq.gz 
	--output pool11_tag2_Shigella --log 
	--gene_db /vlsci/VR0082/shared/srst2_sep/resistance.fasta 
	--mlst_db Escherichia_coli.fasta 
	--mlst_definitions ecoli.txt

------------

Run against multiple read sets in serial

srst2 --input_pe *.fastq.gz
	--output Shigella --log
	--gene_db /vlsci/VR0082/shared/srst2_sep/resistance.fasta 
	--mlst_db Escherichia_coli.fasta 
	--mlst_definitions ecoli.txt

------------

Run against new read sets, merge with previous reports (individual or compiled)

srst2 --input_pe strainsY-Z*.fastq.gz
	--output strainsA-Z --log
	--gene_db /vlsci/VR0082/shared/srst2_sep/resistance.fasta 
	--mlst_db Escherichia_coli.fasta 
	--mlst_definitions ecoli.txt
	--prev_output ShigellaA__genes__resistance__results.txt
          ShigellaA__mlst__Escherichia_coli__results.txt
          ShigellaB__genes__resistance__results.txt
          ShigellaB__mlst__Escherichia_coli__results.txt
          ShigellaC-X__compiledResults.txt

------------

Run against Enterococcus reads, where read names are different from the usual _1.fastq and _2.fastq

srst2 --input_pe strain_R1.fastq.gz strain_R2.fastq.gz 
	--forward _R1 --reverse _R2 
	--output strainA --log 
	--gene_db /vlsci/VR0082/shared/srst2_sep/resistance.fasta 
	--mlst_db Enterococcus_faecium.fasta 
	--mlst_definitions efaecium.txt
	
Compile results from completed runs
====

srst2 --prev_output *compiledResults.txt --output Shigella_report
 
Running lots of jobs and compiling results
====

Run against multiple read sets: submitting 1 job per readset to SLURM queueing system.

The results from all the separate runs can be compiled together using the above command.

slurm_srst2.py --script srst2 
	--output test 
	--input_pe *.fastq.gz 
	--other_args '--gene_db resistance.fasta 
	--mlst_db Escherichia_coli.fasta 
	--mlst_definitions ecoli.txt 
	--save_scores' 
	--walltime 0-1:0 
		> job_sub_list.txt

------------

SLURM job script usage options

```
slurm_srst2.py -h

optional arguments:
  -h, --help            show this help message and exit
  
  --walltime WALLTIME   wall time (default 0-0:30 = 30 minutes)
  
  --memory MEMORY       mem (default 4096 = 4gb)
  
  --rundir RUNDIR       directory to run in (default current dir)
  
  --script SCRIPT       SRST2 script (/vlsci/VR0082/shared/srst2_sep/srst2_150
                        9_reporting2.py)
                        
  --output OUTPUT       identifier for outputs (will be combined with read set
                        identifiers)
                        
  --input_se INPUT_SE [INPUT_SE ...]
                        Single end read file(s) for analysing (may be gzipped)
                        
  --input_pe INPUT_PE [INPUT_PE ...]
                        Paired end read files for analysing (may be gzipped)
                        
  --forward FORWARD     Designator for forward reads (only used if NOT in
                        MiSeq format sample_S1_L001_R1_001.fastq.gz; otherwise
                        default is _1, i.e. expect forward reads as
                        sample_1.fastq.gz)
                        
  --reverse REVERSE     Designator for reverse reads (only used if NOT in
                        MiSeq format sample_S1_L001_R2_001.fastq.gz; otherwise
                        default is _2, i.e. expect forward reads as
                        sample_2.fastq.gz)
                        
  --other_args OTHER_ARGS
                        string containing all other arguments to pass to srst2
```               
                        
Known issues
====

Reference indexing - srst2 uses bowtie2 for mapping reads to reference sequences. To do this, srst2 must first check the index exists, call bowtie2-build to generate the index if it doesn't already exist, and then call bowtie2 to map the reads to this indexed reference. Occasionallly bowtie2 will return an Error message saying that it doesn't like the index. This seems to be due to the fact that if you submit multiple srst2 jobs to a cluster at the same time, they will all test for the presence of the index and, if index files are present, will proceed with mapping... but this doesn't mean the indexing process is actually finished, and so errors will arise. The simple way out of this is, if you are running lots of srst2 jobs, FIRST index your reference(s) for bowtie2 and samtools (using 'bowtie2-build ref.fasta ref.fasta' and 'samtools faidx ref.fasta'), then submit your srst2 jobs. The slurm_srst2.py script takes care of this by formatting the databases before submitting any srst2 jobs.

Example - Shigella sonnei public data
====
This example uses real public data from three Shigella sonnei genomes, two of which were sequenced twice and so allow for reproducibility testing.

As Shigella sonnei is a sublineage of E. coli, we must use the E. coli MLST scheme.

We will also use the resistance genes database provided with srst2, resistance.fasta (modified from ResFinder).

### Download the files for the E. coli MLST scheme, using the script provided

    getmlst.py --species "Escherichia coli"

For SRST2, remember to check what separator is being used in this allele database

```
Looks like --mlst_delimiter '-'

adk-1 --> --> ('adk', '-', '1')
```

Note, this is correctly guessing that we should use the default --mlst_delimiter '-' with this database. The log file will tell you exactly what files were downloaded.

Check that the allele sequences have been downloaded and compiled into a single fasta file: Escherichia_coli#1.fasta

Check that the ST definitions have been downloaded: ecoli.txt

### Download Illumina paired end read sets from the ENA

Note if you have limited download capacity or time to spare, you could download the first two files only (total ~30MB) for basic testing of srst2.

strain 20081885 from ERP000182 (14M, 13M; 51M, 44M)

```
wget ftp://ftp.sra.ebi.ac.uk/vol1/fastq/ERR024/ERR024070/ERR024070_1.fastq.gz

wget ftp://ftp.sra.ebi.ac.uk/vol1/fastq/ERR024/ERR024070/ERR024070_2.fastq.gz

wget ftp://ftp.sra.ebi.ac.uk/vol1/fastq/ERR028/ERR028678/ERR028678_1.fastq.gz

wget ftp://ftp.sra.ebi.ac.uk/vol1/fastq/ERR028/ERR028678/ERR028678_2.fastq.gz
```

strain 20031275 from ERP000182 (89M, 94M; 88M, 74M)

```
wget ftp://ftp.sra.ebi.ac.uk/vol1/fastq/ERR024/ERR024082/ERR024082_1.fastq.gz

wget ftp://ftp.sra.ebi.ac.uk/vol1/fastq/ERR024/ERR024082/ERR024082_2.fastq.gz

wget ftp://ftp.sra.ebi.ac.uk/vol1/fastq/ERR028/ERR028690/ERR028690_1.fastq.gz

wget ftp://ftp.sra.ebi.ac.uk/vol1/fastq/ERR028/ERR028690/ERR028690_2.fastq.gz
```

strain IB694 from ERP000182 (80M, 64M)

```
wget ftp://ftp.sra.ebi.ac.uk/vol1/fastq/ERR024/ERR024619/ERR024619_1.fastq.gz

wget ftp://ftp.sra.ebi.ac.uk/vol1/fastq/ERR024/ERR024619/ERR024619_2.fastq.gz
```

### Try running MLST and gene detection against a tiny read set with low coverage (~3x)
    srst2 --input_pe ERR024070*.fastq.gz --output shigella1 --log --save_scores --mlst_db Escherichia_coli#1.fasta --mlst_definitions ecoli.txt --gene_db resistance.fasta

Note that, as the databases have not been indexed for bowtie2 yet, that srst2 has to run bowtie2-build which spits out a lot of messages to stdout. This is not a problem!

Bowtie2 also reports its mapping stats, it should indicate that there were only 400,882 reads in this data set, of which 0.02% mapped to our MLST/resistance loci. This probably won't be enough to get good quality information.

Check the log file (shigella1.log) to see what happened.

Outputs are printed to: 

shigella1__mlst__Escherichia_coli#1__results.txt - mlst results only 

shigella1__fullgenes__resistance__results.txt - resistance results, one line per gene

shigella1__genes__resistance__results.txt - resistance results only, tabulated 

shigella1__compiledResults.txt - MLST and resistance genes, tabulated 

shigella1.ERR024070.Escherichia_coli#1.scores - full score & alignment info for all MLST alleles

shigella1__ERR024070.resistance.scores - full score & alignment info for resistance genes with >90% coverage

Because we ran with the --save_scores flag, we have also got a scores file for each database, which details the score and mapping information for every allele in that database.

The ST was not called correctly because depth was too low (average read depth 2.19; this is printed in the MLST result table):

```
Sample  ST      adk     fumC    gyrB    icd     mdh     purA    recA    mismatches      uncertainty     depth

ERR024070       NF*?    1*?     63*?    7*?     220*?   14*?    7*?     148*?  [mismatches	[uncertainty]     2.182
```

Only 1 resistance gene was detected with >90% coverage (the default threshold), which looks like a variant of strB (marked with * to indicate it was not a precise match, and ? to indicate uncertainty because some bases weren't covered):

```
Sample  strB

ERR024070       strB1*?
```

The detailed report in shigella1__fullgenes__resistance__results.txt contains more info: 
```
Sample  DB      gene    allele  coverage        depth   diffs   uncertainty     cluster seqid   annotation

ERR024070       resistance      strB    strB1   95.1015531661   3.94353826851   3snp41holes     edge0.0 282     1720
```

If you look in the scores file (ERR024070.resistance.srst2.pileup.table.scores) you will see that two different alleles of strB were covered >90% (both at 95%) and at low depth (~3.9x). 

```
Allele  Score   Avg_depth       Edge1_depth     Edge2_depth     Percent_coverage        Size    Mismatches      Indels  Truncated_bases DepthNeighbouringTruncation     LeastConfident_Rate     LeastConfident_Mismatches       LeastConfident_Depth    LeastConfident_Pvalue

282__strB__strB3__1722  1.07   3.94   4.0     0.0     95.10   837     4       0       41      1       1.0     8       8       6.67e-17

282__strB__strB1__1720  0.82  3.94   4.0     0.0     95.10   837     3       0       41      1       0.33  2       6       0.0007
```

The Truncated_bases column shows 41, indicating there were 41 "holes" in the alignment of reads to these alleles. The next column, "DepthNeighbouringTruncation" contains 1, which tells us that the depth near these holes are 1x, indicating that they are likely to reflect random fluctuation in read depth rather than actual deletions.

This shows us that this level of data (which was considered a failed sequencing run) isn't enough to get good results. Luckily this genome was resequenced; let's see what happens when we analyse that data set.

### Run MLST and gene detection against a proper read set, and compile together with the results from the poor read set for comparison.

    srst2 --input_pe ERR028678*.fastq.gz --output shigella2 --log --mlst_db Escherichia_coli#1.fasta --mlst_definitions ecoli.txt --gene_db srst/resistance.fasta --prev_output shigella1__compiledResults.txt

Check the log file to see what happened.

Outputs are printed to: 

shigella2__mlst__Escherichia_coli#1__results.txt - mlst results only 

shigella2__fullgenes__resistance__results.txt - resistance results, one line per gene

shigella2__genes__resistance__results.txt - resistance results only, tabulated 

shigella2__compiledResults.txt - MLST and resistance genes, tabulated

This time there is a confident ST called, which should look like this:

```
Sample  ST      adk     fumC    gyrB    icd     mdh     purA    recA    mismatches      uncertainty     depth

ERR028678       152?    11?     63      7?      1       14      7       7               adk-11/edge2.0;gyrB-7/edge2.0   13.9307395171
```

There are also lots of resistance genes identified:

```
Sample  aac(3)-II       aadA    blaCTX-M        blaTEM  dfrA    strA    strB    sul2    tet(A)

ERR028678       aac(3)-IId*     aadA1-5 blaCTX-M-15_23  blaTEM-1_1      dfrA1_1 strA4   strB1   sul2_9  tet(A)_4*?
```

Because we supplied a prior results file, shigella2__compiledResults.txt contains the new results as well as the old results.

```
Sample  ST      adk     fumC    gyrB    icd     mdh     purA    recA    mismatches      uncertainty     depth   aac(3)-II       aadA    blaCTX-M        blaTEM  dfrA    strA    strB    sul2    tet(A)

ERR024070       NF*?    1*?     63*?    7*?     220*?   14*?    7*?     148*?   adk-1/1snp106holes;fumC-63/96holes;gyrB-7/2snp80holes;icd-220/1snp129holes;mdh-14/42holes;purA-7/52holes;recA-148/40holes       adk-1/edge0.0;fumC-63/edge0.0;gyrB-7/edge0.0;icd-220/edge0.0;mdh-14/edge1.0;purA-7/edge1.0;recA-148/edge0.0     2.1819729951    -       -       -       -       -       -       strB1*? -       -

ERR028678       152?    11?     63      7?      1       14      7       7       0       adk-11/edge2.0;gyrB-7/edge2.0   13.931  aac(3)-IId*     aadA1-5 blaCTX-M-15_23  blaTEM-1_1      dfrA1_1 strA4   strB1   sul2_9  tet(A)_4*?
```

Now try running on the other three read sets:
    srst2 --input_pe ERR024082*.fastq.gz ERR028690*.fastq.gz ERR024619*.fastq.gz --output shigella3 --log --mlst_db Escherichia_coli#1.fasta --mlst_definitions ecoli.txt --gene_db srst/resistance.fasta

Now compile the results from all 5 read sets:
    srst2 --output all --prev_output shigella2__compiledResults.txt shigella3__compiledResults.txt

This outputs a file, all__compiledResults.txt, containing the compilation of all results for MLST and resistance genes across the 5 strains.

Generating SRST2-compatible clustered database from raw sequences
====

### Gene database format
In addition to MLST, srst2 can do gene/allele detection. This works by mapping reads to each of the reference sequences in a fasta file(s) (provided through --gene_db) and reporting details of all genes that are covered above a certain level (--min_coverage, 90% by default). 

If the input database contains different alelles of the same gene, srst2 can report just the best matching allele for that gene (much like with MLST we report the best matching allele for each locus in the scheme). This makes the output manageable, as you will get one column per gene/locus (e.g. blaCTX-M) which reports the specific allele that was called in each sample (e.g. blaCTX-M-15 in sample A, blaCTX-M-13 in sample B).

To do this properly, srst2 needs to know which of the reference sequences are alleles of the same gene. This is done by adhering to the following format in the naming of the sequences (i.e. the headers in the fasta sequence for the database):

\>[clusterUniqueIdentifier]__[clusterSymbol]__[alleleSymbol]__[alleleUniqueIdentifier]

e.g. in the resistance gene database provided, the first entry is:

\>344__blaOXA__blaOXA-181__1

Note these are separated by two underscores. The individual components are:

clusterUniqueIdentifier = 344;  unique identifier for this cluster (uniquely identifes the cluster)
clusterSymbol = blaOXA;  gene symbol for this cluster (may be shared by multiple clusters)
alleleSymbol = blaOXA-181;  full name of this allele
alleleUniqueIdentifier = 1;  uniquely identifies the sequence

Ideally the alleleSymbol would be unique (as it is in the reference.fasta file provided). However it doesn't have to be: if allele symbols are not unique, then srst2 will use the combination '[alleleSymbol]__[alleleUniqueIdentifier]' to  uniquely identify the sequence in the resulting reports, so that you can trace exactly which sequence was present in each sample.

Additional gene annotation can appear on the header line, after a space. This additional info will be printed in the full genes report, but not in the compiled results files.

e.g. for the blaOXA sequence above, the full header is actually:

\>344__blaOXA__blaOXA-181__1 blaOXA-181_1_HM992946; HM992946; betalactamase


### Sourcing suitable gene databases

To get started, we have provided a resistance gene database (data/resistance.fasta) and code (database_clustering/) to extract virulence factors for a genus of interest from the Virulence Factor DB (detailed instructions below).

If you want to use your own database of allele sequences, with the reporting behaviour described, you will need to assign your sequences to clusters and use this header format. To facilitate this, use the scripts provided in the database_clustering directory provided with srst2, and follow the instructions below.

You can also use unclustered sequences. This is perfectly fine for gene detection applications, where you have one representative allele sequence for each gene, and you simply want to know which samples contain a sequence that is similar to this one (e.g. detecting plasmid replicons, where there is one target sequence per replicon). However, this won't work well for allele typing. If the sequence database contains multiple allele sequences for the same gene, then all of these that are covered above the length threshold (default 90%) will be reported in the output, which makes for messy reporting. If you do this, you would probably find it most useful to look at the full gene results table rather than looking at the compiled results output.

### If you already know which alleles belong to the same gene family and should be clustered for reporting:

Use this info to generate the appropriate headers for srst2 to read. The provided script, csv_to_gene_db.py can help:

```
csv_to_gene_db.py -h

Usage: csv_to_gene_db.py [options]

Options:

  -h, --help            show this help message and exit
  
  -t TABLE_FILE, --table=TABLE_FILE
                        table to read (csv)
                        
  -o OUTPUT_FILE, --out=OUTPUT_FILE
                        output file (fasta)
                        
  -s SEQ_COL, --seq_col=SEQ_COL
                        column number containing sequences
                        
  -f FASTA_FILE, --fasta=FASTA_FILE
                        fasta file to read sequences from (must specify which
                        column in the table contains the sequence names that
                        match the fasta file headers)
                        
  -c HEADERS_COL, --headers_col=HEADERS_COL
                        column number that contains the sequence names that
                        match the fasta file headers
                        
```

The input table should be comma-separated (csv, unix newline characters) and have these columns:

seqID,clusterid,gene,allele,(DNAseq),other....

which will be used to make headers of the required form [clusterID]__[gene]__[allele]__[seqID] [other stuff]

If you have the sequences as a column in the table, specify which column they are in using -s:
csv_to_gene_db.py -t genes.csv -o genes.fasta -s 5

Alternatively, if you have sequences in a separate fasta file, you can provide this file via -f. You will also need to have a column in the table that links the rows to unique sequences, specify which column this is using -c:
csv_to_gene_db.py -t genes.csv -o genes.fasta -f rawseqs.fasta -c 5


### Clustering sequences

If your sequences are not already assigned to gene clusters, you can do this automatically using CD-HIT (http://weizhong-lab.ucsd.edu/cd-hit/).

1 - Run CD-HIT to cluster the sequences at 90% nucleotide identity:

    cdhit-est -i rawseqs.fasta -o rawseqs_cdhit90 -d 0 > rawseqs_cdhit90.stdout

2 - Parse the cluster output and tabulate the results, check for inconsistencies between gene names and the sequence clusters, and generate individual fasta files for each cluster to facilitate further checking:

    python cdhit_to_csv.py --cluster_file rawseqs_cdhit90.clstr --infasta raw_sequences.fasta --outfile rawseqs_clustered.csv

For comparing gene names to cluster assignments, this script assumes very basic nomenclature of the form gene-allele, ie a gene symbol followed by '-' followed by some more specific allele designation. E.g. adk-1, blaCTX-M-15. The full name of the gene (adk-1, blaCTX-M-15) will be stored as the allele, and the bit before the '-' will be stored as the name of the gene cluster (adk, blaCTX). This won't always give you exactly what you want, because there really are no standards for gene nomenclature! But it will work for many cases, and you can always modify the script if you need to parse names in a different way. Note though that this only affects how sensible the gene cluster nomenclature is going to be in your srst2 results, and will not affect the behaviour of the clustering (which is purely sequence based using cdhit) or srst2 (which will assign a top scoring allele per cluster, the cluster name may not be perfect but the full allele name will always be reported anyway).

3 - Convert the resulting csv table to a sequence database using:

    csv_to_gene_db.py -t rawseqs_clustered.csv -o seqs_clustered.fasta -f rawseqs.fasta -c 4

The output file, seqs_clustered.fasta, should now be ready to use with srst2 (--gene_db seqs_clustered.fasta).

If there are potential inconsistencies detected at step 2 above (e.g. multiple clusters for the same gene, or different gene names within the same cluster), you may like to investigate further and change some of the cluster assignments or cluster names. You may find it useful to generate neighbour joining trees for each cluster that contains >2 genes, using align_plot_tree_min3.py

### Screening for resistance genes with SRST2
A preliminary set of resistance genes is in the /data directory of srst2, this is based on the ResFinder database and CARD. The fasta file is ready for use with SRST2. The CSV table contains the same sequence information, but in tabular format for easier parsing/editing.

An easy way to add sequences to this database would be to add new rows to the table, and then generate an updated fasta file using:

    csv_to_gene_db.py -t rawseqs_clustered.csv -o seqs_clustered.fasta -s rawseqs.fasta -c 5

### Using the VFBD Virulence Factor Database with SRST2

The VFDB houses sets of virulence genes for a range of bacterial genera, see http://www.mgc.ac.cn/VFs/.

To type these virulence genes using SRST2, download the full set of sequences from the VFDB website (http://www.mgc.ac.cn/VFs/Down/CP_VFs.ffn.gz) and follow these steps to generate SRST2-compatible files for your genus of interest.

1 - Extract virulence genes by genus from the main VFDB file, CP_VFs.ffn:

    python VFDBgenus.py --infile CP_VFs.ffn --genus Clostridium

or, to get all availabel genera in separate files:

    python VFDBgenus.py --infile CP_VFs.ffn

2 - Run CD-HIT to cluster the sequences for this genus, at 90% nucleotide identity:

    cd-hit -i Clostridium.fsa -o Clostridium_cdhit90 -c 0.9 > Clostridium_cdhit90.stdout

3 - Parse the cluster output and tabulate the results using the specific Virulence gene DB compatible script:

    python VFDB_cdhit_to_csv.py --cluster_file Clostridium_cdhit90.clstr --infile Clostridium.fsa --outfile Clostridium_cdhit90.csv

4 - Convert the resulting csv table to a SRST2-comptaible sequence database using:

    python csv_to_gene_db.py -t Clostridium_cdhit90.csv -o Clostridium_VF_clustered.fasta -s 5
    
The output file, Clostridium_VF_clustered.fasta, should now be ready to use with srst2 (--gene_db Clostridium_VF_clustered.fasta).
