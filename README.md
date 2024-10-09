# Downloading Fantasia

## Install from source from the [Fantasia Github](https://github.com/MetazoaPhylogenomicsLab/FANTASIA/tree/main?tab=readme-ov-file)

This is a large download ~13GB so make sure that you are connected via ethernet or have good wifi

Once downloaded, scp it to your local cluster for analyses, this program takes a fair amount of ram

```
scp -r FANTASIA.tar.gz user@super.computer.edu:/scratch/projects/path/to/directory
```
In the directory you want to decompress this file
```
tar -xvzf FANTASIA.tar.gz
```

Note: This next section is pulled directly from their Github

However, there were a lot of dependency issues, leading to failures along the way. Just read the outputs carefully and it will guide you in how to proceed. 

```
conda create --name gopredsim --file tf_conda_env_spec.txt python=3.9.13

conda activate gopredsim #Check that everything is correctly installed

conda install -c bioconda seqtk #Install seqtk

# 3. Create the python environment and install the required packages

#Note: make sure that you are executing the python from the conda environment. Otherwise, there may be version conflicts

#If encountering any error saying that the version of the package is not available or not compatible with other package's version, modify requirements.txt and remove the version (it is indicated by "==XX.XX.X" and it will automatically find the one that is compatible with the rest of the packages). Be aware that the changing the version of some packages may affect the inference of the embeddings and the validity of results obtained.

#Note2: to deactivate python environments, just type "deactivate"

#General enviromment used for ProTT5 model (and could potentially be used for other models except SeqVec)

python3 -m venv venv

source venv/bin/activate

pip install -r requirements.txt

deactivate

# 4. Change "/path-to-FANTASIA-folder/" to the correct path in the scripts for launching the program.

FANTASIA_PATH=$(pwd) #You should be inside teh FANTASIA folder

sed -i "s|/path-to-FANTASIA-folder|$FANTASIA_PATH|g" launch_*.sh

# 5. Change "/path-to-FANTASIA-folder/" to the correct paths in the script to generate the config files required by the software.

sed -i "s|/path-to-FANTASIA-folder|$FANTASIA_PATH|g" generate_gopredsim_input_files.sh

# 6. GOPredSim can be run using CPUs or GPUs. CPUs is the default option selected in these scripts. However, GPUs can be used to increase the speed when dealing with large amounts of data. In clusters, make sure to select GPU nodes and to load the correspondent CUDA module when launching the pipeline. A sample script for executing the pipeline is included. Paths for execution and conda activation should be double checked.

# 7. By default, ProtT5 model files is included in the GOPredSim/models folder. If those files are removed, GOPredSim will automatically download it in the user .cache. If GOPredSim is used in a cluster, this download is user-specific, and, as files are quite big in some cases, it may rise an error if not enough space. To add more models, download files using the links found in /GOPredSim/venv/lib/python3.9/site-packages/bio_embeddings/utilities/defaults.yml and put them in the GOPredSim/models folder. In addition, this information must be added to the configuration files for the embedding part (.yml files) as explained in https://github.com/sacdallago/bio_embeddings/issues/114.

# 8. The folder includes two additional scripts to convert the format on the output to the input of topGO and to collapse GO annotation of several isoforms into GO annotation per gene. Additional information on how to run the pipeline and these steps can be found on GitHub (https://github.com/MetazoaPhylogenomicsLab/FANTASIA).

```

# Using Fantasia

Fantasia uses an input protein sequence file and annotates GO terms using GOPredSim. 
![FANTASIA_pipeline](https://github.com/user-attachments/assets/bc62f7b9-b9ec-4446-8ed4-6b9c21028956)

Filtered (using transdecoder) protein file should end in .pep and should look something like this
```
>TRINITY_DN10008_c0_g1_i1.p1 TRINITY_DN10008_c0_g1~~TRINITY_DN10008_c0_g1_i1.p1  ORF type:complete len:264 (-),score=53.73 TRINITY_DN10008_c0_g1_i1:64-747(-)
MTADTMTAATLPVASSTWVSPSVQKWTFHQPSDGDGHKGRNLTWSEALELMENSPRFREM
LISKLQESPFNAFFWECTPLTEHTAQHRSFEFVIMEGSHLDSARPDTESFSDYLADFKGQ
PVARAFSNLGGDSSLISPAQATKNPEDYKHIGNFFRRAPMEQRHAVFQTLGQELQRRLAQ
KPEAPYWVSTEGSGVAWLHMRIDPRPKYYHHREYRSPEYGLVQKGEL
>TRINITY_DN10011_c0_g1_i1.p1 TRINITY_DN10011_c0_g1~~TRINITY_DN10011_c0_g1_i1.p1  ORF type:5prime_partial len:218 (+),score=9.82 TRINITY_DN10011_c0_g1_i1:2-655(+)
HLPLAPPSYLAAFPVSTLACTQIFFIRLYACVLHVSAIMDFSAWFEDVRRMNKRQLFYQV
LNFAMIVSSALMIWKGLMVVTGSESPIVVVLSGSMEPAFQRGDLLFLTNYKEDPIRVGEI
VVFKVEGREIPIVHRVLKIHEKKDGAIRFLTKGDNNSVDDRGLYAPGQLWLQRKDVVGRA
RGFVPYVGMVTILMNDYPKFKYAILVALGAFVLLHRE
>TRINITY_DN10011_c0_g1_i2.p1 TRINITY_DN10011_c0_g1~~TRINITY_DN10011_c0_g1_i2.p1  ORF type:5prime_partial len:218 (+),score=9.82 TRINITY_DN10011_c0_g1_i2:2-655(+)
HLPLAPPSYLAAFPVSTLACTQIFFIRLYACVLHVSAIMDFSAWFEDVRRMNKRQLFYQV
LNFAMIVSSALMIWKGLMVVTGSESPIVVVLSGSMEPAFQRGDLLFLTNYKEDPIRVGEI
VVFKVEGREIPIVHRVLKIHEKKDGAIRFLTKGDNNSVDDRGLYAPGQLWLQRKDVVGRA
RGFVPYVGMVTILMNDYPKFKYAILVALGAFVLLHRE
```

Fantasia uses GOPredSim with the protein language model ProtT5 to achieve GO terms used in downstream analyses. 

In order to do this correctly, I needed to build a job script, which would run the .sh

#pstr_generate_gopredsim_input_files.job
```
#!/bin/bash
#BSUB -J pstr_generate_gopredsim_input_files
#BSUB -e pstr_generate_gopredsim_input_files.err
#BSUB -o pstr_generate_gopredsim_input_files.out
#BSUB -q bigmem #this is the queue, we need bigmem for some of this work
#BSUB -P coralma #this is my personal project, use yours
#BSUB -n 8
#BSUB -R "rusage[mem=10000]"
#BSUB -B
#BSUB -W 120:00
#BSUB -N
#BSUB -u user@school.edu #input your user ID

# Change to the working directory
cd /scratch/projects/coralma/fantasia/FANTASIA

# Define variables for paths and options
PEP_FILE="Trinity_filtered.fasta.transdecoder.pep"
OUTPUT_PATH="/scratch/projects/coralma/fantasia/FANTASIA/gopredsim_output/"
PREFIX="pstr"  # Customize this as needed
CONFIG_PATH="/scratch/projects/coralma/fantasia/FANTASIA/"  # Current directory; change if you have a specific config path
MODE="cpu"  # Set to "gpu" if you want to use GPU

# Create the output directory if it doesn't exist
mkdir -p $OUTPUT_PATH

# Generate GOPredSim input files
echo "Generating GOPredSim input files..."
./generate_gopredsim_input_files.sh -i $PEP_FILE -o $OUTPUT_PATH -c $CONFIG_PATH -p -m $MODE --prefix pstr
```

This execution uses CD-HIT to remove identical sequences and any seqs longer than 5000 amino acids (model constraint). The output files should be:
1. [filt_fasta]_cdhit100_5k_removed.pep
2. [filt_fasta]_cdhit100_headers_larger5k.txt
3. [filt_fasta]_cdhit100_headers_smaller5k.txt
4. [filt_fasta]_cdhit100.pep
5. [filt_fasta]_cdhit100.pep.clstr
6. [filt_fasta]_cdhit100_seqs_larger5k.txt

This step also produces a config_files directory with two sub directories, embeddings and gopredsim. In embeddings and gopredsim, you should have a [PREFIX]_prott5.yml

Your job script will also shoot out an .err file (sources of error if there are any) and an .out file (outcome of the job).

Next, we want to actually run the launch_gopredsim_pipeline.sh, so lets build a job script
```
#!/bin/bash
#BSUB -J pstr_launch_gopredsim_pipeline
#BSUB -e pstr_launch_gopredsim_pipeline.err
#BSUB -o pstr_launch_gopredsim_pipeline.out
#BSUB -q bigmem #this is the queue, we need bigmem for some of this work
#BSUB -P coralma #this is my personal project, use yours
#BSUB -n 8
#BSUB -R "rusage[mem=10000]"
#BSUB -B
#BSUB -W 120:00
#BSUB -N
#BSUB -u user@school.edu #input your user ID

# Change to the working directory
cd /scratch/projects/coralma/fantasia/FANTASIA

# Load conda and activate the environment
source /nethome/baw117/miniconda3/etc/profile.d/conda.sh
conda activate gopredsim

# Run the GOPredSim pipeline with the correct path to the config file
./launch_gopredsim_pipeline.sh -c /scratch/projects/coralma/fantasia/FANTASIA/ -x pstr -m prott5 -o /scratch/projects/coralma/fantasia/F$
```

This step is a doozie, I gave it roughly 80GB of RAM and it achieved 1% (250/23787) in an hour.  

