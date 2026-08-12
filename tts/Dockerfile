FROM debian:bookworm-slim

RUN printf 'Acquire::Retries "5";\nAcquire::http::Timeout "30";\n' > /etc/apt/apt.conf.d/80-retries && \
    apt-get update && \
    DEBIAN_FRONTEND=noninteractive apt-get install -y --no-install-recommends \
        wget curl ca-certificates python3 python3-pip python3-dev \
        build-essential ffmpeg libsndfile1 && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

ENV CONDA_DIR=/opt/conda
RUN wget --tries=5 --timeout=30 --quiet \
        https://repo.anaconda.com/miniconda/Miniconda3-py310_22.11.1-1-Linux-x86_64.sh \
        -O /tmp/miniconda.sh && \
    /bin/bash /tmp/miniconda.sh -b -p /opt/conda && \
    rm /tmp/miniconda.sh

ENV PATH=$CONDA_DIR/bin:$PATH

RUN conda install -y -c conda-forge ffmpeg && conda clean -afy

RUN pip install --no-cache-dir Flask waitress pydub omegaconf soundfile "setuptools<70.0.0" && \
    pip install --no-cache-dir torch torchaudio --extra-index-url https://download.pytorch.org/whl/cu118 && \
    pip cache purge

COPY . /root
ENV TTS_HOME=/root/tts_data
WORKDIR /root

RUN conda env config vars set TTS_LD_LIBRARY_PATH=$LD_LIBRARY_PATH && \
    conda env config vars set LD_LIBRARY_PATH=$LD_LIBRARY_PATH:$CONDA_DIR/lib

EXPOSE 5003
ENTRYPOINT ["conda", "run", "--no-capture-output", "-n", "base", "python", "tts.py"]
