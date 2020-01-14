# Micros from nextgen

This is a repo to document steps taken to pull out microsats from Illumina seqs from pukekos.

I am working on graham in this directory:
```
/home/ben/scratch/quinn_stuff
```

# PEAR
First step is to merge the overlapping portion of paired fastq files. Here is a sbatch script for this:
```
#!/bin/sh
#SBATCH --job-name=PEAR
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=1
#SBATCH --time=6:00:00
#SBATCH --mem=16gb
#SBATCH --output=PEAR.%J.out
#SBATCH --error=PEAR.%J.err
#SBATCH --account=def-ben

module load nixpkgs/16.09
module load gcc/5.4.0
module load pear/0.9.10


pear --forward-fastq$1 --reverse-fastq $2 --output $3

```

I ran three commands to do this:
```
sbatch PEAR_sbatch.sh ./60YGXHF/LAN10690/190705_E00386_0372_AH2VYYCCX2/1A_Tawharanui_S1_L001_R1_001.fastq.gz ./60YGXHF/LAN10690/190705_E00386_0372_AH2VYYCCX2/1A_Tawharanui_S1_L001_R2_001.fastq.gz Tawharanui
```
```
sbatch PEAR_sbatch.sh ./60YGXHF/LAN10690/190705_E00386_0372_AH2VYYCCX2/2B_Shakespear_S2_L001_R1_001.fastq.gz ./60YGXHF/LAN10690/190705_E00386_0372_AH2VYYCCX2/2B_Shakespear_S2_L001_R2_001.fastq.gz Shakespear
```
```
sbatch PEAR_sbatch.sh ./60YGXHF/LAN10690/190705_E00386_0372_AH2VYYCCX2/3C_South_Island_S3_L001_R1_001.fastq.gz ./60YGXHF/LAN10690/190705_E00386_0372_AH2VYYCCX2/3C_South_Island_S3_L001_R2_001.fastq.gz South_Island
```
According to the manual: "PEAR outputs four files. A file containing the assembled reads with a assembled.fastq extension, two files containing the forward, resp. reverse, unassembled reads with extensions unassembled.forward.fastq, resp. unassembled.reverse.fastq, and a file containing the discarded reads with a discarded.fastq extension"

# Parsing the overlap

The next step is to process the merged file using a script to search for microsats using a regex.  Depending on discussions with Jim and Blaine, we could look first for "pure" tetramers, or whatever. I found a script that does this here: `https://webblast.ipk-gatersleben.de/misa/`

The output of PEAR is fastq. I can make a sample file to work with like thiis:
```
head -1000 input.fastq > output.fastq
```
I need to convert fastq to fasta format like this:
```
sed -n '1~4s/^@/>/p;2~4p' INFILE.fastq > OUTFILE.fasta
```

Then, to check the fasta file, use grep:
```
grep -A 1 'E00386:372:H2VYYCCX2:1:1101:8633:2030' OUTFILE.fasta
```
