# Nanopore data management rationale

### Here is 

## Transfer from the sequencer associated machine
Transferring files from the sequencer associated machine (SAM) to sequencing data node (SDN) is achieved using a script which utilises rsync. Rsync can unidirectionally synchronise the contents of the SAM data folder with the SDN periodically which minimises the risk of data loss due to hardware failiure or user error. This daemon will be automated using crontab and will require no user input.

Due to the relatively finite storage on the SAM (5 TB) it is essential to periodically purge the data directory of recent experiments to free up space for new data, only after ensuring all data has been successfully transferred to the SDN. For this, rsync has the ```--remove-source-files option```. The user can initiate a rsync process, similar to the automated daemon, which will check the contents of the SAM data folder is correctly uploaded to the SDN, checking MD5 sums, and then it will delete the source files on the SAM. It is important that this is not run during a sequencing experiment, as it will delete files that are being written to for the ongoing experiment which will crash MinKNOW. Logic to prevent this occurrence has been added to the purge script, ensuring no files in the data directory have been created in the last 30 mins, but let's not test that it works.

Always check there is > 100 GB free on the SAM before starting an experiment. Run ```purge_local_data.sh``` to free up space on the SAM.

## Basecalling and QC
To avoid any erroneous deletion or corruptions of raw sequencing data, users will no have write permissions to the data@plum-S8 directories. To perform the basecalling and  QC workflow and onward analysis, the user should access the basecalling environment through ```ssh basecalling@plum-g1``` and use the ```pipeline_v1.2_demultiplex.sh``` script to perform basecalling and QC or ```pipeline_v1.2_R9_demultiplex_nobasecall.sh``` for QC only if the data has already been basecalled with super-high accuracy on the SAM. Remember that as basecalling models improve, the basecalling script will be updated accordingly, so if the data was basecalled a while ago, it might be worth repeating before copying the directory for downstream analysis.

To run the basecalling and QC scripts, input the following fields:

**For basecalling, demultiplexing and QC with a sample sheet**
```
pipeline_v1.2_demultiplex.sh <MinKNOW experiment root directory on SDN> <experiment name> <barcoding kit> <CSV sample sheet absolute path> <FASTA reference sequence absolute path> <basecalling model>
```
**QC only, with a sample sheet**
```
pipeline_v1.2_R9_demultiplex_nobasecall.sh <MinKNOW experiment root directory on SDN> <experiment name> <barcoding kit> <CSV sample sheet absolute path> <FASTA reference sequence absolute path> <basecalling model>
```
**For basecalling, demultiplexing and QC <ins>without</ins> a sample sheet**
Please only use this one if it is absolutely necessary.
```
pipeline_v1.2_demultiplex.sh <MinKNOW experiment root directory on SDN> <experiment name> <barcoding kit> <FASTA reference sequence absolute path> <basecalling model>
```

### Current basecalling models and barcoding kits

Kit 9 model = dna_r9.4.1_450bps_sup.cfg
Kit 12 = dna_r10.4_e8.1_sup.cfg and Kit 14
Kit 14 = dna_r10.4.1_e8.2_260bps_sup.cfg or dna_r10.4.1_e8.2_400bps_sup.cfg depending on temperature

Barcoding kit-9 codes include **EXP-NBD196** (96 barcodes), **EXP-NBD104** (1-12) and **EXP-NBD114** (13-24)
Barcoding for R10.4.1 (Kit 14) kits include **SQK-NBD114-24** and **SQK-NBD114-96**