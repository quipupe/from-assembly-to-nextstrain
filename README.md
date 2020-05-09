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

El archivo se descarga como **SRR11508492.sra**

fasterq-dump separa los archivos en *.1* y *.2*

Modifica el nombre del archivo con el comando  `mv` y luego comprime ambos archivos usando `gzip`

```
mv SRR11508492.sra_1.fastq SRR11508492_1.fastq 
mv SRR11508492.sra_2.fastq SRR11508492_2.fastq 

gzip SRR11508492_1.fastq
gzip SRR11508492_2.fastq
```

Utiliza FastQC para revisar la calidad de las secuencias
```
fastqc SRR11508492_1.fastq.gz
fastqc SRR11508492_2.fastq.gz
 ```
Luego, usa Trimmomatic para hacer el *trimming* o cortar las secuencias segun parametros de calidad

PE: Paired-end reads

-threads: Numero de cores

LEADING y TRAILING cortan las secuencias en los extremos hasta que encuentren secuencias con calidades mayores a 20

SLIDINGWINDOW utiliza una ventana de 4 bases para calcular si dentro de ellas la calidad promedio es 20, si es menor se corta la secuencia

MINLEN es el tamano minimo de las secuencias

```
java -jar -Xms8g ~/Trimmomatic-0.39/trimmomatic-0.39.jar PE -threads 7 SRR11508492_1.fastq.gz SRR11508492_2.fastq.gz out_forward_PE.fq.gz out_forward_unPE.fq.gz out_reverse_PE.fq.gz out_reverse_unPE.fq.gz ILLUMINACLIP:NexteraPE-PE.fa:2:30:10:2:keepBothReads LEADING:20 TRAILING:20 SLIDINGWINDOW:4:20 MINLEN:30  
```

Obtuve estos resultados
```
TrimmomaticPE: Started with arguments:
-threads 7 SRR11508492_1.fastq.gz SRR11508492_2.fastq.gz out_forward_PE.fq.gz out_forward_unPE.fq.gz out_reverse_PE.fq.gz out_reverse_unPE.fq.gzILLUMINACLIP:NexteraPE-PE.fa:2:30:10:2:keepBothReads LEADING:20 TRAILING:20 SLIDINGWINDOW:4:20 MINLEN:30
Using PrefixPair: 'AGATGTGTATAAGAGACAG' and 'AGATGTGTATAAGAGACAG'
Using Long Clipping Sequence: 'GTCTCGTGGGCTCGGAGATGTGTATAAGAGACAG'
Using Long Clipping Sequence: 'TCGTCGGCAGCGTCAGATGTGTATAAGAGACAG'Using Long Clipping Sequence: 'CTGTCTCTTATACACATCTCCGAGCCCACGAGAC'Using Long Clipping Sequence: 'CTGTCTCTTATACACATCTGACGCTGCCGACGA'
ILLUMINACLIP: Using 1 prefix pairs, 4 forward/reverse sequences, 0 forward only sequences, 0 reverse only sequences																			Quality encoding detected as
phred33
Input Read Pairs: 2359909 Both Surviving: 1578896 (66.90%) Forward Only Surviving: 703350 (29.80%) Reverse Only Surviving: 21051 (0.89%) Dropped: 56612 (2.40%)
TrimmomaticPE: Completed successfully
 ```

Vuelve a hacer `fastqc` en los archivos *trimmeados*
```
fastqc out_forward_PE.fq.gz
```

Luego, usa bowtie2 para crear un index a partir de la secuencia fasta. El index se llamara **virus**
```
bowtie2-build -f file.fasta dbname
bowtie2-build -f wuhan_NC_045512.fa virus
```

Use como informacion esta [pagina](http://www.metagenomics.wiki/tools/bowtie2) y esta [otra](https://www.protocols.io/view/week-6-mapping-with-bowtie2-g5zby76)

```
bowtie2 -x ../reference/virus -1 out_forward_PE.fq.gz -2 out_reverse_PE.fq.gz -U out_forward_unPE.fq.gz,out_reverse_unPE.fq.gz -S ./virus.sam --no-unal -p 7
```
Obtuve estos resultados
```
2303297 reads; of these:
1578896 (68.55%) were paired; of these:
1569199 (99.39%) aligned concordantly 0 times
(0.61%) aligned concordantly exactly 1 time
3 (0.00%) aligned concordantly >1 times 
1569199 pairs aligned concordantly 0 times; of these:
(0.02%) aligned discordantly 1 time
1568878 pairs aligned 0 times concordantly or discordantly; of these:
3137756 mates make up the pairs; of these:
(99.99%) aligned 0 times
188 (0.01%) aligned exactly 1 time
140 (0.00%) aligned >1 times
724401 (31.45%) were unpaired; of these:
720372 (99.44%) aligned 0 times
4015 (0.55%) aligned exactly 1 time
14 (0.00%) aligned >1 times
0.63% overall alignment rate 
```

Usa samtools para
1. Tranformar el archivo sam a bam
2. Ordenar el archivo bam
3. Crear un indice a partir de la secuencia de referencia
4. Indexar el archivo bam
```
samtools view -bS virus.sam > virus.bam 
samtools sort virus.bam -o virus.sorted.bam
samtools faidx virus.fa
samtools index virus.sorted.bam
``` 

Con samtools tambien se puede crear un archivo *fastq* con la informacion de los reads pareados que si mapearon y los reads no pareados que si mapearon al genoma de referencia
```
samtools fastq -0 /dev/null -s single.fq -N virus.bam > paired.fq 
```

Si necesitas crear archivos separados por cada paired-end entonces usa este comando, no generara los no pareados
``` 
samtools/bin/samtools fastq -1 paired1.fq -2 paired2.fq -0 /dev/null -s /dev/null -n virus.sorted.bam
[M::bam2fq_mainloop] discarded 16454 singletons
[M::bam2fq_mainloop] processed 23144 reads
````

Usa SPAdes para ensamblar el genoma
Necesitaras los archivos paired y unpaired
```
python3.6 spades.py --12 ../../SRR11508492/reference/paired.fq -s ../../SRR11508492/reference/single.fq -o ../../assembly -t 8
```

Sintaxis para formatos de GitHub
https://help.github.com/es/github/writing-on-github/basic-writing-and-formatting-syntax

# Instalar los programas necesarios para este tutorial

Samtools
http://www.htslib.org/download/
http://quinlanlab.org/tutorials/samtools/samtools.html
```
./configure --prefix=/mnt/c/Users/pipor/Desktop/coronavirus/genome/samtools/ 
make
make install
```
`prefix=` instala el programa en un directorio local 

SPAdes
```
wget http://cab.spbu.ru/files/release3.14.1/SPAdes-3.14.1.tar.gz
tar -xzf SPAdes-3.14.1.tar.gz
cd SPAdes-3.14.1
#sudo apt install cmake en WSL
spades.py --pe1-1 ../share/spades/test_dataset/ecoli_1K_1.fq.gz --pe1-2 ../share/spades/ test_dataset/ecoli_1K_2.fq.gz -o spades_test
```
Mas informacion sobre
SPAdes
https://github.com/ablab/spades#sec2.1
Bowtie2
https://www.protocols.io/view/week-6-mapping-with-bowtie2-g5zby76?step=16
BLAST del nuevo ensamblado contra el genoma de referencia
https://blast.ncbi.nlm.nih.gov/Blast.cgi?CMD=Get&RID=BC7WAC8D114
