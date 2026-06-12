FASTA_URL:= https://ftp.ncbi.nlm.nih.gov/genomes/all/GCF/000/848/505/GCF_000848505.1_ViralProj14703/GCF_000848505.1_ViralProj14703_genomic.fna.gz
GFF_URL:= https://ftp.ncbi.nlm.nih.gov/genomes/all/GCF/000/848/505/GCF_000848505.1_ViralProj14703/GCF_000848505.1_ViralProj14703_genomic.gff.gz

#The name of the genome
GENOME_NAME:= ebola

#The paths to the genome FASTA and GFF files
GENOME_FASTA:= $(GENOME_NAME).fna
GENOME_GFF:= $(GENOME_NAME).gff

#SRR ID of the sample
SRR=SRR1553425

#The name of the alignment BAM file
BAM=bam/$(SRR).bam

#The name of the VCF output file
VCF=vcf/$(SRR).vcf.gz

#The number of reads to download
LIMIT=100000

#The path to the reads files
R1_READS=reads/$(SRR)_1.fastq
R2_READS=reads/$(SRR)_2.fastq

#----------------------------No change below this line----------------------------------------

help: 
	@echo "olvasd a makefile-t!"

#Make the directories that are needed, then download the genome FASTA and GFF files
download:
	mkdir -p data
	wget -O "data/$(GENOME_FASTA).gz" "$(FASTA_URL)"
	wget -O "data/$(GENOME_GFF).gz" "$(GFF_URL)"
	gunzip -f "data/$(GENOME_FASTA).gz"
	gunzip -f "data/$(GENOME_GFF).gz"

#Download the reads from SRA database, limit to the LIMIT reads use the fastq-dump command from the SRA toolkit
fastq:
	mkdir -p reads
	fastq-dump -X $(LIMIT) --outdir reads  --split-files $(SRR)

#Index the reference genome using bwa index
index:
	bwa index data/$(GENOME_FASTA)

#Align the reads to the reference genome using bwa mem, then sort and index the BAM file using samtools
align:
	mkdir -p bam
	bwa mem data/$(GENOME_FASTA) $(R1_READS) $(R2_READS) | samtools view -bS -> $(BAM)
	samtools sort $(BAM) -o $(BAM).sorted.bam
	samtools index $(BAM).sorted.bam
	samtools flagstat $(BAM).sorted.bam

vcf:
	mkdir -p vcf
	make -f src/run/bcftools.mk \
		REF=data/$(GENOME_FASTA) \
		BAM=$(BAM).sorted.bam \
		VCF=$(VCF) \
		run

#Remove all the derived files
clean:
	rm -rf fastq bam data

#Targets that are not files
.PHONY: genome fastq index align vcf clean
