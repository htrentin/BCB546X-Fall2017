# UNIX Assignment - Henrique Trentin

## Step 1) Data Inspection
--> $ for filename in fang_et_al_genotypes.txt snp_position.txt ; do echo "N of lines at: $(wc -l $filename)"; echo  "N of columns at: $(awk -F "\t" '{print NF; exit}' $filename) $filename"; echo "File size of: $(du -h $filename)"; echo "Check if files are ASCII and if they have non-ASCII characters:" $(file $filename); done

**Output:**

N of lines at: 2783 fang_et_al_genotypes.txt

N of columns at: 986 fang_et_al_genotypes.txt

File size of: 11M       fang_et_al_genotypes.txt

Check is files are ASCII and if they have non-ASCII characters: fang_et_al_genotypes.txt: ASCII text, with very long lines, with CRLF line terminators **CR and LF are ASCII characters**

N of lines at: 984 snp_position.txt

N of columns at: 15 snp_position.txt

File size of: 84K       snp_position.txt

Check is files are ASCII and if they have non-ASCII characters: snp_position.txt: ASCII text, with CRLF line terminators **CR and LF are ASCII characters**

## Step 2) Data Processing
### Step 2.1) Extract selected Maize and Teosinte Genotypes from *fang_et_al_genotypes.txt*
--> $ grep -E "(ZMMIL|ZMMLR|ZMMMR)" fang_et_al_genotypes.txt > maize_genotypes.txt


--> $ grep -E "(ZMPBA|ZMPIL|ZMPJA)" fang_et_al_genotypes.txt > teosinte_genotypes.txt

### Step 2.2) Extract the header from the genotype file and add it back to the extracted files

--> $ grep "Group" fang_et_al_genotypes.txt > header.txt

--> $ cat header.txt maize_genotypes.txt > maize_header.txt 

--> $ cat header.txt teosinte_genotypes.txt > teosinte_header.txt

### Step 2.3) Remove *Sample ID* and *JG_OTU* columns so a common column *(SNP ID)* can be used to join the files

--> $ cut -f 3-968 maize_header.txt > maize_cut.txt 

--> $ cut -f 3-968 teosinte_header.txt > teosinte_cut.txt

### Step 2.4) Transpose the data so both the first column of genotype and SNP files have the SNP-ID in the first column

--> $ awk -f transpose.awk maize_cut.txt > transposed_maize.txt

--> $ awk -f transpose.awk teosinte_cut.txt > transposed_teosinte.txt

### Step 2.5) Keep only columns 1, 3 and 4 from snp.position.txt file (SNP ID, Chromosome number and SNP position in the Genome, respectively)
--> $ grep -v "^#" snp_position.txt | cut -f 1,3,4 > snp_position_cut.txt

### Step 2.6) Remove headers from the files
--> $ grep -v "Group" transposed_maize.txt > maize_no_header.txt

--> $ grep -v "Group" transposed_teosinte.txt > teosinte_no_header.txt 

--> $ grep -v "SNP_ID" snp_position.txt_cut > snp_no_header.txt

### Step 2.7) Sort both Genotype and SNP files to appropriately join them 
--> $ sort -k1,1 snp_no_header.txt > snp_sorted.txt 

--> $ sort -k1,1 maize_no_header.txt > maize_sorted.txt

-> $ sort -k1,1 teosinte_no_header.txt > teosinte_sorted.txt

### Step 2.8) Check if files are sorted

--> for filename in snp_sorted.txt maize_sorted.txt teosinte_sorted.txt; do sort -k1,1 $filename | echo $? $filename; done

**Output:**

0 snp_sorted.txt

0 maize_sorted.txt

0 teosinte_sorted.txt

### Step 2.9) Join SNP and Genotype files

--> $ join -t $'\t' -1 1 -2 1 snp_sorted.txt maize_sorted.txt > maize_joined.txt

--> $ join -t $'\t' -1 1 -2 1 snp_sorted.txt teosinte_sorted.txt > teosinte_joined.txt

### Step 2.10) Sort files by chromosome number

$ sort -k2,2n maize_joined.txt > maize_sorted_by_chr.txt 

$ sort -k2,2n teosinte_joined.txt > teosinte_sorted_by_chr.txt

### Step 2.11) Missing data is encoded by "?'. Substitute missing data with '-'

--> $ sed 's/?/-/g' maize_sorted_by_chr.txt > maize_dash.txt

--> $ sed 's/?/-/g' teosinte_sorted_by_chr.txt > teosinte_dash.txt

### Step 2.12) Create separate files for each chromosome

--> $ for i in {1..10}; do awk '$2=='$i'' maize_sorted_by_chr.txt > maize_chr"$i"_questionmark.txt; done

--> $ for i in {1..10}; do awk '$2=='$i'' teosinte_sorted_by_chr.txt > teosinte_chr"$i"_questionmark.txt; done

--> $ for i in {1..10}; do awk '$2=='$i'' maize_dash.txt > maize_chr"$i"_dash.txt; done

--> $ for i in {1..10}; do awk '$2=='$i'' teosinte_dash.txt > teosinte_chr"$i"_dash.txt; done

### Step 2.13) Sort files with missing data encoded by ? based on increasing SNP position values

--> $ for i in {1..10}; do sort -k3,3n maize_chr"$i"_questionmark.txt > maize_chr"$i"_questmark_ordered.txt; done

--> $ for i in {1..10}; do sort -k3,3n teosinte_chr"$i"_questionmark.txt> teosinte_chr"$i"_questmark_ordered.txt; done

### Step 2.14) Sort files with missing data encoded by - based on decreasing SNP position values

--> for i in {1..10}; do sort -k3,3nr maize_chr"$i"_dash.txt > maize_chr"$i"_dash_ordered.txt; done

--> $ for i in {1..10}; do sort -k3,3nr teosinte_chr"$i"_dash.txt > teosinte_chr"$i"_dash_ordered.txt; done

### Step 2.15) Create a file with all SNPs with unknown position in the genome

--> $ grep "unknown" maize_sorted_by_chr.txt > maize_unknown.txt

--> $ grep "unknown" teosinte_sorted_by_chr.txt > teosinte_unknown.txt

### Step 2.16) Create a file with all SNPs with multiple position in the genome

--> $ grep "multiple" maize_sorted_by_chr.txt > maize_multiple.txt

--> $ grep "multiple" teosinte_sorted_by_chr.txt > teosinte_multiple.txt

### Step 2.17) Adding files to folders

--> $ mkdir Results Results/Maize_Questionmark Results/Teosinte_Questionmark Results/Maize_Dash Results/Teosinte_Dash  Results/MaizeandTeosinte_UnknownPositions Results/MaizeandTeosinte_MultiplePositions Intermediate_files

--> $ mv maize_chr*_questmark_ordered.txt Results/Maize_Questionmark

--> $ mv teosinte_chr*_questmark_ordered.txt Results/Teosinte_Questionmark

--> $ mv maize_chr*_dash_ordered.txt Results/Maize_Dash

--> $  mv teosinte_chr*_dash_ordered.txt Results/Teosinte_Dash

--> $ mv maize_unknown.txt teosinte_unknown.txt Results/MaizeandTeosinte_UnknownPositions/

--> $ mv maize_multiple.txt teosinte_multiple.txt Results/MaizeandTeosinte_MultiplePositions/

--> $ mv m*.txt t*.txt snp_position_cut.txt snp_no_header.txt snp_sorted.txt h*.txt Intermediate_files

### Step 2.18) Stage and Commit changes

--> $ git add . 

--> $ git commit -m "Completed assignment" 

--> $ git push origin master

