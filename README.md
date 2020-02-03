# PipeIT
PipeIT is a stand-alone Singularity Container for somatic variant calling on the Ion Torrent platform.

### Introduction
We present PipeIT, an accurate somatic variant calling pipeline specific for Ion Torrent sequencing data. The pipeline has been enclosed into a Singularity image to allow easy and high throughput analyses.

### Software requirements
The only requirement of PipeIT is a working Singularity installation. Please visit the [official website](https://singularity.lbl.gov/) for more information.

### Installation
PipeIT can be downloaded from our laboratory's website: http://oncogenomicslab.org/software-downloads/. 
No further installation is required, all the dependencies needed to perform the whole analysis are already installed within the container.

### Running the pipeline
PipeIT can be executed by simply running this command: 
```
singularity run PipeIT.img -t path/to/tumor.bam -n path/to/normal.bam -e path/to/region.bed 
```
The mandatory input files are the tumor and the normal BAM files, and the BED file of the targeted regions. The BAM files can be obtained directly from the Torrent Server. 
A number of optional parameters have been added in order to give more freedom to the user.
It is possible to:
* Submit the unmerged BED. If this file is not provided, PipeIT will build it.
* Specify the output filename. If not provided PipeIT will use the tumor bam's name.
* Skip the annotation.
* Keep all the intermediate files produced during the execution of the pipeline.
* Use different values for variant calling and filtering.


Alternatively, PipeIT can be executed for a tumor only analysis. Please keep in mind that a tumor-normal analysis will return more accurate results. The standard tumor only analysis can be executed with this command:
```
singularity run PipeIT.img -k path/to/tumor.bam -e path/to/region.bed -c path/to/annovar/humandb/folder 
```
The mandatory input files are the tumor BAM, the BED file of the targeted regions and the folder with Annovar's database files.
Please note that you need to download manually Annovar's database files, either directly with Annovar or using PipeIT.
In the latter case the commands are:
```
singularity exec PipeIT.img annotate_variation.pl -downdb -webfrom annovar -buildver hg19 esp6500siv2_all humandb/
singularity exec PipeIT.img annotate_variation.pl -downdb -webfrom annovar -buildver hg19 1000g2015aug humandb/
singularity exec PipeIT.img annotate_variation.pl -downdb -webfrom annovar -buildver hg19 exac03 humandb/
```

For more information on both the analyses please run:
```
singularity run PipeIT.img --help
```

While specifying input files please note that due to Singularity's nature:
- Paths to input files have to be *Relative*
> ... relative paths will resolve outside the container, and fully qualified paths will resolve inside the container.

Please read [Singularity's FAQ page](http://singularity.lbl.gov/archive/docs/v2-2/faq) for more information about this.
- Singularity automatically mounts some folders inside the container:
> ... Some of the bind paths are automatically derived (e.g. a user’s home directory) and some are statically defined (e.g. bind path in the Singularity configuration file). In the default configuration, the directories $HOME, /tmp, /proc, /sys, and /dev are among the system-defined bind points. 

Input files should be inside these folders to make them accessible to Singularity, otherwise PipeIT cannot find them.
User can also manually mount additional files and folders using the -B flag, in which case the folders and files within these folders are accesile to Singularity. For example:
```
singularity run -B /myHPC/home/username/files/ PipeIT.img -t RELATIVEpath/to/tumor.bam -n RELATIVEpath/to/normal.bam -e RELATIVEpath/to/region.bed
```

For a comprehensive description on which folders are automatically mounted within the container and on how to mount additional folders, users should read [Singularity's official documentation](http://singularity.lbl.gov/docs-mount).

### Output files
PipeIT will locally create a folder that will include both the final output (a VCF file), and the intermediate files . The latter will be automatically deleted by PipeIT at the end of its execution, unless differently indicated.

Please note that an empty final VCF file means that PipeIT have found no mutation in the input sample.

### Tumor-Normal Workflow
<img src="https://github.com/ckynlab/PipeIT/blob/master/images/Workflow.png" align="right" width="300" >

- Step 1 (Variant calling): Variant calling using the Torrent Variant Caller is performed.
- Step 2 (Post-processing multiallelic variants): Multiallelic variants are split, left aligned, trimmed and merged once again with the biallelic variants using BCFtools and GATK.
- Step 3 (Variant filtration): Variants outside of the target regions are removed. Hotspot variants are whitelisted. Variants covered by fewer than 10 reads in either the tumor or the matched normal sample or supported by fewer than 8 reads are removed. Furthermore, variants not likely to be somatic based on the ratio of VAF between tumor and normal (default to minimum 10:1) are also removed. Given the clinical significance of many hotspot mutations, we conservatively whitelist hotspot mutations even if they do not pass all read count and/or VAF filters. We recommend reviewing the whitelisted hotspot variants that did not pass the above read count and/or VAF filters. 
- Step 4 (Variant annotation): SnpEff annotation using canonical transcripts.









### Tumor only Workflow
<img src="https://github.com/ckynlab/PipeIT/blob/master/images/TOdiagram.png" align="left" width="300" >

- Step 1 (Variant calling): Variant calling using the Torrent Variant Caller is performed.
- Step 2 (Post-processing multiallelic variants): Multiallelic variants are split, left aligned, trimmed and merged once again with the biallelic variants using BCFtools and GATK.
- Step 3 (Annotations): Annovar is used to annotate on 3 different databases (1000 Genomes Project, Exome Aggregation Consortium (ExAC) and NHLBI-ESP project) the variants found by TVC. Furthermore, GATK is used to see if the variants found are in homopolymer regions.
- Step 4 (Variant filtration): Variants outside of the target regions are removed.  Variants covered by fewer than 10 reads in either the tumor or the matched normal sample or supported by fewer than 8 reads are removed. Previous annotations are used to remove any variant found in more than the 5% of the whole cohorts in at least one of the three databases or in a homopolymer region longer than 4. Furthermore, variants not likely to be somatic based on the ratio of VAF between tumor and normal (default to minimum 10:1) are also removed. If this ratio is higher than 0.9 or lower than 0.6 the population databases are used again to remove any variant found, no matter how often. 
- Step 5 (Pool of normal filtering): A list of variants found in a pool of normal samples unmatched with the original tumor samples are used to remove possible germline mutations. Hotspot variants are whitelisted. Given the clinical significance of many hotspot mutations, we conservatively whitelist hotspot mutations even if they do not pass any of these filters. We recommend reviewing the whitelisted hotspot variants that did not pass the above read count and/or VAF filters. 



### Versions

* 1.1.0  First release
* 1.1.1-3 Bug fixes
* 1.2.1 Added the option for an anlysis without a paired normal bam file
* 1.2.2-3 Bug fixes
* 1.2.4 Added Exome Aggregation Consortium (ExAC) in the annotation and filtering in the tumor only analysis

***

## Docker version

**Please note that the Docker image may not be up-to-date. Please use the Singularity image or contact us**

A Docker image has also been built for Docker users and can be found on the Docker Hub page: https://hub.docker.com/r/ckynlab/pipeit/. We suggest to use the Singularity version of PipeIT because the pipeline was defined to work on HPC environments, execution times on local machines will not be optimal.

### Running the pipeline
Similar to the Singularity image, the Docker version of PipeIT can also be launched using a simple command. However, due to Docker's behaviour with files external to the container itself, the user needs to mount the folder containing the input file within the container itself.
One easy option could be to create a folder called "data", use it to store all the input files and launch the command: 
```
docker run  --mount type=bind,source="$(pwd)"/data,target=/PipeIT/data,consistency=consistent -it pipeit:latest -t nameoftumor.bam -n nameofnormal.bam -e nameofregion.bed [-u nameofunmerged.bed]
```
Please notice that if you are using the "data" folder you must only use the name of the files, not the path.
PipeIT will create the output files within this directory.
