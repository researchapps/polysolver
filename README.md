# Singularity Polysolver

This will produce a Singularity image (suitable for running in a cluster environment) using [https://hub.docker.com/r/sachet/polysolver/](https://hub.docker.com/r/sachet/polysolver/). We do this by way of a bootstrap file for the Docker image.


## 1. Install Singularity

Instructions can be found on the [singularity site](https://singularityware.github.io).


## 2. Bootstrap the image


    sudo singularity create --size 4000 polysolver.img
    sudo singularity bootstrap polysolver.img Singularity


## 3. Running commands

For each of the following commands, you will want to [mount a local data folder](http://singularity.lbl.gov/docs-mount) to `/data` inside the container, and then provide paths to `/data` to the executable.

      
## Polysolver Documentation

## Description

This software package consists of 3 main tools:


### 1.1 POLYSOLVER (POLYmorphic loci reSOLVER)

This tool can be used for HLA typing based on an input exome BAM file and is currently infers infers alleles for the three major MHC class I  (HLA-A, -B, -C).


      singularity exec polysolver.img /usr/local/libexec/polysolver/scripts/shell_call_hla_type


#### Input:

	
	-bam: path to the BAM file to be used for HLA typing
	-race: ethnicity of the individual (Caucasian, Black, Asian or Unknown)
	-includeFreq: flag indicating whether population-level allele frequencies should be used as priors (0 or 1)
	-build: reference genome used in the BAM file (hg18 or hg19)
	-format: fastq format (STDFQ, ILMFQ, ILM1.8 or SLXFQ; see Novoalign documentation)
	-insertCalc: flag indicating whether empirical insert size distribution should be used in the model (0 or 1)
	-outDir: output directory


#### Output:

- `winners.hla.txt`: file containing the two inferred alleles for each of HLA-A, HLA-B and HLA-C


### 1.2 POLYSOLVER-based mutation detection


This tool works on a tumor/normal pair of exome BAM files and inferred mutations in the tumor file. It assumes that POLYSOLVER has already been run on the normal BAM.


      singularity exec polysolver.img /usr/local/libexec/polysolver/scripts/shell_call_hla_mutations_from_type



#### Input:


	-normal_bam_hla: path to the normal BAM file
	-tumor_bam_hla: path to the tumor BAM file
	-hla: inferred HLA allele file from POLYSOLVER (winners.hla.txt or winners.hla.nofreq.txt)
	-build: reference genome used in the BAM file (hg18 or hg19)
	-format: fastq format (STDFQ, ILMFQ, ILM1.8 or SLXFQ; see Novoalign documentation)
	-outDir: output directory	  

	

#### Output:


	call_stats.$allele.out: Mutect output for each inferred allele in winners.hla.txt
	$allele.all.somatic.indels.vcf: Strelka output for each inferred allele in winners.hla.txt


### 1.3 Annotation of mutations

This tool annotates the predicted mutations from (ii) with gene compartment and amino acid change information


      singularity exec polysolver.img /usr/local/libexec/polysolver/scripts/shell_annotate_hla_mutations


#### Input:


	-indiv: individual ID, used as prefix for output files
	-dir: directory containing the raw call files (Mutect: call_stats*, Strelka: *all.somatic.indels.vcf). Also the output directory	

	
### Output:


        (a). Mutect
	$indiv.mutect.unfiltered.nonsyn.annotated -  list of all unfiltered mutations
	$indiv.mutect.filtered.nonsyn.annotated -  list of cleaned non-synonymous mutations
	$indiv.mutect.filtered.syn.annotated - list of cleaned synonymous changes
	$indiv.mutect.ambiguous.annotated - list of ambiguous calls. This will generally be empty (save for the header). It will be populated if the same mutation (ex. p.A319E) is found in two or more alleles in the individual, with the same allele fractions. In such cases one allele is randomly chosen and included in the .nonysn.annotated file while the complete list of alleles is listed in the .ambiguous.annotated file. If the ethnicity of the individual is known, an alternate method would be to pick the allele with the highest frequency.
 
        (b). Strelka
	$indiv.mutect.unfiltered.nonsyn.annotated -  list of all unfiltered indels (as detected by Strelka)
	$indiv.strelka_indels.filtered.annotated - list of cleaned indels (as detected by Strelka)
	$indiv.strelka_indels.ambiguous.annotated - see description of $indiv.mutect.ambiguous.annotated in (a). above


   
### Testing

Your installation can be tested by running the following command from $PSHOME. **NOTE: this has not been tested - the correct paths to dependencies need to be set in [Singularity](Singularity) and the data folders correctly mounted from the local machine.**


#### 3.1 POLYSOLVER


      singularity exec polysolver.img /usr/local/libexec/polysolver/scripts/shell_call_hla_type test/test.bam Unknown 1 hg19 STDFQ 0 test


If successful, the following command should not yield any differences:

 
      diff test/winners.hla.txt test/orig.winners.hla.txt



#### 3.2 POLYSOLVER-based mutation detection


      singularity exec polysolver.img /usr/local/libexec/polysolver/scripts/shell_call_hla_mutations_from_type test/test.bam test/test.tumor.bam test/winners.hla.txt hg19 STDFQ test

If successful, the following command should not yield any differences:

 
      diff test/call_stats.hla_b_39_01_01_02l.out test/orig.call_stats.hla_b_39_01_01_02l.out 



#### 3.3 Annotation of mutations


      singularity exec polysolver.img /usr/local/libexec/polysolver/scripts/shell_annotate_hla_mutations indiv test



If successful, the following command should not yield any differences:


      diff test/indiv.mutect.filtered.nonsyn.annotated test/orig.indiv.mutect.filtered.nonsyn.annotated



### Running

The tools can be run using the following commands:



#### 4.1 POLYSOLVER


      singularity exec polysolver.img /usr/local/libexec/polysolver/scripts/shell_call_hla_type </path/to/bam> <race> <includeFreq> <build> <format> <insertCalc> </path/to/output_directory>


example:


      singularity exec polysolver.img /usr/local/libexec/polysolver/scripts/shell_call_hla_type test/test.bam Unknown 1 hg19 STDFQ 0 test



#### 4.2 POLYSOLVER-based mutation detection


      singularity exec polysolver.img /usr/local/libexec/polysolver/scripts/shell_call_hla_mutations_from_type </path/to/normal_bam> </path/to/tumor_bam> </path/to/winners.hla.txt> <build> <format> </path/to/output_directory>


example:


      singularity exec polysolver.img /usr/local/libexec/polysolver/scripts/shell_call_hla_mutations_from_type test/test.bam test/test.tumor.bam test/winners.hla.txt hg19 STDFQ test

 
#### 4.3 Annotation of mutations


      singularity exec polysolver.img /usr/local/libexec/polysolver/scripts/shell_annotate_hla_mutations <prefix_to_use> </path/to/directory_with_mutation_detection_output>


example:


      singularity exec polysolver.img /usr/local/libexec/polysolver/scripts/shell_annotate_hla_mutations indiv test
