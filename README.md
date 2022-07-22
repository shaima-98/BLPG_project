# BLPG_project
cd /mnt/c/Genespectrum_BLPG/Project
gunzip chr16.fa.gz

#indexing the ref genome
bwa index chr16.fa

samtools faidx chr16.fa

#mapping the reads on the ref genome
bwa mem -R "@RG\tID:chr16\tSM:Blond_1\tPL:Illumina" chr16.fa Blond_1_R1.fastq Blond_1_R2.fastq > Blond_1.sam
bwa mem -R "@RG\tID:chr16\tSM:Blond_2\tPL:Illumina" chr16.fa Blond_2_R1.fastq Blond_2_R2.fastq > Blond_2.sam
bwa mem -R "@RG\tID:chr16\tSM:Blond_3\tPL:Illumina" chr16.fa Blond_3_R1.fastq Blond_3_R2.fastq > Blond_3.sam
bwa mem -R "@RG\tID:chr16\tSM:Dark_1\tPL:Illumina" chr16.fa Dark_1_R1.fastq Dark_1_R2.fastq > Dark_1.sam
bwa mem -R "@RG\tID:chr16\tSM:Dark_2\tPL:Illumina" chr16.fa Dark_2_R1.fastq Dark_2_R2.fastq > Dark_2.sam
bwa mem -R "@RG\tID:chr16\tSM:Dark_3\tPL:Illumina" chr16.fa Dark_3_R1.fastq Dark_3_R2.fastq > Dark_3.sam

#conversion of sam file to bam file
samtools view -b Blond_1.sam > Blond_1.bam
samtools view -b Blond_2.sam > Blond_2.bam
samtools view -b Blond_3.sam > Blond_3.bam
samtools view -b Dark_1.sam > Dark_1.bam
samtools view -b Dark_2.sam > Dark_2.bam
samtools view -b Dark_3.sam > Dark_3.bam

#sorting and indexing the bam read files
samtools sort Blond_1.bam > Blond_1_sorted.bam
samtools sort Blond_2.bam > Blond_2_sorted.bam
samtools sort Blond_3.bam > Blond_3_sorted.bam
samtools sort Dark_1.bam > Dark_1_sorted.bam
samtools sort Dark_2.bam > Dark_2_sorted.bam
samtools sort Dark_3.bam > Dark_3_sorted.bam
samtools index Blond_1_sorted.bam
samtools index Blond_2_sorted.bam
samtools index Blond_3_sorted.bam
samtools index Dark_1_sorted.bam
samtools index Dark_2_sorted.bam
samtools index Dark_3_sorted.bam

#merge sorted bam files
samtools merge -o Merged_readfiles.bam Blond_1_sorted.bam Blond_2_sorted.bam Blond_3_sorted.bam Dark_1_sorted.bam Dark_2_sorted.bam Dark_3_sorted.bam

#removing duplicates
java -jar gatk-4.2.6.1/gatk-package-4.2.6.1-local.jar MarkDuplicates VALIDATION_STRINGENCY=SILENT I=Merged_readfiles.bam O=Merged_readfiles_rmdup.bam REMOVE_DUPLICATES=true M=Merged_readfiles_rmdup.metrics

#indexing file
samtools index Merged_readfiles_rmdup.bam

#creating dict file for variant calling
java -jar ./gatk-4.2.6.1/gatk-package-4.2.6.1-local.jar CreateSequenceDictionary -R chr16.fa -O chr16.dict

#variant calling using haplotypecaller
java -jar ./gatk-4.2.6.1/gatk-package-4.2.6.1-local.jar HaplotypeCaller -R chr16.fa -I Merged_readfiles_rmdup.bam -O Merged_readfiles_variant.vcf

#variant annotation
java -Xmx6g -jar ./snpEff/snpEff.jar -v GRCh38.105 Merged_readfiles_variant.vcf > Merged_readfiles_variant_ann.vcf







