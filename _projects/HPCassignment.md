---
name: HPC assignment
tools: [Processing, Arduino(C/C++)]
image:
description: Parallel approach for RNA-Seq pipeline to analyze samples of Bladder Cancer
---
# Parallel approach for RNA-Seq pipeline

Course: [High Performance Computing](https://en.unav.edu/web/masters-degree-in-biomedical-engineering/study-program)

#### Proposed problem...
Dear Tecnun HPC consultants,
My name is Geoff Bennet and I am a researcher from the University of Dublin. Over the last years, I have acquired the knowledge to work as a bioinformatician, but still lack all the parts related with usage of clusters, servers and work using parallel techniques to speed up our projects.
We are currently running a complete RNA-Seq pipeline to analyze samples of Bladder Cancer. The samples are available for download via SRA (BioProject PRJNA186504). We have already selected the samples that we are interested in analyzing.

Our pipeline consists of the following steps:
1) Download all the samples using the SRA Toolkit
2) Running quality controls on the .fastq files with the fastqc algorithm
3) Count the reads in each .fastq file (awk)
4) Align the reads to the human genome (hg38)
  - Using STAR
5) Quantify genes by creating a count table
  - Using htseq (Python dependency)
6) Quantify transcripts using Kallisto

In order to run this pipeline, we have different alternatives regarding computational power:
1. A local computer server (Storm)
2. A large-scale cluster (UNAV cluster)
The first option does not have any cost related because we own the machine, but the other option has a cost depending on the time we use the cluster.
With this information we would like to ask you to prepare the following information:

Is it possible to run some of the steps using a parallel approach? Our current pipeline uses for loops in a serial style and it takes a lot of time before completion.
- Also, we would like to know (in general terms) how much resources are required for the different steps. The number of processors, the amount of RAM memory and the required storage space.
- Based on your analysis, which would be the recommended workflow for our pipeline? In other words, we are interested to know where (which cluster) to run each step and why. There is no problem regarding the time taken to transfer data between servers or computers.


#### Project's presentation
<iframe src="https://drive.google.com/file/d/1Gn_kU6Uy8M4Mzd0aKzgs_GQfpQ_ZhJEq/preview" width="640" height="480" allow="autoplay"></iframe>

#### Code
```bash
#!/bin/bash
#############################################################################################################################
## HPC Final Work.
## - Detailed Script for all required steps
#############################################################################################################################

#!/bin/bash
#SBATCH --job-name=HPC_Project
#SBATCH --cpus-per-task=8
#SBATCH -N 1 ## Just one node
#SBATCH --mem=40G
#SBATCH --mail-type=NONE
#SBATCH --mail-user=ccastilla.1@tecnun.es
#SBATCH -o /home/ingbio/logFiles/HPC_Project.log
#SBATCH -p medium


## Load programs here
module load FastQC/0.11.8-Java-1.8
module load STAR/2.7.0d-foss-2018b
module load SAMtools/1.9-foss-2018b
module load R/4.0.0-foss-2018b
module load Python/2.7.15-foss-2018b
module load HTSeq/0.11.0-foss-2018b-Python-2.7.15
module load BEDTools/2.27.1-foss-2018b
module load MACS2/2.1.0.20151222-foss-2018b-Python-2.7.15
module load SRA-Toolkit/2.10.3-centos_linux64
module load kallisto/0.46.0-foss-2018b
##### module load Salmon/1.2.0-foss-2018b

##########################
## 1) Download Samples
##########################

## Install SRA Toolkit

## This lines are used to install the required toolkit. Although it can be also loaded as a module.
cd /home/ingbio/Data/HPC/Software/
##### wget http://ftp-trace.ncbi.nlm.nih.gov/sra/sdk/current/sratoolkit.current-centos_linux64.tar.gz
##### tar -zxvf ./sratoolkit.current-centos_linux64.tar.gz


## Get samples
## Read the .txt and get the sample identifiers
Samples=($(cat /home/ingbio/Data/HPC/UsedSamples.txt))
echo ${Samples[*]}


mkdir /home/ingbio/Data/HPC/Samples


for i in ${Samples[@]}
do
	echo ${i}
	# Absolute path to the fastq-dump command
	fasterq-dump --outdir /home/ingbio/Data/HPC/Samples/ -p -L debug -e 8 ${i}
	# PIGZ is a parallel version of gzip. With the -p option we set the number of cores
	/home/ingbio/Data/HPC/Software/pigz-2.4/pigz /home/ingbio/Data/HPC/Samples/${i}_1.fastq -k -p 8 --force
	## Repeat for the _2.fastq file
	/home/ingbio/Data/HPC/Software/pigz-2.4/pigz /home/ingbio/Data/HPC/Samples/${i}_2.fastq -k -p 8 --force
done

## A folder with .sra files is created. We remove them from our folder.
rm -rf /home/ingbio/ncbi


##########################
## 1) Quality Controls
##########################

## 1.a) FASTQC
#################

## Create output directory
mkdir /home/ingbio/Data/HPC/FastQC

## for loop to run fastqc an all samples
for i in ${Samples[@]}
do
		echo ${i}
		# -o is the directory where to save the output
        fastqc /home/ingbio/Data/HPC/Samples/${i}_1.fastq.gz -o /home/ingbio/Data/HPC/FastQC
        fastqc /home/ingbio/Data/HPC/Samples/${i}_2.fastq.gz -o /home/ingbio/Data/HPC/FastQC
done

## remove .zip files, only keep .html output
rm -rf /home/ingbio/Data/HPC/FastQC/*.zip



## 1.b) Count number of reads per fastq file
###################################################

## Create a two column .txt separated by tab to store sample name and reads
echo -e "Sample\tReads" > /home/ingbio/Data/HPC/Fastq_ReadCounts.txt

for i in ${Samples[@]}
do
		echo ${i}
        reads=$(zcat -cd /home/ingbio/Data/HPC/Samples/${i}_1.fastq.gz | awk 'BEGIN{RS="@"}{}END{print NR}')
        echo -e "${i}_1.fastq.gz\t${reads}" >> /home/ingbio/Data/HPC/Fastq_ReadCounts.txt
        reads=$(zcat -cd /home/ingbio/Data/HPC/Samples/${i}_2.fastq.gz | awk 'BEGIN{RS="@"}{}END{print NR}')
        echo -e "${i}_2.fastq.gz\t${reads}" >> /home/ingbio/Data/HPC/Fastq_ReadCounts.txt
done


##########################
## 2) Alignment
##########################

## 2.a) Generate Genome Index (STAR)
###################################

## Create directory to store Genome Index
mkdir /home/ingbio/Data/HPC/GenomeIndex
mkdir -p /home/ingbio/Data/HPC/GenomeIndex/STAR


## Run STAR to generate index
STAR --runThreadN $SLURM_CPUS_PER_TASK-2 --runMode genomeGenerate --genomeDir /home/ingbio/Data/HPC/GenomeIndex/STAR --genomeFastaFiles  /home/ingbio/Data/HPC/Genomes/Homo_sapiens.GRCh38.dna.primary_assembly.fa  --sjdbGTFfile /home/ingbio/Data/HPC/Genomes/Homo_sapiens.GRCh38.92.gtf --sjdbOverhang 100


#2.b) Run Alignment (STAR)
###################################

## Create directory to store BAM Files
mkdir -p /home/ingbio/Data/HPC/BAMS/STAR

for i in ${Samples[@]}
do
	STAR --runThreadN $SLURM_CPUS_PER_TASK-2 --outSAMtype BAM SortedByCoordinate --outFileNamePrefix /home/ingbio/Data/HPC/BAMS/STAR/${i} --outSAMattributes NH HI AS nM XS --genomeDir /home/ingbio/Data/HPC/GenomeIndex/STAR --readFilesCommand zcat --readFilesIn /home/ingbio/Data/HPC/Samples/${i}_1.fastq.gz /home/ingbio/Data/HPC/Samples/${i}_2.fastq.gz

done



## 2.c) Generate Genome Index (Hisat2)
###################################

## With hisat2 there is no need to create an index. We just download a pre-made version
mkdir -p /home/ingbio/Data/HPC/GenomeIndex/Hisat2
cd /home/ingbio/Data/HPC/GenomeIndex/Hisat2

### wget ftp://ftp.ccb.jhu.edu/pub/infphilo/hisat2/data/grch38.tar.gz
tar -xzvf grch38.tar.gz

## Module load or full route
## module load HISAT2/2.1.0-foss-2018b

## 2.d) Run Alignment (Hisat2)
###################################

## Create directory to save output
##### mkdir -p /home/ingbio/Data/HPC/BAMS/Hisat2

##### for i in ${Samples[@]}
##### do
##### 	/home/ingbio/Data/HPC/Software/hisat2-2.1.0/hisat2 -p $SLURM_CPUS_PER_TASK -x /home/ingbio/Data/HPC/GenomeIndex/Hisat2/grch38/genome -1 /home/ingbio/Data/HPC/Samples/${i}_1.fastq.gz -2 /home/ingbio/Data/HPC/Samples/${i}_2.fastq.gz -S /home/ingbio/Data/HPC/BAMS/Hisat2/${i}.sam

	# Hisat returns .sam files as output. Using samtools we convert them to .bam and then remove the .sam
##### 	samtools view -bS /home/ingbio/Data/HPC/BAMS/Hisat2/${i}.sam | samtools sort -n > /home/ingbio/Data/HPC/BAMS/Hisat2/${i}.bam
#####	rm -rf /home/ingbio/Data/HPC/BAMS/Hisat2/${i}.sam

##### done

cd /home/ingbio/Data/HPC/

##########################
## 3) Quantification
##########################

## Create directory to store quantification output
mkdir -p /home/ingbio/Data/HPC/Quantification/

## 3.a) Using Rsubread (R Script)
###################################

## Export Folder With R Packages
## export R_LIBS=/home/ingbio/R/
##### Rscript /home/ingbio/Data/HPC/RScript_Rsubread.R

## 3.b) Using HT-Seq
###################################

## Count reads using HT-Seq
mkdir -p /home/ingbio/Data/HPC/Quantification/Htseq



for i in ${Samples[@]}
do
	# For htseq we need to specify if BAMS are sorted by read name or by genomic position
	# STAR produces BAMS sorted by coordinate so we specify -r pos
	# Hisat produces BAMS sorted by name (After samtools pre-process) so we specify -r name
	htseq-count -f bam -s yes -i gene_id -r name /home/ingbio/Data/HPC/BAMS/Hisat2/${i}.bam /home/ingbio/Data/HPC/Genomes/Homo_sapiens.GRCh38.92.gtf > /home/ingbio/Data/HPC/Quantification/Htseq/${i}_HTSeqGenes_count.txt
done


## 3.c) Kallisto
###################################

## Install Kallisto

cd /home/ingbio/Data/HPC/


## wget https://github.com/pachterlab/kallisto/releases/download/v0.44.0/kallisto_linux-v0.44.0.tar.gz
## tar -zxvf /home/ingbio/Data/HPC/kallisto_linux-v0.44.0.tar.gz
## mv /home/ingbio/Data/HPC/kallisto_linux-v0.44.0 /home/ingbio/Data/HPC/Kallisto


## Generate Index
mkdir -p /home/ingbio/Data/HPC/GenomeIndex/Kallisto
cd /home/ingbio/Data/HPC/GenomeIndex/Kallisto
## wget https://ftp.ensembl.org/pub/release-92/fasta/homo_sapiens/cdna/Homo_sapiens.GRCh38.cdna.all.fa.gz --no-check-certificate



kallisto index -i /home/ingbio/Data/HPC/GenomeIndex/Kallisto/Hg38_ensembl92.idx /home/ingbio/Data/HPC/GenomeIndex/Kallisto/Homo_sapiens.GRCh38.cdna.all.fa.gz


## Quantification
mkdir -p /home/ingbio/Data/HPC/Quantification/Kallisto
cd /home/ingbio/Data/HPC/Quantification/Kallisto


for i in ${Samples[@]}
do

	kallisto quant -i /home/ingbio/Data/HPC/GenomeIndex/Kallisto/Hg38_ensembl92.idx -o /home/ingbio/Data/HPC/Quantification/Kallisto/${i}.kallisto_out -b 100 -t $SLURM_CPUS_PER_TASK  /home/ingbio/Data/HPC/Samples/${i}_1.fastq.gz /home/ingbio/Data/HPC/Samples/${i}_2.fastq.gz

done


## 3.d) Salmon
###################################

## Download last version of Salmon
cd /home/ingbio/Data/HPC/Software

# wget https://github.com/COMBINE-lab/salmon/releases/download/v1.4.0/salmon-1.4.0_linux_x86_64.tar.gz
# tar -zxvf salmon-1.4.0_linux_x86_64.tar.gz


##### salmon index --index /home/ingbio/Data/HPC/GenomeIndex/salmon/ --transcripts /home/ingbio/Data/HPC/GenomeIndex/Kallisto/Homo_sapiens.GRCh38.cdna.all.fa.gz


##### salmon quant -i /home/ingbio/Data/HPC/GenomeIndex/salmon/ \
#####			 -l A \
#####			 -p $SLURM_CPUS_PER_TASK \
#####			 -1 /home/ingbio/Data/HPC/Samples/${i}_1.fastq.gz -2 /home/ingbio/Data/HPC/Samples/${i}_2.fastq.gz \
#####			 -o /home/ingbio/Data/HPC/Quantification/salmon/${i}.salmon_out \
#####			 --seqBias \
#####			 --useVBOpt \
#####			 --validateMappings
```
