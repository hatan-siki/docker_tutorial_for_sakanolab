FROM python:3.11-bullseye
USER root

RUN mkdir -p /root/workspace
WORKDIR /root/workspace

ENV DEBIAN_FRONTEND=noninteractive

RUN apt-get update && \
    apt-get install -y --no-install-recommends \
    python3-tk && \
    apt-get -y clean && \
    rm -rf /var/lib/apt/lists/*

RUN pip install --upgrade pip
RUN pip install numpy scikit-learn matplotlib
