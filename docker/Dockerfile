ARG CUDA="10.2"
ARG CUDNN="7"

# TODO: conda installs its own version of cuda
FROM nvidia/cuda:${CUDA}-cudnn${CUDNN}-devel-ubuntu18.04

ENV CUDA_PATH /usr/local/cuda
ENV CUDA_ROOT /usr/local/cuda/bin
ENV LD_LIBRARY_PATH /usr/local/nvidia/lib64

RUN apt-get -qq update
# libsm6 and libxext6 are needed for cv2
RUN apt-get update && apt-get install -y libxext6 libsm6 libxrender1 build-essential sudo \
    libgl1-mesa-glx git wget rsync tmux nano dcmtk fftw3-dev liblapacke-dev libpng-dev libopenblas-dev jq && \
  rm -rf /var/lib/apt/lists/*
RUN ldconfig

WORKDIR /tmp
RUN git clone https://github.com/mrirecon/bart.git
WORKDIR bart
RUN make -j4
RUN make install

# install latest cmake 3.*
# RUN add-apt-repository ppa:george-edison55/cmake-3.input_image
# RUN apt-get update && apt-get install -input_image cmake && \
#  rm -rf /var/lib/apt/lists/*

# Make a user
RUN adduser --disabled-password --gecos '' direct
RUN adduser direct sudo
RUN echo '%sudo ALL=(ALL) NOPASSWD:ALL' >> /etc/sudoers
USER direct

WORKDIR /tmp
RUN wget -q https://repo.continuum.io/miniconda/Miniconda3-latest-Linux-x86_64.sh
RUN bash Miniconda3-latest-Linux-x86_64.sh -b

ENV PATH "/home/direct/miniconda3/bin:/tmp/bart/:$PATH:$CUDA_ROOT"

# Setup python packages
RUN conda update -n base conda -yq
RUN conda install python=3.7
RUN conda install numpy pyyaml mkl mkl-include setuptools cmake cffi typing boost
RUN conda install pytorch=1.5 torchvision cudatoolkit=${CUDA} -c pytorch
RUN conda install scipy pandas scikit-learn scikit-image=0.14 -yq
RUN conda install cython tqdm jupyter sqlalchemy -yq
RUN pip install opencv-python simpleitk h5py -q
RUN pip install runstats -q
RUN pip install tb-nightly -q
RUN pip install --pre omegaconf -q

# Install apex for mixed precision training
WORKDIR /tmp
RUN git clone https://github.com/NVIDIA/apex
WORKDIR apex
# A bug in the install creates a exit code 1, blocking docker from continuing, always provide exit code 0.
RUN pip install -v --no-cache-dir --global-option="--cpp_ext" --global-option="--cuda_ext" ./; exit 0

USER root
# Create directories for input and output
RUN mkdir /data
RUN mkdir /direct && chmod 777 /direct
# Directory to copy or symlink data into. Needs to be 777 as DIRECT runs typically without root permissions.
RUN mkdir /input && chmod 777 /input

USER direct

RUN jupyter notebook --generate-config
ENV CONFIG_PATH "/home/direct/.jupyter/jupyter_notebook_config.py"
COPY "jupyter_notebook_config.py" ${CONFIG_PATH}

# Copy files into the docker
COPY [".", "/direct"]

ENV PYTHONPATH /tmp/bart/python:/direct
WORKDIR /direct

# Provide an open entrypoint for the docker
ENTRYPOINT $0 $@
