# virus_assembly
Protocolo para ensamblar un genoma viral

Descargar el genoma SRR11508492, que fue el primer secuenciado por el Instituto Nacional de Salud del Peru
[SRR11508492](https://trace.ncbi.nlm.nih.gov/Traces/sra/?run=SRR11508492).

```
prefetch SRR11508492
fasterq-dump.2.10.5 --skip-technical ../SRA/SRR11508492/SRR11508492.sra -e 8 -m 200 -p -3
```
Ambos programas se encuentran dentro de SRA Toolkit
https://github.com/ncbi/sra-tools

Enlace del archivo con los programas compilados para el sistema Ubuntu
[sratoolkit](http://ftp-trace.ncbi.nlm.nih.gov/sra/sdk/2.10.5/sratoolkit.2.10.5-ubuntu64.tar.gz).

El archivo se descarga como **SRR11508492.sra**. fasterq-dump separa los archivos en *.1* y *.2*. Modifica el nombre con el comando mv `mv` y luego comprime los archivos usango `gzip`

```
mv SRR11508492.sra_1.fastq SRR11508492_1.fastq 
mv SRR11508492.sra_2.fastq SRR11508492_2.fastq 

gzip SRR11508492_1.fastq
gzip SRR11508492_2.fastq
```

#Execute FastQC
fastqc SRR11508492_1.fastq.gz
fastqc SRR11508492_2.fastq.gz
 
java -jar -Xms8g ~/Trimmomatic-0.39/trimmomatic-0.39.jar PE -threads 7 SRR11508492_1.fastq.gz SRR11508492_2.fastq.gz out_forward_PE.fq.gz out_forward_unPE.fq.gz out_reverse_PE.fq.gz out_reverse_unPE.fq.gz ILLUMINACLIP:NexteraPE-PE.fa:2:30:10:2:keepBothReads LEADING:20 TRAILING:20 SLIDINGWINDOW:4:20 MINLEN:30  
 
TrimmomaticPE: Started with arguments:
-threads 7 SRR11508492_1.fastq.gz SRR11508492_2.fastq.gz out_forward_PE.fq.gz out_forward_unPE.fq.gz out_reverse_PE.fq.gz out_reverse_unPE.fq.gzILLUMINACLIP:NexteraPE-PE.fa:2:30:10:2:keepBothReads LEADING:20 TRAILING:20 SLIDINGWINDOW:4:20 MINLEN:30
Using PrefixPair: 'AGATGTGTATAAGAGACAG' and 'AGATGTGTATAAGAGACAG'
Using Long Clipping Sequence: 'GTCTCGTGGGCTCGGAGATGTGTATAAGAGACAG'
Using Long Clipping Sequence: 'TCGTCGGCAGCGTCAGATGTGTATAAGAGACAG'Using Long Clipping Sequence: 'CTGTCTCTTATACACATCTCCGAGCCCACGAGAC'Using Long Clipping Sequence: 'CTGTCTCTTATACACATCTGACGCTGCCGACGA'
ILLUMINACLIP: Using 1 prefix pairs, 4 forward/reverse sequences, 0 forward only sequences, 0 reverse only sequences																			Quality encoding detected as
phred33
Input Read Pairs: 2359909 Both Surviving: 1578896 (66.90%) Forward Only Surviving: 703350 (29.80%) Reverse Only Surviving: 21051 (0.89%) Dropped: 56612 (2.40%)
TrimmomaticPE: Completed successfully

Sintaxis para formatos de GitHub
https://help.github.com/es/github/writing-on-github/basic-writing-and-formatting-syntax
