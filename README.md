# virus_assembly
Protocol to assemble viral genomes


prefetch SRR11508492
fasterq-dump.2.10.5 --skip-technical ../SRA/SRR11508492/SRR11508492.sra -e 8 -m 200 -p -3
