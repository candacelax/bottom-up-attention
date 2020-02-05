FROM nvidia/cuda:10.1-cudnn7-devel-ubuntu18.04
MAINTAINER caffe-maint@googlegroups.com

ENV DEBIAN_FRONTEND=noninteractive
ENV TZ America/New_York
ENV CUDA_PATH /usr/local/cuda
ENV DOCKER_BASE_DIR=/storage/ccross/bias-grounded-bert/vilbert_beta
RUN echo $DOCKER_BASE_DIR $CUDA_PATH
ENV CAFFE_ROOT $DOCKER_BASE_DIR/caffe/

#---- install libraries for Caffe
RUN apt-get update && apt-get install -y --no-install-recommends \
        build-essential \
        git \
        wget \
        libatlas-base-dev \
        libboost-all-dev \
        libgflags-dev \
        libgoogle-glog-dev \
        libhdf5-serial-dev \
        libleveldb-dev \
        liblmdb-dev \
        libopencv-dev \
        libprotobuf-dev \
        libsnappy-dev \
        protobuf-compiler \
        python-dev \
	python-setuptools \
        python-pip && \
	rm -rf /var/lib/apt/lists/*

# add necessary files to image
COPY bottom-up-attention/lib $DOCKER_BASE_DIR/lib/
COPY bottom-up-attention/tools $DOCKER_BASE_DIR/tools/
COPY bottom-up-attention/caffe $DOCKER_BASE_DIR/caffe/
COPY bottom-up-attention/experiments $DOCKER_BASE_DIR/experiments/
COPY bottom-up-attention/models $DOCKER_BASE_DIR/models/
COPY data $DOCKER_BASE_DIR/data/
COPY bottom-up-attention/features $DOCKER_BASE_DIR/features/
COPY bottom-up-attention/experiments $DOCKER_BASE_DIR/experiments/

#--- install Python packages
WORKDIR $CAFFE_ROOT
RUN for req in $(cat python/requirements.txt) pydot; do pip install $req; done && \
    mkdir build && cd build && \
    cmake -DUSE_CUDNN=1 .. && \
    make -j"$(nproc)" && \
    make install
RUN cd ../lib && make -j10
RUN cmake --version

ENV PYCAFFE_ROOT $CAFFE_ROOT/python
ENV PYTHONPATH $PYCAFFE_ROOT:$PYTHONPATH
ENV PATH $CAFFE_ROOT/build/tools:$PYCAFFE_ROOT:$PATH
RUN echo "$CAFFE_ROOT/build/lib" >> /etc/ld.so.conf.d/caffe.conf && ldconfig

WORKDIR /workspace