# EquCab3-Iso-seq-BALF-paper
Code for analysis of equine BALF cells IsoSeq long read sequencing data  
## 1. SMRT Analysis

The input to the Iso-Seq SMRT (v10.1) analysis was the HiFi (CCS) reads and the output was high-quality, full-length transcript sequences (`hq_transcripts.fasta`). The [Iso-Seq SMRTLink Analysis Report PDF can be found here](https://github.com/vetsuisse-unibe/EquCab3-Iso-seq-BALF-paper/blob/main/PB_12_IsoSeq_runQC%26CCSreport_May2021_NGSP.pdfhttps://github.com/vetsuisse-unibe/EquCab3-Iso-seq-BALF-paper/blob/main/PB_12_IsoSeq_runQC%26CCSreport_May2021_NGSP.pdf)
## 2. Mapping to reference and collapsing unique isoforms 

```
minimap2 -ax splice -t 30 -uf --secondary=no -C5 \ 
       GCA_002863925.1_EquCab3.0_genomic.fna \
       hq_transcripts.fasta > \
       hq_isoforms.fasta.sam
       
sort -k 3,3 -k 4,4n hq_isoforms.fasta.sam > hq_isoforms.fasta.sorted.sam

collapse_isoforms_by_sam.py  --input hq_transcripts.fasta \
       -s hq_transcripts.fasta.sorted.sam \
       -c 0.99 -i 0.95 \
       --dun-merge-5-shorter \
       -o hq_ncbi.collapsed
       
get_abundance_post_collapse.py \
       hq_ncbi.collapsed \
       hq_ncbi_transcripts_cluster_report.csv

filter_away_subset.py hq_ncbi.collapsed
```
## 3. Classification and Filtering using SQANTI3
We used [SQANTI3](https://github.com/ConesaLab/SQANTI3/) to classify and filter the collapsed transcripts against the EquCab3.0 NCBI (release 103) annotation.

```
python sqanti3_qc.py \
                 hq.ncbi.collapsed.filtered_classification.filtered_lite.gtf \
                 GCF_002863925.1_EquCab3.0_genomic.gtf \
                 GCA_002863925.1_EquCab3.0_genomic.fna \
                 --fl_count hq_ncbi.collapsed.filtered.mapped_fl_count.txt \
                 
     
             
python sqanti3_RulesFilter.py \
                 --faa hq_ncbi.collapsed.filtered_corrected.faa \
                 hq_ncbi.collapsed.filtered_classification.txt \
                 hq_ncbi.collapsed.filtered_corrected.fasta \
                 hq_ncbi.collapsed.filtered_corrected.gtf
                  -a 0.86 -m 100 -r 8 
```
[The final filtered transcriptome can be found here](https://github.com/vetsuisse-unibe/EquCab3-Iso-seq-BALF-paper/blob/main/annotated.PacBio.transcriptome.based.on.EquCab3.NCBI.v103.gtf)
