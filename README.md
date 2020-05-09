# virus_assembly
Protocolo para ensamblar un genoma viral

Descargar el genoma SRR11508492, que fue el primer secuenciado por el Instituto Nacional de Salud del Peru
[SRR11508492](https://trace.ncbi.nlm.nih.gov/Traces/sra/?run=SRR11508492)

```
prefetch SRR11508492
fasterq-dump.2.10.5 --skip-technical ../SRA/SRR11508492/SRR11508492.sra -e 8 -m 200 -p -3
```
Ambos programas se encuentran dentro de SRA Toolkit
https://github.com/ncbi/sra-tools
Enlace del archivo con los programas compilados para el sistema Ubuntu
[sratoolkit](http://ftp-trace.ncbi.nlm.nih.gov/sra/sdk/2.10.5/sratoolkit.2.10.5-ubuntu64.tar.gz)


