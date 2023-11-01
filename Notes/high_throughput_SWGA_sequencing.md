# Protocol for optimising high-throughput sequencing of SWGA DNA on Nanopore platforms.

Run pipeline as usual. Navigate to the fastq folder.

Using krakentools, we are going to extract and quantify the number of bases in each isolate corresponding to a particular _taxid_. In this case, we have used SWGA to target P._falciparum_ and P._vivax_ genomes, so we will use the __5820__ taxid, which will target the _Plasmodium_ genus.

```
cd .../nanopore_pipeline_out/fastq/

mkdir filtered_reads

for i in $(ls *.fastq.gz | sed 's/.fastq.gz//g'); do 

    extract_kraken_reads.py \
        -k ../krakenqc/${i}.fastq.gz.kraken \
        -r ../krakenqc/${i}.fastq.gz.kreport \
        -s ${i}.fastq.gz \
        -o filtered_reads/${i}.filtered.fastq \
        -t 5820  \
        --include-children \
        --fastq-output >/dev/null;
        seqkit stats filtered_reads/${i}.filtered.fastq | \
        tail -n1 
done | sed 's/filtered_reads\///g' | sed 's/.filtered.fastq//g' | awk '{print $1 "\t" $5}' > ../target_base_summary.tsv

for i in $(ls *.fastq.gz | sed 's/.fastq.gz//g'); do 
        seqkit stats ${i}.fastq.gz | \
        tail -n1 
done | sed 's/filtered_reads\///g' | sed 's/.filtered.fastq//g' | awk '{print $5}' > ../total_bases.tsv

paste ../target_base_summary.tsv ../total_bases.tsv > final_summary.tsv 

```

###Bug fixing edits

```
cd .../nanopore_pipeline_out/fastq/

mkdir filtered_reads

for i in $(ls *.fastq.gz | sed 's/.fastq.gz//g'); do 

    extract_kraken_reads.py \
        -k ../krakenqc/${i}.fastq.gz.kraken \
        -r ../krakenqc/${i}.fastq.gz.kreport \
        -s ${i}.fastq.gz \
        --exclude \
        -o filtered_reads/${i}.filtered.inverse.fastq \
        -t 5820  \
        --include-children \
        --fastq-output >/dev/null;
        seqkit stats filtered_reads/${i}.filtered.inverse.fastq | \
        tail -n1 
done | sed 's/filtered_reads\///g' | sed 's/.filtered.fastq//g'  > ../target_base_summary_inverse.tsv

for i in $(ls *.fastq.gz | sed 's/.fastq.gz//g'); do 
        seqkit stats ${i}.fastq.gz | \
        tail -n1 
done | sed 's/filtered_reads\///g' | sed 's/.filtered.fastq//g' | awk '{print $5}' > ../total_bases.tsv

paste ../target_base_summary.tsv ../total_bases.tsv > final_summary.tsv 

```