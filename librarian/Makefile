PIPELINE_NAME     := TeXP_librarian

LIBRARY_PATH     := /home2/fn64/projects/TeXP/library
EXE_DIR          := ~/tools/manual/

BOWTIE_BIN       := $(EXE_DIR)/bin/bowtie2
BOWTIE_PARAMS    := --sensitive-local -N1 --no-unal
BOWTIE_INDEX     := "/home2/fn64/genomes/Homo_sapiens/hg38/toBowtie2/hg38"

R_BIN            := $(EXE_DIR)/bin/R
SAMTOOLS_BIN     := $(EXE_DIR)/bin/samtools
BEDTOOLS_BIN     := $(EXE_DIR)/bin/bedtools
WGSIM_BIN        := $(EXE_DIR)/bin/wgsim
PYTHON_BIN       := /usr/bin/python

##
## Simulation parameters
##
ERROR_RATE          := 0.1
NUMBER_OF_READS     ?= 100000
NUMBER_OF_LOOPS     ?= 100
MEAN_READ_LEN       := 75

OUTPUT              ?= ./

ELE_REGEX ?= $(ELE_NAME)

USAGE := "make -f Makefile \
	ELE_NAME=[required: Element name LINE1] \
  	REFERENCE_GENOME=[required: absolute path to reference genome .fa] \
  	REPEAT_MASKER_BED=[required: absolute path to repetitive element bed file (.bed)] \
  	ELE_REGEX=[required: Regular expression to select desired elements in the bed file] \
 	ELE_LENGTH=[required: Repetitive element full length]"

ifndef ELE_NAME
$(error Required parameter ELE_NAME missing. Usage: $(USAGE))
endif
ifndef REFERENCE_GENOME
$(error Required parameter REFERENCE_GENOME missing. Usage: $(USAGE))
endif
ifndef REPEAT_MASKER_BED
$(error Required parameter REPEAT_MASKER_BED missing. Usage: $(USAGE))
endif
ifndef ELE_LENGTH
$(error Required parameter ELE_LENGTH missing. Usage: $(USAGE))
endif

LOG_FILE := $(ELE_NAME).log

## Define current time
timestamp := `/bin/date "+%Y-%m-%d(%H:%M:%S)"`

##
## Main make target
##
.PHONY: all
.DEFAULT: all
.INTERMEDIATE: $(OUTPUT)/$(ELE_NAME)/*.ref.bed.tmp

all: create_ref

##
## Compile a sub-bed file containing only the desired elements.
##
$(OUTPUT)/$(ELE_NAME)/ref/$(ELE_NAME).bed: $(REPEAT_MASKER_BED)
	@echo -e "\n\n\n$(timestamp) **************************************************************" >> $(ELE_NAME).log
	@echo -e "$(timestamp) $(PIPELINE_NAME): Creating library folder: ./$(ELE_NAME)" >> $(ELE_NAME).log
	@mkdir -p $(OUTPUT)/$(ELE_NAME)/ref/
	@mkdir -p $(OUTPUT)/$(ELE_NAME)/info/
	@echo -e "$(timestamp) $(PIPELINE_NAME): Creating sub bed file containing only the element of interestet:" >> $(ELE_NAME).log
	egrep -e "$(ELE_REGEX)" $(REPEAT_MASKER_BED) > $(OUTPUT)/$(ELE_NAME)/ref/$(ELE_NAME).bed

##
## Compile a sub-bed file containing only the desired elements.
##
$(OUTPUT)/$(ELE_NAME)/$(ELE_NAME).bed.elements: $(OUTPUT)/$(ELE_NAME)/ref/$(ELE_NAME).bed
	@echo -e "$(timestamp) $(PIPELINE_NAME): Elements to be processed:" >> $(ELE_NAME).log
	@cat $(OUTPUT)/$(ELE_NAME)/ref/$(ELE_NAME).bed | awk '{print $$NF}' | sort | uniq >> $(ELE_NAME).log 
	@cat $(OUTPUT)/$(ELE_NAME)/ref/$(ELE_NAME).bed | awk '{print $$NF}' | sort | uniq > $(OUTPUT)/$(ELE_NAME)/$(ELE_NAME).bed.elements
	@echo -e "$(timestamp) $(PIPELINE_NAME): *** WARNING ***:" >> $(ELE_NAME).log
	@echo -e "$(timestamp) $(PIPELINE_NAME): If there are undesired elements, remove it manually from ./$(OUTPUT)/$(ELE_NAME)/$(ELE_NAME).bed.elements and rerun librarian Makefile:" >> $(ELE_NAME).log
	@echo -e "$(timestamp) $(PIPELINE_NAME): ***         ***:" >> $(ELE_NAME).log

##
## Count number of events and number of bases for each subfamily.
##
$(OUTPUT)/$(ELE_NAME)/$(ELE_NAME).summary: $(OUTPUT)/$(ELE_NAME)/$(ELE_NAME).bed.elements
	@echo -e "$(timestamp) $(PIPELINE_NAME): Creating element summary:" >> $(ELE_NAME).log
	cat $(OUTPUT)/$(ELE_NAME)/ref/$(ELE_NAME).bed | sort -k4,4 | awk 'BEGIN{first=1} {if ( first == 1 ) {id=$$4;first=0}; if ( id != $$4 ) {print id,count,sum;id=$$4;sum=0;count=0}; sum+=($$3-$$2);count+=1}; END{print id,count,sum;}' | fgrep -w -f $(OUTPUT)/$(ELE_NAME)/$(ELE_NAME).bed.elements > $(OUTPUT)/$(ELE_NAME)/$(ELE_NAME).summary

##
## Calculate the proportion of bases for each subfamily.
##
$(OUTPUT)/$(ELE_NAME)/ref/$(ELE_NAME).ref.bases: $(OUTPUT)/$(ELE_NAME)/$(ELE_NAME).bed.elements $(OUTPUT)/$(ELE_NAME)/$(ELE_NAME).summary
	@echo -e "$(timestamp) $(PIPELINE_NAME): Calculating the proportion of bases of each subfamily:" >> $(ELE_NAME).log
	@echo -e "$(ELE_NAME)_Subfamily\t$(ELE_NAME)_Ref_bases" > $(OUTPUT)/$(ELE_NAME)/ref/$(ELE_NAME).ref.bases
	SUM=$$(cat $(OUTPUT)/$(ELE_NAME)/$(ELE_NAME).summary | awk '{sum+=$$3} END{print sum}'); \
	cat $(OUTPUT)/$(ELE_NAME)/$(ELE_NAME).summary | awk -v sum=$$SUM '{print $$1"\t"$$3/sum}' >> $(OUTPUT)/$(ELE_NAME)/ref/$(ELE_NAME).ref.bases

##
## Fetch bases the the reference genome and dump into the ref.fa file.
##
$(OUTPUT)/$(ELE_NAME)/ref/$(ELE_NAME).ref.fa: $(OUTPUT)/$(ELE_NAME)/$(ELE_NAME).bed.elements
	@echo -e "$(timestamp) $(PIPELINE_NAME): Fetching the reference sequence:" >> $(ELE_NAME).log
	@mkdir -p $(OUTPUT)/$(ELE_NAME)/seqs/

	for element in $$(cat $(OUTPUT)/$(ELE_NAME)/$(ELE_NAME).bed.elements); do \
	cat $(REPEAT_MASKER_BED) | awk -v element=$$element '{if ($$4 == element) {print}}' > $(OUTPUT)/$(ELE_NAME)/$$element.ref.bed.tmp; \
	$(BEDTOOLS_BIN) getfasta -fi $(REFERENCE_GENOME) -bed $(OUTPUT)/$(ELE_NAME)/$$element.ref.bed.tmp -fo $(OUTPUT)/$(ELE_NAME)/seqs/$$element.ref.fasta.tmp -s -name; \
	done

	@cat $(OUTPUT)/$(ELE_NAME)/seqs/*.ref.fasta.tmp > $(OUTPUT)/$(ELE_NAME)/ref/$(ELE_NAME).ref.fa
	rm -Rf $(OUTPUT)/$(ELE_NAME)/*.ref.bed.tmp

##
## Fetch bases the the reference genome and dump into the ref.fa file.
##
$(OUTPUT)/$(ELE_NAME)/info/copies_bases.plot.pdf: $(OUTPUT)/$(ELE_NAME)/$(ELE_NAME).summary
	sed 's/TEMPLATE/$(ELE_NAME)/g' templates/copies_bases.R.template > $(OUTPUT)/$(ELE_NAME)/info/$(ELE_NAME)_copies_bases.R
	$(R_BIN) --no-restore --no-save --args $(OUTPUT)/$(ELE_NAME)/$(ELE_NAME).summary $(OUTPUT)/$(ELE_NAME)/info/$(ELE_NAME)_copies_bases.plot.pdf < $(OUTPUT)/$(ELE_NAME)/info/$(ELE_NAME)_copies_bases.R >> $(ELE_NAME).log

##
## Faster simulation of reference reads.
##
$(OUTPUT)/$(ELE_NAME)/simu/general.count: $(OUTPUT)/$(ELE_NAME)/ref/$(ELE_NAME).ref.fa
	@echo -e "======================\n" >> $(ELE_NAME).log
	@echo -e "$(timestamp) $(PIPELINE_NAME): The profile for this study was not found at: $(LIBRARY_PATH)/LTR/$(NUMBER_OF_READS)_$(MEAN_READ_LEN)_$(ERROR_RATE).txt\n" >> $(ELE_NAME).log
	@echo -e "$(timestamp) $(PIPELINE_NAME): Simulating reads with length equal to $(MEAN_READ_LEN)\n" >> $(ELE_NAME).log
	@echo -e "$(timestamp) $(PIPELINE_NAME): Creating reads from based on $(ELE_NAME) reference sequence:\n" >> $(ELE_NAME).log
	mkdir -p $(OUTPUT)/$(ELE_NAME)/simu/
	@for iter in $(shell seq 1 $(NUMBER_OF_LOOPS) ); do \
		$(WGSIM_BIN) -S $$(date "+%N") -1 $(MEAN_READ_LEN) -N $(NUMBER_OF_READS) -d0 -r$(ERROR_RATE) -e 0 -R 0 $(OUTPUT)/$(ELE_NAME)/ref/$(ELE_NAME).ref.fa $(OUTPUT)/$(ELE_NAME)/simu/$(NUMBER_OF_READS)_$(MEAN_READ_LEN)_$(ERROR_RATE)_$$iter.simu /dev/null > /dev/null 2> /dev/null ; \
	done
	@echo -e "$(timestamp) $(PIPELINE_NAME): Aligning simulated reads to the reference genome:\n" >> $(ELE_NAME).log
	@for iter in $(shell seq 1 $(NUMBER_OF_LOOPS) ); do \
		$(BOWTIE_BIN) -p 1 $(BOWTIE_PARAMS) -x $(BOWTIE_INDEX) -U $(OUTPUT)/$(ELE_NAME)/simu/$(NUMBER_OF_READS)_$(MEAN_READ_LEN)_$(ERROR_RATE)_$$iter.simu 2>> $(LOG_FILE) | $(SAMTOOLS_BIN) view -Sb - 2>> $(LOG_FILE) > $(OUTPUT)/$(ELE_NAME)/simu/$(NUMBER_OF_READS)_$(MEAN_READ_LEN)_$(ERROR_RATE)_$$iter.simu.bam; \
		$(SAMTOOLS_BIN) sort -@1 -m 4G $(OUTPUT)/$(ELE_NAME)/simu/$(NUMBER_OF_READS)_$(MEAN_READ_LEN)_$(ERROR_RATE)_$$iter.simu.bam -o $(OUTPUT)/$(ELE_NAME)/simu/$(NUMBER_OF_READS)_$(MEAN_READ_LEN)_$(ERROR_RATE)_$$iter.simu.sorted.bam; \
		rm -R $(OUTPUT)/$(ELE_NAME)/simu/$(NUMBER_OF_READS)_$(MEAN_READ_LEN)_$(ERROR_RATE)_$$iter.simu.bam; \
		$(BEDTOOLS_BIN) intersect -f 0.75 -a $(OUTPUT)/$(ELE_NAME)/simu/$(NUMBER_OF_READS)_$(MEAN_READ_LEN)_$(ERROR_RATE)_$$iter.simu.sorted.bam -b $(OUTPUT)/$(ELE_NAME)/ref/$(ELE_NAME).bed -sorted -bed -wo > $(OUTPUT)/$(ELE_NAME)/simu/$(NUMBER_OF_READS)_$(MEAN_READ_LEN)_$(ERROR_RATE)_$$iter.simu.sorted.bam.bed; \
		cat $(OUTPUT)/$(ELE_NAME)/simu/$(NUMBER_OF_READS)_$(MEAN_READ_LEN)_$(ERROR_RATE)_$$iter.simu.sorted.bam.bed | awk -F "[$$\t ]" '{print $$4,$$(NF-1)}' | sed 's/_[0-9]*_[0-9]*_[0-9]*[:][0-9]*[:][0-9]*_[0-9]*[:][0-9]*[:][0-9]*[_][0-9a-z]*[/][12]//g' | sort -k1,1 -k2,2 | uniq -c > $(OUTPUT)/$(ELE_NAME)/simu/$(NUMBER_OF_READS)_$(MEAN_READ_LEN)_$(ERROR_RATE)_$$iter.simu.sorted.bam.bed.tmpcount; \
	done
	cat $(OUTPUT)/$(ELE_NAME)/simu/$(NUMBER_OF_READS)_$(MEAN_READ_LEN)_$(ERROR_RATE)_*.simu.sorted.bam.bed.tmpcount | sort -k2,2 -k3,3 | sed 's/^[ ]*//g' | awk 'BEGIN{first=1} {if ( first == 1 ) {id=$$2"*"$$3;first=0}; if ( id != $$2"*"$$3 ) {print id,sum/$(NUMBER_OF_LOOPS);id=$$2"*"$$3;sum=0;count=0}; sum+=$$1;count++}; END{print id,sum/$(NUMBER_OF_LOOPS);}' > $@

##
## Edit mismapping summary (Review?)
##
$(OUTPUT)/$(ELE_NAME)/info/mismapping.txt: $(OUTPUT)/$(ELE_NAME)/simu/general.count
	@cat $(OUTPUT)/$(ELE_NAME)/simu/general.count  | sed 's/[*]/ /g' > $(OUTPUT)/$(ELE_NAME)/info/mismapping.txt

##
## Plot element mismapping heatmap.
##
$(OUTPUT)/$(ELE_NAME)/info/mismapping.pdf: $(OUTPUT)/$(ELE_NAME)/info/mismapping.txt
	sed 's/TEMPLATE/$(ELE_NAME)/g' templates/heatmap.crossmapping.R.template > $(OUTPUT)/$(ELE_NAME)/info/$(ELE_NAME)_heatmap.crossmapping.R
	$(R_BIN) --no-restore --no-save --args $(OUTPUT)/$(ELE_NAME)/info/mismapping.txt $(OUTPUT)/$(ELE_NAME)/info/mismapping.pdf < $(OUTPUT)/$(ELE_NAME)/info/$(ELE_NAME)_heatmap.crossmapping.R >> $(ELE_NAME).log

##
## Simulate a specific element.
##
$(OUTPUT)/$(ELE_NAME)/simu/%.slow_count: $(OUTPUT)/$(ELE_NAME)/seqs/%.ref.fasta.tmp
	@echo -e "======================\n" >> $(ELE_NAME).log
	@echo -e "$(timestamp) $(PIPELINE_NAME): The profile for this study was not found at: $(LIBRARY_PATH)/LTR/$(NUMBER_OF_READS)_$(MEAN_READ_LEN)_$(ERROR_RATE).txt\n" >> $(ELE_NAME).log
	@echo -e "$(timestamp) $(PIPELINE_NAME): Simulating reads with length equal to $(MEAN_READ_LEN)\n" >> $(ELE_NAME).log
	@echo -e "$(timestamp) $(PIPELINE_NAME): Creating reads from based on $(ELE_NAME) reference sequence:\n" >> $(ELE_NAME).log
	mkdir -p $(OUTPUT)/$(ELE_NAME)/simu/
	@for iter in $(shell seq 1 $(NUMBER_OF_LOOPS) ); do \
		$(WGSIM_BIN) -S $$(date "+%N") -1 $(MEAN_READ_LEN) -N $(NUMBER_OF_READS) -d0 -r$(ERROR_RATE) -e 0 -R 0 $< $@_$(NUMBER_OF_READS)_$(MEAN_READ_LEN)_$(ERROR_RATE)_$$iter.simu /dev/null > /dev/null 2> /dev/null ; \
	done
	@echo -e "$(timestamp) $(PIPELINE_NAME): Aligning simulated reads to the reference genome:\n" >> $(ELE_NAME).log
	@for iter in $(shell seq 1 $(NUMBER_OF_LOOPS) ); do \
		$(BOWTIE_BIN) -p 1 $(BOWTIE_PARAMS) -x $(BOWTIE_INDEX) -U $@_$(NUMBER_OF_READS)_$(MEAN_READ_LEN)_$(ERROR_RATE)_$$iter.simu 2>> $(LOG_FILE) | $(SAMTOOLS_BIN) view -Sb - 2>> $(LOG_FILE) > $@_$(NUMBER_OF_READS)_$(MEAN_READ_LEN)_$(ERROR_RATE)_$$iter.simu.bam; \
		$(SAMTOOLS_BIN) sort -@1 -m 4G $@_$(NUMBER_OF_READS)_$(MEAN_READ_LEN)_$(ERROR_RATE)_$$iter.simu.bam $@_$(NUMBER_OF_READS)_$(MEAN_READ_LEN)_$(ERROR_RATE)_$$iter.simu.sorted; \
		rm -R $@_$(NUMBER_OF_READS)_$(MEAN_READ_LEN)_$(ERROR_RATE)_$$iter.simu.bam; \
		$(BEDTOOLS_BIN) intersect -f 0.75 -a $@_$(NUMBER_OF_READS)_$(MEAN_READ_LEN)_$(ERROR_RATE)_$$iter.simu.sorted.bam -b $(OUTPUT)/$(ELE_NAME)/ref/$(ELE_NAME).bed -sorted -bed -wo > $@_$(NUMBER_OF_READS)_$(MEAN_READ_LEN)_$(ERROR_RATE)_$$iter.simu.sorted.bam.bed; \
		cat $@_$(NUMBER_OF_READS)_$(MEAN_READ_LEN)_$(ERROR_RATE)_$$iter.simu.sorted.bam.bed | awk -F "[$$\t ]" '{print $$4,$$(NF-1)}' | sed 's/_[0-9]*_[0-9]*_[0-9]*[:][0-9]*[:][0-9]*_[0-9]*[:][0-9]*[:][0-9]*[_][0-9a-z]*[/][12]//g' | sort -k1,1 -k2,2 | uniq -c > $@_$(NUMBER_OF_READS)_$(MEAN_READ_LEN)_$(ERROR_RATE)_$$iter.simu.sorted.bam.bed.tmpcount; \
	done
	cat $@_$(NUMBER_OF_READS)_$(MEAN_READ_LEN)_$(ERROR_RATE)_*.simu.sorted.bam.bed.tmpcount | sort -k2,2 -k3,3 | sed 's/^[ ]*//g' | awk 'BEGIN{first=1} {if ( first == 1 ) {id=$$2"*"$$3;first=0}; if ( id != $$2"*"$$3 ) {print id,sum/$(NUMBER_OF_LOOPS);id=$$2"*"$$3;sum=0;count=0}; sum+=$$1;count++}; END{print id,sum/$(NUMBER_OF_LOOPS);}' > $@

##
## Call slow but detailed simulation for all elements.
##
$(OUTPUT)/$(ELE_NAME)/info/$(ELE_NAME)_mismapping_slow.txt: $(OUTPUT)/$(ELE_NAME)/$(ELE_NAME).bed.elements
	for element in $$(cat $(OUTPUT)/$(ELE_NAME)/$(ELE_NAME).bed.elements); do \
		make -f Makefile ELE_NAME=$(ELE_NAME) REFERENCE_GENOME=$(REFERENCE_GENOME) $(OUTPUT)/$(ELE_NAME)/simu/$$element.count; \
	done
	@cat $(OUTPUT)/$(ELE_NAME)/simu/*.slow_count  | sed 's/[*]/ /g' > $(OUTPUT)/$(ELE_NAME)/info/$(ELE_NAME)_mismapping_slow.txt

##
## Plot slow but detailed simulation for all elements.
##
$(OUTPUT)/$(ELE_NAME)/info/$(ELE_NAME)_mismapping_slow.pdf: $(OUTPUT)/$(ELE_NAME)/info/$(ELE_NAME)_mismapping_slow.txt templates/heatmap.crossmapping.R.template
	sed 's/TEMPLATE/$(ELE_NAME)/g' templates/heatmap.crossmapping.R.template > $(OUTPUT)/$(ELE_NAME)/info/$(ELE_NAME)_heatmap.crossmapping.R
	$(R_BIN) --no-restore --no-save --args $(OUTPUT)/$(ELE_NAME)/info/$(ELE_NAME)_mismapping_slow.txt $(OUTPUT)/$(ELE_NAME)/info/$(ELE_NAME)_mismapping_slow.pdf < $(OUTPUT)/$(ELE_NAME)/info/$(ELE_NAME)_heatmap.crossmapping.R >> $(ELE_NAME).log

##
## Generate alignment quality heatmap summary.
##
$(OUTPUT)/$(ELE_NAME)/info/heatmap.alignmentqual.txt: $(OUTPUT)/$(ELE_NAME)/simu/general.count
	@for bamfile in $$(ls --color=never $(OUTPUT)/$(ELE_NAME)/simu/*.bam); do \
		samtools view $$bamfile; \
	done | awk '{print $$1,$$5}' | sed 's/_[0-9]*_[0-9]*_[0-9]*[:][0-9]*[:][0-9]*_[0-9]*[:][0-9]*[:][0-9]*[_][0-9a-z]*[/][12]//g' | awk '{print $$1,int($$2/10)}' | sort -k1,1 -k2,2n | uniq -c > $(OUTPUT)/$(ELE_NAME)/info/heatmap.alignmentqual.txt

##
## Plot alignment quality heatmap.
##
$(OUTPUT)/$(ELE_NAME)/info/heatmap.alignmentqual.pdf: $(OUTPUT)/$(ELE_NAME)/info/heatmap.alignmentqual.txt
	sed 's/TEMPLATE/$(ELE_NAME)/g' templates/heatmap.alignmentqual.R.template > $(OUTPUT)/$(ELE_NAME)/info/$(ELE_NAME)_heatmap.alignmentqual.R
	$(R_BIN) --no-restore --no-save --args $(OUTPUT)/$(ELE_NAME)/info/heatmap.alignmentqual.txt $(OUTPUT)/$(ELE_NAME)/info/heatmap.alignmentqual.pdf < $(OUTPUT)/$(ELE_NAME)/info/$(ELE_NAME)_heatmap.alignmentqual.R >> $(ELE_NAME).log

##
## Create generic sub makefile.
##
$(OUTPUT)/$(ELE_NAME)/ref/$(ELE_NAME).makefile.sub: templates/makefile_rule_template
	sed 's/TEMPLATE/$(ELE_NAME)/g;s/REGEX/$(ELE_REGEX)/g' templates/makefile_rule_template > $(OUTPUT)/$(ELE_NAME)/ref/$(ELE_NAME).makefile.sub 

##
## Create generic Mixed Membership Model script.
##
#DEPRECATED - 2017-02-14 - Only lasso script is generated
#$(OUTPUT)/$(ELE_NAME)/ref/$(ELE_NAME).mixedMembership.R: templates/mixedMembership.R.template
#	sed 's/TEMPLATE/$(ELE_NAME)/g;s/TEMP_LENGTH/$(ELE_LENGTH)/g' templates/mixedMembership.R.template > $(OUTPUT)/$(ELE_NAME)/ref/$(ELE_NAME).mixedMembership.R

##
## Create generic Ridge regression script.
##
#DEPRECATED - 2017-02-14 - Only lasso script is generated
#$(OUTPUT)/$(ELE_NAME)/ref/$(ELE_NAME).ridge.R: templates/ridge.R.template
#	sed 's/TEMPLATE/$(ELE_NAME)/g;s/TEMP_LENGTH/$(ELE_LENGTH)/g' templates/ridge.R.template > $(OUTPUT)/$(ELE_NAME)/ref/$(ELE_NAME).ridge.R

##
## Create generic LASSO regression script.
##
$(OUTPUT)/$(ELE_NAME)/ref/$(ELE_NAME).lasso.R: templates/lasso.R.template
	sed 's/TEMPLATE/$(ELE_NAME)/g;s/TEMP_LENGTH/$(ELE_LENGTH)/g' templates/lasso.R.template > $(OUTPUT)/$(ELE_NAME)/ref/$(ELE_NAME).lasso.R

#DEPRECATED - 2017-02-14 - Only lasso script is generated
#create_ref: $(OUTPUT)/$(ELE_NAME)/$(ELE_NAME).summary $(OUTPUT)/$(ELE_NAME)/ref/$(ELE_NAME).ref.bases $(OUTPUT)/$(ELE_NAME)/ref/$(ELE_NAME).ref.fa $(OUTPUT)/$(ELE_NAME)/ref/$(ELE_NAME).makefile.sub $(OUTPUT)/$(ELE_NAME)/ref/$(ELE_NAME).mixedMembership.R $(OUTPUT)/$(ELE_NAME)/ref/$(ELE_NAME).ridge.R $(OUTPUT)/$(ELE_NAME)/ref/$(ELE_NAME).lasso.R
create_ref: $(OUTPUT)/$(ELE_NAME)/$(ELE_NAME).summary $(OUTPUT)/$(ELE_NAME)/ref/$(ELE_NAME).ref.bases $(OUTPUT)/$(ELE_NAME)/ref/$(ELE_NAME).ref.fa $(OUTPUT)/$(ELE_NAME)/ref/$(ELE_NAME).makefile.sub $(OUTPUT)/$(ELE_NAME)/ref/$(ELE_NAME).lasso.R
create_plots: $(OUTPUT)/$(ELE_NAME)/info/copies_bases.plot.pdf $(OUTPUT)/$(ELE_NAME)/info/heatmap.alignmentqual.pdf $(OUTPUT)/$(ELE_NAME)/info/mismapping.pdf
