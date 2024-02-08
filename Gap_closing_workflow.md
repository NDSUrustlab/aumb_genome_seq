## Manual gap closing with blastn
- 2kb region from both sides of the gap was extracted
- hifi and nanopore sequencing reads were mapped to this 4100 bp sequences (4kb bases and 100 N's) using blastn
	- hifi and nanopore sequencing reads were used to make the blast db
- Then, based on blastn results saved in tsv file, we highlighted the read by opening the file in microsoft excel that span the 4100 bases 
- These highlighted reads were extracted using samtools faidx
- Then we extracted the exact gap region and gap region with 100bp on each side to visualize in Geneious for confirmation
- We filled the exact sequences manually by opening the scaffolded assembly at gaps (100N's)

#### 1) making the blastdb
```bash
makeblastdb -in hifi_reads_with_underscore.fasta -dbtype nucl
```
#### 2) extracting the reads
```bash
#!/bin/bash

echo $0
gap_number=$1
unfilled_gap_location=$2

scaf_number=1
save_in_folder=../fasta_files/scaffold_${scaf_number}

blast_db=/mmfs1/scratch/jatinder.singh/hifi_gapclosing_manual_checking/hifi_underscore_blast_db/hifi_reads_with_underscore.fasta
for_unfilled=/mmfs1/scratch/jatinder.singh/hifi_gapclosing_manual_checking/aumb_7scaffold.fasta

echo "Step: unfilled_gap sequence extraction"
samtools faidx $for_unfilled $unfilled_gap_location > $save_in_folder/${gap_number}_unfilled_gap.fasta

echo "Step: blastn"
blastn -query $save_in_folder/${gap_number}_unfilled_gap.fasta -db $blast_db -out ../tsv_files/scaffold_${scaf_number}/${gap_number}_unfilled_gap.tsv -outfmt 6 -max_hsps 2
```
#### 3) highlighted the desired reads in excel file
#### 4) getting the read that span the 2kb region on each side
```bash
#!/bin/bash

reads_file=/mmfs1/scratch/jatinder.singh/hifi_gapclosing_manual_checking/hifi_underscore_blast_db/hifi_reads_with_underscore.fasta

gap_number=$1
whcih_read=$2

scaf_number=1
save_in_folder=../extracted_hifi_reads/scaffold_${scaf_number}

echo "getting your read"
samtools faidx $reads_file $whcih_read > $save_in_folder/${gap_number}_gap_read_${whcih_read}.fasta
```
#### 5) Visualizing the reads in Geneious
#### 6) Opening the assembly with custom script made in python
```python
import re

with open("scaffold1.fasta", "r") as file:
    data = file.read().replace("\n", "")

lst = (re.split("(nnnnnnnnnnnnnnnnnnnnnnnnnnnnnnnnnnnnnnnnnnnnnnnnnnnnnnnnnnnnnnnnnnnnnnnnnnnnnnnnnnnnnnnnnnnnnnnnnnnn)", data))

lst2 = [ ''.join(x) for x in zip(lst[1::2], lst[2::2]) ]

lst2.insert(0, lst[0])

with open(r'scaf1_filled.fasta', 'w') as fp:
    for item in lst2:
        # write each item on a new line
        fp.write("%s\n" % item)
    print('Done')
```
#### 7) Then we copy pasted the extracted exact gap region in the row by counting the gap number in vs code.
#### 8) We confirmed the accuracy of filled gap region by comparing the length of scaffolds before and after gap closing and length of the gap region
