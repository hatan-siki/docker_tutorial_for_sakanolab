FROM ubuntu:22.04
USER root

ENV DEBIAN_FRONTEND=noninteractive

RUN apt-get update && \
    apt-get install -y --no-install-recommends \
    python3.11-dev \
    python3-pip \
    python3-tk && \
    apt-get -y clean && \
    rm -rf /var/lib/apt/lists/*

RUN pip install --upgrade pip
RUN pip install numpy scikit-learn matplotlib
