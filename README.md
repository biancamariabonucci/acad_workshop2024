# acad_workshop2024
Here all the commands for the workshop

*Quality control of trimmed and merged sequences When handling large data and mapping against large reference genome collections, it can be important to remove duplicates, to save cpu and run time.* 
For this, we can use vsearch https://github.com/torognes/vsearch which is a fast tool that screens for 100% identical sequences (most likely caused by PCR duplication). You can use vsearch --help to familiarize yourself with its options.
```
srun -n 1 --mem 1GB  vsearch --fastx_uniques euks_data/ERR10493277_small-FINAL.fq.gz --fastqout ./ERR10493277_small-FINAL.vs.fq --minseqlength 30 --strand both
```

Then what we did was to clean our data from low complexity regions.

```
srun -n 1 --mem 5GB sga preprocess -m 30 --dust-threshold=1 ERR10493277_small-FINAL.vs.fq  -o ERR10493277_small-FINAL.vs.d1.fq
```

Lets check what sequences was filtered out.
```
# we can make this into a text file with all readIDs that parsed
for file in ERR10493277_small-FINAL.vs.d1.fq; do grep 'M_A00706' $file | cut -f2 -d@ > $file.readID.tmp ; done
grep 'M_A00706' ERR10493277_small-FINAL.vs.fq | grep -f readID.tmp -v | cut -f2 -d@ > $file.readID_lowcom.tmp
grep 'M_A00706' ERR10493277_small-FINAL.vs.fq | grep -f $file.readID.tmp -v | cut -f2 -d@ > $file.readID_lowcom.tmp
# how many reads did we filter out, and does it correspond to the stdout sga printed?
wc -l ERR10493277_small-FINAL.vs.d1.fq.readID_lowcom.tmp
# we can use seqtk to grep out all sequences that where categorized as low complex
seqtk subseq ERR10493277_small-FINAL.vs.fq ERR10493277_small-FINAL.vs.d1.fq.readID_lowcom.tmp

#count how many low complexity reads have been cut out
wc -l ERR10493277_small-FINAL.vs.d1.fq.readID_lowcom.tmp 

#create a file for the readlength distribution
cat ERR10493277_small-FINAL.vs.d1.fq | awk '{if(NR%4==2) print length($1)}' | sort -n | uniq -c > $file.read_length.txt &

```




