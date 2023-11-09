# NF-JaBbA (Nextflow - Junction Balance Analysis Pipeline)
```
▐▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▀▌
▐                                                                                                  ▌
▐                  ██████                   █████           ███████████  █████       █████████     ▌
▐                 ███░░███                 ░░███           ░░███░░░░░███░░███       ███░░░░░███    ▌
▐    ████████    ░███ ░░░                   ░███   ██████   ░███    ░███ ░███████  ░███    ░███    ▌
▐   ░░███░░███  ███████    ██████████       ░███  ░░░░░███  ░██████████  ░███░░███ ░███████████    ▌
▐    ░███ ░███ ░░░███░    ░░░░░░░░░░        ░███   ███████  ░███░░░░░███ ░███ ░███ ░███░░░░░███    ▌
▐    ░███ ░███   ░███                 ███   ░███  ███░░███  ░███    ░███ ░███ ░███ ░███    ░███    ▌
▐    ████ █████  █████               ░░████████  ░░████████ ███████████  ████████  █████   █████   ▌
▐   ░░░░ ░░░░░  ░░░░░                 ░░░░░░░░    ░░░░░░░░ ░░░░░░░░░░░  ░░░░░░░░  ░░░░░   ░░░░░    ▌
▐                                                                                                  ▌
▐▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▄▌
```

[![Cite with Zenodo](http://img.shields.io/badge/DOI-10.5281/zenodo.XXXXXXX-1073c8?labelColor=000000)](https://doi.org/10.5281/zenodo.XXXXXXX)

[![Nextflow](https://img.shields.io/badge/nextflow%20DSL2-%E2%89%A523.04.0-23aa62.svg)](https://www.nextflow.io/)
[![run with docker](https://img.shields.io/badge/run%20with-docker-0db7ed?labelColor=000000&logo=docker)](https://www.docker.com/)
[![run with singularity](https://img.shields.io/badge/run%20with-singularity-1d355c.svg?labelColor=000000)](https://sylabs.io/docs/)
[![Launch on Nextflow Tower](https://img.shields.io/badge/Launch%20%F0%9F%9A%80-Nextflow%20Tower-%234256e7)](https://tower.nf/launch?pipeline=https://github.com/mskilab-org/nf-jabba)

## Introduction

**mskilab-org/nf-JaBbA** is a new state-of-art bioinformatics pipeline from [`mskilab-org`](https://www.mskilab.org/) that is intended to run [`JaBbA`](https://github.com/mskilab-org/JaBbA/tree/master), a MIP based joint inference of copy number and rearrangement state in cancer whole genome sequence data. It runs all the pre-requisite modules in `mskilab-org` necessary for JaBbA. This pipeline is built to handle tumor-normal pairs as input and is designed and tested to run on Human samples. 

We drew our inspiration and ideas from [`nf-core/Sarek`](https://github.com/nf-core/sarek), a workflow designed to detect variants in whole genome or targeted sequencing data. **`nf-jabba`** is built using [`Nextflow`](https://www.nextflow.io/) and is implemented using `Nextflow DSL2`. All the modules use [`Docker`](https://www.docker.com/) and [`Singularity`](https://sylabs.io/docs/) containers, making the pipeline easily reproducible. Some of the modules/processes are derived from open source [`nf-core/modules`](https://github.com/nf-core/modules).

This pipeline has been designed to start from **FASTQ** files or directly from **BAM** files. Paths to these files should be supplied in a **CSV** file (*please refer to the documentation below for the input format of the .csv file*). We incorporated a modified version of the `Alignment` step from `nf-core/Sarek` for the `nf-JaBbA` pipeline. 

## Workflow Summary:
1. Alignment to Reference Genome (currently supports `BWA-MEM` & `BWA-MEM2`)
2. Quality Control (using [`FastQC`](https://www.bioinformatics.babraham.ac.uk/projects/fastqc/))
3. Trimming (must turn on using `--trim_fastq`) (using `fastp`)
4. Marking Duplicates (using `GATK MarkDuplicates`)
5. Base recalibration (using `GATK BaseRecalibrator`)
6. Applying BQSR (using `GATK ApplyBQSR`)
7. Performing structural variant calling (using [`SVABA`](https://github.com/walaj/svaba) and/or [`GRIDSS`](https://github.com/PapenfussLab/gridss); must mention using `--tools`)
8. Perform pileups (using mskilab's custom `HetPileups` module; must mention using `--tools`)
9. Generate raw coverages and correct for GC & Mappability bias (using [`fragCounter`](https://github.com/mskilab-org/fragCounter); must mention using `--tools`)
10. Remove biological and technical noise from coverage data. (using [`Dryclean`](https://github.com/mskilab-org/dryclean); must mention using `--tools`)
11. Perform segmentation using tumor/normal ratios of corrected read counts, (using the `CBS` (circular binary segmentation) algorithm; must mention using `--tools`)
12. Purity & ploidy estimation (currently supports [`ASCAT`](https://www.crick.ac.uk/research/labs/peter-van-loo/software) to pass ploidy values to JaBbA; must mention using `--tools`)
13. Execute JaBbA (using inputs from `Dryclean`, `CBS`, `HetPileups` and/or `ASCAT`; must mention using `--tools`)


## Usage

> **Note**
> If you are new to Nextflow and nf-core, please refer to [this page](https://nf-co.re/docs/usage/installation) on how
> to set-up Nextflow. Make sure to [test your setup](https://nf-co.re/docs/usage/introduction#how-to-run-a-pipeline)
> with `-profile test` before running the workflow on actual data.

### Setting up the ***samplesheet.csv*** file for input:

You need to create a samplesheet with information regarding the samples that you want to run the pipeline on. You need to specify the path of your **samplesheet** using the `--input` flag to specify the location. Make sure the input file is a *comma-separated* file and contains the headers discussed below. *It is highly recommended to provide the **absolute path** for inputs inside the samplesheet rather than relative paths.*

To mention a sample as paired tumor-normal, it has to be specified with the same `patient` ID, a different `sample`, and their respective `status`. A **1** in the `status` field indicates a tumor sample, while a **0** indicates a normal sample. If there are multiple `sample` IDs, `nf-jabba` will consider them as separate samples and output the results in separate folders based on the `patient` attribute. All the runs will be separated by `patient`, to ensure that there is no mixing of outputs.

You need to specify the desired output root directory using `--outdir` flag. The outputs will then be stored in your designated folder, organized by `tool` and `sample`.

To run the pipeline from the beginning, first create an `--input` `sampleSheet.csv` file with your file paths. A typical input whould look like this:

```csv
patient,sex,status,sample,lane,fastq_1,fastq_2
TCXX49,XX,0,TCXX49_N,lane_1,/path/to/fastq_1.fq.gz,/path/to/fastq_2.gz
```
Each row represents a pair of fastq files (paired end) for each sample.
After the input file is ready, you can run the pipeline using:

```bash
nextflow run mskilab-org/nf-jabba \
   -profile <docker/singularity/.../institute> \
   --input samplesheet.csv \
   --outdir <OUTDIR> \
   --tools <svaba,fragcounter,dryclean,cbs,hetpileups,ascat,jabba> \
   --genome <GATK.GRCh37/GATK.GRCh38>
```
> **Warning:**
> Please provide pipeline parameters via the CLI or Nextflow [`-params-file`](https://www.nextflow.io/blog/2020/cli-docs-release.html) option. Custom config files including those
> provided by the `-c` Nextflow option can be used to provide any configuration _**except for parameters**_;
> see [docs](https://nf-co.re/usage/configuration#custom-configuration-files).


### Discussion of expected fields in input file and expected inputs for each `--step`

A typical sample sheet should populate with the column names as shown below:

|   Column Name   |                                               Description                                                                                                 |
|-----------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------|
|   patient       | Patient or Sample ID. This should differentiate each patient/sample. *Note*: Each patient can have multiple sample names.                                 |
|   sample        | Sample ID for each Patient. Should differentiate between tumor and normal. Sample IDs should be unique to Patient IDs                                     |
|   lane          | If starting with FASTQ files, and if there are multiple lanes for each sample for each patient, mention lane name. **Required for `--step alignment`.         | 
|   sex           | If known, please provide the sex for the patient. For instance if **Male** type XY, else if **Female** type XX, otherwise put NA.     |
|   status        | This should indicate if your sample is **tumor** or **normal**. For **normal**, write 0, and for **tumor**, write 1.                                          |
|   fastq_1       | Full Path to FASTQ file read 1. The extension should be `.fastq.gz` or `.fq.gz`. **Required** for `--step alignment`.                                     |
|   fastq_2       | Full Path to FASTQ file read 2. The extension should be `.fastq.gz` or `.fq.gz`. **Required** for `--step alignment`.                                     |
|   bam           | Full Path to BAM file. The extension should be `.bam`. **Required** for `--step sv_calling`.                                                              |
|   bai           | Full Path to BAM index file. The extension should be `.bam.bai`. **Required** for `--step sv_calling`.                                                    |
|   cram          | Full Path to CRAM file. The extension should be `.cram`. **Required** for `--step sv_calling` if file is of type `CRAM`.                                  |
|   crai          | Full Path to CRAM index file. The extension should be `.cram.crai`. **Required** for `--step sv_calling` if file is of type `CRAM`.                       |
|   table         | Full path to Recalibration table file. **Required** for `--step recalibrate`.                                                                             |
|   vcf           | Full path to VCF file. **Required** for `--step jabba`.                                                                                                   |
|   hets          | Full path to HetPileups .txt file. **Required** for `--step jabba`.                                                                                       |


For more information regarding the pipeline usage and the inputs necesaary for each step, please follow the [Usage](docs//usage.md) documentation.

### Helpful Core Nextflow Commands:

#### `-resume`
If a process of the pipeline fails or is interrupted at some point, Nextflow can resume from that point without having to start over from the beginning. You must specify this in the `CLI` or on the `command-line` when restarting a pipeline. You can also supply a run name to resume a specific run using: `-resume` [run-name]. Use the `nextflow log` command to show previous run names.

#### `-profile`
Use this parameter for choosing a configuration profile. Profiles contain configuration presets for different computing environments.

Several generic profiles have been provided by default which instruct the pipeline to use software packaged using different methods. You can use this option to run the pipeline via containers (singularity/Docker) (**highly recommended**) 

#### `-c`
You can mention custom configuration scripts to run the pipeline with using the `-c` flag and providing a path to the `.config` file. This is advised when you want to submit processes into an executor like `slurm` or `LSF`.

#### `-bg`
The Nextflow `-bg` flag launches the Nextflow pipeline as a background process. This allows you to detach or exit your terminal without interrupting the run. A log of the run will be saved inside a file upon completion. You can also use `screen` or `tmux` sessions to persist runs.

## Debugging any step/process:

To debug any step or process that failed, first check your current `execution_trace*.txt` file inside the `<outdir>/pipeline_info/` folder and gather the `hash` number for that process. Then navigate to the `work` directory and use that `hash` number to locate that process's working directory. This directory will contain multiple `.command.*` files that correspond to your run and contain valuable information that can help you debug your error. You can also run the `.command.sh` script to do a manual, isolated execution of the offending process.

## Credits

`nf-jabba` was originally written by [`Tanubrata Dey`](https://github.com/tanubrata) and [`Shihab Dider`](https://github.com/shihabdider) at the Perlmutter Cancer Center and the New York Genome Center.

We thank the following people for their extensive guidance in the development of this pipeline:
- [Marcin Imielinski](https://github.com/imielinski)
- [Joel Rosiene](https://github.com/jrosiene)


## Contributions and Support

If you would like to contribute to this pipeline, please see the [contributing guidelines](.github/CONTRIBUTING.md).

## Citations

An extensive list of references for the tools used by the pipeline can be found in the [`CITATIONS.md`](CITATIONS.md) file.

This pipeline uses code and infrastructure developed and maintained by the [nf-core](https://nf-co.re) community, reused here under the [MIT license](https://github.com/nf-core/tools/blob/master/LICENSE).

> **The nf-core framework for community-curated bioinformatics pipelines.**
>
> Philip Ewels, Alexander Peltzer, Sven Fillinger, Harshil Patel, Johannes Alneberg, Andreas Wilm, Maxime Ulysse Garcia, Paolo Di Tommaso & Sven Nahnsen.
>
> _Nat Biotechnol._ 2020 Feb 13. doi: [10.1038/s41587-020-0439-x](https://dx.doi.org/10.1038/s41587-020-0439-x).
