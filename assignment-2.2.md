## 1. Get the data files
**Question 1:** Download the following data files from the internet using the curl command: 
http://eaton-lab.org/pdsb/test.fastq.gz and http://eaton-lab.org/pdsb/iris-data-dirty.csv. 
Use the less or zless commands to look at each file. Describe what these commands do. Finally, 
use the head command to print the first 5 lines of the file iris-data-dirty.csv.

**My solution:** I am very familiar with `curl` but I always forget if it wants `-O` or `-o` or
`-0` (zero) so I had to look at the curl manual to remember: `man curl`

```bash
## -O Tells curl to use the 'remote name' as the output file name
## -L Tells curl to 'follow redirects'
$ curl -L -O http://eaton-lab.org/pdsb/test.fastq.gz
$ curl -L -O http://eaton-lab.org/pdsb/iris-data-dirty.csv

## What happens if you don't give the -O argument?
## What happens if you don't give the -L argument?
```

## 2. Clean the data
**Question 2:** Use grep, uniq, and sed for this question. Check that all of the species names 
are spelled correctly in the file iris-data-dirty.csv. Also check for missing values stored as NA. 
Create a new file where mispelled names are replaced with the correct values, and lines with NA are 
excluded, and save it as iris-data-clean.csv. Use cut, sort and uniq to list the number of data values 
there are for each species in the new cleaned data file. Describe your work.

The Iris metadata looks like this:

```
$ head -n 5 iris-data-dirty.csv 
5.1,3.5,1.4,0.2,Iris-setosa
4.9,3.0,1.4,0.2,Iris-setosa
4.7,3.2,1.3,0.2,Iris-setosa
4.6,3.1,1.5,0.2,Iris-setosa
5.0,3.6,1.4,0.2,Iris-setosa
```
### First check for missing values with grep
```
$ grep NA iris-data-dirty.csv
6.3,NA,4.9,1.5,Iris-versicolor
5.6,NA,4.1,1.3,Iris-versicolor

# There are two rows that contain NA. We can also use grep to find 'inverted matches'
# which means finding lines that DON'T include NA
$ grep -v NA iris-data-dirty.csv
```

### Next search for misspelled species names
We can use sed, grep, and uniq to isolate the sample names and 'count' the uniq occurences.
```bash
$ sed 's/,/\n/g' iris-data-dirty.csv | grep Iris | uniq -c
  11 Iris-setosa
   1 Iris-setsa
  38 Iris-setosa
   1 Iris-versicolour
  49 Iris-versicolor
  50 Iris-virginic
```
**Bonus question:** Why are there two batches of 'Iris-setosa' (which appear to be spelled correctly)?

Use `sed` to find and replace one of the bad species names:

```
## The resulting cleaned data is piped to stdout, and we can grep for setosa and count the number of resulting lines.
$ sed 's/Iris-setsa/Iris-setosa/g' iris-data-dirty.csv | grep setosa | wc -l
50
```

### Putting it all together to create a 'clean' data file
```
## Find the lines that don't have NA
## Clean the names with sed
## Redirect the output to a new file called iris-data-clean.csv
$ grep -v NA iris-data-dirty.csv | sed 's/Iris-setsa/Iris-setosa/g' | sed 's/Iris-versicolour/Iris-versicolor/g' > iris-data-clean.csv
```

Take a look at the cleaned data to doublecheck that it looks good:
```
## Cut `-f` selects the 'field` and `-d` selects the delimiter to split on
cut -f 5 -d ',' iris-data-clean.csv| sort | uniq -c
  50 Iris-setosa
  50 Iris-versicolor
  50 Iris-virginica
```
BAM!

## 3. Find a sequence
**Question 3:** Find how many lines in the data file test.fastq.gz start with "TGCAG" and end 
with "GAG". Describe your work.

This is how I would do it, but there are definitely many more ways than this. I would unpack
the data with gunzip using `-c` to push the data to stdout (rather than unzipping the file in place).
Then pass the data through two pipes, the first finds the TGCAG at the beginning of each line, with
the '^', then piping only those lines that meet the first condition through a second grep to find
GAG at the end of those lines with '$'.
```bash
$ gunzip -c test.fastq.gz | grep "^TGCAG" | grep "GAG$"

## How many lines are there (out of a total of 13553)?
$ gunzip -c test.fastq.gz| grep "^TGCAG" | grep "GAG$" | wc -l
44
```

## 4. Find a sequence chunk
**Question 4:** Using grep and other tools if necessary find all lines that contain the sequence
"AAAACCCC" and for each print that line, the line above it, and two lines below it (so that a 4-line 
chunk around each search hit is printed). Describe your work.

Lets start by just simply finding the lines that have the given sequence:
```bash
$ gunzip -c test.fastq.gz | grep AAAACCCC
TGCAGAATAGATAGGAAACGTTTTGGCGCTGTAGACATTAAAACCCCAGTAGGACACGGGTATCACAACGTACA
TGCAGTGGATCGAAAACCCCGAGGCTCAAGGTCACGCCACCGTCTTCGTGGCCAAGTTCTTCGGCCGCGCCGGC

# Looks like there are only 2 of these
```

Now, I know that grep can print lines before and after a target line, but I never remember how to do this so i had to search through the manual (`man grep`). In the man page I searched for 'before' and found this helpful info about "context":

```
     -B num, --before-context=num
             Print num lines of leading context before each match.  See also
             the -A and -C options.
```

```
# Lets try it with just -B first
$ gunzip -c test.fastq.gz| grep AAAACCCC -B 1     
@32082_przewalskii.98 GRC13_0027_FC:4:1:5669:1669 length=74
TGCAGAATAGATAGGAAACGTTTTGGCGCTGTAGACATTAAAACCCCAGTAGGACACGGGTATCACAACGTACA
--
@33413_thamno.59 GRC13_0027_FC:4:1:5000:1620 length=74
TGCAGTGGATCGAAAACCCCGAGGCTCAAGGTCACGCCACCGTCTTCGTGGCCAAGTTCTTCGGCCGCGCCGGC
```

Adding 2 lines of "after" context to complete the assignment:

```
$ gunzip -c test.fastq.gz| grep AAAACCCC -B 1 -A 2
@32082_przewalskii.98 GRC13_0027_FC:4:1:5669:1669 length=74
TGCAGAATAGATAGGAAACGTTTTGGCGCTGTAGACATTAAAACCCCAGTAGGACACGGGTATCACAACGTACA
+32082_przewalskii.98 GRC13_0027_FC:4:1:5669:1669 length=74
IIIIIIIIIIIIIIIIIIHIHIIIIIIIIGIIIGIIIIIIHIIIIIIIHIIIIHIIIIIIEHIHHIIIIICIHI
--
@33413_thamno.59 GRC13_0027_FC:4:1:5000:1620 length=74
TGCAGTGGATCGAAAACCCCGAGGCTCAAGGTCACGCCACCGTCTTCGTGGCCAAGTTCTTCGGCCGCGCCGGC
+33413_thamno.59 GRC13_0027_FC:4:1:5000:1620 length=74
IIIIIIIIIIIIIIIIIIIIDHIIHHIIIIIEIBGBGGGIIHEHHHIEBBHHIEGGDGIGGHAEFDBFBDDB?D
```



