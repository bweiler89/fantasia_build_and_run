# Fantasia_build_and_run
Full start to finish install and code to run Fantasia on a containerize Docker Singularity image using a Mac M1 chip

## Docker
#First download Docker, this is needed to create the architecture (Ubuntu) needed for singularity. If using HPC, singularity does not need root priviledges and can be done without Docker, as long as the architecture is linux
#I like the docker desktop GUI, so I use went to docs.docker.com for the mac install

#Once docker is installed, build a dockerfile so that you can build singularity within the container. This may take some work to get the packages and dependencies needed
#This was created using a Mac OS M1 chip, which requires Rosetta
#Dockerfile.txt
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
#The entire build took about 10minutes

## Singularity

#To enter the docker singularity container

```
docker run -it --entrypoint /bin/bash singularity-ce
```

#Once inside the container, check singularity is working

```
singularity --version
```

#Great, now that singularity works and our build is complete, we can DL the image
#Here we are pulling by a unique ID

```
singularity pull --arch amd64 library://gemma.martinezredondo/fantasia/fantasia:sha256.06f759be1e48bf4f72aed0d4bb4fe2fd6e05774bb58131b131f0128c7b0efc84
```

#This is a 10GB download, so be patient. 

## Fantasia
