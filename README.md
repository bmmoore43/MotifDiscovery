# MotifDiscovery
Original pipeline used python for processing dataframes and R for running Random Forest. For those instructions see bottom of the document (Python & R Pipeline)

New pipeline uses SciPy to determine enrichment, Numpy and Pandas for dataframe management, and SciKit Learn for RandomForest

## Updates to the Pipeline:
Nov 25 2015 : Simplify and standardize outputs (gives stdev and SE), also changed the random_state generator in RandomForestClassifier from an int to the default which is np.random.

Nov 24 2015 : Make Fisher's Exact Test 1-Tailed (only looking at enrichment in positive class) & remove Training/Testing data split since using cross-validation

Nov 13 2015 : Alter script so that enriched kmers are lengthened by 1 bp until they are no longer enriched

Oct 26 2015 : Switch from Python+R Pipeline to running everything in Python, this included changing to include reverse complement information, and to run the ML using 20 sets of random negative example genes. 

## How to Run New Pipeline
1. Anytime you log in to HPC and want to use the pipeline you have to first run:
    - export   PATH=/mnt/home/azodichr/miniconda3/bin:$PATH
2. Import positive example and negative example FASTA files - the pipeline finds enriched kmers, lengthens kmers if possible, and runs 20 RandomForest models each with a different random negative set so that the ML is balanced. 
    - python /mnt/home/azodichr/GitHub/MotifDiscovery/RandomForest_v2.0.py -pos [FASTA FILE] -neg [FASTA FILE] -k /mnt/home/azodichr/ML_Python/6mers.txt (or 5mers.txt) -imp yes -save NAME -pval 0.01

Example of short runcc.txt file to submit to hpc:
cd /mnt/home/azodichr/01_DualStress_At/12_RF_Python/13_OneTailed/01_p01; export PATH=/mnt/home/azodichr/miniconda3/bin:$PATH;python /mnt/home/azodichr/GitHub/MotifDiscovery/RandomForest_v2.0.py -pos /mnt/home/azodichr/01_DualStress_At/12_RF_Python/01_Upreg_6merGrow/Fasta_Files/UNU_k1_promoter.fa -neg /mnt/home/azodichr/01_DualStress_At/12_RF_Python/01_Upreg_6merGrow/Fasta_Files/NNN_1000_promoter.fa -save UNU_k1_pro -k /mnt/home/azodichr/01_DualStress_At/12_RF_Python/01_Upreg_6merGrow/6mers.txt -imp yes -pval 0.01
cd /mnt/home/azodichr/01_DualStress_At/12_RF_Python/13_OneTailed/01_p01; export PATH=/mnt/home/azodichr/miniconda3/bin:$PATH;python /mnt/home/azodichr/GitHub/MotifDiscovery/RandomForest_v2.0.py -pos /mnt/home/azodichr/01_DualStress_At/12_RF_Python/01_Upreg_6merGrow/Fasta_Files/UNU_k1_3UTR.fa -neg /mnt/home/azodichr/01_DualStress_At/12_RF_Python/01_Upreg_6merGrow/Fasta_Files/NNN_1000_3UTR.fa -save UNU_k1_3UTR -k /mnt/home/azodichr/01_DualStress_At/12_RF_Python/01_Upreg_6merGrow/6mers.txt -imp yes -pval 0.01
cd /mnt/home/azodichr/01_DualStress_At/12_RF_Python/13_OneTailed/01_p01; export PATH=/mnt/home/azodichr/miniconda3/bin:$PATH;python /mnt/home/azodichr/GitHub/MotifDiscovery/RandomForest_v2.0.py -pos /mnt/home/azodichr/01_DualStress_At/12_RF_Python/01_Upreg_6merGrow/Fasta_Files/UNU_k1_int1.fa -neg /mnt/home/azodichr/01_DualStress_At/12_RF_Python/01_Upreg_6merGrow/Fasta_Files/NNN_1000_int1.fa -save UNU_k1_int1 -k /mnt/home/azodichr/01_DualStress_At/12_RF_Python/01_Upreg_6merGrow/6mers.txt -imp yes -pval 0.01




#Python & R Pipeline
Pairwise_kmers.py: Contains functions to make lists of paired kmers, make data frames of what genes contain those motifs, and run Fisher's Exact test to determine enrichment of those kmers/kmer pairs in the positive genes. 
RandomForest.R: Runs Random Forest on input dataframe. 10 replicates and 10 fold cross validation. 

## What you need:
•	File with all your positive examples (naming scheme will be based off the name of the positive example file, so make sure that makes sense and isn't too long)
•	File with all your negative examples
•	Fasta file with all gene promoter regions. 

If you already have fasta files of positive and neg examples, skip steps 2-3.

In HPC load:  Python3, Biopython, and SciPy

Scripts: /mnt/home/azodichr/GitHub/MotifDiscovery/

## Set Up Your Files:
1. Inside directory for Pairwise experiment make directories for FASTA files and Motif Lists:
  - mkdir FastaFiles
  - mkdir MotifLists

2. Put cluster file in FastaFiles dir and get promoter sequence:
  - cd FastaFiles/
  - cp [pos_examples] .
  - python /mnt/home/shius/codes/FastaManager.py -f getseq2 -fasta [promoter sequences] -name [pos examples]
  - *For arabidopsis you can use: /mnt/home/azodichr/01_DualStress_At/TAIR10_upstream1000_Alex_20101104.mod.fa*

3. Put negative example file in FastaFiles dir and get promoter sequence. 
  - cp [neg_examples] .     #For Random Forest you want a 1:1 ratio of positive and negative examples, if 73 genes in cluster, randomly select 73 negative examples.
  - python /mnt/home/shius/codes/FastaManager.py -f getseq2 -fasta [promoter sequences] -name [neg examples]
  - *For arabidopsis you can use: /mnt/home/azodichr/01_DualStress_At/TAIR10_upstream1000_Alex_20101104.mod.fa*

4. Make singleton and paired kmer list. Reverse complement sequences are separated by “.”, pairs separated by a space (k = length of kmer you want):
  - cd ../MotifLists
  - python Pairwise_kmers.py -f make_pairs2 –k 5
  - python Pairwise_kmers.py -f make_pairs2 –k 6


## Make presence/absence dataframes and do enrichment
The work flow here is 1) Make df with all kmers/pairs. 2) Make list of enriched kmers/pairs. 3) Remake df with just those enriched kmers/pairs.

1. Make data frame with presence or absence of all kmer/kmer pair:
  - python Pairwise_kmers.py -f make_df –k [ListOfKmers] -p [positive fasta files] -n [negative fasta files]
  - *If you want to add DNA structure information to your prediction ask me. It didn't add much to my prediction...*

2. If you want all the motifs move on to step 4, otherwise parse your motifs using Fisher's Exact Test:
  - python Pairwise_kmers.py -f parse_df –df [output df from step 5]
  OPTIONAL: -pval <Default is 0.05>

3. Re-make data frame with only enriched motifs:
  - python Pairwise_kmers.py -f make_df –k [output from step 6, ending in: “_sig_0.05.txt”] -p [positive fasta files] -n [negative fasta files]

4. Run Random Forest in R. If randomForest is not in your library yet, see the end of this document.
  - export R_LIBS_USER=~/R/library
  - R --vanilla --slave --args [df*] < RandomForest.R
  - *Can use output df from step 1 or 3*

This will output two files:
  - .imp.txt: Open in exel, sort by "Mean Decrease Accuracy" - Make sure you shift the column headers over by one- they skip the motif name heading...
  - .Results.txt: Output with F-measure, stdev, sterror, and 95% confidence intervals.



### Getting RandomForest onto HPC:
  - Rscript -e "install.packages(‘LIBRARY_NAME',lib='~/R/library',contriburl=contrib.url('http://cran.r-project.org/'))”
  - export R_LIBS_USER=~/R/library      *you will need to run this line every time you run RandomForest.R*
  - R
  - Sys.getenv("R_LIBS_USER")
  - library(“LIBRARY_NAME")
  - q()
