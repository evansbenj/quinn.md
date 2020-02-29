# For Jim and Blaine
to connect to graham
```
ssh ben@graham.comoputecanada.ca
```

To change a directory type `cd <directory name.`
To list the contents type `ls`
To print the working diretory type `pwd`


# Micros from nextgen

This is a repo to document steps taken to pull out microsats from Illumina seqs from pukekos.

I am working on graham in this directory:
```
/home/ben/scratch/quinn_stuff
```

# PEAR
First step is to merge the overlapping portion of paired fastq files. I used a program called PEAR for this (https://cme.h-its.org/exelixis/web/software/pear/doc.html).

Here is a sbatch script for this:
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


pear --forward-fastq $1 --reverse-fastq $2 --output $3

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
sed -n '1~4s/^@/>/p;2~4p' Tawharanui.assembled.fastq > Tawharanui.assembled.fasta
sed -n '1~4s/^@/>/p;2~4p' Shakespear.assembled.fastq > Shakespear.assembled.fasta
sed -n '1~4s/^@/>/p;2~4p' South_Island.assembled.fastq > South_Island.assembled.fasta
```
To run misa, I am working in this directory:
```
/home/ben/scratch/quinn_stuff/misa_sourcecode_22092015
```
To execute misa I typed this: 
```
./misa.pl ../test.fasta
./misa.pl ../Tawharanui.assembled.fasta
```
or from this directory: `/home/ben/scratch/quinn_stuff/misa_sourcecode_22092015/Shakespear_microsats`
```
sbatch misa_sbatch.sh ../../Shakespear.assembled.fasta
```
where `misa_sbatch.sh` is this:
```
#!/bin/sh
#SBATCH --job-name=misa
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=1
#SBATCH --time=12:00:00
#SBATCH --mem=16gb
#SBATCH --output=misa.%J.out
#SBATCH --error=misa.%J.err
#SBATCH --account=def-ben

../misa $1 
```

./misa.pl ../South_Island.assembled.fasta
```
This generated one file per microsat in the wording directory.  I moveed these to their own directory like this:

```
mkdir Tawharanui_microsats
mkdir Shakespear_microsats
mkdir South_Island_microsats
```

Then, to check the fasta file for an individual microsatellite, use grep:
```
grep -A 1 'E00386:372:H2VYYCCX2:1:1101:8633:2030' OUTFILE.fasta
```

# Example
I ran misa with the config file with 4:4 in it.  A quick check identifes a potential microsatellite with flanking sequences with which to design primers:
```
[ben@gra-login2 misa_sourcecode_22092015]$ grep -A 1 'E00386:372:H2VYYCCX2:1:1101:7943:5300' ../test.fasta
>E00386:372:H2VYYCCX2:1:1101:7943:5300 1:N:0:2
GTGCCCATCGCTTCATGTCCCATCAGTGGGCACTAGTGAGAAGAGCACACCATCTCCTCTTTCCTTCCTTCCTTCCCATCAGATACTTAGAAACAGAGATACAACTACCCCTGAGTCTTCACTTCCTCAGGCTGAACACTGAAACACAGAACTGTCTTTGTTGGACAGGACCTTTCAGATCACCACGACTAACCATAACCCTAATGTGCCAAAACCACCACTAACTCCCATCCCTAAGCACCTCATCTACCTGTCTTTTATA
[ben@gra-login2 misa_sourcecode_22092015]$ more E00386:372:H2VYYCCX2:1:1101:7943:5300.gff
##gff-version 3
##sequence-region E00386:372:H2VYYCCX2:1:1101:7943:5300 1 262
#!Date 2020-01-14
#!Type DNA
#!Source-version MISA 2.0
E00386:372:H2VYYCCX2:1:1101:7943:5300	MISA	region	1	262	.	
.	.	ID=E00386:372:H2VYYCCX2:1:1101:7943:5300.1
E00386:372:H2VYYCCX2:1:1101:7943:5300	MISA	microsatellite	61	76	
.	.	.	Note=tetranucleotide_repeat_microsatellite_feature,(TTCC
)4;ID=E00386:372:H2VYYCCX2:1:1101:7943:5300.2
```

# Parsing the microsatellites

This script (parses_misa_output.pl) will go through all of the microsat descriptor files within a directory and print to an output file the names of the microsats that have characteristics we like such as being between 4-8 repeats and having flanking regions that are at least 40 bp long.

```perl
#!/usr/bin/perl
use warnings;
use strict;

# This script will parse output files from the program misa 
# It will print to an output file the names of the microsats
# that have certain criteria (length and positioon in sequence) 

# execute this script in a directory like this: "perl parses_misa_output.pl"

# First generate an output file to which we will print the
# names of the good microsatellites.
# Later we can use this file to extract the sequences for each 
# of these microsats 
my $outputfile = "good_micros.txt";
unless (open(OUTFILE, "> $outputfile"))  {
	print "I can\'t write to $outputfile   $!\n\n";
	exit;
}
print "Creating output file: $outputfile\n";

# define variables to use for filtering results
my $max_repeats_allowed = 8;
my $min_repeats_allowed = 4;
my $min_flanking_size = 40;

# define variables to use in parsing
my @files; 
my $lines=0;
my $length;
my $type;
my $micro_start;
my $micro_end;
my @temp;
my @temp1;

# print the header of the data we are extracting to screen
print "name\tseq_length\tmicro_start\tmicro_end\tnum_repeats\n";
# also print this header to our output file
print OUTFILE "name\tseq_length\tmicro_start\tmicro_end\tnum_repeats\n";

# read in all the names of files in a directory that befin with 'E00386'
@files = glob("E00386*");

# cycle through each file, check the length, and check if it has a 
# microsat that is the correct number of repeats
foreach(@files){
	# open the file
	unless (open DATAINPUT, $_) {
		print "Can not find the data input file!\n";
		exit;
	}
	# look at each line of the file
	while ( my $line = <DATAINPUT>) {
		# put the values of the line in an array called @temp using a whitespace separator
		@temp = split(/\s+/,$line);
		# check if we are on line # 5 - this has details about the length of the sequence
	    if($lines == 5){ 
			$length = $temp[4]; # this is the length of the sequence
	    }
	    # check if we are on line # 6 - this has information on the (first) microsat
	    if($lines == 6){ 
			$type = $temp[2]; # this is the type of repeat (microsat or repeat)
			$micro_start = $temp[3]; # this is where the microsat starts in the sequence
			$micro_end = $temp[4]; # this is where the microsat ends in the sequence
			@temp1 = split(/[);]/,$temp[8]); # this allows us to determine how many repeats are in the microsat
	    }
	    # count each line
	    $lines +=1;
		# check length of file
	} # end while
	close DATAINPUT;
	if($lines == 7){ # ignore any seqs with more than one micro; these have a longer file tseq_length
		# only print to screen and the output file the mmicrosats that have characteristics we want
		if(($type eq 'microsatellite') && ($micro_start >= $min_flanking_size) && (($length-$micro_end) >= $min_flanking_size) && 
			($temp1[1] <= $max_repeats_allowed) && ($temp1[1] >= $min_repeats_allowed)) {
			print $_,"\t",$length,"\t",$micro_start,"\t",$micro_end,"\t",$temp1[1],"\n";
			print OUTFILE $_,"\t",$length,"\t",$micro_start,"\t",$micro_end,"\t",$temp1[1],"\n";
		}
	}
	# reset $line variable so it can be used in the next file
	$lines=0;
} # end of cycling though each file
# close the output file
close OUTFILE;
```
