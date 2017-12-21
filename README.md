# TeXP
TeXP is a pipeline to evaluate the transcription level of transposable elements in short read RNA-seq data

#About
TeXP is a pipeline for quantifying abundances of Transposable Elements transcripts from RNA-Seq data. TeXP is based on the assumption that RNA-seq reads overlapping Transposable Elements is a composition of pervasive transcription signal and autonomous transcription of Transposable Elements.

[[REF]]

# Requirements
 - Bowtie2 (2.3+)
 - Bedtools (2.26+)
 - Fastx-toolkit (0.0.14+)
 - perl (5.24+)
 - python (2.7)
 - R (3.3+)
  - Penalized package (0.49+)
 - samtools (1.3+)
 - wgsim (a12da33 on Oct 17, 2011)
---
 - Bowtie2 hg38 reference index (http://homes.gersteinlab.org/people/fn64/TeXP/rep_annotation.hg38.tar.bz2)
 - Hg38 repetitive element annotation (http://homes.gersteinlab.org/people/fn64/TeXP/rep_annotation.hg38.tar.bz2)
 
# Download
 $> git clone https://github.com/gersteinlab/texp.git

 Edit TeXP.sh and Update INSTALL_DIR variable to the path where TeXP was cloned 

# Docker image
docker pull fnavarro/texp
https://hub.docker.com/r/fnavarro/texp/ for futher instructions

# TeXP config
 A few paramaters must be set in order to work outside the docker enviroment; Parameters are set on opts.mk

 - LIBRARY_PATH: Should be pointing to TeXP library
 - EXT_LIBRARY_PATH: Path containing the bowtie2 reference index and Transposable element annotation bed file
 - EXE_DIR: Path to binaries


# Running TeXP
 $> ./TeXP.sh -f=[FILE_NAME] -t=[INT] -o=[OUTPUT_PATH] n=[SAMPLE_ID]

 -f: Input file (fastq,fastq.gz,sra)

 -t: Number of threads

 -o: Output path (i.e. ./ or ./processed)

 -n: Sample name (i.e. SAMPLE01)
 
 ---
 
 # Installing TeXP dependencies
apt-get update

- Install binaries dependencies

apt-get install -y \
	bedtools=2.26.0+dfsg-3 \
	bowtie2=2.3.0-2 \
	fastx-toolkit=0.0.14-3 \
	gawk=1:4.1.4+dfsg-1 \
	git \
	perl=5.24.1-3+deb9u1 \
	python=2.7.13-2 \
	r-base=3.3.3-1 \
	r-base-dev=3.3.3-1 \
	samtools=1.3.1-3 \
	wget 


- Install Wgsim

mkdir -p /src; \ 
	cd /src ; \
	git clone https://github.com/lh3/wgsim.git; \
	cd wgsim; \
	gcc -g -O2 -Wall -o wgsim wgsim.c -lz -lm; \
	mv wgsim /usr/bin/; \
	cd /;


- Download Libraries

Fix path (/data/library) to the a proper location at your computation enviroment

mkdir -p /data/library/rep_annotation; \
	cd /data/library/rep_annotation; \
	wget -c -t0 "http://homes.gersteinlab.org/people/fn64/TeXP/rep_annotation.hg38.tar.bz2" -O rep_annotation.hg38.tar.bz2; \
	tar xjvf rep_annotation.hg38.tar.bz2; \
	rm -Rf rep_annotation.hg38.tar.bz2
	
mkdir -p /data/library/bowtie2; \
	cd /data/library/bowtie2; \
	wget -c -t0 "http://homes.gersteinlab.org/people/fn64/TeXP/bowtie2.hg38.tar.bz2" -O bowtie2.hg38.tar.bz2; \
	tar xjvf bowtie2.hg38.tar.bz2; \
	rm -Rf bowtie2.hg38.tar.bz2



- Install R packages dependencies

echo 'install.packages(c("penalized"), repos="http://cloud.r-project.org", dependencies=TRUE)' > /tmp/packages.R \
    && Rscript /tmp/packages.R

