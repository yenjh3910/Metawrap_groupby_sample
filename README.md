# Metawrap_groupby_sample

## Binning pipeline
### Co-assembly (MEGAHIT)
```
#use bbnorm read
megahit -t 16 -m 0.95 -1 AT1_1.fastq,AT2_1.fastq,AT3_1.fastq,AT4_1.fastq,AT5_1.fastq -2 AT1_2.fastq,AT2_2.fastq,AT3_2.fastq,AT4_2.fastq,AT5_2.fastq --min-contig-len 1000 -o ~/megahit/megahit_coassembly_groupby_sample/AT_coassembly

#quast
~/quast-5.1.0rc1/quast.py ~/megahit/megehit_coassembly/final.contigs.fa -o ~/megahit/megehit_coassembly/coassembly_quast
```   

### metaWRAP (use fastq format except metabat2)
#### metabat2    
```   
cd ~/bowtie2    
bowtie2-build  ~/megahit/megehit_coassembly/final.contigs.fa coassembly_contig   
```   
copy clean reads to $bowtie2    
```   
bowtie2 -x coassembly_contig -1 {read1_1.fastq.gz,read2_1.fastq.gz,...} -2  {read1_2.fastq.gz,read2_2.fastq.gz,...} | samtools sort -o coassembly.sort.bam  
```
delete clean reads in $bowtie2    
```
conda activate metawrap-env
cd ~/metabat2
runMetaBat.sh ~/megahit/megehit_coassembly/final.contigs.fa ~/bowtie2/coassembly.sort.bam   
mv {final.contigs.fa.metabat-bins-20220508_131755} metabat2_bins
rm ~/metabat2/final.contigs.fa.depth.txt  
mv ~/metabat2/metabat2_bins ~/metawrap_run/initial_binning
```   
#### metaWRAP binning (maxbin2, concoct)   
 ```    
 metawrap binning -o ~/metawrap_run/initial_binning -t 16 -a ~/megahit/megehit_coassembly/final.contigs.fa --maxbin2 --concoct ~/clean_read/*fastq    
 ```
#### metaWRAP refinement  
```  
metawrap bin_refinement -o ~/metawrap_run/bin_refinement -t 16 -A ~/metawrap_run/initial_binning/metabat2_bins -B ~/metawrap_run/initial_binning/maxbin2_bins -C  ~/metawrap_run/initial_binning/concoct_bins -c 50 -x 10 -m 56 

#If error, maybe re-run concoct binning
metawrap binning -o ~/metawrap_run/initial_binning -t 16 -a ~/megahit/megehit_coassembly/final.contigs.fa --concoct ~/clean_read/*fastq
```  
#### metaWRAP quant_bins
```
metawrap quant_bins -b ~/metawrap_run/bin_refinement/metawrap_50_10_bins -o ~/metawrap_run/bin_quant -a ~/megahit/megehit_coassembly/final.contigs.fa ~/clean_read/*fastq -t 16  
conda deactivate
```

### GTBDK

**Usage**   
```   
conda activate gtdbtk-1.5.0   
gtdbtk classify_wf --genome_dir ~/metawrap_run/bin_refinement/metawrap_50_10_bins -x fa --out_dir ~/bins_gtdbdk --scratch_dir scratch.tempory --cpus 16    
conda deactivate
```  

### Bin Prodigal & CD-HIT  
Put all bin file in $\~/metawrap_run/bin_refinement/metawrap_50_10_bins  

**Usage**      
```  
#Edit shell script  
vim ~/bioinformatic_shell_script/bin_prodigal_cdhit.sh  
for i in {1..[number]}  

~/bioinformatic_shell_script/bin_prodigal_cdhit.sh  
```

### DIAMOND-blastx (ARGs, MGEs) & abricate (VFs)  
  
**Usage**  
```  
#Edit shell script  
vim ~/bioinformatic_shell_script/DIAMOND-blastx_abricate.sh 
for i in {1..[number]} 

~/bioinformatic_shell_script/DIAMOND-blastx_abricate.sh  
```  
