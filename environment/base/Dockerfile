FROM ubuntu:24.04

FROM ubuntu:24.04

# Non-interactive mode for APT
ENV DEBIAN_FRONTEND=noninteractive

# Add necessary repositories and install dependencies
RUN apt-get update -qq && apt-get install -y -qq software-properties-common \
    && add-apt-repository -y ppa:deadsnakes/ppa \
    && apt-get update -qq && apt-get install -y -qq \
    autoconf automake apt-utils bedtools build-essential \
    curl default-jre git infernal libopenjp2-7 libz-dev \
    libtiff-dev mafft ncbi-blast+ python3.11 python3.11-venv \
    python3.11-distutils python3.11-dev samtools wget vim \
    && rm -rf /var/lib/apt/lists/*

# Ajustar o Python3 padrão para apontar para o Python3.11
RUN update-alternatives --install /usr/bin/python3 python3 /usr/bin/python3.11 1

# Instalar pip para o Python3.11
RUN curl -sS https://bootstrap.pypa.io/get-pip.py | python3

# Agora pip3 aponta para a instalação do Python3.11
# Certifique-se de que a versão do biopython agora seja a mais recente
RUN pip3 install --no-cache-dir --upgrade pip \
    && pip3 install --no-cache-dir biopython pandas Pillow matplotlib entrezpy dna_features_viewer bcbio-gff

WORKDIR /opt

# Instalar minimap2
RUN curl -L https://github.com/lh3/minimap2/releases/download/v2.24/minimap2-2.24_x64-linux.tar.bz2 \
    | tar -jxvf - --no-same-owner

# Instalar hifiasm
RUN curl -L https://github.com/chhylp123/hifiasm/archive/refs/tags/0.16.1.tar.gz \
    | tar -xzvf - \
    && cd hifiasm-0.16.1 \
    && make \
    && wget -P /usr/local/src https://bootstrap.pypa.io/pip/2.7/get-pip.py \
    && python2 /usr/local/src/get-pip.py \
    && python2 -m pip --no-cache-dir install biopython==1.70

# Instalar MitoFinder
RUN git clone https://github.com/RemiAllio/MitoFinder.git \
    && cd MitoFinder \
    && ./install.sh \
    && sed -i 's/\/usr\/bin\/python/\/usr\/bin\/env python/' mitofinder \
    && chmod -R 755 /opt/MitoFinder

# Instalar CD-HIT
RUN git clone https://github.com/weizhongli/cdhit.git && \
    cd cdhit && \
    make MAX_SEQ=10000000 

RUN mkdir -p /opt/wrappers

COPY mitos_wrapper.sh /opt/wrappers/runmitos.py
COPY mitofinder_wrapper.sh /opt/wrappers/mitofinder
RUN chmod -R 755 /opt/wrappers

ARG CONDA_DIR=/opt/conda
RUN curl -L -O "https://github.com/conda-forge/miniforge/releases/latest/download/Miniforge3-$(uname)-$(uname -m).sh" \
    && bash Miniforge3-$(uname)-$(uname -m).sh -bfp $CONDA_DIR \
    && rm -rf Miniforge3-$(uname)-$(uname -m).sh

# Criar ambientes Conda, se necessário
RUN $CONDA_DIR/bin/conda create -n mitos_env -c bioconda -c conda-forge mitos=2.1.0 "r-base>4"
RUN $CONDA_DIR/bin/conda create -n mitofinder_env python=2.7
RUN $CONDA_DIR/bin/conda clean -a

RUN mkdir -p /opt/databases
WORKDIR /opt/databases
RUN curl -L https://zenodo.org/record/4284483/files/refseq89m.tar.bz2?download=1 | tar -jxvf - \
    && curl -L https://zenodo.org/record/4284483/files/refseq89f.tar.bz2?download=1 | tar -jxvf -

WORKDIR /opt
