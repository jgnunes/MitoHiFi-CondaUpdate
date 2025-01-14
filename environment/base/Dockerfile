FROM ubuntu:20.04

ENV DEBIAN_FRONTEND=noninteractive

RUN apt-get -qq -y update \
    && apt-get -qq -y install \
    software-properties-common \
    && add-apt-repository ppa:deadsnakes/ppa \
    && apt-get -qq -y install \
    python3.13 \
    python3.13-dev \
    python3.13-venv \
    autoconf \
    automake \
    bedtools \
    build-essential \
    curl \
    default-jre \
    git \
    infernal \
    libopenjp2-7 \
    libtiff5 \
    libz-dev \
    singularity-container \
    mafft \
    ncbi-blast+ \
    samtools \
    wget \
    vim \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

# Update alternatives to use python3.13 as default
RUN update-alternatives --install /usr/bin/python3 python3 /usr/bin/python3.13 1 \
    && update-alternatives --set python3 /usr/bin/python3.13

RUN curl -sS https://bootstrap.pypa.io/get-pip.py | python3.13

WORKDIR /opt

# Install other software and dependencies (minimap2, hifiasm, MitoFinder, etc.)
RUN curl -L https://github.com/lh3/minimap2/releases/download/v2.24/minimap2-2.24_x64-linux.tar.bz2 \
    | tar -jxvf  - --no-same-owner

RUN curl -L https://github.com/chhylp123/hifiasm/archive/refs/tags/0.16.1.tar.gz \
    | tar -xzvf - \
    && cd hifiasm-0.16.1 \
    && make

RUN git clone https://github.com/weizhongli/cdhit.git \
    && cd cdhit \
    && make MAX_SEQ=10000000

RUN mkdir -p /opt/wrappers

COPY mitos_wrapper.sh /opt/wrappers/runmitos.py

RUN chmod -R 755 /opt/wrappers

# Pull the MitoFinder Singularity image
RUN singularity pull --arch amd64 /opt/mitofinder_v1.4.2.sif library://remiallio/default/mitofinder:v1.4.2

# Create a wrapper script for MitoFinder
RUN echo '#!/bin/bash\nsingularity exec /opt/mitofinder_v1.4.2.sif mitofinder "$@"' > /usr/bin/mitofinder \
    && chmod +x /usr/bin/mitofinder

ARG CONDA_DIR=/opt/conda

RUN curl -L -O "https://github.com/conda-forge/miniforge/releases/latest/download/Miniforge3-$(uname)-$(uname -m).sh" \
    && bash Miniforge3-$(uname)-$(uname -m).sh -bfp $CONDA_DIR \
    && rm -rf Miniforge3-$(uname)-$(uname -m).sh

RUN $CONDA_DIR/bin/conda create -n mitos_env -c bioconda -c conda-forge mitos=2.1.0 "r-base>4"

RUN $CONDA_DIR/bin/conda clean -a

RUN mkdir -p /opt/databases

WORKDIR /opt/databases

RUN curl -L https://zenodo.org/record/4284483/files/refseq89m.tar.bz2?download=1 | tar -jxvf - \
    && curl -L https://zenodo.org/record/4284483/files/refseq89f.tar.bz2?download=1 | tar -jxvf -

WORKDIR /opt

# Ensure Python 3.13 is used and reinstall dependencies
RUN python3.13 -m pip install --no-cache-dir --upgrade pip \
    && python3.13 -m pip install --no-cache-dir --force-reinstall six python-dateutil biopython>=1.80 pandas \
    Pillow matplotlib entrezpy dna_features_viewer bcbio-gff

# Debug to validate installation
RUN python3.13 -c "import sys; print(sys.path)" \
    && python3.13 -c "import six; print(six.__file__)" \
    && python3.13 -c "import dateutil; print(dateutil.__file__)" \
    && python3.13 -c "import Bio; print(Bio.__file__)"
