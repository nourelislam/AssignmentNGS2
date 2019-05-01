# AssignmentNGS2

###STAR alignment

##I have begin with the whole genome but that takes much longer time to index the genome, so I've used chr22 as a reference and their GTFfile which is recommended by STAR developer to use the annotation file besides the reference genome and here is the code for genome indexing
...


STAR --runMode genomeGenerate --genomeDir ./  --runThreadN 20 --genomeFastaFiles chr22_with_ERCC92.fa --sjdbGTFfile gencode.v30lift37.annotation.gtf

##then mapping step


R1= ~/Downloads/assignment/ngs2-assignment-data/SRR8797509_1.part_001.part_001.fastq

R2= ~/Downloads/assignment/ngs2-assignment-data/SRR8797509_2.part_001.part_001.fastq

STAR --runMode alignReads --genomeDir ~/Downloads/genome/ --runThreadN 20 --readFilesIn $R1 $R2 --readFilesType Fastx --outFileNamePrefix SRR8797509_

but the Log.final.out file has a unique mappping quality of 3.4% and unmapped short reads 96% which is a very bad mapping quality, and this is due to we have used only chr22 as a reference, but I continue to convert the SAM output file into BAM then sorting it 

samtools view -hbo SRR8797509.bam SRR8797509_Aligned.out.sam  

samtools sort SRR8797509.bam -o SRR8797509.sorted.bam

Then we have to mark the duplicate by Picard tools


picard_path=$CONDA_PREFIX/share/picard-2.19.0-0

java  -Xmx2g -jar $picard_path/picard.jar MarkDuplicates INPUT=SRR8797509.sorted.bam OUTPUT=SRR8797509.dedup.bam METRICS_FILE=SRR8797509.metrics.txt

then cutting the overhang by spliNtrim

Split N-trim did not run with the given code in the page of RNA-seq variant calling on GATK website due to the absence of jar file, then I've realized that is due to this code is from the old version of GATK3, so when I used the new GATK code version, the code run.

gatk SplitNCigarReads -R chr22_with_ERCC92.fa -I SRR8797509.dedup.bam -O SRR8797509_splitrim.bam

so this is the time for haplotypecaller 

gatk --java-options "-Xmx2G" HaplotypeCaller -R chr22_with_ERCC92.fa -I SRR8797509_splitrim.bam --emit-ref-confidence GVCF --pcr-indel-model NONE -O SRR8797509.gvcf

but it did not work having this error A USER ERROR has occurred: Argument --emit-ref-confidence has a bad value: Can only be used in single sample mode currently. Use the --sample-name argument to run on a single sample out of a multi-sample BAM file.

and when I used the recommended msg to add sample-name the following error has appeared 
A USER ERROR has occurred: Argument --sample_name has a bad value: Specified name does not exist in input bam files
so I searched more and more to fix this issue in biostar but the recommendation did not work which is making reindexing to the BAM file.
