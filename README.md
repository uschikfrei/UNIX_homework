# UNIX Assignment

## Inspecting the data files:

To inspect the datafiles I use the following commands:
```
$ wc -l datafile.txt
$ awk _F '{print NF; exit}' datafile.txt
$ ls- lh datafile.txt
$ du -h datafile.txt
file datafile.txt
```
*snp_positions.txt:*
lines         984
columns        15
size           81k
disk usage     84k
encoding      ASCII text

*fang_et_al_genotypes.txt:*
lines         2783
columns        986
size            11M
disk usage      11M
encoding      ASCII text, with very long lines

## Data Processing

### generating sub data sets for maize or teosinte individuals only:
```
$ grep 'Sample_ID' fang_et_al_genotypes > maize_genotypes.txt && grep 'ZMM' fang_et_al_genotypes.txt >> maize_genotypes.txt
$ grep 'Sample_ID' fang_et_al_genotypes > maize_genotypes.txt && grep 'ZMP' fang_et_al_genotypes.txt >> teosinte_genotypes.txt
```
--->should add the header line and the respective lines into sub data sets

to check:
```
$ cut -f3 maize_genotypes.txt | sort | uniq
```
---> should only list the three groups starting with 'ZMM' and the header group
```
$ wc -l maize_genotypes.txt
```
----> 1574
```
cut -f3 teosinte_genotypes.txt | sort | uniq'
```
---> should only list the three groups starting with 'ZMP' and the header group
```
$ wc -l teosinte_genotypes.txt
```
--->976

### transpose the genotype data files:
```
$ awk -f transpose.awk maize_genotypes.txt > transposed_maize_genotypes.txt
$ awk -f transpose.awk teosinte_genotypes.txt > transposed_teosinte_genotypes.txt
```
to check:
```
$ wc -l transposed_maize_genotypes.txt
```
---> 986 
```
$ wc -l transposed_teosinte_genotypes.txt
```
---> 986
```
$ awk -F '{print NF; exit}' transposed_maize_genotypes.txt
$ awk -F '{print NF; exit}' transposed_teosinte_genotypes.txt
```
---> both times I get:
awk: cmd. line:1: transposed_maize_genotypes.txt
awk: cmd. line:1:                           ^syntax error



### joining the data sets
```
$ tail -n +4 transposed_maize_genotypes.txt > red-transposed_maize_genotypes.txt
$ tail -n +4 transposed_maize_genotypes.txt > red-transposed_teosinte_genotypes.txt
$ cut -f1,2,3 snp_position.txt | tail -n +2 > res-snp_position.txt
$ join -t $'\t' -1 1 -2 1 red-snp_position.txt red-transposed_maize_genotypes.txt > maize_combined.txt
$ join -t $'\t' -1 1 -2 1 red-snp_position.txt red-transposed_teosinte_genotypes.txt > teosinte_combined.txt
$ head -n 3 transposed_maize_genotypes.txt | cat > maize_data.txt && cat maize_combined.txt >> maize_data.txt
$ head -n 3 transposed_teosinte_genotypes.txt | cat > teosinte_data.txt && cat maizeteosinte_combined.txt >> maize_data.txt
```

---> first I remove he headers, then I join the data, then I add the headers again

to check:
```
$ wc -l maize_data.txt
```
---> 986 
```
$ wc -l teosinte_data.txt
```
---> 986
checking for number of columns fails as before!
I try the following instead:
```
$ awk -f transpose.awk maize_data.txt | wc -l
```
---> 1574
```
$ awk -f transpose.awk teosinte_data.txt | wc -l
```
---> 976

### generating the 40 files
first I generate the chromosome specific files:
```
$ awk '$3 == "1" {print}' maize_data.txt | cat > chr1_maize_data.txt
```
...
There is probbly a way to do that for all chromosomes in one step - I will think about it later.

to check:
```
$ cut -f3 maize_data.txt | sort  | uniq -c
```
---> gives me the number of markers in each chromosome
```
$ wc -l chr1_maize_data.txt 
```
---> gives me the number of lines which should be the same


To do the substitution of the ?/? into ? or -, I create a file try_out.txt to play around:
```
$ awk '{gsub("?/?", "-"); print $0}' try_out.txt
```
---> replaces "?/?" always with "--"  - strange!
```
$ awk '{gsub("?/?", "-"); print $0}' try_out.txt | sed 's/--/-/g'
$ awk '{gsub("?/?", "-"); print $0}' try_out.txt | sed 's/--/-/g' | sed 's/-/?/g'
```
---> that works in my try_out.txt
 
I tried to write a script: subst-for-dashes.sh, but it does not work.
So I do it one by one...

### sorting the files: ? increasing, - decreasing
```
$ sort -k2,2n chr1_maize_data-with-dashes.txt > chr1_maize_sorted-data-with-dashes.txt
$ sort -k2,2nr chr1_maize_data-with-qm.txt > chr1_maize_sorted-data-with-qm.txt
```
---> works for single files, I try to find a way to do it for 10 at a time:

 ```
$ for i in {1,2,3,4,5,6,7,8,9,10}; do sort -2k,2n chr[$i]_maize_data-with-dashes.txt > chr[$i]_maize_sorted-data-with-dashes.txt; done
```
---> sorts all ten files, although the file name does not end up to be as I wanted it....
```
$ for i in {1,2,3,4,5,6,7,8,9,10}; do sort -2k,2nr chr[$i]_maize_data-with-qm.txt > chr[$i]_maize_sorted-data-with-qm.txt; done
```

### adding headers to the files:
in case the headers are needed, I created a file header_teosinte.txt, which can be added to the tesinte sorted files
and a header_maize.txt for the maize files
