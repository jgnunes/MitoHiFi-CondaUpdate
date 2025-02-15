FROM ghcr.io/jgnunes/mitohifi-condaupdate-base:master

WORKDIR /opt

# Updates MitoFinder wrapper to "see" /opt/MitoHiFi/tests directory (that contains files to test mitohifi image through github actions)
RUN echo '#!/bin/bash\nsingularity run --bind /opt/MitoHiFi/tests:/opt/MitoHiFi/tests /opt/mitofinder_v1.4.2.sif "$@"' > /usr/bin/mitofinder \
    && chmod +x /usr/bin/mitofinder

# /opt/MitoHiFi
RUN git clone https://github.com/marcelauliano/MitoHiFi.git \
    && cd MitoHiFi/src \
    && chmod +x *.py \
    && sed -i '1i#!/usr/bin/env python3' mitohifi.py findFrameShifts.py fixContigHeaders.py \
    && sed -i '1 s/python\>/python3/' *.py \
    && sed -i 's/(mitofinder_cmd, stderr=subprocess.DEVNULL, stdout=subprocess.DEVNULL)/(mitofinder_cmd)/' parallel_annotation.py \
    && sed -i 's/"python2",\s//' parallel_annotation_mitos.py \
    && sed -i 's/MITOS\/data/\/opt\/databases/' parallel_annotation_mitos.py

WORKDIR /tmp

ENV CONDA_DIR=/opt/conda

ENV PATH=/opt/wrappers:/opt/cdhit/:/opt/hifiasm-0.16.1/:/opt/MitoHiFi/src/:/opt/MitoFinder/:/opt/minimap2-2.24_x64-linux/:${PATH}
