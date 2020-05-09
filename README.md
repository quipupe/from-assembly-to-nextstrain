# virus_assembly
Protocol to assemble viral genomes

Para descargar el genoma SRR11508492 que fue el primer secuenciado por el Instituto Nacional de Salud del Peru
https://trace.ncbi.nlm.nih.gov/Traces/sra/?run=SRR11508492
```
prefetch SRR11508492
fasterq-dump.2.10.5 --skip-technical ../SRA/SRR11508492/SRR11508492.sra -e 8 -m 200 -p -3
```


