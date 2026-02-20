# UNIX Assignment 
### _Yining Fu_ 

## Data Inspection

### Attributes of `fang_et_al_genotypes.txt`

1. `wc fang_et_al_genotypes.txt`  
	* it has a total of 2783 lines, 2744038 words, and 11051939 characters. 

2. `awk -F "\t" '{print NF; exit}' fang_et_al_genotypes.txt`  
	* it has 986 columns. The column number stays the same when skipping header or go to the last few rows in the file. Because 2744038/2783 = 986, every entry in the file has data in it. 

3. `cat -T fang_et_al_genotypes.txt`  
	* it is tab-delimited

4. `head -n +5 fang_et_al_genotypes.txt` & `tail -n 5 fang_et_al_genotypes.txt`  
	* the header consists of `Sample_ID`, `JG_OTU`, `Group`, and all the SNP positions, which correspond to entries in column 1 named `SNP_ID` in `snp_position.txt`. Combining the result of `head` and `tail`, I know that the format of this file is consistent throughout. The data, in other words, SNPs, in the file are in the form of "nucleotide/nucleotide".

5. `du -h fang_et_al_genotypes.txt`  
	* file size is 6.7M

6. `file fang_et_al_genotypes.txt`  
	* it is encoded in ASCII with very long lines

### Attributes of `snp_position.txt`

1. `wc snp_position.txt`  
	* it has a total of 984 lines, 13198 words, and 82763 characters. 

2. `awk -F "\t" '{print NF; exit}' snp_position.txt`  
	* it has 15 columns. The column number stays the same when skipping header or go to the last few rows in the file. Because 13198/984 = 13.41 != 15, some entries in this file are empty. 

3. `cat -T snp_position.txt`  
	* it is tab-delimited

4. `head -n +5 snp_position.txt` & `tail -n 5 snp_position.txt`  
	* the header consists of `SNP_ID`, `cdv_marker_id`, `Chromosome`, `Position`, `alt_pos`, `mult_positions`, `amplicon`, `cdv_map_feature.name`, `gene`, `candidate/random`, `Genaissance_daa_id`, `Sequenom_daa_id`, `count_amplicons`, `count_cmf  `, `count_gene`. Combining the result of `head` and `tail`, I know that the format of this file is consistent throughout. 

6. `cut -f snp_position.txt | sort | wc`  
	* repeat the command on all 15 columns, I discovered that column 4, 5, and 6 are the ones with incomplete data. Column 8 and 9 have more words than the number of rows, meaning that some entries contain multiple space-delimited words. This shows that `wc` can count space-delimited words in one entry as multiple words. Also, this explains the discrepancy find in point 2. 

7. `du -h snp_position.txt`  
	* file size is 49K

8. `file snp_position.txt`  
	* it is encoded in ASCII characters


## Data Processing

1. extract maize and teosinte data from `fang_et_al_genotypes.txt` respectively  
	* `grep -w -e "Group" -e "ZMMIL" -e "ZMMLR" -e "ZMMMR" fang_et_al_genotypes.txt > maize_genotypes.txt`  
	* `grep -w -e "Group" -e "ZMPBA" -e "ZMPIL" -e "ZMPJA" fang_et_al_genotypes.txt > teosinte_genotypes.txt`

2. transpose maize and teosinte data  
	* `awk -f transpose.awk maize_genotypes.txt > transposed_maize_genotypes.txt`  
	* `awk -f transpose.awk teosinte_genotypes.txt > transposed_teosinte_genotypes.txt`

3. change the `Sample_ID` header in `transposed_maize_genotypes.txt` and `transposed_teosinte_genotypes.txt` to `SNP_ID` so they can better merge with the snp positions file  
	* `vi transposed_maize_genotypes.txt` and `vi transposed_teosinte_genotypes.txt`
	* use `i` to make changes and `:x` to save changes

4. trim `snp_position.txt` so that it only has columns "SNP_ID", "Chromosome", and "Position"  
	* `cut -f 1,3,4 snp_position.txt > trim_snp_position.txt`
	
5. trim `transposed_maize_genotypes.txt` and `transposed_teosinte_genotypes.txt` so that they do not have the line "JG_OTU" and "Group"
	* `grep -v -e "^JG_OTU" -e "^Group" transposed_maize_genotypes.txt > trim_tp_maize_genotypes.txt`  
	* `grep -v -e "^JG_OTU" -e "^Group" transposed_teosinte_genotypes.txt > trim_tp_teosinte_genotypes.txt`

6. sort `trim_snp_position.txt` based on column 1 which is `SNP_ID`; keep the header intact  
	* `(head -n 1 trim_snp_position.txt && tail -n +2 trim_snp_position.txt | sort -k1,1V -g -f) > sort_trim_snp_position.txt` 

7. sort `trim_tp_maize_genotypes.txt` and `trim_tp_teosinte_genotypes.txt` based on column 1 which is `SNP_ID`; keep the header intact  
	* `(head -n 1 trim_tp_maize_genotypes.txt && tail -n +2 trim_tp_maize_genotypes.txt | sort -k1,1V -g -f) > sort_trim_tp_maize_genotypes.txt`  
	* `(head -n 1 trim_tp_teosinte_genotypes.txt && tail -n +2 trim_tp_teosinte_genotypes.txt | sort -k1,1V -g -f) > sort_trim_tp_teosinte_genotypes.txt` 

8. check if `sort_trim_snp_position.txt`, `sort_trim_tp_maize_genotypes.txt`, and `sort_trim_tp_teosinte_genotypes.txt` have the same dimension  
	* `wc filename.txt` shows that they all have 984 lines, as expected

9. join `sort_trim_snp_position.txt` with `sort_trim_tp_maize_genotypes.txt` and `sort_trim_tp_teosinte_genotypes.txt` respectively. `sort_trim_snp_position.txt` is the larger file, and the resulting files should have the first column as `SNP_ID`, the second column as `Chromosome`, the third column as `Position`, and subsequent columns as genotype data from either maize or teosinte individuals. The header line of the genotype data columns is `Sample_ID` found in `fang_et_al_genotypes.txt`  
	* `join --header -1 1 -2 1 -a 1 -t $'\t' sort_trim_snp_position.txt sort_trim_tp_maize_genotypes.txt > maize_join.txt`  
	* `join --header -1 1 -2 1 -a 1 -t $'\t' sort_trim_snp_position.txt sort_trim_tp_teosinte_genotypes.txt > teosinte_join.txt`

10. check the output of join  
	* `wc maize_join.txt` and `wc teosinte_join.txt`: both have 984 lines  
	* `grep -q $'\t' maize_join.txt | echo $?` and `grep -q $'\t' teosinte_join.txt | echo $?`: both are tab-delimited

11. make three files from `maize_join.txt` with values for position, unknown positions, and multiple positions, respectively  
	* `grep -v -e "unknown" -e "multiple" maize_join.txt > maize_have_position.txt`
	* `grep -e "Position" -e "unknown" maize_join.txt > maize_unknown_position.txt`
	* `grep -e "Position" -e "multiple" maize_join.txt > maize_multiple_position.txt`

12. check the results of step 11  
	* `cut -f3 maize_join.txt | sort | uniq -c`: there are 11 rows with multiple positions, 27 rows with unknown positions, and 6 rows with no entry in the position column
	* `cut -f3 maize_unknown_position.txt | sort | uniq -c`: the output file does have 27 rows with "unknown" in the position column
	* `cut -f3 maize_multiple_position.txt | sort | uniq -c`: the output file does have all 11 rows with "multiple" in the position column, but also have the 6 rows with no entry. By using a combination of `head`, `tail`, and `cut`, I found that those 6 rows have "multiple" in the second column, which is the chromosome column 
	* `wc maize_have_position.txt`: it has 940 rows. 984-27-11-6=940, meaning that step 11 is successful

13. separate `maize_have_position.txt` by chromosome number  
	* `awk '$2 == 1 || $2 ~ /Chromosome/' maize_have_position.txt > maize_chr1.txt` repeat the command for 10 times by replacing the number after "==" to the corresponding chromosome number

14. sort `maize_chr?.txt` in increasing order (in for increasing)
	* `(head -n 1 maize_chr1.txt && tail -n +2 maize_chr1.txt | sort -k3,3g) > maize_chr1_in.txt`  
	Because the missing data is alreay represented by "?", the resulting files here are final for requirement: "10 files (1 for each chromosome) with SNPs ordered based on increasing position values and with missing data encoded by this symbol: ?".
	
14. sort `maize_chr?_de.txt` in decreasing order and replace missing data with  "-"  
	* `(head -n 1 maize_chr10.txt && tail -n +2 maize_chr10.txt | sort -k3,3gr) | sed 's/?/-/g' > maize_chr10_de.txt`  
	The resulting files from this step are final for the requirement: "10 files (1 for each chromosome) with SNPs ordered based on decreasing position values and with missing data encoded by this symbol: -"
	
15. repeat step 11 - step 14 to perform the same operation on teosinte data  
	* `grep -v -e "unknown" -e "multiple" teosinte_join.txt > teosinte_have_position.txt`
	* `grep -e "Position" -e "unknown" teosinte_join.txt > teosinte_unknown_position.txt`
	* `grep -e "Position" -e "multiple" teosinte_join.txt > teosinte_multiple_position.txt`  
	* `awk '$2 == 1 || $2 ~ /Chromosome/' teosinte_have_position.txt > teosinte_chr1.txt`
	* `(head -n 1 teosinte_chr1.txt && tail -n +2 teosinte_chr1.txt | sort -k3,3g) > teosinte_chr1_in.txt`
	* `(head -n 1 teosinte_chr1.txt && tail -n +2 teosinte_chr1.txt | sort -k3,3gr) | sed 's/?/-/g' > teosinte_chr1_de.txt`

16. move all 44 final files into directory `final_products/`  
	* `mv maize_chr*_* teosinte_chr*_* final_products/`
	* `mv maize_multiple_position.txt maize_unknown_position.txt teosinte_multiple_position.txt teosinte_unknown_position.txt final_products/`
	* inside the directory, check that it has 44 files: `ls -1 | wc -l`

17. move two source files, transpose script, and all intermediate files to `starting_intermediate_files/`  
	* `mv *.txt transpose.awk starting_intermediate_files/`

18. move this README file to Nova  
	* home directory in the Files dropdown tab in Nova OnDemand
	
18. update to github  
	* `git add .`, `git commit -m`, `git remote add origin git@github.com:ynfu23/BCB5460_AS1.git`
	* `git push origin master`