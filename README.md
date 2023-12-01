# Notes on the metagenomics profiling using Nextflow in csc-puhti

## Working directory configuration

Let's first configure the working directoy, for example:
```
├── config
├── DATA
├── SCRIPTS
├── DB
│   ├── db_mOTU
│   │   ├── db_mOTU_test
│   │   └── public_profiles
│   ├── human_CHM13
│   └── human_GRCh38
|			  └── db_MetaPhlan4	
├── RESULTS
```

- `config` to store all configuartion files needed for the pipeline to run
- `DATA` to store the the input raw sequence (in fastq or fastq.gz)
- `DATA` to store the pre-build reference database needed by the pipeline
- `RESULTS` placeholder for the output file
- `SCRIPTS` to store scripts needed for analysis

## Data sources

We will use the paired-end reads generated by Lee et al., 2017 study:https://microbiomejournal.biomedcentral.com/articles/10.1186/s40168-017-0270-x#Sec2.
The full set of samples available here: https://www.ebi.ac.uk/ena/browser/view/PRJNA353655
(total size 45 gb)

For simplicity we will only use the before and after (4 weeks) FMT data from 2 participants.

To download the data run this command:

```
sbatch ./SCRIPTS/download.sh
```


## Nextflow

We will be using the taxprofiler: https://nf-co.re/taxprofiler/1.1.2

### Set-up and submitting the job:

1. prepare the databse needed and create the full database sheet of tools you want to use

	- for metaphlan4 puhti followed set-up here: https://docs.csc.fi/apps/metaphlan/
	- cretae the database sheet:
		- config/database.csv


2. Prepare the samplesheet as requested by the workflow

	- for example data: config/samplesheet.csv

3. Select configurations of the optional params (read more here: https://nf-co.re/taxprofiler/1.1.2/parameters)

	- Read processing, activated with `--perform_shortread_qc`
			- Let's use fastp and use the tool's default adapter unless the adapter is diferent
			- no need to merge pairs if using metaphlan4 or motus
			- save the resulting processed reads with : `--save_preprocessed_reads`
	
	- Let's activate the complexity filtering using bbduk step using: `--perform_shortread_complexityfilter  --shortread_complexityfilter_tool bbduk`

	- Activate the host-read removal using current DB CHM13 : `--perform_shortread_hostremoval`
		- specify the DB of FASTA file downloaded from here: https://www.ncbi.nlm.nih.gov/data-hub/genome/GCA_009914755.4/
		Optional to provide the indexed file.
		`--hostremoval_reference /scratch/project_xxx/USER_WORKSPACES/xxx/exercise/nextflow_metagenome/DB/T2T-CHM13v2.0.zip`
		If you downloaded it from local dir copy it to puhti using `scp`


	- We can skip the Run merging for now
	
	- let's activate the option for table generation: `--run_profile_standardisation`

	- for profile let's use singularity using oprtion `-profile SIngularity`

	- tools that we want to run, let's try with metaphlan4 and motus using this option 

4. Be sure to update the pipeline regularly `nextflow pull nf-core/taxprofiler`

5. Let's create the sbatch script to submit the job, for sample less than 100, hyperqueue is not needed --> confirm again

	See example: https://yetulaxman.github.io/containers-workflows/hands-on/day4/nextflow-containers.html
	In case we need hyperqueue: 
	https://yetulaxman.github.io/containers-workflows/hands-on/day4/nf-core-hyperqueue.html

	see example: SCRIPTS/tax_profiler.sh

Submit the job using:

```
sbatch SCRIPTS/tax_profiler.sh
```

### Additional concerns to address:

- resources allocation in puhti
- customaize the nf-core pipeline if additional step needed --> functional annotation using humann3

