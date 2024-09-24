# Fantasia_build_and_run
Full start to finish install and code to run Fantasia on a containerize Docker Singularity image using a Mac M1 chip

## Docker
First download Docker, this is needed to create the architecture (Ubuntu) needed for singularity. If using HPC, singularity does not need root priviledges and can be done without Docker, as long as the architecture is linux

I like the docker desktop GUI, so I use went to docs.docker.com for the mac install

Once docker is installed, build a dockerfile so that you can build singularity within the container. This may take some work to get the packages and dependencies needed

This was created using a Mac OS M1 chip, which requires Rosetta

Go to the directory you wish to have this, I called mine singularity-docker

```
mkdir singularity-docker
cd singularity-docker
nano Dockerfile
```

### Dockerfile.txt
```
# Use an ARM64 base image
FROM arm64v8/ubuntu:22.04

# Set the DEBIAN_FRONTEND to noninteractive to avoid prompts during package installation
ENV DEBIAN_FRONTEND=noninteractive

# Install dependencies including FUSE headers and build tools
RUN apt-get update && apt-get install -y \
    build-essential \
    libseccomp-dev \
    pkg-config \
    squashfs-tools \
    cryptsetup \
    curl \
    git \
    uuid-dev \
    libgpgme-dev \
    wget \
    libssl-dev \
    uidmap \
    libglib2.0-dev \
    libfuse-dev \
    libfuse3-dev \
    autoconf \
    automake \
    libtool

# Install Go version 1.20.7 which <1.20.0 is required for singularity
ENV GO_VERSION=1.20.7
ENV OS=linux
ENV ARCH=arm64
RUN wget https://golang.org/dl/go${GO_VERSION}.${OS}-${ARCH}.tar.gz && \
    tar -C /usr/local -xzf go${GO_VERSION}.${OS}-${ARCH}.tar.gz && \
    rm go${GO_VERSION}.${OS}-${ARCH}.tar.gz

# Add Go to the PATH
ENV PATH="/usr/local/go/bin:${PATH}"

# Verify Go installation
RUN go version

# Download and install Singularity
RUN export SINGULARITY_VERSION=4.0.2 && \
    cd /tmp && \
    wget https://github.com/sylabs/singularity/releases/download/v${SINGULARITY_VERSION}/singularity-ce-${SINGULARITY_VERSION}.tar.gz && \
    tar -xzf singularity-ce-${SINGULARITY_VERSION}.tar.gz && \
    cd singularity-ce-${SINGULARITY_VERSION} && \
    ./mconfig && \
    make -C builddir && \
    make -C builddir install

# Set the entrypoint to Singularity
ENTRYPOINT ["singularity"]
```
**Note: The entire build took about 10minutes after a plethora of failures and fatal crashes**

## Singularity

To enter the docker singularity container

```
docker run -it --entrypoint /bin/bash singularity-ce
```

Once inside the container, check singularity is working

```
singularity --version
```

Great, now that singularity works and our build is complete, we can DL the image

Here we are pulling by a unique ID

```
singularity pull --arch amd64 library://gemma.martinezredondo/fantasia/fantasia:sha256.06f759be1e48bf4f72aed0d4bb4fe2fd6e05774bb58131b131f0128c7b0efc84
```

This is a 10GB download, so be patient. 

## [Fantasia](https://github.com/MetazoaPhylogenomicsLab/FANTASIA?tab=readme-ov-file)
![FANTASIA_pipeline](https://github.com/user-attachments/assets/c8d464ad-a3bd-4031-80cc-8c4d46917218)

FANTASIA (Functional ANnoTAtion based on embedding space SImilArity) is a pipeline for annotating GO terms in protein sequence files using GOPredSim (to know more) with the protein language model ProtT5. FANTASIA takes as input a proteome file (either the longest isoform or the full set of isoforms for all genes), removes identical sequences using CD-HIT (ref) and sequences longer than 5000 amino acids (due to a length constraint in the model), and executes GOPredSim-ProtT5 for all sequences. Then, it converts the standard GOPredSim output file to the input file format for topGO (ref) to facilitate its application in a wider biological workflow.

Here is the syntax of FANTASIA from the vignette and also some of the flags you can use
```
Syntax: ./fantasia --infile protein.fasta [--outpath output_path] [--allisoforms gene_isoform_conversion.txt] [--keepintermediate]
options:
-i/--infile           Input protein fasta file.
-h/--help             Print this Help.
-o/--outpath          (Optional) Output directory. If not provided, input file directory will be used.
-a/--allisoforms      (Optional) Tab-separated conversion file specifying the correspondance between gene and isoform IDs for obtaining a per-gene annotation using all isoforms.
-p/--prefix           (Optional) Prefix to add to output folders and files (e.g. the species code). If not provided, input file name will be used.
```

### Using Fantasia
Fantasia uses an input protein sequence file and annotates GO terms using GOPredSim. 

#Filtered (using transdecoder) protein file should end in .pep and should look something like this
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

Using our protein fasta file, we can run it in Fantasia to predict annotations, which will produce GO terms. This outfile can supplement our eggnogg mapper file for any proteins that did not have any associated GO terms

```
./fantasia --infile Trinity_filtered.fasta.transdecoder.pep [--outpath output_path] [--allisoforms gene_isoform_conversion.txt] [--keepintermediate]
```


