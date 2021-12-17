FROM ubuntu:20.04

ENV DEBIAN_FRONTEND noninteractive
ENV PATH /opt/amazon/openmpi/bin:$PATH
ENV LD_LIBRARY_PATH /opt/amazon/openmpi/lib64:$LD_LIBRARY_PATH

# Install basic tools
RUN apt-get update && apt-get install -y vim htop \
    software-properties-common groff jq libc6-dev \
    build-essential gcc g++ cmake wget curl zip

# Customize bash
RUN echo "alias ll='ls -alh --color=auto'" | tee -a /root/.bashrc

# Install aws cli
RUN curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip" && \
    unzip awscliv2.zip && \
    ./aws/install && \
    rm -f awscliv2.zip && \
    rm -rf aws && \
    aws --version

# Install efa
RUN wget -q --timeout=20 https://s3-us-west-2.amazonaws.com/aws-efa-installer/aws-efa-installer-latest.tar.gz -O /tmp/aws-efa-installer-latest.tar.gz && \
    tar -xf /tmp/aws-efa-installer-latest.tar.gz -C /tmp && \
    cd /tmp/aws-efa-installer && \
    ./efa_installer.sh -y -d --skip-kmod --skip-limit-conf --no-verify && \
    rm -f /tmp/aws-efa-installer-latest.tar.gz

# Install osu-micro-benchmarks
RUN export PATH=/opt/amazon/openmpi/bin:/opt/amazon/efa/bin/:$PATH && \
    export LD_LIBRARY_PATH=/opt/amazon/openmpi/lib64:/opt/amazon/efa/lib64:$LD_LIBRARY_PATH && \
    wget https://mvapich.cse.ohio-state.edu/download/mvapich/osu-micro-benchmarks-5.7.1.tgz -O /tmp/osu-micro-benchmarks-5.7.1.tgz && \
    tar zxvf /tmp/osu-micro-benchmarks-5.7.1.tgz -C /tmp && \
    cd /tmp/osu-micro-benchmarks-5.7.1/ && \
    ./configure CC=mpicc CXX=mpicxx && \
    make -j 4 && \
    cat config.log && \
    rm -f /tmp/osu-micro-benchmarks-5.7.1.tgz
