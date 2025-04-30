# 2025 Workshop on Genome Annotation (Barcelona)

This repository contains course materials for a workshop on structural genome annotation with BRAKER, GALBA, and TSEBRA.

Authors: Katharina Hoff & Natalia Nenasheva

Contact: katharina.hoff@uni-greifswald.de

Link to wall with tool names: xxx

## Course contents

   * theory: repeat library generation and repeat masking with RepeatModeler2/RepeatMasker
   * theory: short read RNA-Seq to genome alignment with Hisat2
   * theory: sorting an RNA-Seq alignment file with Samtools
   * practice: application of BRAKER3 (structural genome annotation with RNA-Seq alignments and a large protein data base)
   * practice: application of BRAKER1 (structural genome annotation with short read RNA-Seq alignments)
   * practice: application of BRAKER2 (structural genome annotation with protein database)
   * practice: application of GALBA (structural genome annotation with proteins of a closely related species, suitable for e.g. vertebrate genomes)
   * practice: merging BRAKER1 and BRAKER2 (or GALBA) gene sets with TSEBRA
   * practice: BUSCO assessment of predicted gene set
   * practice: preparing an assembly hub for the UCSC Genome Browser with MakeHub 
   * for advanced learners: annotate a chromsome of *Basesia duncani*

## Instructions for Setup in Barcelona (for Particpants)

1. Open a terminal and ssh into your instance with port forwarding:

   `ssh -L 9002:localhost:9002 user@server`
   
   Keep this ssh connection open for the entire workshop. If the connection breaks, just reconnect with the same command.

2. In your terminal, in your home (~), make a new directory for the git clone

```
cd ~
git clone https://github.com/KatharinaHoff/GenomeAnnotation_Workshop_Barcelona.git
```

This will create a folder called `GenomeAnnotation_Workshop_Barcelona` in your home directory. This folder contains the JupyterNotebook for this course (GenomeAnnotation.ipynb) ☺️

### Running Jupyter Server

The organizers of the Barcelona Genome Annotation Workshop have already compiled a singularity file called `genome_annotation.sif` for you. You can find this file at `/data/classes/ibe_genann2025/shared_data/hoff_classes/`.

With the cloned data and the singularity file (`genome_annotation.sif`), you can run the image for JupyterNotebook display in the terminal. Please start screen before proceeding to ensure that the server will keep running even if you close your terminal window:

```
screen
```
and hit enter 2x. (Here is a randomly picket tutorial on the general usage of screen: https://linuxize.com/post/how-to-use-linux-screen/ .)

Afterwards, in that screen session, execute the following:

```
conda activate singularity
IMAGE=/data/classes/ibe_genann2025/shared_data/hoff_classes/genome_annotation.sif
DATA=/data/classes/ibe_genann2025/shared_data/hoff_classes/GenomeAnnotation_Workshop_Barcelona/DATA
GIT=~/GenomeAnnotation_Workshop_Barcelona
cd ~ # just to be sure you are not anywhere else!
singularity exec --cleanenv --bind ${DATA}:${DATA} --bind ${PWD}:${PWD} --bind $PWD:/home/jovyan ${IMAGE} jupyter notebook --no-browser --ip=127.0.0.1 --port=9002
```

This will display 3 links in your terminal. The links will look something like this:

```
http://127.0.0.1:9002/?token=b4ecdf0c807aad10aaa8fe12e71d9c3d0bf875c16d135527
```

Copy one of these links and paste it into the address line of your local Chrome browser (e.g. on your notebook).

Click on the folder `GenomeAnnotation_Workshop_Barcelona` to access the workshop content. Double click to open the GenomeAnnotation.ipynb. Welcome to the starting point of this lab 🤓

#### A few notes about running code in Jupyter Notebooks:

You can run code by clicking "Run" at the top of the screen

<img width="650" alt="Screenshot 2023-05-14 at 12 33 24" src="https://github.com/KatharinaHoff/GenomeAnnotation_Workshop/assets/38511308/d9374ebe-a77d-493d-95b5-d5ad1668e992">

Alternatively, you can click inside any box which starts with

```
%%script bash
```

and type Shift + Enter

You can stop a code block running by pressing "Stop" (the black square)

<img width="567" alt="Screenshot 2023-05-14 at 12 44 32" src="https://github.com/KatharinaHoff/GenomeAnnotation_Workshop/assets/38511308/45e6d335-1476-408c-8ca5-ecf6a7456ba3">



When a code block (the kernel) is running, it will look like this:

```
In [*]
```

When a code block has finished running:

```
In [num]
```

where num is an index of the job

When a code block has not yet been run:

```
In[ ]
```

To create a new code block, you click on "Insert"

## Obtaining Data

You do **not** have to download the data onto the Barcelona Workshop infrastructure. It has already been downloaded for you.
If you want to run the JupyterNotebooks of this Workshop on your infrastructure, you can download the data as follows (this will require **9 GB** of free space). 
Let's say $DATA is the location where you want to store all data for this session, you may have to modify that variable:

```
DATA=/data/classes/ibe_genann2025/shared_data/hoff_classes/GenomeAnnotation_Workshop_Barcelona/DATA/ # will require modification!
cd $DATA
wget https://raw.githubusercontent.com/KatharinaHoff/GenomeAnnotation_Workshop_Barcelona/refs/heads/master/obtain_data.sh
bash obtain_data.sh
```

## Obtaining the Singularity Image File

You do **not** have to build the image file for usage on the Barcelona Workshop infrastructure. It has already been built for you. If you want to obtain the same image for using it after the course, you can do so as follows (with singularity-ce version 3.11.2, available from https://github.com/sylabs/singularity, find their installation instructions at https://github.com/sylabs/singularity/blob/main/INSTALL.md, make sure you are not using an older version of singularity, as this may cause problems):

```
singularity build genome_annotation.sif docker://katharinahoff/bioinformatics-notebook
```

## Moving from JupyterLab to SLURM

The exact same container that is used for rendering our teaching materials with JupyterLab can be used on any HPC (that has singularity support) with a SLURM scheduler. In this case, you will not execute JupyterLab, but submit a task with SLURM for computation. Example for calling BRAKER3 with SLURM:

#### Script contents: braker3.sh

```
#!/bin/bash                                  
#SBATCH -o braker.%j.%N.out
#SBATCH -e braker.%j.%N.err
#SBATCH --get-user-env
#SBATCH --time=72:00:00
#SBATCH -N 1 # number of nodes, BRAKER does not scale across multiple nodes
#SBATCH -n 48 # number of threads on that node, BRAKER does not scale well to hundreds of threads, we often execute on 8-48 threads

module load singularity

(singularity exec -B $PWD:$PWD genome_annotation.sif braker.pl --genome=genome.fasta.masked --prot_seq=proteins.fa --bam=rnaseq.bam --threads 48 ) &> braker3.log
```

Any tasks from our JupyterNotebook cells can be implemented in such scripts.

#### Submit the script

Simply submit the job with sbatch:

``` 
sbatch braker3.sh
```

## Acknowledgements

Stefan Kemnitz from The University Compute Center at University of Greifswald (https://rz.uni-greifswald.de/dienste/allgemein/sonstiges/high-performance-computing/) kindly assisted in building docker containers for genome annotation with methods developed at University of Greifswald.

Josie Paris from Università Politecnica delle Marche provided very helpful instructions on how to use the cloud computing infrastructure during the Cesky Krumlov workshop.

